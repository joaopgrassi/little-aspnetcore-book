## Autorização (authorization) com roles

Roles são uma abordagem comum para lidar com autorização e permissões em uma aplicação web. Por exemplo, é comum criar uma role de Administrador que ofereça aos usuários administradores mais permissões ou poder do que aos usuários normais.

Neste projeto, você adicionará uma página chamada "Gerenciar Usuários" que somente os administradores poderão ver. Se usuários normais tentarem acessá-la, verão um erro.

### Adicionando a página "Gerenciar Usuários"

Primeiro, crie um novo controller:

**Controllers/ManageUsersController.cs**

```csharp
using System;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Identity;
using AspNetCoreTodo.Models;
using Microsoft.EntityFrameworkCore;

namespace AspNetCoreTodo.Controllers
{
    [Authorize(Roles = "Administrator")]
    public class ManageUsersController : Controller
    {
        private readonly UserManager<ApplicationUser>
            _userManager;
        
        public ManageUsersController(
            UserManager<ApplicationUser> userManager)
        {
            _userManager = userManager;
        }

        public async Task<IActionResult> Index()
        {
            var admins = (await _userManager
                .GetUsersInRoleAsync("Administrator"))
                .ToArray();

            var everyone = await _userManager.Users
                .ToArrayAsync();

            var model = new ManageUsersViewModel
            {
                Administrators = admins,
                Everyone = everyone
            };

            return View(model);
        }
    }
}
```

Definir a propriedade `Roles` no atributo `[Authorize]` garantirá que o usuário esteja logado **e** tenha a role Administrador para visualizar a página.

Em seguida, crie uma view model:

**Models/ManageUsersViewModel.cs**

```csharp
using System.Collections.Generic;
using AspNetCoreTodo.Models;

namespace AspNetCoreTodo.Models
{
    public class ManageUsersViewModel
    {
        public ApplicationUser[] Administrators { get; set; }

        public ApplicationUser[] Everyone { get; set;}
    }
}
```

Por último, crie o diretório `Views/ManageUsers` e a view para a action `Index`:

**Views/ManageUsers/Index.cshtml**

```html
@model ManageUsersViewModel

@{
    ViewData["Title"] = "Manage users";
}

<h2>@ViewData["Title"]</h2>

<h3>Administrators</h3>

<table class="table">
    <thead>
        <tr>
            <td>Id</td>
            <td>Email</td>
        </tr>
    </thead>
    
    @foreach (var user in Model.Administrators)
    {
        <tr>
            <td>@user.Id</td>
            <td>@user.Email</td>
        </tr>
    }
</table>

<h3>Everyone</h3>

<table class="table">
    <thead>
        <tr>
            <td>Id</td>
            <td>Email</td>
        </tr>
    </thead>
    
    @foreach (var user in Model.Everyone)
    {
        <tr>
            <td>@user.Id</td>
            <td>@user.Email</td>
        </tr>
    }
</table>
```

Rode a aplicação e tente acessar a rota `/ManageUsers` logado com um usuário normal. Você verá a página de acesso negado:

![Erro de acesso negado](access-denied.png)

Isso ocorre porque ainda não temos nenhum usuário administrador.

### Criando uma conta de administrator para testes

Por razões óbvias de segurança, não é possível para ninguém registrar uma nova conta de administrador. Na verdade, o papel de administrador nem existe no banco de dados ainda!

Você pode adicionar a role Administrador mais uma conta de administrador de teste ao banco de dados na primeira vez que o aplicativo for iniciado. A adição de dados pela primeira vez ao banco de dados é chamada de inicialização ou **seeding** do banco de dados.

Crie uma nova class na raiz do projeto chamada `SeedData`:

**SeedData.cs**

```csharp
using System;
using System.Threading.Tasks;
using AspNetCoreTodo.Models;
using Microsoft.AspNetCore.Identity;
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.DependencyInjection;

namespace AspNetCoreTodo
{
    public static class SeedData
    {
        public static async Task InitializeAsync(
            IServiceProvider services)
        {
            var roleManager = services
                .GetRequiredService<RoleManager<IdentityRole>>();
            await EnsureRolesAsync(roleManager);

            var userManager = services
                .GetRequiredService<UserManager<ApplicationUser>>();
            await EnsureTestAdminAsync(userManager);
        }
    }
}
```

O método `InitializeAsync()` usa um `IServiceProvider` (a coleção de serviços que é configurada no método `Startup.ConfigureServices()`) para obter o `RoleManager` e o `UserManager` do ASP.NET Core Identity.

Adicione mais dois métodos abaixo do método `InitializeAsync()`. Primeiro, o método `EnsureRolesAsync()`:

```csharp
private static async Task EnsureRolesAsync(
    RoleManager<IdentityRole> roleManager)
{
    var alreadyExists = await roleManager
        .RoleExistsAsync(Constants.AdministratorRole);
    
    if (alreadyExists) return;

    await roleManager.CreateAsync(
        new IdentityRole(Constants.AdministratorRole));
}
```

Este método verifica se existe uma role `Administrador` no banco de dados. Se não, cria uma. Em vez de digitar repetidamente a string `"Administrator"`, crie uma pequena classe chamada "Constants" para conter o valor:

**Constants.cs**

```csharp
namespace AspNetCoreTodo
{
    public static class Constants
    {
        public const string AdministratorRole = "Administrator";
    }
}
```

> Se você quiser, você pode atualizar o `ManageUsersController` para usar este valor constante também.

Em seguida, escreva o método `EnsureTestAdminAsync()`:

**SeedData.cs**

```csharp
private static async Task EnsureTestAdminAsync(
    UserManager<ApplicationUser> userManager)
{
    var testAdmin = await userManager.Users
        .Where(x => x.UserName == "admin@todo.local")
        .SingleOrDefaultAsync();

    if (testAdmin != null) return;

    testAdmin = new ApplicationUser
    {
        UserName = "admin@todo.local",
        Email = "admin@todo.local"
    };
    await userManager.CreateAsync(
        testAdmin, "NotSecure123!!");
    await userManager.AddToRoleAsync(
        testAdmin, Constants.AdministratorRole);
}
```

Se já não houver um usuário com o nome de usuário `admin@todo.local` no banco de dados, esse método criará um e atribuirá uma senha temporária. Depois de fazer o login pela primeira vez, você deve alterar a senha da conta para uma senha segura!

Em seguida, você precisa dizer a sua aplicação para executar essa lógica quando for iniciada. Modifique a classe `Program.cs` e atualize o método `Main()` para chamar o novo método `Initialize Database()` que você irá criar:

**Program.cs**

```csharp
public static void Main(string[] args)
{
    var host = BuildWebHost(args);
    InitializeDatabase(host);
    host.Run();
}
```

Em seguida, adicione o novo método abaixo de `Main()`:

```csharp
private static void InitializeDatabase(IWebHost host)
{
    using (var scope = host.Services.CreateScope())
    {
        var services = scope.ServiceProvider;

        try
        {
            SeedData.InitializeAsync(services).Wait();
        }
        catch (Exception ex)
        {
            var logger = services
                .GetRequiredService<ILogger<Program>>();
            logger.LogError(ex, "Error occurred seeding the DB.");
        }
    }
}
```

Adicione um novo comando `using` ao início do arquivo:

```csharp
using Microsoft.Extensions.DependencyInjection;
```

Esse método obtém a coleção de serviços que `SeedData.InitializeAsync()` precisa e, em seguida, executa o método de seed do banco de dados. Se algo der errado, um erro será registrado.

> Como `InitializeAsync()` retorna uma `Task`, o método `Wait()` deve ser usado para garantir que seja finalizado antes que a aplicação seja inicializada. Você normalmente usaria `await` para isso, mas por razões técnicas você não pode usar `await` na classe `Program`. Esta é uma exceção rara. Você deve usar o `await` em qualquer outro lugar!

Quando você iniciar a aplicação, a conta `admin@todo.local` será criada e atribuída à role Administrador. Tente fazer login com essa conta e navegue até `http://localhost:5000/ManageUsers`. Você verá uma lista de todos os usuários registrados na aplicação.

> Como um desafio extra, tente adicionar mais recursos de administração a esta página. Por exemplo, você pode adicionar um botão que permita ao administrador excluir uma conta de usuário.

### Verificando autorização (authorization) em uma view

O atributo `[Authorize]` facilita a execução de uma verificação de autorização em um controller ou action method, mas e se você precisar verificar a autorização em uma view? Por exemplo, seria bom exibir um link "Gerenciar usuários" na barra de navegação se o usuário logado for um administrador.

Você pode injetar o `UserManager` diretamente em uma view para realizar esses tipos de verificações. Para manter suas views limpas e organizadas, crie uma nova partial view que adicionará um item à barra de navegação no layout:

**Views/Shared/_AdminActionsPartial.cshtml**

```html
@using Microsoft.AspNetCore.Identity
@using AspNetCoreTodo.Models

@inject SignInManager<ApplicationUser> signInManager
@inject UserManager<ApplicationUser> userManager

@if (signInManager.IsSignedIn(User))
{
    var currentUser = await UserManager.GetUserAsync(User);

    var isAdmin = currentUser != null
        && await userManager.IsInRoleAsync(
            currentUser,
            Constants.AdministratorRole);

    if (isAdmin)
    {
        <ul class="nav navbar-nav navbar-right">
            <li>
                <a asp-controller="ManageUsers" 
                   asp-action="Index">
                   Manage Users
                </a>
            </li>
        </ul>
    }
}
```

> É convencional nomear partial views compartilhadas começando com um sublinhado `_`, mas não é obrigatório.

Esta partial view usa o `SignInManager` para determinar se o usuário está logado. Se não estiver, o resto do código da view pode ser ignorado. Se **é** um usuário logado, o `UserManager` é usado para procurar seus detalhes e executar uma verificação de autorização com `IsInRoleAsync()`. Se todas as verificações forem bem-sucedidas e o usuário for um administrador, um link **Gerenciar usuários** será adicionado à barra de navegação.

Para incluir esta partial view no layout principal, edite `_Layout.cshtml` e adicione-a na seção navbar:

**Views/Shared/_Layout.cshtml**

```html
<div class="navbar-collapse collapse">
    <ul class="nav navbar-nav">
        <!-- existing code here -->
    </ul>
    @await Html.PartialAsync("_LoginPartial")
    @await Html.PartialAsync("_AdminActionsPartial")
</div>
```

Quando você fizer login com uma conta de administrador, você verá um novo item no canto superior direito:

![Manage Users link](manage-users.png)


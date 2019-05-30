## Usando identity na aplicação

Os itens da lista de tarefas a fazer ainda são compartilhados entre todos os usuários, porque as entidades de tarefas armazenadas não estão vinculadas a um usuário específico. Agora que o atributo `[Authorize]` garante que você deve estar logado para ver a view de tarefas, você pode filtrar a consulta do banco de dados com base em quem está logado.

Primeiro, injete a `UserManager<ApplicationUser>` no `TodoController`:

**Controllers/TodoController.cs**

```csharp
[Authorize]
public class TodoController : Controller
{
    private readonly ITodoItemService _todoItemService;
    private readonly UserManager<ApplicationUser> _userManager;

    public TodoController(ITodoItemService todoItemService,
        UserManager<ApplicationUser> userManager)
    {
        _todoItemService = todoItemService;
        _userManager = userManager;
    }

    // ...
}
```

Você precisará adicionar um novo comando `using` no início do arquivo:

```csharp
using Microsoft.AspNetCore.Identity;
```

A classe `UserManager` é parte do ASP.NET Core Identity. Você pode utilizá-la para recuperar o usuário logado na action `Index`:

```csharp
public async Task<IActionResult> Index()
{
    var currentUser = await _userManager.GetUserAsync(User);
    if (currentUser == null) return Challenge();

    var items = await _todoItemService
        .GetIncompleteItemsAsync(currentUser);

    var model = new TodoViewModel()
    {
        Items = items
    };

    return View(model);
}
```

O novo código no topo do action método usa o `UserManager` para procurar o usuário logado a partir da propriedade `User` disponível na action:

```csharp
var currentUser = await _userManager.GetUserAsync(User);
```

Se houver um usuário logado, a propriedade `User` contém um objeto com algumas (mas não todas) informações do usuário. O `UserManager` usa isto para procurar os detalhes completos do usuário no banco de dados através do método `GetUserAsync()`.

O valor de `currentUser` nunca deve ser nulo, porque o atributo `[Authorize]` está presente no controller. No entanto, é uma boa idéia fazer uma verificação a mais para garantir. Você pode usar o método `Challenge()` para forçar o usuário a efetuar login novamente se suas informações estiverem faltando.

```csharp
if (currentUser == null) return Challenge();
```

Já que você está passando um parâmetro `ApplicationUser` para` GetIncompleteItemsAsync()`, você precisará atualizar a interface `ITodoItemService`:

**Services/ITodoItemService.cs**

```csharp
public interface ITodoItemService
{
    Task<TodoItem[]> GetIncompleteItemsAsync(
        ApplicationUser user);
    
    // ...
}
```

Como você mudou a interface `ITodoItemService`, você também precisa atualizar a assinatura do método `GetIncompleteItemsAsync()` no `TodoItemService`:

**Services/TodoItemService**

```csharp
public async Task<TodoItem[]> GetIncompleteItemsAsync(
    ApplicationUser user)
```

O próximo passo é atualizar a consulta do banco de dados e adicionar um filtro para mostrar apenas os itens criados pelo usuário atual. Antes de fazer isso, você precisa adicionar uma nova propriedade ao banco de dados.

### Atualizando o banco de dados

Você precisará adicionar uma nova propriedade ao entity model `TodoItem` para que cada tarefa possa "lembrar" o usuário que o possui:

**Models/TodoItem.cs**

```csharp
public string UserId { get; set; }
```

Como você atualizou o entity model usado pelo context do banco de dados, também é necessário executar o migrate do banco de dados. Crie uma nova migration usando o comando `dotnet ef` no terminal:

```
dotnet ef migrations add AddItemUserId
```

Isso cria uma nova migration chamada `AddItemUserId` que adicionará uma nova coluna à tabela `Items`, espelhando a alteração que você fez no model `TodoItem`.

Use o comando `dotnet ef` novamente para aplicá-lo ao banco de dados:

```
dotnet ef database update
```

### Atualizando a service class

Com o banco de dados e seu contexto atualizados, você pode atualizar o método `GetIncompleteItemsAsync()` no `TodoItemService` e adicionar outra cláusula à instrução `Where`:

**Services/TodoItemService.cs**

```csharp
public async Task<TodoItem[]> GetIncompleteItemsAsync(
    ApplicationUser user)
{
    return await _context.Items
        .Where(x => x.IsDone == false && x.UserId == user.Id)
        .ToArrayAsync();
}
```

Se você executar a aplicatção e se registrar ou efetuar login, verá uma lista de tarefas vazia novamente. Infelizmente, todos os itens que você tenta adicionar desaparecem porque você ainda não atualizou a action `AddItem` para tratar os usuários das tarefas ainda.

### Atualizando as actions AddItem e MarkDone

Você precisará usar o `UserManager` para obter o usuário atual nas actions `AddItem` e `MarkDone`, assim como você fez em `Index`.

Aqui estão os dois métodos atualizados:

**Controllers/TodoController.cs**

```csharp
[ValidateAntiForgeryToken]
public async Task<IActionResult> AddItem(TodoItem newItem)
{
    if (!ModelState.IsValid)
    {
        return RedirectToAction("Index");
    }

    var currentUser = await _userManager.GetUserAsync(User);
    if (currentUser == null) return Challenge();

    var successful = await _todoItemService
        .AddItemAsync(newItem, currentUser);

    if (!successful)
    {
        return BadRequest("Could not add item.");
    }

    return RedirectToAction("Index");
}

[ValidateAntiForgeryToken]
public async Task<IActionResult> MarkDone(Guid id)
{
    if (id == Guid.Empty)
    {
        return RedirectToAction("Index");
    }

    var currentUser = await _userManager.GetUserAsync(User);
    if (currentUser == null) return Challenge();

    var successful = await _todoItemService
        .MarkDoneAsync(id, currentUser);
    
    if (!successful)
    {
        return BadRequest("Could not mark item as done.");
    }

    return RedirectToAction("Index");
}
```

Ambos service methods agora devem aceitar um parâmetro `ApplicationUser`. Atualize a definição da interface em `ITodoItemService`:

```csharp
Task<bool> AddItemAsync(TodoItem newItem, ApplicationUser user);

Task<bool> MarkDoneAsync(Guid id, ApplicationUser user);
```

E por último, atualize as implementações do service method no `TodoItemService`. No método `AddItemAsync`, defina a propriedade `UserId` ao construir um `new TodoItem`:

```csharp
public async Task<bool> AddItemAsync(
    TodoItem newItem, ApplicationUser user)
{
    newItem.Id = Guid.NewGuid();
    newItem.IsDone = false;
    newItem.DueAt = DateTimeOffset.Now.AddDays(3);
    newItem.UserId = user.Id;

    // ...
}
```

A cláusula `Where` no método `MarkDoneAsync` também precisa verificar o ID do usuário, de forma que um usuário não autorizado não possa completar as tarefas de outra pessoa adivinhando seus IDs:

```csharp
public async Task<bool> MarkDoneAsync(
    Guid id, ApplicationUser user)
{
    var item = await _context.Items
        .Where(x => x.Id == id && x.UserId == user.Id)
        .SingleOrDefaultAsync();

    // ...
}
```

Pronto! Tente usar a aplicação com duas contas de usuário diferentes. As tarefas agora estão privadas para cada conta de usuário.

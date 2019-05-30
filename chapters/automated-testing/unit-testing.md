## Testes unitários

Testes unitários são pequenos testes que verificam o comportamento de um método ou classe. Quando o código que você está testando depende de outros métodos ou classes, os testes unitários dependem do **mocking** dessas outras classes, de modo que o teste se concentre apenas em uma coisa por vez.

Por exemplo, a classe `TodoController` tem duas dependências: uma `ITodoItemService` e o `UserManager`. O `TodoItemService`, por sua vez, depende do `ApplicationDbContext`. (A idéia de que você pode desenhar uma linha de `TodoController` > `TodoItemService` > `ApplicationDbContext` é chamada de **dependency graph [gráfico de dependência]**).

Quando a aplicação é executada normalmente, o contêiner de serviço do ASP.NET Core e o sistema de injeção de dependência injetam cada um desses objetos no dependency graph quando o `TodoController` ou o` TodoItemService` é criado.

Quando você escreve um teste unitário, você precisa manipular o dependency graph sozinho. É típico fornecer versões somente de teste ou "mockadas" dessas dependências. Isso significa que você pode isolar apenas a lógica da classe ou método que está testando. (Isto é importante! Se você está testando um serviço, você não quer **também** acidentalmente gravar dados no seu banco de dados.)

### Criando um projeto de testes

É uma boa prática criar um projeto separado para seus testes, para que eles sejam mantidos separados do código da aplicação. O novo projeto de teste deve estar em um diretório próximo (não dentro) do diretório do seu projeto principal.

Se você está atualmente no diretório do seu projeto, o comando `cd` sobe um nível. (Este diretório raiz também será chamado `AspNetCoreTodo`). Em seguida, use este comando para criar um novo projeto de teste:

```
dotnet new xunit -o AspNetCoreTodo.UnitTests
```

O xUnit.NET é um framework de teste popular para código .NET que pode ser usado para escrever testes unitários e de integração. Como todos os outros, é um conjunto de pacotes NuGet que podem ser instalados em qualquer projeto. O modelo `dotnet new xunit` já inclui tudo o que você precisa.

Sua estrutura de diretórios deve estar assim:

```
AspNetCoreTodo/
    AspNetCoreTodo/
        AspNetCoreTodo.csproj
        Controllers/
        (etc...)

    AspNetCoreTodo.UnitTests/
        AspNetCoreTodo.UnitTests.csproj
```

Como o projeto de teste usará as classes definidas em seu projeto principal, você precisará adicionar uma referência ao projeto `AspNetCoreTodo`:

```
dotnet add reference ../AspNetCoreTodo/AspNetCoreTodo.csproj
```

Exclua o arquivo `UnitTest1.cs` criado automaticamente. Você está pronto para escrever seu primeiro teste.

> Se você estiver usando o Visual Studio Code, talvez seja necessário fechar e reabrir a janela do Visual Studio Code para que o autocomplete volte a funcionar no novo projeto.

### Escrevendo um teste de serviço

Dê uma olhada na lógica do método `AddItemAsync()` do `TodoItemService`:

```csharp
public async Task<bool> AddItemAsync(
    TodoItem newItem, ApplicationUser user)
{
    newItem.Id = Guid.NewGuid();
    newItem.IsDone = false;
    newItem.DueAt = DateTimeOffset.Now.AddDays(3);
    newItem.UserId = user.Id;

    _context.Items.Add(newItem);

    var saveResult = await _context.SaveChangesAsync();
    return saveResult == 1;
}
```

Esse método faz uma série de decisões ou suposições sobre o novo item (em outras palavras, executa a lógica de negócio no novo item) antes de realmente salvá-lo no banco de dados:

* A propriedade `UserId` deve ser configurada para o ID do usuário
* Novos itens devem estar sempre incompletos (`IsDone = false`)
* O título do novo item deve ser copiado de `newItem.Title`
* Novos itens devem sempre ser finalizados daqui a 3 dias

Imagine se você ou alguém refatorasse o método `AddItemAsync()` e se esquecesse dessa lógica de negócios. O comportamento do seu aplicativo pode mudar sem você perceber! Você pode evitar isso escrevendo um teste que verifique duas vezes se essa lógica de negócio não foi alterada (mesmo que a implementação interna do método seja alterada).

> Pode parecer improvável agora que você possa introduzir uma mudança na lógica de negócio sem perceber, mas fica muito mais difícil acompanhar as decisões e suposições em um projeto grande e complexo. Quanto maior o seu projeto, mais importante é ter verificações automatizadas que garantem que nada mudou!

Para escrever um teste unitário que irá verificar a lógica no `TodoItemService`, crie uma nova classe no seu projeto de teste:

**AspNetCoreTodo.UnitTests/TodoItemServiceShould.cs**

```csharp
using System;
using System.Threading.Tasks;
using AspNetCoreTodo.Data;
using AspNetCoreTodo.Models;
using AspNetCoreTodo.Services;
using Microsoft.EntityFrameworkCore;
using Xunit;

namespace AspNetCoreTodo.UnitTests
{
    public class TodoItemServiceShould
    {
        [Fact]
        public async Task AddNewItemAsIncompleteWithDueDate()
        {
            // ...
        }
    }
}
```

> Existem muitas maneiras diferentes de nomear e organizar testes, todos com diferentes prós e contras. Eu gosto de usar postfixing em minhas classes de teste com `Should` para criar uma frase legível com o nome do método de teste, mas sinta-se livre para usar seu próprio estilo!

O atributo `[Fact]` vem do pacote Nuget xUnit.NET e marca este método como um método de teste.

O `TodoItemService` requer um `ApplicationDbContext`, que normalmente está conectado ao seu banco de dados. Você não vai querer usar isso para testes. Em vez disso, você pode usar o provider do Entity Framework Core's in-memory database em seu código de teste. Como todo o banco de dados existe na memória, ele é apagado toda vez que o teste é reiniciado. E, como é um provider do Entity Framework Core, o `TodoItemService` não saberá a diferença!

Use um `DbContextOptionsBuilder` para configurar o in-memory database provider e, em seguida faça uma chamada para `AddItemAsync()`:

```csharp
var options = new DbContextOptionsBuilder<ApplicationDbContext>()
    .UseInMemoryDatabase(databaseName: "Test_AddNewItem").Options;

// Set up a context (connection to the "DB") for writing
using (var context = new ApplicationDbContext(options))
{
    var service = new TodoItemService(context);

    var fakeUser = new ApplicationUser
    {
        Id = "fake-000",
        UserName = "fake@example.com"
    };

    await service.AddItemAsync(new TodoItem
    {
        Title = "Testing?"
    }, fakeUser);
}
```

A última linha cria um novo to-do item chamado `Testing?`, E diz ao serviço para salvá-lo no banco de dados (in-memory).

Para verificar se a lógica de negócios foi executada corretamente, escreva mais alguns códigos abaixo do bloco `using` existente:

```csharp
// Use a separate context to read data back from the "DB"
using (var context = new ApplicationDbContext(options))
{
    var itemsInDatabase = await context
        .Items.CountAsync();
    Assert.Equal(1, itemsInDatabase);
    
    var item = await context.Items.FirstAsync();
    Assert.Equal("Testing?", item.Title);
    Assert.Equal(false, item.IsDone);

    // Item should be due 3 days from now (give or take a second)
    var difference = DateTimeOffset.Now.AddDays(3) - item.DueAt;
    Assert.True(difference < TimeSpan.FromSeconds(1));
}
```

A primeira assertion é uma verificação de integridade: nunca deve haver mais de um item salvo no in-memory database. Supondo que isso seja verdade, o teste recupera o item salvo com "FirstAsync" e, em seguida, afirma que as propriedades estão definidas para os valores esperados.

> Tanto os testtes unitários quanto os de integração geralmente seguem o padrão AAA (Arrange-Act-Assert): objetos e dados são configurados primeiro, depois alguma ação é executada e, finalmente, o teste verifica (confirma) que o comportamento esperado ocorreu.

Fazer Asserting em valores de data e hora é um pouco complicado, uma vez que comparar duas datas para igualdade falhará se até mesmo os milissegundos forem diferentes. Em vez disso, o teste verifica se o valor de `DueAt` está a menos de um segundo do valor esperado.

### Executando o teste

No terminal, execute este comando (verifique se você ainda está no diretório `AspNetCoreTodo.UnitTests`):

```
dotnet test
```

O comando `test` examina o projeto atual para testes (marcado com atributos `[Fact]` neste caso), e executa todos os testes que encontrar. Você verá uma saída semelhante a:

```
Starting test execution, please wait...
 Discovering: AspNetCoreTodo.UnitTests
 Discovered:  AspNetCoreTodo.UnitTests
 Starting:    AspNetCoreTodo.UnitTests
 Finished:    AspNetCoreTodo.UnitTests

Total tests: 1. Passed: 1. Failed: 0. Skipped: 0.
Test Run Successful.
Test execution time: 1.9074 Seconds
```

Agora você tem um teste fornecendo cobertura de teste do `TodoItemService`. Como um desafio extra, tente escrever testes de unidade que garantam:

* O método `MarkDoneAsync()` retorna false se for passado um ID que não existe
* O método `MarkDoneAsync()` retorna true quando um item válido é marcado como completo
* O método `GetIncompleteItemsAsync()` retorna apenas os itens pertencentes a um usuário em particular

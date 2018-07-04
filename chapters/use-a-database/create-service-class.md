## Criando uma nova service class

No capítulo *Introdução ao MVC*, você criou a classe `FakeTodoItemService` com tarefas a fazer hardcoded (fixas). Agora que você possui um `DbContext`, você pode criar uma nova service class que utilizará o Entity Framework Core para recuperar tarefas a fazer reais do banco de dados.

Exclua o arquivo `FakeTodoItemService.cs` e crie um novo conforme abaixo:

**Services/TodoItemService.cs**

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using AspNetCoreTodo.Data;
using AspNetCoreTodo.Models;
using Microsoft.EntityFrameworkCore;

namespace AspNetCoreTodo.Services
{
    public class TodoItemService : ITodoItemService
    {
        private readonly ApplicationDbContext _context;

        public TodoItemService(ApplicationDbContext context)
        {
            _context = context;
        }

        public async Task<TodoItem[]> GetIncompleteItemsAsync()
        {
            return await _context.Items
                .Where(x => x.IsDone == false)
                .ToArrayAsync();
        }
    }
}
```

Repare que utilizamos dependency injection assim como fizemos no capítulo *Introdução ao MVC*, a diferença é que agora é o `ApplicationDbContext` que está sendo injetado. O `ApplicationDbContext` já está sendo adicionado ao service container no método `ConfigureServices`, então ele está disponível para dependency injection aqui.

Vamos olhar mais de perto o código do método `GetIncompleteItemsAsync`. Primeiro, ele utiliza a propriedade `Items` do `DbContext` para acessar todas as tarefas a fazer no `DbSet`:

```csharp
var items = await _context.Items
```

Em seguida, o método `Where` é utilizado para filtrar apenas tarefas não finalizadas:

```csharp
.Where(x => x.IsDone == false)
```

O método `Where` é um recurso do C# chamado LINQ (**l**anguage **in**tegrated **q**uery), inspirado na programação funcional e torna fácil a escrita de queries SQL via código C#. Por baixo dos panos, o Entity Framework Core traduz o método `Where` em uma instrução SQL similar à `SELECT * FROM Items WHERE IsDone = 0`, ou a uma query de documento equivalente para bancos de dados NoSQL.

Finalmente, o método `ToArrayAsync` diz ao Entity Framework Core para recuperar todas as entidades que correspondem ao filtro e retorná-las como um array. O método `ToArrayAsync` é assíncrono (tipo de retorno `Task`), então é preciso esperá-lo finalizar.

Para que o método fique menor, você pode remover a variável intermediária `items` e apenas retornar o resultado da query diretamente (o que dá no mesmo):

```csharp
public async Task<TodoItem[]> GetIncompleteItemsAsync()
{
    return await _context.Items
        .Where(x => x.IsDone == false)
        .ToArrayAsync();
}
```

### Atualizando o service container

Como você excluiu a classe `FakeTodoItemService`, será necessário atualizar a linha que conecta à interface `ITodoItemService` no `ConfigureServices`:

```csharp
services.AddScoped<ITodoItemService, TodoItemService>();
```

O método `AddScoped` adiciona seu serviço ao service container utilizando o lifecycle **scoped**. Isto significa que novas instâncias da classe `TodoItemService` serão criadas durante cada requisição web. Isso é necessário para service classes que interagem com bancos de dados.

> Adicionar uma service class que interage com o Entity Framework Core (e seu banco de dados) com o lifecycle singleton (ou outro diferente de scoped) pode causar problemas por causa da forma que o Entity Framework Core gerencia conexões com bancos de dados. Por isso, utilize sempre o lifecycle scoped para serviços que utilizem o Entity Framework Core.

O `TodoController`, que depende de um` ITodoItemService` injetado, estará feliz e inconsciente da mudança nas services class, mas por baixo dos panos ele estará utilizando o Entity Framework Core e comunicando com um banco de dados real!

### Testando

Inicie a aplicação e navegue para `http://localhost:5000/todo`. Os items falsos (fake) sumiram e agora sua aplicação está rodando queries reais no banco de dados. Como ainda não há tarefas a fazer salvas no banco de dados, não há nada sendo exibido.

No próximo capítulo você adicionará mais recursos à nossa aplicação, iniciando com a inclusão de novas tarefas a fazer.

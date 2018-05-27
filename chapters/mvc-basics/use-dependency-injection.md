## Usar injeção de dependência
De volta ao `TodoController`, adicione algum código para trabalhar com `ITodoItemService`:

```csharp
public class TodoController : Controller
{
    private readonly ITodoItemService _todoItemService;

    public TodoController(ITodoItemService todoItemService)
    {
        _todoItemService = todoItemService;
    }

    public IActionResult Index()
    {
        // Get to-do items from database

        // Put items into a model

        // Pass the view to a model and render
    }
}
```

Como `ITodoItemService` está no namespace `Services`, você precisará adicionar uma declaração `using` no topo do arquivo:

```csharp
using AspNetCoreTodo.Services;
```

A primeira linha da classe declara uma variável privada para guardar uma referência à `ITodoItemService`. Esta variável permitirá usar o service a partir do método action `Index` (você verá como em um minuto).

A linha `public TodoController(ITodoItemService todoItemService)` define um **construtor** para a classe. O construtor é um método especial que é chamado quando você quer criar uma nova instância para a classe (a classe `TodoController`, neste caso). Ao adicionar um parâmetro `ITodoItemService` ao construtor, você declarou que para criar o `TodoController`, precisará fornecer um objeto que corresponda a interface `ITodoItemService`.

> Interfaces são incríveis porque ajudam a desacoplar (separar) a lógica da sua aplicação. Como o controller depende da interface `ITodoItemService`, e não de uma classe *específica*, ele não sabe ou se importa com qual classe realmente será recebida. Poderia ser a `FakeTodoItemService`, uma diferente que se comunique com um banco de dados, ou alguma outra! Contanto que implemente a interface, o controller pode usá-la. Isso torna realmente fácil testar partes da sua aplicação separadamente. Falarei sobre testes em detalhes no capítulo *Teste Automatizado*.

Agora você pode finalmente usar o `ITodoItemService` (através da variável privada que você declarou) no seu método action para obter itens de tarefa da sua camada service:

```csharp
public IActionResult Index()
{
    var items = await _todoItemService.GetIncompleteItemsAsync();

    // ...
}
```

Lembra que o método `GetIncompleteItemsAsync` retornou um `Task<TodoItem[]>`? Retornar um `Task` significa que o método não terá necessariamente um resultado imediato, mas você pode usar a keyword `await` para garantir que seu código espere até que o resultado esteja pronto antes de continuar.

O padrão `Task` é comum quando seu código chama um banco de dados ou um serviço de API, porque ele não será capaz de retornar um resultado real até que o banco de dados (ou rede) responda. Se você já usou promises ou callbacks em JavaScript ou outras linguagens, `Task` é a mesma ideia: a promessa de que haverá um resultado - em algum tempo no futuro.

> Se você já teve que lidar com "callback hell" em código JavaScript antigo, você está com sorte. Lidar com asynchronous code em .NET é muito mais fácil graças a mágica da keyword `await`! `await` permite que seu código pause em uma operação assíncrona, e então continue de onde parou quando o banco de dados ou solicitação de rede terminar. Enquanto isso, sua aplicação não fica bloqueada, pois, pode processar outras requisições conforme necessário. Este padrão é simples, mas leva um tempo para se acostumar, então não se preocupe se isso não fizer sentido agora. Apenas continue acompanhando!

O único problema é que você precisa atualizar a assinatura do método `Index` para retornar um `Task<IActionResult>` em vez de apenas `IActionResult`, e marcá-lo como `async`:

```csharp
public async Task<IActionResult> Index()
{
    var items = await _todoItemService.GetIncompleteItemsAsync();

    // Put items into a model

    // Pass the view to a model and render
}
```

Você está quase lá! Fez o `TodoController` depender da inteface `ITodoItemService`, mas ainda não disse ao ASP.NET Core que quer que o `FakeTodoItemService` seja o serviço real usado. Pode parecer óbvio agora já que você só tem uma classe que implementa `ITodoItemService`, mas depois terá múltiplas classes implementando a mesma interface, então ser explícito é preciso.

Declarar (ou "conectar") que classe concreta usar para cada interface é feito no método `ConfigureServices` da classe `Startup`. Neste momento, parece algo como isso:

**Startup.cs**

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // (... some code)

    services.AddMvc();
}
```

O trabalho do método `ConfigureServices` é adicionar coisas ao **service container**, ou a coleção de services que o ASP.NET Core conhece. A linha `services.AddMvc` adiciona os services que os sistemas internos do ASP.NET Core precisam (como um experimento, tente comentar esta linha). Quaisquer outros services que você quiser usar na sua aplicação precisam ser adicionados ao service container em `ConfigureServices`.

Adicione a linha seguinte em qualquer lugar dentro do método `ConfigureServices`:

```csharp
services.AddSingleton<ITodoItemService, FakeTodoItemService>();
```

Esta linha diz ao ASP.NET Core para usar o `FakeTodoItemService` sempre que a interface `ITodoItemService` for requisitada em um construtor (ou em qualquer outro lugar).

`AddSingleton` adiciona o seu service ao service container como um **singleton**. Isso significa que apenas uma cópia de `FakeTodoItemService` é criada, e será reutilizada sempre que o service for solicitado. Mais tarde, quando você escrever uma classe service diferente que fale com um banco de dados, usará uma outra abordagem (chamada **scoped**). Eu explicarei o motivo no capítulo *Use um banco de dados*.

É isso! Quando uma solicitação é recebida e encaminhada para o `TodoController`, o ASP.NET Core irá olhar para os services disponíveis e automaticamente fornecerá o `FakeTodoItemService` quando o controller pedir um `ITodoItemService`. Porque services são "injetados" pelo service container, esse padrão é chamado **dependency injection**.

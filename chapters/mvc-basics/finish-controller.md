## Finalizar o controller
O último passo é finalizar o código do controller. O controller agora tem uma to-do list da camada service, e é preciso colocar esses itens em um `TodoViewModel` e vincular esse model à view que você criou mais cedo:

**Controllers/TodoController.cs**

```csharp
public async Task<IActionResult> Index()
{
    var items = await _todoItemService.GetIncompleteItemsAsync();

    var model = new TodoViewModel()
    {
        Items = items
    };

    return View(model);
}
```

Caso ainda não tenha feito, certifique-se de que estas declarações `using` estão incluídas no começo do arquivo:

```csharp
using AspNetCoreTodo.Services;
using AspNetCoreTodo.Models;
```

Se você está usando Visual Studio ou Visual Studio Code, o editor irá sugerir estas declarações `using` quando você colocar o seu cursor em uma linha vermelha ondulada.

## Testar
Para iniciar a aplicação, pressione F5 (se você está usando Visual Studio ou Visual Studio Code), ou apenas digite `dotnet run` no terminal. Se o código compilar sem erros, o servidor irá iniciar na porta 5000 por padrão.

Se o seu navegador não abrir automaticamente, obra-o e navegue para http://localhost:5000/todo. Você verá a view criada, com os dados extraídos do seu falso banco de dados (por enquanto).

Embora seja possível ir direto para `http://localhost:5000/todo`, seria melhor adicionar um item chamado **My to-dos** ao navbar. Para fazer isso, você pode editar o arquivo de layout compartilhado (shared layout).

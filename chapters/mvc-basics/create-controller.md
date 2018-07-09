## Create a controller

Já existem alguns controllers no diretório Controllers do projeto, incluindo o `HomeController` que renderiza a tela padrão de boas-vindas que você vê quando visita `http://localhost:5000`. Você pode ignorar esse controllers por enquanto.

Crie um novo controller para a funcionalidade de to-do list, chamado `TodoController` e adicione o code seguinte:

**Controllers/TodoController.cs**

``` csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;

namespace AspNetCoreTodo.Controllers
{
    public class TodoController : Controller
    {
        // Actions go here
    }
}
```

Rotas que são tratadas por controllers são chamadas de **actions**, e são representadas por métodos na classe controller. Por exemplo, o `HomeController` contêm três métodos action (`Index`, `About`, e `Contact`) que são mapeados pelo ASP.NET Core para essas rotas URLs:

```
localhost:5000/Home         -> Index()
localhost:5000/Home/About   -> About()
localhost:5000/Home/Contact -> Contact()
```

Existe um número de convenções (padrões comuns) usados pelo ASP.NET Core, tal como o padrão que `FooController` se torna `/Foo`, e o nome do action `Index` pode ser deixado de fora da URL. Você pode customizar esse comportamento se quiser, mas por enquanto vamos seguir a convenção padrão.

Adicione um novo action chamado `Index` ao `TodoController`, substituindo o comentário `// Actions go here`:

```csharp
public class TodoController : Controller
{
    public IActionResult Index()
    {
        // Get to-do items from database

        // Put items into a model

        // Render view using the model
    }
}
```

Métodos action podem retornar views, dados JSON, ou códigos de estado HTTP como `200 OK` e `404 Not Found`. O tipo de retorno `IActionResult` lhe dá a flexibilidade de retornar qualquer um desses a partir do action.

É uma prática recomendada manter os controllers tão leves quanto possível. Neste caso, o controller será responsável por obter os itens da to-do list do banco de dados, colocar esses itens em um model que a view consiga entender e enviar a view de volta para o navegador do usuário.

Antes de escrever o restante do código do controller, você precisa criar um model e uma view.

## Adicionar uma classe de serviço
Você criou um model, uma view, e um controller. Antes de usar o model e visualizar no controller, você também precisa escrever um código que irá obter a to-do list do usuário de uma base de dados.

Você pode escrever este código  de banco de dados diretamente no controller, mas é uma prática melhor manter seu código separado. Por que? Em uma grande aplicação do mundo real, você terá que lidar com muitas preocupações:

* **Renderizar views** e manipular dados recebidos: isso é o que seu controller já faz.
* **Executar regra de negócio**, ou código e lógica relacionados ao propósito e "negócio" da sua aplicação. Em uma aplicação to-do list, regras de negócio significam decisões como definir uma data de vencimento padrão para novas atividades, ou apenas exibir tarefas que estão incompletas. Outros exemplos de regras de negócio incluem calcular o custo total baseado no preço do produto e taxas de imposto, ou checar se um jogador tem pontos suficientes para subir de nível em um jogo.
* **Salvar e recuperar** itens de um banco de dados.

Novamente, é possível fazer todas essas coisas em um único e grande controller, mas isso rapidamente se torna muito difícil de gerenciar e testar. Ao invés disso, é comum ver aplicações separadas em duas, três, ou mais "camadas" ou níveis onde cada camada lida com um (e apenas um) interesse. Isso ajuda a manter os controllers tão simples quanto possível, e facilita a realização de testes e alterações na regra de negócio e código de banco de dados mais tarde.

Separar sua aplicação dessa forma é, às vezes, chamado de **multi camadas** ou **arquiterura N-camadas**. Em alguns casos, os níveis (camadas) são isolados em projetos completamente separados, mas em outros casos apenas se refere a como as classes são organizadas e usadas. O importante é pensar sobre como dividir sua aplicação em pedaços gerenciáveis, e evitar ter controllers ou classes inchadas que tentam fazer todas as coisas.

Para este projeto, você usará duas camadas de aplicação: a **camada de apresentação** composta pelos controllers e views que interagem com o usuário, e a **camada de serviço** que contém regras de negócio e código de banco de dados. A camada de apresentação já existe, então o próximo passo é criar um serviço que manipule regras de negócio da to-do list e salve seus itens no banco de dados.

> A maioria dos grandes projetos usa uma arquitetura 3-camadas: uma camada de apresentação, uma camada de lógica de serviço e uma camada de repositório de dados. Um **repositório** é uma classe focada apenas em código de banco de dados (sem regra de negócio). Nesta aplicação, você vai juntar estas em uma única camada de serviço, pela simplicidade, mas sinta-se livre para experimentar diferentes maneiras de arquitetar o código.

### Criar uma interface

A linguagem C# inclui o conceito de **interfaces**, onde a definição dos métodos e propriedades de um objeto é separada da classe que realmente contém o código para esses métodos e propriedades. Interfaces facilitam manter suas classes desacopladas e fáceis de testar, como você verá aqui (mais tarde no capítulo *Teste Automatizado*). Você usará uma interface para representar o serviço que pode interagir com itens da to-do list no banco de dados.

Por convenção, interfaces são prefixadas com "I". Crie um novo arquivo no diretório Services:

**Services/ITodoItemService.cs**

```csharp
using System;
using System.Collections.Generic;
using System.Threading.Tasks;
using AspNetCoreTodo.Models;

namespace AspNetCoreTodo.Services
{
    public interface ITodoItemService
    {
        Task<TodoItem[]> GetIncompleteItemsAsync();
    }
}
```

Observe que o namespace desse aquivo é `AspNetCoreTodo.Services`. Namespaces são maneiras de organizar arquivos de código .NET, e é costume para o namespace seguir o diretório em que o arquivo está armazenado (`AspNetCoreTodo.Services` para arquivos no diretório `Services`, e assim por diante).

Porque esse arquivo (no namespace `AspNetCoreTodo.Services`) referencia a classe `TodoItem` (no namespace `AspNetCoreTodo.Models`), é necessário incluir uma declaração `using` no começo do arquivo para importar aquele namespace. Sem a declaração `using`, você verá um erro como:

```
The type or namespace name 'TodoItem' could not be found (are you missing a using directive or an assembly reference?)
```

Como isso é uma interface, não existe de fato nenhum código aqui, apenas a definição (ou **assinatura de método**) do método `GetIncompleteItemsAsync`. Este método não exige parâmetros e retorna um `Task<TodoItem[]>`.

> Se essa sintaxe parecer confusa, pense: "uma Task que contém array de TodoItems".

O tipo `Task` é semelhante a um future ou uma promise, e é utilizado aqui porque este método será **assíncrono**. Em outras palavras, o método pode não estar apto a retornar a to-do list imediatamente, pois é necessário ir conversar com o banco de dados primeiro. (Você vai ver mais sobre isso depois.)

### Criar a classe de serviço

Agora que a interface está definida, você está pronto para realmente criar a classe de serviço. Eu irei falar sobre o código de banco de dados em profundidade no capítulo *Use um banco de dados*, então por enquanto você o simulará e sempre irá retornar dois itens implementados:

**Services/FakeTodoItemService.cs**

```csharp
using System;
using System.Collections.Generic;
using System.Threading.Tasks;
using AspNetCoreTodo.Models;

namespace AspNetCoreTodo.Services
{
    public class FakeTodoItemService : ITodoItemService
    {
        public Task<TodoItem[]> GetIncompleteItemsAsync()
        {
            var item1 = new TodoItem
            {
                Title = "Learn ASP.NET Core",
                DueAt = DateTimeOffset.Now.AddDays(1)
            };

            var item2 = new TodoItem
            {
                Title = "Build awesome apps",
                DueAt = DateTimeOffset.Now.AddDays(2)
            };

            return Task.FromResult(new[] { item1, item2 });
        }
    }
}
```

Este `FakeTodoItemService` implementa a interface `ITodoItemService`, mas sempre retorna o mesmo array com dois `TodoItem`s. Você usará isso para testar o controller e a view, e então adicionar código de banco de dados real em *Use um banco de dados*.

## Adicionando novas tarefas a fazer

O usuário adicionará novas tarefas a fazer com um form (formulário) simples como o abaixo:

![Form final](final-form.png)

Implementar esta funcionalidade requer alguns passos:

* Adicionar um form na view
* Cria nova action no controller para manipular o form
* Adicionar código na camada de serviço para atualizar o banco de dados

### Adicionando o form

A view `Views/Todo/Index.cshtml` possui um placeholder para o form de inclusão de itens:

```html
<div class="panel-footer add-item-form">
  <!-- TODO: Add item form -->
</div>
```

Para manter as coisas organizadas e separadas, você criará o form como uma **partial view** (view parcial). Uma partial view é um pequeno pedaço de uma view maior codificada em um arquivo separado.

Crie a view `AddItemPartial.cshtml`:

**Views/Todo/AddItemPartial.cshtml**

```html
@model TodoItem

<form asp-action="AddItem" method="POST">
    <label asp-for="Title">Add a new item:</label>
    <input asp-for="Title">
    <button type="submit">Add</button>
</form>
```

O tag helper `asp-action` pode gerar uma URL para o form, assim como quando você o usa em um elemento `<a>`. Neste caso, o helper `asp-action` é substituído pelo caminho real para a rota `AddItem` que você criará:

```html
<form action="/Todo/AddItem" method="POST">
```

Adicionar um tag helper `asp-` ao elemento `<form>` também adiciona um campo oculto ao form que contém um token de verificação. Esse token de verificação pode ser usado para evitar ataques cross-site request forgery (CSRF). Você verificará o token quando escrever o código da action.

Isso cuida da criação da partial view. Agora, faça referência à visualização principal da view Todo:

**Views/Todo/Index.cshtml**

```html
<div class="panel-footer add-item-form">
  @await Html.PartialAsync("AddItemPartial", new TodoItem())
</div>
```

### Adicionando uma action

Quando um usuário clica em Adicionar no form que você acabou de criar, o navegador cria uma solicitação POST para `/Todo/AddItem` em seu aplicativo. Isso não funcionará agora, porque não há nenhuma action (ação) que possa manipular a rota `/Todo/AddItem`. Se você tentar agora, o ASP.NET Core retornará um erro "404 Not Found" (Não Encontrado).

Você precisará criar uma nova action chamada `AddItem` no` TodoController`:

```csharp
[ValidateAntiForgeryToken]
public async Task<IActionResult> AddItem(TodoItem newItem)
{
    if (!ModelState.IsValid)
    {
        return RedirectToAction("Index");
    }

    var successful = await _todoItemService.AddItemAsync(newItem);
    if (!successful)
    {
        return BadRequest("Could not add item.");
    }

    return RedirectToAction("Index");
}
```

Viu como a nova action `AddItem` aceita um parâmetro `TodoItem`? Este é o mesmo modelo `TodoItem` que você criou no capítulo _MVC basics_ para armazenar informações sobre um item de tarefa a fazer. Quando usado aqui como um parâmetro action, o ASP.NET Core executará automaticamente um processo chamado **model binding** (binding de modelo).

Model binding examina os dados em uma solicitação e tenta corresponder de maneira inteligente os campos de entrada com propriedades no model. Em outras palavras, quando o usuário envia este form e seus POSTs de navegador para essa action, o ASP.NET Core irá capturar as informações do form e colocá-las na variável `newItem`.

O atributo `[ValidateAntiForgeryToken]` antes da action diz ao ASP.NET Core que ele deve procurar (e verificar) o token de verificação oculto que foi adicionado ao form pelo tag helper `asp-action`. Essa é uma medida de segurança importante para evitar ataques cross-site request forgery (CSRF), em que os usuários podem ser induzidos a enviar dados de um site mal-intencionado. O token de verificação garante que seu aplicativo é realmente aquele que processou e enviou o form.

Dê uma olhada na view `AddItemPartial.cshtml` mais uma vez. A linha `@model TodoItem` na parte superior do arquivo informa ao ASP.NET Core que a visualização deve estar emparelhada com o modelo `TodoItem`. Isto torna possível usar `asp-for="Title"` na tag `<input>` para que o ASP.NET Core saiba que este elemento de entrada é para a propriedade `Title`.

Por causa da linha `@ model`, a partial view esperará passar um objeto` TodoItem` quando for renderizada. Passar um `new TodoItem` através de `Html.PartialAsync` inicializa o form com um item vazio. (Tente passar junto `{ Title = "hello" }` e veja o que acontece!)

Durante o model binding, todas as propriedades de modelo que não podem ser correspondidas com campos no request são ignoradas. Como o form inclui apenas o elemento de entrada `Title`, você pode esperar que as outras propriedades em `TodoItem` (o flag `IsDone`, a data de` DueAt`) estarão vazias ou conterão valores default (padrão).

> Em vez de reutilizar o model `TodoItem`, outra abordagem seria criar um model separado (como `NewTodoItem`) que é usado apenas para esta action e possui apenas as propriedades específicas (Title) necessárias para adicionar uma novoa tarefa a fazer. Model binding ainda é usado, mas dessa forma você separou o model usado para armazenar um item de tarefa a fazer no banco de dados a partir do model usado para vincular dados de request de entrada. Às vezes, isso é chamado de **binding model** ou **data transfer object** (DTO). Esse padrão de projetos (design pattern) é comum em projetos maiores e mais complexos.


Depois de vincular os dados do request ao model, o ASP.NET Core também realiza **validação de modelo** (model validation). A validação verifica se os dados ligados ao model do request recebido fazem sentido ou são válidos. Você pode adicionar atributos ao model para informar ao ASP.NET Core como ele deve ser validado.

O atributo `[Required]` na propriedade `Title` informa ao validador do model do ASP.NET Core para considerar o título inválido se estiver faltando ou em branco. Dê uma olhada no código da action `AddItem`: o primeiro bloco verifica se o `ModelState` (o resultado da validação do model) é válido. É costume fazer essa verificação de validação logo no início da action:

```csharp
if (!ModelState.IsValid)
{
    return RedirectToAction("Index");
}
```

Se o `ModelState` for inválido por qualquer motivo, o navegador será redirecionado para a rota `/Todo/Index`, que atualiza a página.

Em seguida, o controller chama o service layer para realizar a operação real do banco de dados de salvar a nova tarefa a fazer:

```csharp
var successful = await _todoItemService.AddItemAsync(newItem);
if (!successful)
{
    return BadRequest(new { error = "Could not add item." });
}
```

O método `AddItemAsync` retornará` true` ou `false` dependendo se a tarefa foi adicionada com sucesso ao banco de dados. Se falhar por algum motivo, a action retornará um erro HTTP `400 Bad Request` juntamente com um objeto que contenha uma mensagem de erro.

Finalmente, se tudo for concluído sem erros, a action redireciona o navegador para a rota `/Todo/Index`, que atualiza a página e exibe a nova lista atualizada de tarefas a fazer para o usuário.

### Adicionando um método de serviço (service method)

Se você estiver usando um editor de código que entenda C#, você verá linhas rabiscadas em `AddItemAsync` porque o método ainda não existe.

Como último passo, você precisa adicionar um método ao service layer. Primeiro, adicione-o à definição da interface em `ITodoItemService`:

```csharp
public interface ITodoItemService
{
    Task<TodoItem[]> GetIncompleteItemsAsync();

    Task<bool> AddItemAsync(TodoItem newItem);
}
```

Depois, a implementação real em `TodoItemService`:

```csharp
public async Task<bool> AddItemAsync(TodoItem newItem)
{
    newItem.Id = Guid.NewGuid();
    newItem.IsDone = false;
    newItem.DueAt = DateTimeOffset.Now.AddDays(3);

    _context.Items.Add(newItem);

    var saveResult = await _context.SaveChangesAsync();
    return saveResult == 1;
}
```

A propriedade `newItem.Title` já foi definida pelo model binder do ASP.NET Core, portanto, este método só precisa atribuir um ID e definir os valores padrão para as outras propriedades. Em seguida, a nova tarefa é adicionada ao contexto do banco de dados. Na verdade, ela não é salvo até que você chame `SaveChangesAsync()`. Se a operação for bem sucedida, o método `SaveChangesAsync()` retornará 1.

### Experimente

Execute a aplicação e adicione algumas tarefas à sua lista com o formulário. Como as tarefas estão sendo armazenadas no banco de dados, elas ainda estarão lá mesmo depois que a aplicação for parada e iniciada novamente.

> Como um desafio extra, tente adicionar um selecionador de data (date picker) usando HTML e JavaScript e deixe o usuário escolher uma data (opcional) para a propriedade "DueAt". Em seguida, use essa data em vez de sempre criar novas tarefas com prazo de 3 dias.

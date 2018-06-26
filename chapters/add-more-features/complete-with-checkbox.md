## Finalizando tarefas com um checkbox

Adicionar tarefas à sua lista de tarefas a fazer é ótimo, mas eventualmente você precisará concluí-las também. Na view `Views/Todo/Index.cshtml`, um checkbox é renderizado para cada tarefa a fazer:

```html
<input type="checkbox" class="done-checkbox">
```

Clicar no checkbox não faz nada (ainda). Assim como no último capítulo, você adicionará esse comportamento usando forms e actions. Nesse caso, você também precisará de um pequeno trecho de código JavaScript.

### Adicionando elementos de form à view

Primeiro, atualize a view e coloque cada checkbox entre um elemento `<form>`. Em seguida, adicione um elemento oculto (hidden) contendo o ID do item:

**Views/Todo/Index.cshtml**

```html
<td>
    <form asp-action="MarkDone" method="POST">
        <input type="checkbox" class="done-checkbox">
        <input type="hidden" name="id" value="@item.Id">
    </form>
</td>
```

Quando o loop `foreach` é executado na view e imprime uma linha para cada tarefa a fazer, uma cópia deste form existirá em cada linha. O campo hidden que contém o ID dtarefa permite que o código do controller indique qual checkbox foi marcado. (Sem isso, você seria capaz de dizer que *alguns* checkboxes foram marcadas, mas não quais.)

Se você executar sua aplicação agora, os checkboxes ainda não farão nada, porque não há nenhum botão de envio (submit) para dizer ao navegador para criar uma solicitação POST com os dados do form. Você poderia adicionar um botão de envio em cada checkbox, mas isso seria uma experiência ruim para o usuário. O idela é que ao clicar no checkbox o form seja enviado automaticamente. Você pode implementar este recurso utilizando JavaScript.

### Adicionando código JavaScript

Encontre o arquivo `site.js` no diretório` wwwroot/js` e adicione este código:

**wwwroot/js/site.js**

```javascript
$(document).ready(function() {

    // Wire up all of the checkboxes to run markCompleted()
    $('.done-checkbox').on('click', function(e) {
        markCompleted(e.target);
    });
});

function markCompleted(checkbox) {
    checkbox.disabled = true;

    var row = checkbox.closest('tr');
    $(row).addClass('done');

    var form = checkbox.closest('form');
    form.submit();
}
```

Este código utiliza jQuery (uma biblioteca JavaScript) para anexar código ao `click` de todos os checkboxes na página com a classe CSS `done-checkbox`. Quando um checkbox é clicado, o método `markCompleted()` é executado.

O método `markCompleted()` faz o seguinte:
* Adiciona o atributo `disabled` ao checkbox para que ele não seja clicado novamente
* Adiciona a classe CSS `done` à linha pai que contém o checkbox, que muda a aparência da linha com base nas regras CSS em `style.css`
* Envia (submete) o formulário para o servidor

Isso cuida da view e do código de frontend. Agora vamos adicionar uma nova action!

### Adicionando uma action ao controller

Como você imaginou, você precisa adicionar uma action chamada `MarkDone` no` TodoController`:

```csharp
[ValidateAntiForgeryToken]
public async Task<IActionResult> MarkDone(Guid id)
{
    if (id == Guid.Empty)
    {
        return RedirectToAction("Index");
    }

    var successful = await _todoItemService.MarkDoneAsync(id);
    if (!successful)
    {
        return BadRequest("Could not mark item as done.");
    }

    return RedirectToAction("Index");
}
```

Let's step through each line of this action method. First, the method accepts a `Guid` parameter called `id` in the method signature. Unlike the `AddItem` action, which used a model and model binding/validation, the `id` parameter is very simple. If the incoming request data includes a field called `id`, ASP.NET Core will try to parse it as a guid. This works because the hidden element you added to the checkbox form is named `id`.

Since you aren't using model binding, there's no `ModelState` to check for validity. Instead, you can check the guid value directly to make sure it's valid. If for some reason the `id` parameter in the request was missing or couldn't be parsed as a guid, `id` will have a value of `Guid.Empty`. If that's the case, the action tells the browser to redirect to `/Todo/Index` and refresh the page.

Next, the controller needs to call the service layer to update the database. This will be handled by a new method called `MarkDoneAsync` on the `ITodoItemService` interface, which will return true or false depending on whether the update succeeded:

```csharp
var successful = await _todoItemService.MarkDoneAsync(id);
if (!successful)
{
    return BadRequest("Could not mark item as done.");
}
```

Finally, if everything looks good, the browser is redirected to the `/Todo/Index` action and the page is refreshed.

With the view and controller updated, all that's left is adding the missing service method.

### Adicionando um método de serviço (service method)

First, add `MarkDoneAsync` to the interface definition:

**Services/ITodoItemService.cs**

```csharp
Task<bool> MarkDoneAsync(Guid id);
```

Then, add the concrete implementation to the `TodoItemService`:

**Services/TodoItemService.cs**

```csharp
public async Task<bool> MarkDoneAsync(Guid id)
{
    var item = await _context.Items
        .Where(x => x.Id == id)
        .SingleOrDefaultAsync();

    if (item == null) return false;

    item.IsDone = true;

    var saveResult = await _context.SaveChangesAsync();
    return saveResult == 1; // One entity should have been updated
}
```

This method uses Entity Framework Core and `Where()` to find an item by ID in the database. The `SingleOrDefaultAsync()` method will either return the item or `null` if it couldn't be found.

Once you're sure that `item` isn't null, it's a simple matter of setting the `IsDone` property:

```csharp
item.IsDone = true;
```

Changing the property only affects the local copy of the item until `SaveChangesAsync()` is called to persist the change back to the database. `SaveChangesAsync()` returns a number that indicates how many entities were updated during the save operation. In this case, it'll either be 1 (the item was updated) or 0 (something went wrong).

### Experimente

Run the application and try checking some items off the list. Refresh the page and they'll disappear completely, because of the `Where()` filter in the `GetIncompleteItemsAsync()` method.

Right now, the application contains a single, shared to-do list. It'd be even more useful if it kept track of individual to-do lists for each user. In the next chapter, you'll add login and security features to the project.

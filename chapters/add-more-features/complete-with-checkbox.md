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

Quando o loop `foreach` é executado na view e imprime uma linha para cada tarefa a fazer, uma cópia deste form existirá em cada linha. O campo hidden que contém o ID da tarefa permite que o código do controller indique qual checkbox foi marcado. (Sem isso, você seria capaz de dizer que *alguns* checkboxes foram marcadas, mas não quais.)

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

Vamos percorrer cada linha desse action method. Primeiro, o método aceita um parâmetro `Guid` chamado `id` na assinatura do método. Ao contrário da action `AddItem`, que usou um model e um model binding/validation, o parâmetro `id` é muito simples. Se os dados do request de entrada incluírem um campo chamado `id`, o ASP.NET Core tentará analisá-lo como um guid. Isso funciona porque o elemento hidden que você adicionou ao checkbox do form é chamado `id`.

Como você não está usando model binding, não há 'ModelState` para verificar a validade. Em vez disso, você pode verificar o guid diretamente para certificar-se de que é válido. Se por alguma razão o parâmetro `id` estiver faltando no request ou não puder ser analisado como um guid, o `id` terá um valor de `Guid.Empty`. Se for esse o caso, a action diz ao navegador para redirecionar para `/Todo/Index` e atualizar a página.

Em seguida, o controller precisa chamar a camada de serviço para atualizar o banco de dados. Isso será tratado por um novo método chamado `MarkDoneAsync` na interface `ITodoItemService`, que retornará verdadeiro ou falso dependendo se a atualização foi bem-sucedida:

```csharp
var successful = await _todoItemService.MarkDoneAsync(id);
if (!successful)
{
    return BadRequest("Could not mark item as done.");
}
```

Finalmente, se tudo correr bem, o navegador será redirecionado para a action `/Todo/Index` e a página é atualizada.

Com a view e o controller atualizados, tudo o que resta é adicionar o service method.

### Adicionando um método de serviço (service method)

Primeiro, adicione `MarkDoneAsync` à definição da interface:

**Services/ITodoItemService.cs**

```csharp
Task<bool> MarkDoneAsync(Guid id);
```

Depois, adicione a implementação concreta ao `TodoItemService`:

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

Esse método usa o Entity Framework Core e `Where()` para localizar uma tarefa por ID no banco de dados. O método `SingleOrDefaultAsync()` retornará a tarefa ou `null` se não puder ser encontrada.

Uma vez que você tenha certeza de que `item` não é nulo, é uma simples questão de configurar a propriedade `IsDone`:

```csharp
item.IsDone = true;
```

A alteração da propriedade afeta somente a cópia local da tarefa até que `SaveChangesAsync()` seja chamado para persistir a alteração no banco de dados. `SaveChangesAsync()` retorna um número que indica quantas entidades foram atualizadas durante a execução do método de salvar no banco de dados. Nesse caso, será 1 (a tarefa foi atualizada) ou 0 (algo deu errado).

### Experimente

Execute a aplicação e tente concluir algumas tarefas da lista. Atualize a página e elas desaparecerão completamente por causa do filtro `Where()` no método `GetIncompleteItemsAsync()`.

No momento, a aplicação contém uma única lista de tarefas compartilhada. Seria ainda mais útil ainda se pudéssemos ter o controle de listas de tarefas individuais para cada usuário. No próximo capítulo, você adicionará recursos de login e segurança ao projeto.

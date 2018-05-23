## Criar uma view
Views no ASP.NET Core são construídas utilizando a linguagem de template Razor, que combina HTML e código C#. (Se você já escreveu páginas usando Handlebars moustaches, ERB no Ruby on Rails, ou Thymeleaf no Java, você já tem a ideia básica.)

A maior parte do código da view é apenas HTML, com adições ocasionais de declarações C# para extrair dados do view model e transformá-los em texto ou HTML. As declarações C# são prefixadas com o símbolo `@`.

A view renderizada pela action `Index` do `TodoController` precisa pegar os dados do view model (uma sequência de itens de tarefa) e exibi-los em uma tabela agradável para o usuário. Por convenção, views são colocadas no diretório `Views` em um subdiretório correspondente ao nome do controller. O nome do arquivo da view é o nome da action com uma extensão `.cshtml`.

Crie um diretório `Todo` dentro do diretório `Views`, e adicione este arquivo:

**Views/Todo/Index.cshtml**

```html
@model TodoViewModel

@{
    ViewData["Title"] = "Manage your todo list";
}

<div class="panel panel-default todo-panel">
  <div class="panel-heading">@ViewData["Title"]</div>

  <table class="table table-hover">
      <thead>
          <tr>
              <td>&#x2714;</td>
              <td>Item</td>
              <td>Due</td>
          </tr>
      </thead>
      
      @foreach (var item in Model.Items)
      {
          <tr>
              <td>
                <input type="checkbox" class="done-checkbox">
              </td>
              <td>@item.Title</td>
              <td>@item.DueAt</td>
          </tr>
      }
  </table>

  <div class="panel-footer add-item-form">
    <!-- TODO: Add item form -->
  </div>
</div>
```

No começo do arquivo, a diretiva `@model` diz ao Razor qual model esperar que esta view seja ligada. O model é acessado através da propriedade `Model`.

Assumindo que existam itens de tarefa em `Model.Items`, a declaração `foreach` irá iterar sobre cada item de tarefa e renderizar uma linha de tabela (elemento `<tr>`) contendo o nome do item e sua data de vencimento. O checkbox também é renderizado, o que irá permitir que o usuário marque o item como completo.

### O arquivo de layout
Você deve estar se perguntando onde está o resto do HTML: o que aconteceu com a tag `<body>`, ou o cabeçalho e o rodapé da página? O ASP.NET Core usa uma layout view que define a estrutura básica onde todas as outras view's são renderizadas. Ela fica armazenada em `Views/Shared/_Layout.cshtml`.

O template padrão do ASP.NET Core inclui Bootstrap e jQuery neste arquivo de layout, então você pode criar rapidamente uma aplicação web. Naturalmente, você pode usar suas próprias bibliotecas CSS e JavaScript se assim quiser.

### Personalizando o stylesheet

O template padrão também inclui um stylesheet com algumas regras CSS básicas. O stylesheet é armazenado no diretório `wwwroot/css`. Adicione algumas novas regras de estilo CSS no fim do arquivo `site.css`:

**wwwroot/css/site.css**

```css
div.todo-panel {
  margin-top: 15px;
}

table tr.done {
  text-decoration: line-through;
  color: #888;
}
```

Você pode usar regras CSS como estas para modificar completamente a aparência das suas páginas.

ASP.NET Core e Razor podem fazer muito mais, como partial views e server-rendered view components, mas um simples layout e view são tudo que você precisa por enquanto. A documentação oficial do ASP.NET Core (em https://docs.asp.net) contém vários exemplos se você quiser aprender mais.

## Atualizar o layout

O arquivo de layout em `Views/Shared/_Layout.cshtml` contém o HTML "base" para cada view. Isso inclui a navbar, que é renderizado no topo de cada página.

Para adicionar um novo item na navbar, encontre o código HTML para os itens existentes:

**Views/Shared/_Layout.cshtml**

```html
<ul class="nav navbar-nav">
    <li><a asp-area="" asp-controller="Home" asp-action="Index">
        Home
    </a></li>
    <li><a asp-area="" asp-controller="Home" asp-action="About">
        About
    </a></li>
    <li><a asp-area="" asp-controller="Home" asp-action="Contact">
        Contact
    </a></li>
</ul>
```

Adicione o seu item que apontará para o controler `Todo` ao invés de `Home`:

```html
<li>
    <a asp-controller="Todo" asp-action="Index">My to-dos</a>
</li>
```

Os atributos `asp-controller` e `asp-action` no elemento `<a>` são chamados de **tag helpers**. Antes de a view ser renderizada, o ASP.NET Core substitui esses tag helpers por atributos HTML reais. Neste caso, uma URL para a rota `/Todo/Index` é gerada e adicionada ao elemento `<a>` como um atributo `href`. Isso significa que você não precisa implementar a rota para o `TodoController`. Ao invés disso, o ASP.NET Core a gera automaticamente para você.

> Se você já usou Razor no ASP.NET 4.x, você notará algumas mudanças de sintaxe. Em vez de usar `@Html.ActionLink()` para gerar um link para um action, tag helpers agora são a forma recomendada de criar links nas suas views. Tag helpers são úteis para forms, também (você verá porque em um capítulo posterior). Você pode aprender sobre outras tag helpers na documentação em https://docs.asp.net.

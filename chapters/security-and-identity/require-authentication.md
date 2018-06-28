## Solicitando autenticação

Muitas vezes você vai querer exigir que o usuário faça login antes de acessar certas partes de sua aplicação. Por exemplo, faz sentido mostrar a home page para todos (esteja você logado ou não), mas a lista de tarefas só deve ser exibida para quem fizer login.

Você pode usar o atributo `[Authorize]` no ASP.NET Core para exigir que o usuário esteja logado para executar uma action específica ou um controller inteiro. Para requerer autenticação para todas as ações do `TodoController`, adicione o atributo acima da primeira linha do controller:

**Controllers/TodoController.cs**

```csharp
[Authorize]
public class TodoController : Controller
{
    // ...
}
```

Adicione este comando `using` ao início do arquivo:

```csharp
using Microsoft.AspNetCore.Authorization;
```

Rode a aplicação e tente acessar `/ todo` sem estar logado. Você será redirecionado para a página de login automaticamente.

> O atributo `[Authorize]` está fazendo uma verificação de autenticação aqu e não uma verificação de autorização (apesar do nome do atributo). Posteriormente, você usará o atributo para verificar **a autenticação e a autorização**.

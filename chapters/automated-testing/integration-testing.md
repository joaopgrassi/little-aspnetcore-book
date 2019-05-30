## Testes de integração

Em comparação com testes unitários, os testes de integração são muito maiores em escopo e exercitam toda a pilha da aplicação. Em vez de isolar uma classe ou método, os testes de integração asseguram que todos os componentes do seu aplicativo estejam funcionando adequadamente: roteamento, controllers, services, banco de dados e assim por diante.

Os testes de integração são mais lentos e envolve mais do que os testes unitários, portanto, é comum que um projeto tenha muitos testes unitários pequenos, mas apenas um projeto de teste de integração.

Para testar a pilha inteira (incluindo o roteamento dos controllers), os testes de integração geralmente fazem chamadas HTTP para a aplicação da mesma forma que um navegador Web faria.

Para escrever testes de integração que fazem solicitações HTTP, você pode iniciar manualmente sua aplicação e testes ao mesmo tempo e escrever seus testes para fazer requisições para `http://localhost:5000`. No entanto, o ASP.NET Core oferece uma maneira mais agradável de hospedar sua aplicação para testes: a classe `TestServer`. `TestServer` pode hospedar sua aplicação pela duração do teste e interrompê-la automaticamente quando o teste for concluído.

### Criando um projeto de testes de integração

Se você está atualmente no diretório do seu projeto, o comando `cd` sobe um nível para o diretório root` AspNetCoreTodo`. Utilize este comando para criar um novo projeto de teste:

```
dotnet new xunit -o AspNetCoreTodo.IntegrationTests
```

Sua estrutura de diretórios deve estar assim:

```
AspNetCoreTodo/
    AspNetCoreTodo/
        AspNetCoreTodo.csproj
        Controllers/
        (etc...)

    AspNetCoreTodo.UnitTests/
        AspNetCoreTodo.UnitTests.csproj

    AspNetCoreTodo.IntegrationTests/
        AspNetCoreTodo.IntegrationTests.csproj
```

> Se preferir, você pode manter seus testes unitários e de integração no mesmo projeto. Para projetos grandes, é comum dividi-los, pois isso permite facilmente executá-los separadamente.

Como o projeto de teste usará as classes definidas em seu projeto principal, você precisará adicionar uma referência ao projeto principal:

```
dotnet add reference ../AspNetCoreTodo/AspNetCoreTodo.csproj
```

Você também precisará adicionar o pacote NuGet `Microsoft.AspNetCore.TestHost`:

```
dotnet add package Microsoft.AspNetCore.TestHost
```

Exclua o arquivo `UnitTest1.cs` criado por `dotnet new`. Você está pronto para escrever um teste de integração.

### Escrevendo um teste de integração

Existem algumas coisas que precisam ser configuradas no servidor de teste antes de cada teste. Em vez de sobrecarregar o teste com este código de configuração, você pode manter essa configuração em uma classe separada. Crie uma nova classe chamada `TestFixture`:

**AspNetCoreTodo.IntegrationTests/TestFixture.cs**

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Net.Http;
using Microsoft.AspNetCore.Hosting;
using Microsoft.AspNetCore.TestHost;
using Microsoft.Extensions.Configuration;

namespace AspNetCoreTodo.IntegrationTests
{
    public class TestFixture : IDisposable  
    {
        private readonly TestServer _server;

        public HttpClient Client { get; }

        public TestFixture()
        {
            var builder = new WebHostBuilder()
                .UseStartup<AspNetCoreTodo.Startup>()
                .ConfigureAppConfiguration((context, config) =>
                {
                    config.SetBasePath(Path.Combine(
                        Directory.GetCurrentDirectory(),
                        "..\\..\\..\\..\\AspNetCoreTodo"));
                    
                    config.AddJsonFile("appsettings.json");
                });

            _server = new TestServer(builder);

            Client = _server.CreateClient();
            Client.BaseAddress = new Uri("http://localhost:8888");
        }

        public void Dispose()
        {
            Client.Dispose();
            _server.Dispose();
        }
    }
}
```

Esta classe cuida da criação de um `TestServer` e ajudará a manter os testes limpos e ordenados.

Agora você está (finalmente) pronto para escrever um teste de integração. Crie uma nova classe chamada `TodoRouteShould`:

**AspNetCoreTodo.IntegrationTests/TodoRouteShould.cs**

```csharp
using System.Net;
using System.Net.Http;
using System.Threading.Tasks;
using Xunit;

namespace AspNetCoreTodo.IntegrationTests
{
    public class TodoRouteShould : IClassFixture<TestFixture>
    {
        private readonly HttpClient _client;

        public TodoRouteShould(TestFixture fixture)
        {
            _client = fixture.Client;
        }

        [Fact]
        public async Task ChallengeAnonymousUser()
        {
            // Arrange
            var request = new HttpRequestMessage(
                HttpMethod.Get, "/todo");

            // Act: request the /todo route
            var response = await _client.SendAsync(request);

            // Assert: the user is sent to the login page
            Assert.Equal(
                HttpStatusCode.Redirect,
                response.StatusCode);

            Assert.Equal(
                "http://localhost:8888/Account" +
                "/Login?ReturnUrl=%2Ftodo",
                response.Headers.Location.ToString());
        }
    }
}
```

Esse teste faz uma requisição anônima (sem login) à rota `/todo` e verifica se o navegador é redirecionado para a página de login.

Esse cenário é um bom candidato para um teste de integração, porque envolve vários componentes da aplicação: o sistema de roteamento, o controller, o fato de o controller estar marcado com `[Authorize]` e assim por diante. Também é um bom teste, pois garante que você nunca removerá acidentalmente o atributo `[Authorize]` e tornará a visão dos to-do items acessível a todos.

## Executando o teste

Execute o teste no terminal com o comando `dotnet test`. Se tudo estiver funcionando corretamente, você verá uma mensagem de sucesso:

```
Starting test execution, please wait...
 Discovering: AspNetCoreTodo.IntegrationTests
 Discovered:  AspNetCoreTodo.IntegrationTests
 Starting:    AspNetCoreTodo.IntegrationTests
 Finished:    AspNetCoreTodo.IntegrationTests

Total tests: 1. Passed: 1. Failed: 0. Skipped: 0.
Test Run Successful.
Test execution time: 2.0588 Seconds
```

## Finalizando

Testes é um tópico amplo e há muito mais a aprender. Este capítulo não aborda testes de interface do usuário (UI testing) ou testes de frontend (JavaScript), que provavelmente merecem livros inteiros próprios. Você deve, no entanto, ter as habilidades e os conhecimentos básicos necessários para aprender mais sobre testes e para praticar escrita de testes para suas próprias aplicações.

A documentação do ASP.NET Core (https://docs.asp.net) e o Stack Overflow são ótimos recursos para aprender mais e encontrar respostas quando você fica confuso.

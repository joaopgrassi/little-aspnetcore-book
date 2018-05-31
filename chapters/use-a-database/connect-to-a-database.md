## Conectando-se a um banco de dados

Existem algumas coisas que você precisa para utilizar o Entity Framework Core para se conectar a um banco de dados. Desde quando você utilizou o comando `dotnet new` e o modelo MVC + Individual Auth para definir seu projeto, você os já tem:

* **O pacote NuGet Entity Framework Core**. Este pacote já está incluso por padrão em todos os projetos ASP.NET Core.

* **Um banco de dados** (naturalmente). O arquivo `app.db` no diretório raiz do projeto é um pequeno banco de dados SQLite criado para você através do comando `dotnet new`. O SQLite é um motor de banco de dados leve que é executado sem a necessidade de instalar ferramentas extras em seu computador, possibilitando um desenvolvimento fácil e rápido.

* **Uma classe de contexto de banco de dados (database context class ou `DbContext`)**. Uma classe de contexto de banco de dados é uma classe C# de interface com seu banco de dados. É ela que possui o código para interagir com um banco de dados para ler e salvar informações. Uma classe de contexto já existe no arquivo `Data/ApplicationDbContext.cs`.

* **Uma connection string (string de conexão)**. Independentemente de você estar se conectando a um banco de dados local em arquivo (como o SQLite) ou a um banco de dados remoto, você definirá uma connection string com o nome ou endereço do banco de dados que deseja se conectar. Isto já está configurado para você no arquivo `appsettings.json`: a connection string para o banco de dados SQLite de nosso projeto é `DataSource=app.db`.

O Entity Framework Core utiliza a classe de contexto de banco de dados em conjunto com a connection string para estabelecer uma conexão com um banco de dados. Você precisa dizer ao Entity Framework Core qual contexto, qual connection string e qual provedor de banco de dados utilizar no método `ConfigureServices` da classe `Startup`. Aqui está o que foi definido para você, graças ao template que utilizamos:

```csharp
services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlite(
        Configuration.GetConnectionString("DefaultConnection")));
```

Este código adiciona o `ApplicationDbContext` ao service container e diz ao Entity Framework Core para utilizar o provedor de banco de dados SQLite com a connection string configurada no arquivo `appsettings.json`.

Como você pode ver, o comando `dotnet new` cria uma série de coisas para você! O banco de dados está configurado e pronto para uso. No entanto, ele não possui nenhuma tabela para armazenar nossas tarefas a fazer. Para armazenar a entidade `TodoItem`, será necessário atualizar a classe de contexto e migrar (Entity Framework Migrations) nosso banco de dados.

## Utilizando Entity Framework Migrations

O Entity Framework Migrations monitora as alterações na estrutura do banco de dados ao longo do tempo. Com ele é possível desfazer (rollback) mudanças no banco de dados ou criar um novo banco de dados com base na estrutura de outro banco de dados já existente. Com o Migrations você tem um log completo de modificações no seu banco de dados, como inclusão e remoção de colunas e tabelas, por exemplo.

No capítulo anterior você adicionou a tabela `Items` ao `DbContext`. Como agora o `DbContext` manipula uma tabela que não existe no banco de dados, você precisar criar o Migrations para atualizar o banco de dados:

```
dotnet ef migrations add AddItems
```

O comando acima cria um Migrations chamado `AddItems` analisando todas as alterações que você fez no `DbContext`.

> Se por acaso ocorrer o erro `No executable found matching command "dotnet-ef"`, verifique se você está no diretório correto. O comando acima precisa ser executado no diretório raiz do projeto (o diretório no qual o arquivo `Program.cs` está).

Agora verifique o diretório `Data/Migrations` e você verá uma série de novos arquivos criados:

![Migrations Criadas](migrations.png)

O primeiro arquivo Migrations (com nome `00_CreateIdentitySchema.cs`) foi criado e aplicado com o comando `dotnet new`. Seu novo Migrations `AddItem` está com a data/hora de criação no nome do arquivo.

> Você pode ver a lista de Migrations criados com o comando `dotnet ef migrations list`.

Se você abrir seu arquivo Migrations, verá dois métodos chamados `Up` e `Down`:

**Data/Migrations/<date>_AddItems.cs**

```csharp
protected override void Up(MigrationBuilder migrationBuilder)
{
    // (... some code)

    migrationBuilder.CreateTable(
        name: "Items",
        columns: table => new
        {
            Id = table.Column<Guid>(nullable: false),
            DueAt = table.Column<DateTimeOffset>(nullable: true),
            IsDone = table.Column<bool>(nullable: false),
            Title = table.Column<string>(nullable: true)
        },
        constraints: table =>
        {
            table.PrimaryKey("PK_Items", x => x.Id);
        });

    // (some code...)
}

protected override void Down(MigrationBuilder migrationBuilder)
{
    // (... some code)

    migrationBuilder.DropTable(
        name: "Items");

    // (some code...)
}
```

O método `Up` é executado quando você aplica o Migrations no banco de dados. Como você incluiu um `DbSet<TodoItem>` ao `DbContext`, o Entity Framework Core criará uma tabela `Items` (com colunas equivalentes às propriedades da entidade `TodoItem`) quando você aplicar este Migrations.

O método `Down` faz o contrário: se você precisar desfazer (rollback) o Migrations, a tabela `Items` será apagada.

### Solução Alternativa (Workaround) para as Limitações do SQLite

O SQLite possui algumas limitações com o Migrations. Até que estas limitações sejam solucionadas, utilize a solução alternativa abaixo:

* Comente ou remova as linhas com o comando `migrationBuilder.AddForeignKey` no método `Up`.
* Comente ou remova quaisquer linhas com o comando `migrationBuilder.DropForeignKey` no método `Down`.

Se você utilizar um banco de dados relacional completo, como o SQL Server ou o MySQL, esta solução alternativa (vulgo "hack") não é necessária.

### Aplicando o Migrations

O último passo para criar Migrations é efetivamente aplicá-los ao banco de dados:

```
dotnet ef database update
```

O comando acima fará com que o Entity Framework Core de fato crie a tabela `Items` no banco de dados.

> Se você precisar efetuar rollback do banco de dados, você terá de fornecer o nome do Migrations anterior:
> `dotnet ef database update CreateIdentitySchema`
> Este comando executará os métodos `Down` de todos os Migrations mais recentes do que o Migrations que você especificou.

> Se você precisar apagar o banco de dados por completo para iniciar do zero no Migrations atual, utilize o comando `dotnet ef database drop` seguido pelo comando `dotnet ef database update`.

Pronto! Agora o banco de dados e o `DbContext` estão preparados para uso. A seguir, você utilizará o `DbContext` na camada de serviço.

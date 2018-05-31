## Atualizando o contexto do banco de dados (`DbContext`)

Ainda não há muita coisa acontecendo em nosso contexto do banco de dados:

**Data/ApplicationDbContext.cs**

```csharp
public class ApplicationDbContext 
             : IdentityDbContext<ApplicationUser>
{
    public ApplicationDbContext(
        DbContextOptions<ApplicationDbContext> options)
        : base(options)
    {
    }

    protected override void OnModelCreating(ModelBuilder builder)
    {
        base.OnModelCreating(builder);
        // ...
    }
}
```

Adicione uma propriedade com tipo `DbSet` à classe `ApplicationDbContext`, logo abaixo de seu construtor:

```csharp
public ApplicationDbContext(
    DbContextOptions<ApplicationDbContext> options)
    : base(options)
{
}

public DbSet<TodoItem> Items { get; set; }

// ...
```

Um `DbSet` representa uma tabela do banco de dados. Ao criar uma propriedade com tipo `DbSet<TodoItem>` chamada `Items`, você está dizendo ao Entity Framework Core que você quer armazenar entidades `TodoItem` em uma tabela de banco de dados chamada `Items`.

Você atualizou a classe de contexto de banco de dados, mas ainda há um pequeno problema: o contexto e o banco de dados agora estão dessincronizados, pois ainda não há uma tabela chamada `Items` em nosso banco de dados. (Apenas atualizar o código da classe de contexto não altera o banco de dados em si.)

Para atualizar o banco de dados refletindo as alterações que você acabou de fazer, você precisa utilizar um recurso chamado "migration".

> Se você já tiver um banco de dados existente, procure na web pelo termo "scaffold-dbcontext existing database" e leia a documentação da Microsoft sobre como utilizar a ferramenta `Scaffold-DbContext` para fazer engenharia reversa automática de seu banco de dados nos respectivos `DbContext` e modelo de classes.

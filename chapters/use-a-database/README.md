# Utilizando banco de dados

Escrever código de banco de dados pode ser um problema. A não ser que você realmente saiba o que está fazendo, incluir SQL "hardcoded" em sua aplicação é uma péssima ideia. Um **ORM** (object-relational mapper) torna muito mais fácil a escrita de código para interação com banco de dados adicionando uma camada de abstração entre seu código fonte e o banco de dados em si. O Hibernate do Java e o ActiveRecord do Ruby são exemplos de ORMs bem conhecidos.

Existem muitos ORMs para .NET, incluindo um criado pela Microsoft e incluso com o ASP.NET Core por padrão: o Entity Framework Core. O Entity Framework Core torna fácil se conectar com diversos tipos de bancos de dados, e permite que você crie queries com C# que podem ser mapeadas para classes POCO.

> Lembra-se de como criamos uma interface de serviço para desacoplar o código do controller da classe de serviço concreta? O Entity Framework Core é como uma grande interface de seu banco de dados. Seu código C# torna-se agnóstico ao banco de dados e você pode mudar para diferentes tipos de bancos de dados sem afetar o código de sua aplicação.

Entity Framework Core permite conexões com bancos de dados relacionais como SQL Server, PostgreSQL, MySQL e também com bancos de dados NoSQL, como Mongo. Durante o desenvolvimento de nossa aplicação, utilizaremos o SQLite por ser fácil de configurar.

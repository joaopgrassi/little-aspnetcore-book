# Adicionando Pacotes (Packages) Nuget
Uma das grandes vantagens da utilização de um ecossistema maduro como o .NET é a ampla disponibilidade de plugins e pacotes de terceiros. Similar a outros sistemas de pacotes, você pode fazer download e instalar pacotes .NET que o ajudarão a resolver quaisquer problemas imagináveis.

O NuGet (https://www.nuget.org) é, ao mesmo tempo, o gerenciador e o repositório de pacotes oficial do .NET. Você pode pesquisar pacotes NuGet na web e instalá-los em sua máquina local através do terminal de seu sistema operacional (ou por GUI, se estiver utilizando o Visual Studio).

## Instalando o Pacote "Humanizer"
No final do último capítulo, nossa aplicação exibe as tarefas a fazer no formato abaixo:

![Datas em formato ISO 8601](iso8601.png)

A coluna de datas de vencimento (due date) está em um formato excelente para máquinas (ISO 8601), porém péssimo para humanos. Não seria melhor se simplesmente exibirmos "X days from now"?

Você mesmo poderia escrever o código de conversão de datas no formato ISO 8601 para um formato mais amigável, porém existe um caminho mais rápido.

O pacote NuGet Humanizer soluciona este problema disponibilizando métodos para humanizar praticamente qualquer coisa: datas, horas, durações, números etc. É um projeto open-source fantástico e útil publicado sob a licença MIT permissiva.

Para adicioná-lo ao seu projeto, execute o comando abaixo em seu terminal:

```
dotnet add package Humanizer
```

Se você olhar o arquivo de projeto `AspNetCoreTodo.csproj` após executar este comando, verá uma nova linha `PackageReference` que referencia o pacote NuGet `Humanizer`.

## Utilizando o Humanizer nas views

Para utilizar um pacote NuGet em seu código, geralmente basta adicionar uma instrução `using` importando o pacote no início de seu arquivo fonte.

Como o Humanizer será utilizado para formatar datas nas views, você pode adicioná-lo diretamente nas próprias views. Primeira, adicione a instrução `@using` no topo da view:

**Views/Todo/Index.cshtml**

```html
@model TodoViewModel
@using Humanizer

// ...
```

Em seguida, atualize a linha que gera a propriedade `DueAt` para utilizar o método `Humanize` do pacote NuGet Humanizer:

```html
<td>@item.DueAt.Humanize()</td>
```

Agora as datas estão muito mais legíveis:

![Datas Legíveis para Humanos](friendly-dates.png)

Existem pacotes disponíveis no NuGet para tudo, desde realizar parse de XML à machine learning e até a postar no Twitter. O próprio ASP.NET Core não é nada mais do que uma coleção de pacotes NuGet adicionados ao seu projeto.

> O arquivo de projeto criado com o comando `dotnet new mvc` inclui uma simples referência ao pacote `Microsoft.AspNetCore.All`, que é um conveniente "metapackage" que referencia todos os pacotes ASP.NET Core necessários para um projeto padrão nesta tecnologia. Desta maneira, não são necessárias grandes quantidades de referências a pacotes em seu arquivo de projeto.

No próximo capítulo, você utilizará outro pacote NuGet (chamado Entity Framework Core) para interagir com bancos de dados.

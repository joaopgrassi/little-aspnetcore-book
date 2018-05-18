## Hello World em C# #
Antes de aprofundarmos no ASP.NET Core, tente criar e executar uma aplicação C# simples.

Você pode executar todos os passos a seguir através da linha de comando. Primeiro, abra o Terminal (ou PowerShell no WIndows). Navegue até a pasta que você quer armazenar os projetos, como o seu diretorio de Documentos:

```
cd Documents
```

Utilize o comando `dotnet` para criar um novo projeto:

```
dotnet new console -o CsharpHelloWorld
```

O comando `dotnet new` cria um novo projeto .NET em C# como padrão. O parâmetro `console` seleciona o template para um console application (um programa que exibe textos no seu terminal). O parâmetro `-o CsharpHelloWorld` diz ao comando `dotnet net` para criar um novo diretorio chamado `CsharpHelloWorld` com todos os arquivos do projeto. Entre neste diretório:

```
cd CsharpHelloWorld
```

`dotnet new console` cria um programa em C# básico que imprime o texto `Hello World!` no seu terminal. O programa é composto por dois arquivos: Um arquivo de projeto (com a extensão `.csproj`) e um arquivo com o código C# (com a extensão `.cs`). Se você abrir os arquivos no seu editor, você vai ver isso:

**CsharpHelloWorld.csproj**

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>netcoreapp2.0</TargetFramework>
  </PropertyGroup>

</Project>
```

O arquivo do projeto é feito em XML e define todas as propriedades sobre o projeto. Posteriormente, quando você referenciar outros pacotes, eles serão listados aqui (muito proximo ao arquivo `package.json` do npm). Você não precisa editar este arquivo com muita frequência.

**Program.cs**

```csharp
using System;

namespace CsharpHelloWorld
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Hello World!");
        }
    }
}
```

`static void Main` é o ponto de entrada de um programa C#, e por convenção ele é inserido em uma classe (um tipo de estrutura do codigo ou modulo) chamado `Program`. A declaração `using` no inicio é para importar a classe 'System' e disponibilizar para o seu código na sua classe.

Dentro do diretorio do seu projeto, execute o comando `dotnet run` para executar o seu programa. Você vai ver escrito no seu console após a execução do programa:

```
dotnet run

Hello World!
```

Isso é tudo para criar a estrutura e executar um programa em .NET! À seguir, você vai fazer exatamente a mesma coisa para uma aplicação ASP.NET Core.

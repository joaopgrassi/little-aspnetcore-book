## Criar um projeto em ASP.NET Core
Se voce ainda está no diretorio que criou para o exemplo Hello World, volte uma pasta acima para a sua pasta Documentos ou o diretório padrão:

```
cd ..
```

Em seguida, crie um novo diretorio para armazenar todo o seu projeto, e entre neste diretório:

```
mkdir AspNetCoreTodo
cd AspNetCoreTodo
```

Em seguida, crie o seu projeto com o comando `dotnet new`, desta vez com algumas opções a mais:

```
dotnet new mvc --auth Individual -o AspNetCoreTodo
cd AspNetCoreTodo
```

Este comando vai criar um novo projeto com o template `mvc`, e adicionar a autenticação e segurança para o projeto. (Eu vou falar sobre segurança no capitulo *Segurança e identidade*.)

> Você deve estar se perguntando porque eu tenho um diretório chamado `AspNetCoreTodo` dentro de outro diretorio chamado `AspNetCoreTodo`. O diretório principal pode conter um ou mais diretorios de projetos, algumas vezes, ele é chamado de **diretório de solução **. Mais para frente, você vai adicionar mais diretorios de projetos, todos dentro de um diretório de soluções.

Você está vendo alguns arquivos dentro deste novo diretório do projeto. Tudo o que você precisa fazer para executar o projeto é:

```
dotnet run

Now listening on: http://localhost:5000
Application started. Press Ctrl+C to shut down.
```

Ao invés de imprimir no console e sair do programa, ele inicia um servidor web e aguarda as requisições na porta 5000.

Abra o seu navegador e acesse a página `http://localhost:5000`. Voce vai ver a página padrão do ASP.NET Core, o que indica que o seu projeto está funcionando! Quando você finalizar, pressione Ctrl-C na janela do terminal e o servidor web será finalizado.

### As partes de um projeto ASP.NET Core
O template `dotnet new mvc` inicializa todos os arquivos e diretorios para você. Aqui está algumas informações importantes que você vai precisar:

* Os arquivos **Program.cs** e **Startup.cs** são responsaveis por inicializar o webserver e o pipeline do ASP.NET Core. A classe `Startup` é onde você pode adicionar middleware que gerencia e modifica a chegada das requisições, e também provê os conteudos estaticos e as paginas de erros. É onde voce vai adicionar o seu próprio serviço para o conteiner de injeção de dependencia (vamos tratar sobre isso mais para frente).

* O diretório **Models**, **Views**, e **Controllers** contém os componentes da arquitetura Model-View-Controller (MVC). Você vai conhecer os tres no próximo capitulo.

* O diretório **wwwroot** contém todos os arquivos estaticos, como CSS, JavaScript e as imagens. Arquivos dentro do `wwwroot` serão oferecidos como um conteudo estatico, e podem ser empacotados e minificados automaticamente.

* O arquivo **appsettings.json** contém todas as configurações do ASP.NET Core que serão carregadas na inicialização. Você pode utilizar para armazenar a sua string de conexão com a base de dados ou outras coisas que você não quer deixar hard-code no seu código.

### Dicas para o Visual Studio Code

If you're using Visual Studio Code for the first time, here are a couple of helpful tips to get you started:

* **Open the project root folder**: In Visual Studio Code, choose File - Open or File - Open Folder. Open the `AspNetCoreTodo` folder (the root directory), not the inner project directory. If Visual Studio Code prompts you to install missing files, click Yes to add them.

* **F5 to run (and debug breakpoints)**: With your project open, press F5 to run the project in debug mode. This is the same as `dotnet run` on the command line, but you have the benefit of setting breakpoints in your code by clicking on the left margin:

![Breakpoint in Visual Studio Code](breakpoint.png)

* **Lightbulb to fix problems**: If your code contains red squiggles (compiler errors), put your cursor on the code that's red and look for the lightbulb icon on the left margin. The lightbulb menu will suggest common fixes, like adding a missing `using` statement to your code:

![Lightbulb suggestions](lightbulb.png)

* **Compile quickly**: Use the shortcut `Command-Shift-B` or `Control-Shift-B` to run the Build task, which does the same thing as `dotnet build`.

> These tips apply to Visual Studio (not Code) on Windows too. If you're using Visual Studio, you'll need to open the `.csproj` project file directly. Visual Studio will later prompt you to save the Solution file, which you should save in the root directory (the first `AspNetCoreTodo` folder). You can also create an ASP.NET Core project directly within Visual Studio using the templates in File - New Project.

### A note about Git

If you use Git or GitHub to manage your source code, now is a good time to do `git init` and initialize a Git repository in the project root directory:

```
cd ..
git init
```

Make sure you add a `.gitignore` file that ignores the `bin` and `obj` directories. The Visual Studio template on GitHub's gitignore template repo (https://github.com/github/gitignore) works great.

There's plenty more to explore, so let's dive in and start building an application!

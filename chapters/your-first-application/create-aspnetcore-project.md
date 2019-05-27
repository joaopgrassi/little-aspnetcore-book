## Criar um projeto em ASP.NET Core
Se voce ainda está no diretorio que criou para o exemplo Hello World, volte uma pasta acima para a sua pasta Documentos ou o diretório padrão:

```
cd ..
```

Em seguida, crie um novo diretorio para armazenar o seu projeto "TODO", e entre neste diretório:

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

> Você deve estar se perguntando porque eu tenho um diretório chamado `AspNetCoreTodo` dentro de outro diretorio chamado `AspNetCoreTodo`. O diretório principal pode conter um ou mais diretorios de projetos, algumas vezes, ele é chamado de **diretório de solução**. Mais para frente, você vai adicionar mais diretorios de projetos, todos dentro de um diretório de soluções.

Você está vendo alguns arquivos dentro deste novo diretório do projeto. Tudo o que você precisa fazer para executar o projeto é:

```
dotnet run

Now listening on: http://localhost:5000
Application started. Press Ctrl+C to shut down.
```

Ao invés de imprimir no console e sair do programa, ele inicia um servidor web e aguarda as requisições na porta 5000.

Abra o seu navegador e acesse a página `http://localhost:5000`. Voce vai ver a página padrão do ASP.NET Core, o que indica que o seu projeto está funcionando! Quando você quiser finalizar o servidor, pressione Ctrl-C na janela do terminal e o servidor web será finalizado.

### As partes de um projeto ASP.NET Core
O template `dotnet new mvc` inicializa todos os arquivos e diretorios para você. Aqui está algumas informações importantes que você vai precisar:

* Os arquivos **Program.cs** e **Startup.cs** são responsaveis por inicializar o webserver e o pipeline do ASP.NET Core. A classe `Startup` é onde você pode adicionar o middleware que gerencia e modifica a chegada das requisições, e também provê os conteudos estaticos e as paginas de erros. É onde voce vai adicionar o seu próprio serviço para o conteiner de injeção de dependencia (vamos tratar sobre isso mais para frente).

* O diretório **Models**, **Views**, e **Controllers** contém os componentes da arquitetura Model-View-Controller (MVC). Você vai conhecer os tres no próximo capitulo.

* O diretório **wwwroot** contém todos os arquivos estaticos, como CSS, JavaScript e as imagens. Arquivos dentro do `wwwroot` serão oferecidos como um conteudo estatico, e podem ser empacotados e minificados automaticamente.

* O arquivo **appsettings.json** contém todas as configurações do ASP.NET Core que serão carregadas na inicialização. Você pode utilizar para armazenar a sua string de conexão com a base de dados ou outras coisas que você não quer deixar hard-code no seu código.

### Dicas para o Visual Studio Code

Se você está utiliznado o Visual Studio Code pela primeira vez, aqui vão algumas dicas para te ajudar neste inicio:

* **Abra a pasta raiz do projeto**: No Visual Studio Code, selecione Arquivo - Abrir ou Arquivo - Abrir Pasta. Abra a pasta `AspNetCoreTodo` (a pasta raiz), e não a pasta de dentro do projeto. Se o Visual Studio Code exibir uma mensagem para você instalar alguns arquivos que estão faltando, clique em Sim e adicione eles.

* **F5 para executar (e debugar os breakpoints)**: Com o seu projeto aberto, pressione F5 para executar o seu projeto em modo debug. Isso é o mesmo que `dotnet run` na linha de comando, mas você tem o beneficio de configurar os breakpoints no seu codigo clicando na margem da esquerda:

![Breakpoint no Visual Studio Code](breakpoint.png)

* **Lampada para resolver os problemas**: Se o seu código contem linhas vermelhas (erros de compilação), coloque o cursor no código que está com a linha vermelha e mova o mouse para o icone de lampada na margem da esquerda. O menu vai sugerir algumas correções, como adicionar uma declaração `using` que está faltando no seu código:

![Sugestões da Lampada](lightbulb.png)

* **Compilar rapidamente**: Use as teclas de atalho `Command-Shift-B` ou `Control-Shift-B` para executar a tarefa de Build, isso é o mesmo que `dotnet build`.

> Estas dicas aplicam também para o Visual Studio no Windows também. Se você está utilizando o Visual Studio, voce pode abrir o arquivo `.csproj` diretamente. O Visual Studio vai te perguntar para salvar o arquivo de Solução do projeto, que voce deve salvar da pasta raiz do projeto (A primeira pasta `AspNetCoreTodo`). Você pode criar um projeto ASP.NET Core diretamente do Visual Studio usando os templates que estão em Arquivo - Novo Projeto (File - New Project).

### Uma nota sobre o Git

Se voce utiliza Gi ou GitHub para gerenciar o seu código fonte, agora é uma boa hora para realizar um `git init` e inicializar o seu repositorio Git na pasta raiz do seu projeto:

```
cd ..
git init
```

Não se esqueca de adicionar o arquivo `.gitignore` para não considerar as pastas `bin` e `obj`. O template do gitignore, no repositorio do Visual Studio, no GitHub(https://github.com/github/gitignore) atende perfeitamente.

Há muito mais para explorar, então vamos mergulhar de cabeça e iniciar uma nova aplicação!
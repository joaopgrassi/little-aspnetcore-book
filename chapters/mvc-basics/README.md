# MVC noções básicas
Neste capítulo, você irá explorar o sistema MVC no ASP.NET Core. **MVC** (Model-View-Controller) é um padrão para construir aplicações web utilizado em quase todo framework web (Ruby on Rails e Express são exemplos famosos), além de frameworks JavaScript frontend como Angular. Aplicativos móveis para iOS e Android, também usam uma variação do MVC.

Como o nome sugere, o MVC possui três componentes: models, views, e controllers. **Controllers** lidam com requisições recebidas de um cliente ou navegador web e tomam decisões sobre qual código executar. **Views** são templates (geralmente HTML mais um processador de templates como Handlebars, Pug, ou Razor) que recebem os dados adicionados a eles e então os exibem ao usuário. **Models** mantêm os dados que são adicionados as views, ou os dados que são informados pelo usuário.

Um padrão comum para código MVC é:

* O controller recebe uma requisição e procura alguma informação na base de bados
* O controller cria um model com a informação e o atribui a uma view
* A view é renderizada e exibida no navegador do usuário
* O usuário clica em um botão ou submete um formulário, que envia uma nova requisição ao controller, e o ciclo se repete

Se você já trabalhou com MVC em outras linguagens, vai se sentir à vontade no ASP.NET Core MVC. Se você é novo no MVC, este capítulo vai lhe ensinar as noções básicas e ajudará você a começar.

## O que você vai construir
O exercício "Olá mundo" de MVC está construindo uma aplicação de lista de tarefas. É um ótimo projeto já que é pequeno e possui escopo simples, mas passa por cada parte do MVC e cobre muitos dos conceitos que você usaria em uma aplicação maior.

Neste livro, você vai construir uma aplicação de tarefas que permite que o usuário adicione itens à sua lista e os marque como concluídos quando estiverem completos. Mais especificamente, você criará:

* Uma aplicação web servidor (as vezes chamada de "backend") utilizando ASP.NET Core, C#, e o padrão MVC
* Uma base de dados para armazenar os itens de tarefas do usuário utilizando o mecanismo de banco de dados SQLite e um sistema chamado de Entity Framework Core
* Páginas web e uma interface com a qual o usuário irá interagir através do seu browser, usando HTML, CSS, e JavaScript (chamada de "frontend")
* Um formulário de login e verificações de segurança para que a lista de tarefas de cada usuário se mantenha privada

Parece bom? Vamos contruí-la! Se você ainda não criou um novo projeto ASP.NET Core usando `dotnet new mvc`, siga os passos do capítulo anterior. Você deve ser capaz de construir e executar o projeto e ver a tela de boas-vindas padrão.

# The Little ASP.NET Core Book

*by Nate Barbettini*

Copyright &copy; 2018. Todos os direitos reservados.

ISBN: 978-1-387-75615-5

Publicado sobre a licença Creative Commons Attribution 4.0. Você é livre para compartilhar, copiar e redistribuir este livro em qualquer formato, ou remixar e transformá-lo para qualquer propósito (até mesmo comercial). Você deve forcener os créditos apropriados e disponiblizar um link para a licença

Para mais informações, visite https://creativecommons.org/licenses/by/4.0/

## Introdução
Obrigado por obter o Little ASP.NET Core Book! Eu escrevi esse resumido livro com intuito em ajudar desenvolvedores e pessoas interessadas em desenvolvimento web a aprender sobre ASP.NET Core, um novo framework para desenvolver aplicações web e APIs.

O Little ASP.NET Core Book é organizado como um tutorial. Você vai desenvolver uma aplicação do início ao fim e aprender:

* O básico sobre o padrão MVC (Model-View-Controller)
* Como código front-end (HTML, CSS, Javascript) funciona em conjunto com código back-end
* O que é _dependency injection_ e porque é útil
* Como consultar e salvar dados em um banco de dados
* Como adicionar log-in, registro de usuários e segurança
* Como fazer deploy da aplicação na web

Não se preocupe, você não precisa conhecer nada sobre ASP.NET Core (ou quaisquer um acima) para começar.

## Antes de começar
O código da versão final da aplicação que você irá desenvolver está disponível no GitHub:

https://www.github.com/nbarbettini/little-aspnetcore-todo

Sinta-se livre em fazer o download se você quiser ver o projeto final ou comparar enquanto você trabalha em seu código.

O livro em si é atualizado frequentemente com correções de erros e novo conteúdo. Se você está lendo um PDF, e-book ou versão impressa, verifique o site oficial ([littleasp.net/book](http://www.littleasp.net/book)) para informa-se se existe uma versão atualizada disponível. Na última página do livro você encontra informações sobre versão e log das alterações.

### Lendo na sua própria língua
Graças a alguns fantásticos contribuidores multilíngues, o Little ASP.NET Core Book foi traduzido em outras línguas:

* [**ASP.NET Core El Kitabı**](https://sahinyanlik.gitbooks.io/kisa-asp-net-core-kitabi/) (Turkish)
 	 
* [**简明 ASP.NET Core 手册**](https://windsting.github.io/little-aspnetcore-book/book/) (Chinese)

* **ASP.NET Core - Guia de bolso** (Brazilian Portuguese) (in-progress)


## Para quem é esse livro
Se você é novo em programação, esse livro irá apresentar-lhe padrões e conceitos usados para criar aplicações web modernas. Você vai aprender como criar uma web app (e como todas as partes se integram) construindo uma do zero! Enquanto esse pequeno livro não será capaz de cobrir absolutamente tudo que você precisa saber sobre progração, ele oferecerá um ponto de início para que você possa aprofundar-se sobre tópicos mais avançados.

Se você já programa em alguma linguagem back-end como Node, Python, Ruby, Go ou Java, você notará muitas idéias familiares como _MVC_, _view templates_ e _dependency injection_. O código será em C#, mas não será muito diferente do que você já conhece.

Se você é um desenvolvedor ASP.NET, você vai se sentir em casa! ASP.NET Core adiciona novas ferramentas e re-utiliza (e simplifica) coisas que você já conhece. Abaixo você encontrará algumas das diferenças.

Não importa sua experiência anterior com desenvolvimento web, esse livro vai ensinar-lhe tudo que você precisa para criar uma aplicação web simples e útil usando ASP.NET Core. Você aprenderá como criar funcionalidades usando código back-end e front-end, como interagir com um banco de dados e como publicar sua aplicação para o mundo todo acessar.

## O que é ASP.NET Core?
O ASP.NET Core é um framework web desenvolvido pela Microsoft com o intuito de criar applicações web, APIS e micro-serviços. O ASP.NET Core utiliza padrões comuns como o MVC (_Model-View-Controller_), _dependency injection_ e um _request pipeline_ composto de middlewares. É open-source sobre a licença Apache 2.0, a qual significa que o código fonte é abertamente disponível e a comunidade é encorajada em contribuir com correção de bugs e novas funcionalidades.

ASP.NET Core roda sobre o .NET runtime, similar a JVM (Java Virtual Machine) do Java, ou o interpretador do Ruby. Você pode criar aplicações ASP.NET Core em diversas linguages (C#, Visual Basic, F#). C# é a escolha mais popular, e será a que iremos utilizar nesse livro. Você pode criar e executar aplicações ASP.NET Core em Windows, Mac e Linux.

## Porque precisamos de mais um framework web?
Existem diversos ótimos frameworks web que podemos utilizar: Node/Express, Spring, Ruby on Rails, Django, Laravel, entre outors. Então, quais são as vantagens do ASP.NET Core?

* **Velocidade.** ASP.NET Core é rápido. Devido ao código .NET ser compilado, ele é executado muito mais rápido do que código escritos em linguagens interpretadas, como por exemplo Javascript ou Ruby. O ASP.NET Core é também otimizado para tarefas _async_ e _multithreading_. É comum verificar um ganho de performance de 5-10x mais em relação código escrito em Node.js.

* **Ecosistema.** ASP.NET Core pode ser novo, mas .NET já existe por um longo tempo. Existem milhares de pacotes disponíveis no NuGet (O gerenciador de pacotes do .NET, assim npm, Ruby gems ou Maven). Existem por exemplo pacotes para deserializar JSON, para conectar-se a banco de dados, gerar PDF's e praticamente qualquer outra coisa que você possa imaginar.

* **Segurança.** E equipe da Microsoft responsável pelo ASP.NET Core leva segurança a sério, com isso, tudo é construído e pensado para ser seguro desde o início. O ASP.NET Core cuida de diversos items de segurança, como sanitização de dados de entrada (_input sanitization_) e ataques cross-site request forgery (CSRF), tudo para que dessa forma os desenvolvedores não precisem se preocupar.
Você também tem o grande benefício do _static typing_ do compilador do .NET, que é como ter um _linter_ paranóico ligado o tempo todo. Com isso, fazer algo de errado ou não intencionado em seu código se torna muito mais difícil

## .NET Core and .NET Standard
Throughout this book, you'll be learning about ASP.NET Core (the web framework). I'll occasionally mention the .NET runtime, the supporting library that runs .NET code. If this already sounds like Greek to you, just skip to the next chapter!

You may also hear about .NET Core and .NET Standard. The naming gets confusing, so here's a simple explanation:

**.NET Standard** is a platform-agnostic interface that defines features and APIs. It's important to note that .NET Standard doesn't represent any actual code or functionality, just the API definition. There are different "versions" or levels of .NET Standard that reflect how many APIs are available (or how wide the API surface area is). For example, .NET Standard 2.0 has more APIs available than .NET Standard 1.5, which has more APIs than .NET Standard 1.0.

**.NET Core** is the .NET runtime that can be installed on Windows, Mac, or Linux. It implements the APIs defined in the .NET Standard interface with the appropriate platform-specific code on each operating system. This is what you'll install on your own machine to build and run ASP.NET Core applications.

And just for good measure, **.NET Framework** is a different implementation of .NET Standard that is Windows-only. This was the only .NET runtime until .NET Core came along and brought .NET to Mac and Linux. ASP.NET Core can also run on Windows-only .NET Framework, but I won't touch on this too much.

If you're confused by all this naming, no worries! We'll get to some real code in a bit.

## A note to ASP.NET 4 developers
If you haven't used a previous version of ASP.NET, skip ahead to the next chapter.

ASP.NET Core is a complete ground-up rewrite of ASP.NET, with a focus on modernizing the framework and finally decoupling it from System.Web, IIS, and Windows. If you remember all the OWIN/Katana stuff from ASP.NET 4, you're already halfway there: the Katana project became ASP.NET 5 which was ultimately renamed to ASP.NET Core.

Because of the Katana legacy, the `Startup` class is front and center, and there's no more `Application_Start` or `Global.asax`. The entire pipeline is driven by middleware, and there's no longer a split between MVC and Web API: controllers can simply return views, status codes, or data. Dependency injection comes baked in, so you don't need to install and configure a container like StructureMap or Ninject if you don't want to. And the entire framework has been optimized for speed and runtime efficiency.

Alright, enough introduction. Let's dive in to ASP.NET Core!

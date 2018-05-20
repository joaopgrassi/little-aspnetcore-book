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
O ASP.NET Core é um framework web desenvolvido pela Microsoft com o intuito de criar applicações web, APIs e micro-serviços. O ASP.NET Core utiliza padrões comuns como o MVC (_Model-View-Controller_), _dependency injection_ e um _request pipeline_ composto de middlewares. É open-source sobre a licença Apache 2.0, a qual significa que o código fonte é abertamente disponível e a comunidade é encorajada em contribuir com correção de bugs e novas funcionalidades.

ASP.NET Core é executado sobre o .NET runtime, similar a JVM (Java Virtual Machine) do Java, ou o interpretador do Ruby. Você pode criar aplicações ASP.NET Core em diversas linguagens (C#, Visual Basic, F#). C# é a escolha mais popular, e será a que iremos utilizar nesse livro. Você pode criar e executar aplicações ASP.NET Core em sistemas Windows, Mac e Linux.

## Porque precisamos de mais um framework web?
Existem diversos ótimos frameworks web que já podemos utilizar: Node/Express, Spring, Ruby on Rails, Django, Laravel, entre outors. Então, quais são as vantagens do ASP.NET Core?

* **Velocidade.** ASP.NET Core é rápido. Devido ao código .NET ser compilado, ele é executado muito mais rápido do que código escrito em linguagens interpretadas, como por exemplo o Javascript ou Ruby. O ASP.NET Core é também otimizado para tarefas _async_ e ambientes _multithreading_. É comum verificar um ganho de performance de 5-10x mais em relação código escrito em Node.js.

* **Ecosistema.** ASP.NET Core pode ser novo, mas o .NET já existe por um longo tempo. Existem milhares de pacotes disponíveis no NuGet (gerenciador de pacotes do .NET, assim como o npm, Ruby gems ou Maven). Existem por exemplo pacotes para deserializar JSON, para conectar-se a um banco de dados, gerar PDF's e praticamente qualquer outra coisa que você possa imaginar.

* **Segurança.** E equipe da Microsoft responsável pelo ASP.NET Core leva segurança a sério, com isso, tudo é construído e pensado para ser seguro desde o início. O ASP.NET Core cuida de diversos items de segurança, como sanitização de dados de entrada (_input sanitization_) e ataques cross-site request forgery (CSRF). Tudo para que dessa forma os desenvolvedores não precisem se preocupar.
Você também tem o grande benefício do _static typing_ do compilador do .NET, que é como ter um _linter_ paranóico ligado o tempo todo. Com isso, fazer algo de errado ou não intencionado em seu código se torna muito mais difícil

## .NET Core e .NET Standard
Ao longo desse livro, você irá aprender sobre ASP.NET Core (o framework web). Ocasionalmente eu mencionarei o .NET runtime, a biblioteca de suporte que roda o código .NET. Se isso já soa grego para você, pule para o próximo capítulo!

Você talvez também tenha ouvido algo sobre .NET Core e .NET Standard. Os nomes podem ser um pouco confusos, então segue uma explicação:

**.NET Standard** é uma interface de platforma-agnóstica que define funcionalidades e APIs. É importante notar que o .NET Standard não representa nenhum código ou funcionalidade de fato, somente a definição de API. Existem diferentes "versões" ou níveis do .NET Standard que refletem quantas APIs estão disponíveis (ou quão ampla é a área que a API cobre). Por exemplo, o .NET Standard 2.0 possuí mais APIs disponíveis do que o .NET Standard 1.5, que por sua vez possuí mais APIs do que o .NET Standard 1.0.

**.NET Core** é o .NET runtime que pode ser instalado em Windows, Mac ou Linux. Ele implementa as APIs definidas na interface do .NET Standard com o apropriado código para cada plataforma e sistema operacional. É isso que você irá instalar em seu computador para fazer build e executar applicações ASP.NET Core.

E somente como boa medida, **.NET Framework** é uma diferente implementação do .NET Standard disponível somente para Windows. Esse era o único .NET runtime existente até a chegada do .NET Core, que trouxe .NET para Mac e Linux. ASP.NET Core também pode rodar somente no Windows com .NET Framework, mas não iremos abordar esse assunto.

Se você está confuso com todas essas nomenclaturas, não se preocupe! Nós iremos colocar a mão na massa em breve.

## Uma nota para desenvolvedores ASP.NET 4
Se você nunca usou uma versão antiga do ASP.NET, pule para o próximo capítulo.

ASP.NET Core é uma completa re-escrita do ASP.NET, com foco em modernizar o framework e finalmente desacoplá-lo do System.Web, IIS e Windows. Se você lembra todas as coisas do OWIN/Katana do ASP.NET 4, você já tem meio caminho andado: O projeto Katana se tornou o ASP.NET 5, que no final, foi renomeado para ASP.NET Core.

Devido ao legado do Katana, a classe `Startup` é o centro de tudo; não existe mai `Application_Start` ou `Global.asax`. Todo o pipeline é movido por middlewares, e não existe mais separação entre MVC e Web API: Controllers podem simplesmente retornar views, status codes ou dados. Dependency injection está disponível no framework, então você não é obrigado a instalar e configurar outro container como StructureMap ou Ninject. Outro importante fator é que todo o framework foi otimizado para velocidade e eficiência do runtime.

Certo, já chega de introdução. Vamos começar com ASP.NET Core!

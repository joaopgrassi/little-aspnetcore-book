# The Little ASP.NET Core Book (ASP.NET Core - Guia de Bolso)

*por Nate Barbettini*

Copyright &copy; 2018. Todos os direitos reservados.

ISBN: 978-1-387-75615-5

Publicado sobre a licença Creative Commons Attribution 4.0. Você é livre para compartilhar, copiar e redistribuir este livro em qualquer formato, ou remixar e transformá-lo para qualquer propósito (até mesmo comercial). Você deve fornecer os créditos apropriados e disponibilizar um link para a licença

Para mais informações, visite https://creativecommons.org/licenses/by/4.0/

## Introdução
Obrigado por obter o Little ASP.NET Core Book! Eu escrevi esse resumido livro com o intuito de ajudar desenvolvedores e pessoas interessadas em desenvolvimento web a aprender sobre ASP.NET Core, um novo framework para desenvolver aplicações web e APIs.

"The Little ASP.NET Core Book" é organizado como um tutorial. Você desenvolverá uma aplicação do início ao fim e aprenderá:

* O básico sobre o padrão MVC (Model-View-Controller)
* Como código front-end (HTML, CSS, Javascript) funciona em conjunto com código back-end
* O que é dependency injection e porque ela é útil
* Como consultar e salvar dados em um banco de dados
* Como adicionar log-in, registro de usuários e segurança
* Como fazer deploy da aplicação na web

Não se preocupe, você não precisa conhecer nada sobre ASP.NET Core (ou qualquer um dos itens acima) para começar.

## Antes de começar
O código da versão final da aplicação que você irá desenvolver está disponível no GitHub:

https://www.github.com/nbarbettini/little-aspnetcore-todo

Sinta-se livre para fazer download caso queira ver o produto final, ou comparar com seu código enquanto segue os tutoriais do livro.

O livro em si é atualizado frequentemente com correções de erros e novos conteúdos. Se você está lendo um PDF, e-book, ou versão impressa, verifique o site oficial ([littleasp.net/book](http://www.littleasp.net/book)) para informar-se sobre novas versões atualizadas. Na última página do livro existem informações sobre versões e log de alterações.

### Lendo em seu próprio idioma
Graças a alguns fantásticos contribuidores multilíngues, The Little ASP.NET Core Book foi traduzido nos seguintes idiomas:

* [**ASP.NET Core El Kitabı**](https://sahinyanlik.gitbooks.io/kisa-asp-net-core-kitabi/) (Turco)
 	 
* [**简明 ASP.NET Core 手册**](https://windsting.github.io/little-aspnetcore-book/book/) (Chinês Simplificado)

* [**ASP.NET Core - Guia de Bolso**](https://github.com/joaopgrassi/little-aspnetcore-book) (Português do Brasil - em andamento)

## Para quem é esse livro
Se você é novo em programação, esse livro irá lhe apresentar padrões e conceitos usados para criar aplicações web modernas. Você vai aprender como criar uma aplicação web (e como todas as partes se integram) construindo uma do zero! Enquanto esse pequeno livro não será capaz de cobrir absolutamente tudo que você precisa saber sobre programação, ele oferecerá um ponto de partida para que você possa aprofundar-se sobre tópicos mais avançados após completar sua leitura.

Se você já programa em alguma linguagem back-end como Node, Python, Ruby, Go ou Java, você notará muitas ideias familiares como MVC, view templates e dependency injection. O código será em C#, mas não será muito diferente do que você já conhece.

Se você é um desenvolvedor ASP.NET, se sentirá em casa! ASP.NET Core adiciona novas ferramentas e reutiliza (e simplifica) coisas que você já conhece. Abaixo você encontrará algumas das diferenças.

Independentemente de sua experiência anterior com desenvolvimento web, esse livro lhe ensinará tudo o que você precisa para criar uma aplicação web simples e útil usando ASP.NET Core. Você aprenderá como criar funcionalidades usando código back-end e front-end, como interagir com um banco de dados e como publicar sua aplicação para o mundo todo acessar.

## O que é ASP.NET Core?
ASP.NET Core é um framework web criado pela Microsoft para desenvolvimento de aplicações web, APIs e micro serviços. Ele utiliza padrões comuns como o MVC (Model-View-Controller), dependency injection e um request pipeline composto por middlewares. É open-source sobre a licença Apache 2.0, o que significa que o código fonte é aberto e disponibilizado gratuitamente e que a comunidade de desenvolvedores é encorajada a contribuir com correções de erros e novas funcionalidades.

ASP.NET Core é executado sobre o runtime do .NET, similar à JVM (Java Virtual Machine) e ao interpretador do Ruby. Você pode criar aplicações ASP.NET Core em diversas linguagens de programação (C#, Visual Basic, F#). C# é a escolha mais popular, e será a que iremos utilizar nesse livro. Você pode criar e executar aplicações ASP.NET Core em Windows, Mac e Linux.

## Porque precisamos de mais um framework web?
Existem diversos ótimos frameworks web que já podemos utilizar: Node/Express, Spring, Ruby on Rails, Django, Laravel, entre outros. Então, quais são as vantagens do ASP.NET Core?

* **Velocidade.** ASP.NET Core é rápido. Devido ao código .NET ser compilado, ele é executado muito mais rápido do que código escrito em linguagens interpretadas, como Javascript e Ruby, por exemplo. O ASP.NET Core também é otimizado para tarefas assíncronas e para ambientes com processamentos paralelos. É comum verificar um ganho de performance de 5x à 10x com ASP.NET Core em comparação à códigos escritos em Node.js.

* **Ecosistema.** ASP.NET Core pode ser novo, mas o .NET já existe há bastante tempo. Existem milhares de pacotes disponíveis no NuGet (gerenciador de pacotes do .NET, similar ao Npm, ao Ruby Gems e ao Maven). Existem, por exemplo, pacotes para deserializar JSON, para conectar-se com diversos bancos de dados, gerar PDF e praticamente qualquer outra coisa que você possa imaginar.

* **Segurança.** A Microsoft leva a segurança à sério, e o ASP.NET Core foi construído pensando em segurança desde o início. ASP.NET Core cuida de diversos itens de segurança, como sanitização de dados de entrada e ataques cross-site request forgery (CSRF - Falsificação de Solicitação Entre Sites), de forma que o desenvolvedor não precise se preocupar com isso. Você também tem o grande benefício da static typing (tipagem estática) do compilador .NET, que é como ter um linter (ferramenta para verificar erros em códigos fontes) paranoico ligado o tempo todo. Com isso, fazer algo de errado ou não intencionado em seu código se torna muito mais difícil.

## .NET Core e .NET Standard
Ao longo desse livro, você irá aprender sobre ASP.NET Core (o framework web). Ocasionalmente, o .NET runtime (biblioteca que executa códigos .NET) também será mencionado. Se soar como Grego para você, apenas pule para o próximo capítulo!

Talvez você também tenha ouvido sobre .NET Core e .NET Standard. Os nomes podem ser um pouco confusos, então segue uma explicação simplificada:

**.NET Standard** é uma interface de plataforma agnóstica que define funcionalidades e APIs. É importante notar que o .NET Standard não representa nenhum código ou funcionalidade de fato, somente a definição de API. Existem diferentes "versões" ou níveis do .NET Standard que refletem quantas APIs estão disponíveis (ou quão ampla é a área que a API cobre). Por exemplo, o .NET Standard 2.0 possuí mais APIs disponíveis do que o .NET Standard 1.5, que por sua vez possui mais APIs do que o .NET Standard 1.0.

**.NET Core** é o .NET runtime que pode ser instalado em Windows, Mac ou Linux. Ele implementa as APIs definidas na interface do .NET Standard com código apropriado para cada plataforma dos sistemas operacionais suportados. É isso que você irá instalar em seu computador para compilar e executar aplicações ASP.NET Core.

E somente para constar, **.NET Framework** é uma implementação específica do .NET Standard para Windows. Esse era o único runtime .NET existente até a chegada do .NET Core, que trouxe o .NET para Mac e Linux. ASP.NET Core também pode rodar somente em Windows com o .NET Framework, mas não iremos abordar esse assunto neste.

Se você está confuso com todas essas nomenclaturas, não se preocupe! Nós iremos colocar a mão na massa em breve.

## Uma nota para desenvolvedores ASP.NET 4
Se você nunca usou uma versão antiga do ASP.NET, pule para o próximo capítulo.

ASP.NET Core é uma completa reescrita do ASP.NET, com foco em modernizar o framework e finalmente desacoplá-lo do System.Web, do IIS e do Windows. Se você lembra todas as coisas do OWIN/Katana do ASP.NET 4, você já tem meio caminho andado: O Projeto Katana se tornou o ASP.NET 5 que, no final, foi renomeado para ASP.NET Core.

Devido ao legado do Katana, a classe `Startup` é o centro de tudo; não existe mais `Application_Start` ou `Global.asax`. Todo o pipeline é movido por middlewares, e não existe mais separação entre MVC e Web API: controllers podem simplesmente retornar views, códigos de status ou dados. Injeção de dependência está disponível nativamente no framework, então você não é obrigado a instalar e configurar outros containers de injeção de dependência, como StructureMap ou Ninject. E todo o framework foi otimizado para velocidade e eficiência de execução.

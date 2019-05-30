## Mais recursos

ASP.NET Core Identity ajuda você a adicionar recursos de segurança e identity, como login e registro em sua aplicação. Os templates `dotnet new` oferecem templates de views e controllers que lidam com esses cenários comuns para que você possa começar a trabalhar rapidamente.

Há muito mais que o ASP.NET Core Identity pode fazer, como redefinição de senha e login social. A documentação disponível em http://docs.asp.net é um recurso fantástico para aprender como adicionar esses recursos.

### Alternativas ao ASP.NET Core Identity

ASP.NET Core Identity não é a única maneira de adicionar funcionalidade de identity. Outra abordagem é usar um serviço de identity hospedado na nuvem, como o Azure Active Directory B2C ou o Okta, para manipular as identidades de sua aplicação. Você pode pensar nessas opções como parte de uma progressão:

* **Faça você mesmo**: Não recomendado, a menos que você seja um especialista em segurança!
* **ASP.NET Core Identity**: você obtém muito código de graça com os templates, o que torna muito fácil o início da codificação de sua aplicação. Você ainda precisará escrever algum código para cenários mais avançados e manter um banco de dados para armazenar informações do usuário.
* **Serviços de identidade hospedados em nuvem**. Estes serviços lidam com cenários simples e avançados (autenticação de múltiplos fatores, recuperação de contas, federação) e reduz significativamente a quantidade de código que você precisa escrever para sua aplicação. Além disso, os dados confidenciais do usuário não são armazenados em seu próprio banco de dados.

Para este projeto, o ASP.NET Core Identity cai muito bem. Para projetos mais complexos, recomendo fazer algumas pesquisas e experimentar as duas opções para entender qual é o melhor para o seu caso de uso.

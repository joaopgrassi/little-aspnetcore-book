# Segurança e identity (identidade)

Segurança é uma das principais preocupações de qualquer aplicação ou API web moderna. É importante manter os seus usuarios ou os dados de seus clientes seguros e fora do alcance dos invasores. Este é um tópico muito abrangente, envolvendo itens como:

* Remover os dados sensiveis de entrada para evitar ataques de injeção de SQL
* Prevenção de ataques cross-domain (CSRF) nas paginas web
* Utilização de HTTPS (criptografia de conexão) para que os dados não possam ser interceptados enquanto trafegam pela Internet
* Disponibilizar aos usuários uma maneira de autenticar com segurança, utilizando uma senha ou qualquer outra credencial.
* Criação de fluxos de autenticação multifato, de redefinição de senha e de recuperação de conta

ASP.NET Core pode ajudar a tornar tudo isso mais fácil de implementar. Os dois primeiros (proteção contra injeção de SQL e ataques cross-domain) já estão incorporados e você pode adicionar algumas linhas de código para habilitar o suporte a HTTPS. Este capítulo focará principalmente nos aspectos de **identity** da segurança: manipulação de contas de usuário, autenticação (logging in) de seus usuários com segurança e tomada de decisões de autorização após a autenticação.

> Autenticação e autorização são ideias distintas que são frequentemente confundidas. **Autenticação** lida com o fato de um usuário estar logado, enquanto **autorização** lida com o que ele tem permissão para fazer *depois de* fazer login. Você pode pensar na autenticação como a pergunta "Eu sei quem este usuário é?" enquanto a autorização pergunta "Este usuário tem permissão para fazer *X*?"

O modelo MVC + Individual Authentication que você usou para estruturar o projeto inclui várias classes baseadas no ASP.NET Core Identity, um sistema de autenticação e identidade que faz parte do ASP.NET Core. E também já adiciona a capacidade de efetuar login com um email e uma senha.

## O que é ASP.NET Core Identity?

O ASP.NET Core Identity é o sistema de identidade fornecido com o ASP.NET Core. Como tudo no ecossistema do ASP.NET Core, é um conjunto de pacotes NuGet que podem ser instalados em qualquer projeto (e já estão incluídos se você usar o modelo padrão).

O ASP.NET Core Identity cuida do armazenamento de contas de usuário, hashing e armazenamento de senhas e gerenciamento de roles para usuários. Ele suporta login por e-mail/senha, autenticação de múltiplos fatores, login social com provedores como Google e Facebook, bem como conexão a outros serviços usando protocolos como OAuth 2.0 e OpenID Connect.

As views Register e Login fornecidas com o modelo MVC + Individual Authentication já aproveitam o ASP.NET Core Identity e já funcionam! Tente registrar-se para uma conta e fazer login.

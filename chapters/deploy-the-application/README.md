# Deploy (implantação) da aplicação
Você percorreu um longo caminho, mas ainda não terminou. Depois de criar uma ótima aplicação, você precisa compartilhá-la com o mundo!

Como as aplicações ASP.NET Core podem ser executados no Windows, Mac ou Linux, há várias maneiras diferentes de realizar o deploy de sua aplicação. Neste capítulo, mostrarei as formas mais comuns (e mais fáceis) de go live.

## Opções de deploy

O deploy de aplicações ASP.NET Core normalmente é realizado em um desses ambientes:

* **Um host do Docker**. Qualquer máquina capaz de hospedar contêineres Docker pode ser usada para hospedar uma aplicação ASP.NET Core. Criar uma imagem do Docker é uma maneira muito rápida de implantar sua aplicação, especialmente se você estiver familiarizado com o Docker. (Se você não for, não se preocupe! Eu cobrirei os passos mais tarde.)

* **Azure**. O Microsoft Azure tem suporte nativo para aplicações ASP.NET Core. Se você tiver uma assinatura do Azure, basta criar um Web App e fazer o upload de seus arquivos de projeto. Eu cobrirei como fazer isso com o CLI do Azure na próxima seção.

* **Linux (com Nginx)**. Se você não quiser ir para o caminho do Docker, ainda poderá hospedar sua aplicação em qualquer servidor Linux (isso inclui as máquinas virtuais Amazon EC2 e DigitalOcean). É típico associar o ASP.NET Core ao proxy reverso Nginx. (Mais sobre Nginx abaixo.)

* **Windows**. Você pode usar o servidor da Web IIS no Windows para hospedar aplicações ASP.NET Core. Geralmente, é mais fácil (e mais barato) implantar apenas no Azure, mas se você preferir gerenciar os servidores Windows por conta própria, tudo vai funcionar bem.

## Kestrel e proxies reversos

> Se você não se importa com a coragem de hospedar as aplicações ASP.NET Core e quiser apenas as instruções passo a passo, sinta-se à vontade para pular para uma das próximas duas seções.

O ASP.NET Core inclui um servidor web rápido e leve chamado Kestrel. É o servidor que você usa toda vez que você roda o comando `dotnet run` e navega para` http: // localhost: 5000`. Quando você implanta sua aplicação em um ambiente de produção, ele ainda usa o Kestrel nos bastidores. No entanto, é recomendável que você coloque um proxy reverso na frente do Kestrel, porque o Kestrel ainda não possui balanceamento de carga e outros recursos que os servidores da Web mais maduros têm.

No Linux (e em contêineres do Docker), você pode usar o Nginx ou o servidor da Web Apache para receber requisições de entrada da Internet e encaminhá-las para sua aplicação hospedada com o Kestrel. Se você estiver no Windows, o IIS faz a mesma coisa.

Se você estiver usando o Azure para hospedar sua aplicação, tudo isso será feito automaticamente. Eu cobrirei a configuração do Nginx como um proxy reverso na seção do Docker.

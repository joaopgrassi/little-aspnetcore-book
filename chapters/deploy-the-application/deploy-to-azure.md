## Deploy para o Azure

O deploy da aplicação ASP.NET Core no Azure leva apenas alguns passos. Você pode fazê-lo através do portal Web do Azure ou na linha de comando usando a CLI do Azure. Eu cobrirei o último.

### O que você precisará

* Git (use o comando `git --version` para ter certeza de que está instalado)
* A CLI do Azure (siga as instruções de instalação em https://github.com/Azure/azure-cli)
* Uma assinatura do Azure (a assinatura gratuita é válida)
* Um arquivo de configuração de deployment na raiz do seu projeto

### Criando um arquivo de configuração de deployment

Como existem vários projetos em sua estrutura de diretório (a aplicação Web e dois projetos de teste), o Azure não saberá qual publicar. Para corrigir isso, crie um arquivo chamado `.deployment` no topo da sua estrutura de diretórios:

**.deployment**

```ini
[config]
project = AspNetCoreTodo/AspNetCoreTodo.csproj
```

Certifique-se de salvar o arquivo como `.deployment` sem nenhuma outra parte no nome. (No Windows, você pode precisar colocar aspas ao redor do nome do arquivo, como `" .deployment "`, para evitar que uma extensão `.txt` seja adicionada.)

Se você executar o comando `ls` ou` dir` no seu diretório de nível superior, você deve ver estes itens:

```
.deployment
AspNetCoreTodo
AspNetCoreTodo.IntegrationTests
AspNetCoreTodo.UnitTests
```

### Configurando os recursos do Azure

Se você acabou de instalar a CLI do Azure pela primeira vez, execute

```
az login
```

e siga as instruções para efetuar login na sua máquina. Em seguida, crie um novo Resource Group (grupo de recursos) para esta aplicação:

```
az group create -l westus -n AspNetCoreTodoGroup
```

Isso cria um Resource Group na região Oeste dos EUA (West US). Se você estiver longe do Oeste dos EUA, use `az account list-locations` para obter uma lista de locais e encontrar um mais próximo de você.

Em seguida, crie um App Service plan (plano de serviço de aplicação) no grupo que você acabou de criar:

```
az appservice plan create -g AspNetCoreTodoGroup -n AspNetCoreTodoPlan --sku F1
```

> F1 é o plano de aplicação gratuito. Se você quiser usar um nome de domínio personalizado com sua aplicação, use o plano D1 ($10/mês) ou superior.

Agora crie uma aplicação Web no App Service plan:

```
az webapp create -g AspNetCoreTodoGroup -p AspNetCoreTodoPlan -n MyTodoApp
```

O nome da aplicação acima (`MyTodoApp`) deve ser globalmente exclusivo no Azure. Uma vez que a aplicação é criada, ele terá uma URL padrão no formato: http://mytodoapp.azurewebsites.net

### Deploy dos seus arquivos de projeto para o Azure

Você pode usar o Git para enviar os arquivos de sua aplicação para o Azure Web App. Se o seu diretório local não estiver sendo rastreado como repositório do Git, execute estes comandos para configurá-lo:

```
git init
git add .
git commit -m "First commit!"
```

Em seguida, crie um nome de usuário e senha do Azure para deployment:

```
az webapp deployment user set --user-name nate
```

Siga as instruções para criar uma senha. Então use `config-local-git` para gerar uma URL do Git:

```
az webapp deployment source config-local-git -g AspNetCoreTodoGroup -n MyTodoApp --out tsv

https://nate@mytodoapp.scm.azurewebsites.net/MyTodoApp.git
```

Copie a URL para a área de transferência e use-a para adicionar o repositório remoto do Git ao seu repositório local:

```
git remote add azure <paste>
```

Você só precisa seguir estas etapas uma vez. Agora, sempre que você quiser enviar os arquivos de sua aplicação para o Azure, verifique-os com o Git e execute

```
git push azure master
```

Você verá um fluxo de mensagens de log à medida que o deploy da aplicação é feito para o Azure.

Quando estiver completo, navegue até http://yourappname.azurewebsites.net para verificar a aplicação!

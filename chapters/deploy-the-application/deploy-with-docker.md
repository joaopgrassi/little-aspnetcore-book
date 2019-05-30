## Deploy para o Docker

Se você não estiver usando uma plataforma como o Azure, tecnologias de contêiner como o Docker podem facilitar a implantação de aplicações web em seus próprios servidores. Em vez de gastar tempo configurando um servidor com as dependências necessárias para executar a aplicação, copiar arquivos e reiniciar processos, basta criar uma imagem do Docker que descreva tudo o que o sua aplicação precisa para rodar e transformá-la em um contêiner em qualquer host Docker.

O Docker também pode facilitar o dimensionamento de sua aplicação em vários servidores. Depois de ter uma imagem, usá-la para criar um contêiner é o mesmo processo que criar 100 contêineres.

Antes de começar, você precisa do Docker CLI instalado em sua máquina de desenvolvimento. Procure por "obter docker para (mac/windows/linux)" e siga as instruções no site oficial do Docker. Você pode verificar se o Docker está instalado corretamente com

```
docker version
```

### Adicionando um Dockerfile

A primeira coisa que você precisa é de um Dockerfile, que é como uma receita que informa ao Docker o que sua aplicação precisa para compilar e executar.

Crie um arquivo chamado `Dockerfile` (sem extensão) no diretório raiz `AspNetCoreTodo`. Abra no seu editor favorito. Escreva a seguinte linha:

```dockerfile
FROM microsoft/dotnet:2.0-sdk AS build
```

Isso diz ao Docker para usar a imagem `microsoft/dotnet:2.0-sdk` como ponto de partida. Esta imagem é publicada pela Microsoft e contém as ferramentas e dependências necessárias para executar `dotnet build` e compilar sua aplicação. Ao usar essa imagem pré-construída como ponto de partida, o Docker pode otimizar a imagem produzida para sua aplicação e mantê-la pequena.

Em seguida, adicione esta linha:

```dockerfile
COPY AspNetCoreTodo/*.csproj ./app/AspNetCoreTodo/
```

O comando `COPY` copia o arquivo de projeto `.csproj` para a imagem no caminho `/app/AspNetCoreTodo/`. Note que nenhum dos códigos atuais (arquivos `.cs`) foram copiados para a imagem ainda. Você verá porque em um minuto.

```dockerfile
WORKDIR /app/AspNetCoreTodo
RUN dotnet restore
```

`WORKDIR` é o equivalente do comando `cd` no Docker. Isto significa que qualquer comando executado a seguir será executado dentro do diretório `/app/AspNetCoreTodo` que o comando` COPY` criou na última etapa.

Executar o comando `dotnet restore` restaura os pacotes NuGet que a aplicação precisa, definidos no arquivo `.csproj`. Ao restaurar pacotes dentro da imagem **antes** de adicionar o restante do código, o Docker pode armazenar em cache os pacotes restaurados. Então, se você fizer alterações de código (mas não alterar os pacotes definidos no arquivo de projeto), a reconstrução da imagem do Docker será super rápida.

Agora é hora de copiar o resto do código e compilar a aplicação:

```dockerfile
COPY AspNetCoreTodo/. ./AspNetCoreTodo/
RUN dotnet publish -o out /p:PublishWithAspNetCoreTargetManifest="false"
```

O comando `dotnet publish` compila o projeto, e o sinalizador `-o out` coloca os arquivos compilados em um diretório chamado `out`.

Esses arquivos compilados serão usados ​​para executar a aplicação com os poucos comandos finais:

```dockerfile
FROM microsoft/dotnet:2.0-runtime AS runtime
ENV ASPNETCORE_URLS http://+:80
WORKDIR /app
COPY --from=build /app/AspNetCoreTodo/out ./
ENTRYPOINT ["dotnet", "AspNetCoreTodo.dll"]
```

O comando `FROM` é usado novamente para selecionar uma imagem menor que tenha apenas as dependências necessárias para executar a aplicação. O comando `ENV` é usado para definir variáveis ​​de ambiente no container, e a variável de ambiente `ASPNETCORE_URLS` diz ao ASP.NET Core qual interface de rede e porta ele deve ligar (neste caso, porta 80).

O comando `ENTRYPOINT` permite ao Docker saber que o container deve ser iniciado como um executável executando o `dotnet AspNetCoreTodo.dll`. Isto diz `dotnet` para iniciar sua aplicação a partir do arquivo compilado criado pelo `dotnet publish` anteriormente. (Quando você executa `dotnet run` durante o desenvolvimento, você está realizando a mesma coisa em uma única etapa).

O Dockerfile completo é assim:

**Dockerfile**

```dockerfile
FROM microsoft/dotnet:2.0-sdk AS build
COPY AspNetCoreTodo/*.csproj ./app/AspNetCoreTodo/
WORKDIR /app/AspNetCoreTodo
RUN dotnet restore

COPY AspNetCoreTodo/. ./
RUN dotnet publish -o out /p:PublishWithAspNetCoreTargetManifest="false"

FROM microsoft/dotnet:2.0-runtime AS runtime
ENV ASPNETCORE_URLS http://+:80
WORKDIR /app
COPY --from=build /app/AspNetCoreTodo/out ./
ENTRYPOINT ["dotnet", "AspNetCoreTodo.dll"]
```

### Criando uma imagem

Verifique se o Dockerfile está salvo e, em seguida, use `docker build` para criar uma imagem:

```
docker build -t aspnetcoretodo .
```

Não esqueça o ponto final! Isso diz ao Docker para procurar por um Dockerfile no diretório atual.

Depois que a imagem é criada, você pode executar `docker images` para listar todas as imagens disponíveis em sua máquina local. Para testá-la em um contêiner, execute

```
docker run --name aspnetcoretodo_sample --rm -it -p 8080:80 aspnetcoretodo
```

O sinalizador `-it` diz ao Docker para executar o container no modo interativo (saída para o terminal, ao invés de rodar em background). Quando você quiser parar o contêiner, pressione Ctrl-C.

Lembra-se da variável `ASPNETCORE_URLS` que disse ao ASP.NET Core para escutar na porta 80? A opção `-p 8080:80` diz ao Docker para mapear a porta 8080 na *sua* máquina para a porta 80 do *container*. Abra seu navegador e navegue para http://localhost:8080 para ver a aplicação sendo executada no container!

### Configurando o Nginx

No início deste capítulo, mencionei que você deve usar um proxy reverso como Nginx para fazer requisições de proxy ao Kestrel. Você também pode usar o Docker para isso.

A arquitetura geral consistirá em dois contêineres: um contêiner Nginx que atende na porta 80, encaminhando requisições para o contêiner recém-construído que hospeda sua aplicação com o Kestrel.

O contêiner Nginx precisa de seu próprio Dockerfile. Para evitar que ele entre em conflito com o Dockerfile que você acabou de criar, crie um novo diretório na raiz da aplicação web:

```
mkdir nginx
```

Crie um novo Dockerfile e adicione estas linhas:

**nginx/Dockerfile**

```dockerfile
FROM nginx
COPY nginx.conf /etc/nginx/nginx.conf
```

Em seguida, crie um arquivo `nginx.conf`:

**nginx/nginx.conf**

```
events { worker_connections 1024; }

http {
    server {
        listen 80;
        location / {
          proxy_pass http://kestrel:80;
          proxy_http_version 1.1;
          proxy_set_header Upgrade $http_upgrade;
          proxy_set_header Connection 'keep-alive';
          proxy_set_header Host $host;
          proxy_cache_bypass $http_upgrade;
        }
    }
}
```

Este arquivo de configuração informa ao Nginx para fazer o proxy das requisições recebidas em `http://kestrel:80`. (Já já você verá porque o `kestrel` funciona como um hostname.)

> Ao implementar sua aplicação em um ambiente de produção, você deve adicionar a diretiva `server_name` e validar e restringir o header do host a valores válidos. Para mais informações, veja:

> https://github.com/aspnet/Announcements/issues/295

### Configurando o Docker Compose

Há mais um arquivo para criar. No diretório raiz, crie o `docker-compose.yml`:

**docker-compose.yml**

```yaml
nginx:
    build: ./nginx
    links:
        - kestrel:kestrel
    ports:
        - "80:80"
kestrel:
    build: .
    ports:
        - "80"
```

O Docker Compose é uma ferramenta que ajuda a criar e executar aplicações com vários contêineres. Este arquivo de configuração define dois containers: `nginx` a partir do `./Nginx/Dockerfile` e `kestrel` a partir do`./Dockerfile`. Os contêineres são explicitamente vinculados entre si para que possam se comunicar.

Você pode tentar rodar toda a aplicação multi-contêiner executando:

```
docker-compose up
```

Tente abrir um navegador e navegue até http://localhost (porta 80, não 8080!). O Nginx está escutando na porta 80 (a porta HTTP padrão) e enviando requisições de proxy para sua aplicação ASP.NET Core hospedada pelo Kestrel.

### Configurando um servidor Docker

Instruções específicas de configuração estão fora do escopo deste livro, mas qualquer versão moderna do Linux (como o Ubuntu) pode ser utilizado para configurar um host do Docker. Por exemplo, você pode criar uma máquina virtual com o Amazon EC2 e instalar o serviço Docker. Você pode procurar por "amazon ec2 set up docker" (por exemplo) para obter instruções.

Eu gosto de usar a DigitalOcean porque eles tornaram realmente fácil começar. A DigitalOcean tem uma máquina virtual Docker pré-construída e tutoriais detalhados para colocar o Docker em funcionamento (procure por "docker digitalocean").

---
Exercise:
  title: 'Exercício: criar imagens de contêiner do Linux e do Windows e armazená-las no Registro de Contêiner do Azure'
  module: Guided Project - Deploy applications to Azure Kubernetes Service
---
# Exercício 2 – implantar aplicativos no Serviço de Kubernetes do Azure (AKS)

## Objetivos

Este projeto guiado consiste nos seguintes exercícios:

+ Exercício 1: Provisionar Registro de Contêiner do Azure (ACR) e Serviço de Kubernetes do Azure (AKS).
+ **Exercício 2: criar imagens de contêiner do Linux e do Windows e armazená-las no Registro de Contêiner do Azure.**
+ Exercício 3: implantar imagens de contêiner no Serviço de Kubernetes do Azure.
+ Exercício 4: Examinar a implantação e desprovisionar todos os recursos.

Neste exercício, você cria imagens de contêiner do Linux e do Windows e as armazena no Registro de Contêiner do Azure.

## Exercício 2: criar imagens de contêiner do Linux e do Windows e armazená-las no ACR
Neste exercício, você criará imagens do Docker baseadas em Linux e Windows e as enviará por push para o Registro de Contêiner do Azure criado anteriormente neste laboratório.


>**Observação**: para concluir este exercício, você precisará de uma [assinatura do Azure](https://azure.microsoft.com/free/).
> Para quaisquer propriedades que não sejam especificadas, use o valor padrão.


### Tarefa 1: criar uma imagem de contêiner do Linux e armazená-la no ACR
Nesta tarefa, você usará uma tarefa do ACR para criar uma imagem de contêiner do Linux e efetuar push dela automaticamente no ACR.

1. No portal do Azure, selecione o ícone **Cloud Shell**.
1. Se for solicitado que você selecione **Bash** ou **PowerShell**, selecione **Bash**. 
1. Se solicitado, selecione **Criar armazenamento** e aguarde até que o painel do Azure Cloud Shell seja exibido. 
1. Verifique se o **Bash** aparece no menu suspenso no canto superior esquerdo do painel do Cloud Shell.
1. Na sessão Bash no Cloud Shell, crie um diretório que hospedará o Dockerfile para a imagem do Linux e altere para ele do diretório atual executando os seguintes comandos:

   ```bash
   mkdir ~/image-l01
   cd ~/image-l01
   ```

1. Na sessão Bash do Azure Cloud Shell, use o editor interno para criar um arquivo chamado server.js no diretório image-l01 e copie para ele o seguinte conteúdo:

   ```js
   const http = require('http')
   const port = 80
   const server = http.createServer((request, response) => {
     response.writeHead(200, {'Content-Type': 'text/plain'})
     response.write('Hello World from Node\n')
     response.end('Version: ' + process.env.NODE_VERSION + '\n')
   })
   server.listen(port)
   console.log(`Server running at http://localhost: ${port}`)
   ```

   > **Observação:** após a execução, o código de Node.js resultante exibirá a mensagem **Olá, mundo do Node**.

1. Na sessão Bash do Azure Cloud Shell, use o editor interno novamente para criar um arquivo chamado package.json no diretório image-l01 e copie para ele o seguinte conteúdo:

   ```json
   {
     "name": "helloworld",
     "version": "1.0.0",
     "description": "Sample app for ACR Build",
     "main": "server.js",
     "scripts": {
       "test": "echo \"Error: no test specified\" && exit 1",
       "start": "node server.js"
     },
     "license": "MIT"
   }
   ```

1. Salve as alterações no arquivo e feche-as para retornar ao prompt do Bash.
1. Na sessão Bash do Azure Cloud Shell, use o editor interno novamente para criar um arquivo chamado Dockerfile no diretório image-l01 e copie para ele o seguinte conteúdo:

   ```Dockerfile
   FROM node:20.2-alpine
   COPY . /src
   RUN cd /src && npm install
   EXPOSE 80
   CMD ["node", "/src/server.js"]
   ```

1. Salve as alterações no arquivo e feche-as para retornar ao prompt do Bash.
1. Na sessão Bash do Azure Cloud Shell, identifique o nome do registro de contêiner do Azure criado anteriormente neste exercício e armazene-o em uma variável chamada **$ACRNAME** executando os seguintes comandos:

   ```azurecli
   ACR_RGNAME='acr-01-RG'
   ACR_NAME=$(az acr list --resource-group $ACR_RGNAME --query "[].name" --output tsv)
   ```

1. Na sessão Bash do Azure Cloud Shell, crie uma imagem do Docker com base no Dockerfile armazenado no diretório atual e envie-a automaticamente para o Registro de Contêiner do Azure cujo nome é armazenado na variável **$ACRNAME** executando o seguinte comando (certifique-se de incluir o ponto à direita):

   ```azurecli
   az acr build --registry $ACR_NAME --image hellofromnode:v1.0 .
   ```

   > **Observação:** acompanhe o progresso do build e certifique-se de que ele seja concluído com êxito. Isso deverá levar menos de 1 minuto.

### Tarefa 2: criar uma imagem de contêiner do Windows e armazená-la no ACR
Nesta tarefa, você usará uma tarefa do ACR para criar uma imagem de contêiner do Windows e efetuar push dela automaticamente para o ACR.

1. Na sessão Bash dentro do Cloud Shell, crie um diretório que hospedará o Dockerfile da imagem do Windows e altere para ele do diretório atual executando os seguintes comandos:

   ```bash
   mkdir ~/image-w01
   cd ~/image-w01
   ```

1. Na sessão Bash do Azure Cloud Shell, clone um repositório público do GitHub que hospeda os arquivos que você usará para criar a imagem do Windows e alternar o diretório atual para o repositório clonado executando os seguintes comandos:

   ```git
   git clone https://github.com/Azure-Samples/dotnetcore-docs-hello-world.git
   cd ~/image-w01/dotnetcore-docs-hello-world
   ```

   > **Observação:** o repositório contém o código-fonte de um aplicativo Web .NET 7 que exibe a mensagem **Olá, mundo do .NET 7**.

1. Na sessão Bash do Azure Cloud Shell, crie uma imagem do Docker com base no Dockerfile armazenado no diretório atual e envie-a automaticamente para o Registro de Contêiner do Azure cujo nome é armazenado na variável **$ACRNAME** executando o seguinte comando (certifique-se de incluir o ponto à direita):

   ```azurecli
   az acr build --registry $ACR_NAME --image hellofromdotnet:v1.0 --platform windows --file Dockerfile.windows .
   ```

   > **Observação:** o nome do Dockerfile que define o build de imagem do Windows é **Dockerfile.windows**.

   > **Observação:** acompanhe o progresso do build e certifique-se de que ele seja concluído com êxito. Isso deve levar menos de 3 minutos.

1. Feche o painel do Azure Cloud Shell.
1. No portal do Azure, navegue até a página **Registros de contêiner** e selecione a entrada que representa o Registro de contêiner para o qual você efetuou push de ambas as imagens.
1. Na página Registro de contêiner, no menu do hub vertical, selecione **Repositórios** e verifique se **hellofromnode** e **hellofromdotnet** aparecem na lista de repositórios.

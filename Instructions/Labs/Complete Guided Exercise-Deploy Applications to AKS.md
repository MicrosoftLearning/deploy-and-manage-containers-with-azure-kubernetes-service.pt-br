---
Guided Exercise:
  title: Exercício Guiado Implantar Aplicativos no AKS
---
# Exercício Guiado – implantar aplicativos no Serviço de Kubernetes do Azure (AKS)

## Objetivos

Este exercício guiado consiste nas seguintes atividades:

+ Exercício 1: provisionar o Registro de Contêiner do Azure (ACR) e o Serviço de Kubernetes do Azure (AKS)
+ Exercício 2: criar imagens de contêiner do Linux e do Windows e armazená-las no ACR
+ Exercício 3: implantar as imagens de contêiner no AKS 
+ Exercício 4: examinar a implantação e desprovisionar todos os recursos

## Exercício 1: provisionar o Registro de Contêiner do Azure (ACR) e o Serviço de Kubernetes do Azure (AKS)
Neste exercício, você criará um Registro de Contêiner do Azure e um cluster do AKS.


>**Observação**: para concluir este exercício, você precisará de uma [assinatura do Azure](https://azure.microsoft.com/free/).
> Para quaisquer propriedades que não sejam especificadas, use o valor padrão.


### Tarefa 1: criar um registro de contêiner do Azure
Nesta tarefa, você criará um Registro de Contêiner do Azure

1. No computador, abra uma janela do navegador da Web e navegue até o portal do Azure em https://portal.azure.com.
1. Quando solicitado, entre com uma conta de usuário que tenha a função Proprietário na assinatura do Azure que você usará neste exercício. 
1. Entre no portal do Azure. 
1. No portal do Azure, na caixa de texto **Pesquisar**, pesquise e selecione **Registros de contêiner**.
1. Na página **Registros de contêiner**, selecione **+ Criar** e especifique as seguintes configurações:

    |Configuração|Valor|
    |---|---|
    |Assinatura|O nome da assinatura do Azure que você usará neste exercício|
    |Grupo de recursos|O nome de um novo grupo de recursos **acr-01-RG**|
    |Nome do registro|Qualquer nome válido e globalmente exclusivo que contenha entre 5 e 50 caracteres alfanuméricos|
    |Region|Qualquer região do Azure na qual você pode criar um Registro de Contêiner do Azure e um cluster do AKS|
    |Zonas de disponibilidade|**Nenhuma**|
    |SKU|**Basic**|

1. Na página **Registros de contêiner**, selecione **Examinar + criar** e, na guia **Examinar + criar**, selecione **Criar**.

   > **Observação:** prossiga para o próximo exercício sem aguardar a conclusão do provisionamento do Registro de Contêiner do Azure.

### Tarefa 2: criar uma rede virtual do Azure e um cluster do AKS
Nesta tarefa, você criará uma rede virtual do Azure e implantará um cluster do AKS, incluindo um pool de nós do Windows nessa rede virtual.

> **Observação:** embora você possa criar uma rede virtual ao provisionar um cluster do AKS, esse não é um cenário típico. Mais importante, implantar clusters do AKS em uma rede virtual existente requer algumas considerações adicionais com as quais você deve estar familiarizado.

1. No portal do Azure, na caixa de texto **Pesquisar**, pesquise e selecione **Redes virtuais**.
1. Na página **Redes virtuais**, selecione **+ Criar** e, na guia **Noções básicas** da página **Criar rede virtual**, especifique as seguintes configurações:

   |Configuração|Valor|
   |---|---|
   |Assinatura|O nome da assinatura do Azure selecionada no primeiro exercício|
   |Grupo de recursos|O nome de um novo grupo de recursos **aks-01-RG**|
   |Nome da rede virtual|**vnet-01**|
   |Region|A mesma região do Azure selecionada no primeiro exercício deste exercício|

1. Na guia **Noções básicas** da página **Criar rede virtual**, selecione **Avançar**.
1. Na guia **Segurança** da página **Criar rede virtual**, aceite as configurações padrão e selecione **Avançar**.
1. Na guia **Endereços IP** da página **Criar rede virtual**, verifique se o espaço de endereço IP está definido como **10.0.0.0/16**, exclua a sub-rede **padrão**, selecione **Examinar + criar** e, na guia **Examinar + criar**, selecione **Criar**.

   > **Observação:** a criação de uma rede virtual deve levar apenas alguns segundos, portanto, você deve ser capaz de prosseguir diretamente para a próxima etapa.

1. No portal do Azure, na caixa de texto **Pesquisar**, pesquise e selecione **Serviços do Kubernetes**.
1. Na página **Serviços do Kubernetes**, selecione **+ Criar**, na lista suspensa, escolha **Criar um cluster do Kubernetes** e, na guia **Informações básicas** da página **Criar cluster do Kubernetes**, especifique as seguintes configurações e selecione **Próximo**:

   |Configuração|Valor|
   |---|---|
   |Assinatura|O nome da assinatura do Azure selecionada no primeiro exercício deste laboratório|
   |Grupo de recursos|**aks-01-RG**|
   |Configuração predefinida do cluster|**Desenvolvimento/Teste**|
   |Nome do cluster do Kubernetes|**aks-01**|
   |Region|A mesma região do Azure selecionada no primeiro exercício deste laboratório|
   |Zonas de disponibilidade|**Nenhuma**|
   |Tipo de preço do AKS|**Gratuito**|
   |Versão do Kubernetes|Aceitar o valor padrão|
   |Atualização automática|Desabilitado|
   |Tipo de canal de segurança do nó|**Nenhuma**|
   |Autenticação e autorização|**Contas locais com o RBAC do Kubernetes**|

1. Na guia **Pools de nós** da página **Criar cluster do Kubernetes**, realize as seguintes tarefas:

   - Na seção **Pools de nós**, selecione o link do **agentpool**.
   - Na página **Atualizar pool de nós**, na seção **Tamanho do nó**, selecione o link **Escolher um tamanho**.
   - Na página **Selecionar um tamanho de VM**, na lista de tamanhos de VM, escolha **B4ms** e clique em **Selecionar**.
   - De volta à página **Atualizar pool de nós**, defina o **Método de escala** como **Manual** e **Contagem de nós** como **2**.
   - Na página **Atualizar pool de nós**, selecione **Atualizar**.

   > **Observação:** talvez seja necessário aumentar as cotas de vCPU ou alterar a SKU da VM para acomodar o tamanho do nó e os valores de contagem de nós. Para obter informações sobre o procedimento para aumentar as cotas de vCPU, confira o artigo do Microsoft Learn [Aumentar cotas de vCPU da família de VMs](https://learn.microsoft.com/en-us/azure/quotas/per-vm-quota-requests).

   > **Observação:** Você adicionará um pool de nós do Windows ao cluster. Isso exigia a alteração da configuração de rede para a **CNI do Azure** do **Kubenet** padrão. A configuração de rede do Kubenet não dá suporte a pools de nós do Windows.

1. De volta à guia **Pools de nós** da página **Criar cluster do Kubernetes**, selecione **Próximo**.
1. Na guia **Rede** da página **Criar cluster do Kubernetes**, selecione a opção **CNI do Azure**, marque a caixa de seleção **Traga sua própria rede virtual**; na lista suspensa **Rede virtual**, selecione **vnet-01** e, abaixo da caixa de texto **Sub-rede do cluster**, selecione **Gerenciar configuração de sub-rede**.
1. Na página **vnet-01 \| Sub-redes**, selecione **+ Sub-rede**.
1. Na página **Adicionar sub-redes**, especifique as seguintes configurações e selecione **Salvar**:

   |Configuração|Valor|
   |---|---|
   |Nome|**aks-subnet**|
   |Intervalo de endereços da sub-rede|**10.0.0.0/20**|

1. De volta à página **vnet-01 \| Sub-redes**, na trilha breadcrumb na parte superior esquerda da página, selecione **Criar cluster do Kubernetes**. 
1. De volta à guia **Rede** da página **Criar cluster do Kubernetes**, especifique as seguintes configurações:

   |Configuração|Valor|
   |---|---|
   |Rede virtual|**vnet-01**|
   |Sub-rede de cluster|**aks-subnet (10.0.0.0/20)**|
   |Intervalo de endereços de serviço do Kubernetes|**172.16.0.0/22**|
   |Endereço IP do serviço DNS do Kubernetes|**172.16.3.254**|
   |Prefixo do nome DNS|**aks-01-dns**|
   |Política de rede|**Nenhuma**|

1. Na guia **Rede** da página **Criar cluster do Kubernetes**, selecione **Anterior**.
1. De volta à guia **Pools de nós** da página **Criar cluster do Kubernetes**, selecione **+ Adicionar pool de nós**.
1. Na página **Adicionar pool de nós**, especifique as seguintes configurações:

   |Configuração|Valor|
   |---|---|
   |Nome do pool de nós|**w1pool**|
   |Modo|**Usuário**|
   |Tipo do SO|**Windows 2022**|
   |Zona de disponibilidade|**Nenhuma**|
   |Habilitar instâncias spot do Azure|Desabilitadas|
   |Tamanho do nó|**B4ms**|
   |Método de dimensionamento|**Manual**|
   |Contagem de nós|**2**|
   |Pods máx por nó|**30**|
   |Habilitar IP público por nó|Desabilitadas|

   > **Observação:** Talvez seja necessário aumentar as cotas de vCPU ou alterar o SKU da VM para acomodar o tamanho do nó e os valores de contagem de nós.

1. Na página **Adicionar pool de nós**, selecione **Adicionar**.
1. De volta à guia **Pools de nós** da página **Criar cluster do Kubernetes**, selecione **Próximo**.
1. Na guia **Rede** da página **Criar cluster do Kubernetes**, selecione **Próximo**.
1. Na guia **Integração** da página **Criar cluster do Kubernetes**, na lista suspensa do **Registro de contêiner**, selecione a entrada que representa o Registro de Contêiner do Azure que você criou no exercício anterior, desabilite a caixa de seleção **Habilitar as regras de alerta recomendadas**, certifique-se de que a opção **Azure Policy** esteja desabilitada e selecione **Avançar**.
1. Na guia **Monitoramento** da página **Criar cluster do Kubernetes**, desmarque a caixa de seleção **Habilitar métricas do Prometheus** e selecione **Examinar + criar**.
1. Na guia **Revisar + criar** da página **Criar cluster do Kubernetes**, selecione **Criar**.

   > **Observação:** prossiga para o próximo exercício sem aguardar a conclusão do provisionamento do cluster do AKS. O processo de provisionamento pode levar cerca de 5 minutos.

## Exercício 2: criar imagens de contêiner do Linux e do Windows e armazená-las no ACR
Neste exercício, você criará imagens do Docker baseadas em Linux e Windows e as enviará por push para o Registro de Contêiner do Azure criado anteriormente neste exercício.

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

## Exercício 3: implantar as imagens de contêiner no AKS 
Neste exercício, você implantará duas imagens de contêiner criadas anteriormente neste exercício no cluster do AKS.

> **Observação:** Antes de prosseguir com este exercício, certifique-se de que o provisionamento do cluster do AKS tenha sido concluído com êxito.

### Tarefa 1: criar namespaces personalizados do AKS
Nesta tarefa, você criará dois namespaces no cluster do AKS criado anteriormente neste exercício.

1. No portal do Azure, na caixa de texto **Pesquisar**, pesquise e selecione **Serviços do Kubernetes**.
1. Na página **Serviços do Kubernetes**, selecione **aks-01**.
1. Na página **aks-01**, no menu vertical do hub, selecione **Namespaces**.
1. Na página **aks-01 \| Namespaces**, selecione **+ Criar** e, no menu suspenso, selecione **Namespace**.
1. No painel **Criar um namespace**, na caixa de texto **Nome**, insira **dev-node** e selecione **Criar**.
1. Na página **aks-01 \| Namespaces**, selecione **+ Criar** e, no menu suspenso, selecione **Namespace**.
1. No painel **Criar um namespace**, na caixa de texto **Nome**, insira **dev-dotnet** e selecione **Criar**.

### Tarefa 2: criar um manifesto do Kubernetes para implantar a imagem do Linux
Nesta tarefa, você criará manifestos do Kubernetes para implantar a imagem do Linux no pool de nós do Linux

1. No portal do Azure, selecione o ícone **Cloud Shell**.
1. Verifique se o **Bash** aparece no menu suspenso no canto superior esquerdo do painel do Cloud Shell.
1. Na sessão Bash do Azure Cloud Shell, crie um diretório que hospedará o manifesto de implantação para provisionar pods com base na imagem do Linux e altere para ele do diretório atual executando os seguintes comandos:

   ```bash
   mkdir ~/aks-l01
   cd ~/aks-l01
   ```

1. Na sessão Bash do Azure Cloud Shell, use o editor interno para criar um arquivo chamado aks-deployment-l01.yaml no diretório aks-l01 e copie para ele o seguinte conteúdo:

   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: hellofromnode-deployment
     labels:
       environment: dev
       app: hellofromnode
   spec:
     replicas: 1
     template:
       metadata:
         name: hellofromnode
         labels:
           app: hellofromnode
       spec:
         nodeSelector:
           "kubernetes.io/os": linux
         containers:
         - name: hellofromnode
           image: ACR_NAME.azurecr.io/hellofromnode:v1.0
           resources:
             limits:
               cpu: 1
               memory: 800M
           ports:
             - containerPort: 80
     selector:
       matchLabels:
         app: hellofromnode
   ---
   apiVersion: v1
   kind: Service
   metadata:
     name: hellofromnode-service
   spec:
     type: LoadBalancer
     ports:
     - protocol: TCP
       port: 80
     selector:
       app: hellofromnode
   ```

   > **Observação:** a implantação criará pods com base na imagem de contêiner do Linux para o pool de nós do Linux no cluster do AKS. Além disso, o manifesto inclui um serviço, que fornecerá um acesso com balanceamento de carga aos pods na implantação por meio de um endereço IP público na porta 80.

1. Salve as alterações no arquivo e feche-as para retornar ao prompt do Bash.
1. Na sessão Bash do Azure Cloud Shell, substitua o espaço reservado **ACR_NAME** no arquivo aks-deployment-l01.yaml executando os seguintes comandos:

   ```azurecli
   ACR_RGNAME='acr-01-RG'
   ACR_NAME=$(az acr list --resource-group $ACR_RGNAME --query "[].name" --output tsv)
   sed -i "s/ACR_NAME/$ACR_NAME/" ./aks-deployment-l01.yaml
   ```

### Tarefa 3: criar um manifesto do Kubernetes para implantar a imagem do Windows
Nesta tarefa, você criará manifestos do Kubernetes para implantar a imagem do Windows no pool de nós do Windows

1. Na sessão Bash do Azure Cloud Shell, crie um diretório que hospedará o manifesto de implantação para provisionar pods com base na imagem do Windows e altere para ele do diretório atual executando os seguintes comandos:

   ```bash
   mkdir ~/aks-w01
   cd ~/aks-w01
   ```

1. Na sessão Bash do Azure Cloud Shell, use o editor interno para criar um arquivo chamado aks-deployment-w01.yaml no diretório aks-w01 e copie para ele o seguinte conteúdo:

   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: hellofromdotnet-deployment
     labels:
       environment: dev
       app: hellofromdotnet
   spec:
     replicas: 1
     template:
       metadata:
         name: hellofromdotnet
         labels:
           app: hellofromdotnet
       spec:
         nodeSelector:
           "kubernetes.io/os": windows
         containers:
         - name: hellofromdotnet
           image: ACR_NAME.azurecr.io/hellofromdotnet:v1.0
           resources:
             limits:
               cpu: 1
               memory: 800M
           ports:
             - containerPort: 80
     selector:
       matchLabels:
         app: hellofromdotnet
   ---
   apiVersion: v1
   kind: Service
   metadata:
     name: hellofromdotnet-service
   spec:
     type: LoadBalancer
     ports:
     - protocol: TCP
       port: 80
     selector:
       app: hellofromdotnet
   ```

   > **Observação:** a implantação criará pods com base na imagem de contêiner do Windows para o pool de nós do Windows no cluster do AKS. Além disso, o manifesto inclui um serviço, que fornecerá um acesso com balanceamento de carga aos pods na implantação por meio de um endereço IP público na porta 80.

1. Salve as alterações no arquivo e feche-as para retornar ao prompt do Bash.
1. Na sessão Bash do Azure Cloud Shell, substitua o espaço reservado **ACR_NAME** no arquivo aks-deployment-l01.yaml executando o seguinte comando:

   ```azurecli
   sed -i "s/ACR_NAME/$ACR_NAME/" ./aks-deployment-w01.yaml
   ```

### Tarefa 4: executar implantações do AKS usando arquivos de manifesto YAML
Nesta tarefa, você implantará as duas imagens de contêiner em seus respectivos namespaces e pools de nós no cluster do AKS de destino.

1. Na sessão Bash do Azure Cloud Shell, conecte-se ao cluster do AKS executando os seguintes comandos:

   ```azurecli
   AKSRG='aks-01-RG'
   AKSNAME='aks-01'
   az aks get-credentials --resource-group $AKSRG --name $AKSNAME
   ```

1. Na sessão Bash do Azure Cloud Shell, verifique se a conexão foi bem-sucedida executando o seguinte comando:

   ```kubectl
   kubectl get nodes
   ```

   > **Observação:** a saída do comando deve incluir a listagem de todos os nós do AKS (quatro em nosso caso).

1. Na sessão Bash do Azure Cloud Shell, crie a primeira implantação definida no arquivo de manifesto YAML correspondente no namespace **dev-node** executando os seguintes comandos:

   ```kubectl
   cd ~/aks-l01
   kubectl apply -f aks-deployment-l01.yaml -n=dev-node
   ```

   > **Observação:** prossiga para a próxima etapa sem aguardar a conclusão da implantação. O provisionamento de todos os recursos pode levar alguns minutos.

1. Na sessão Bash do Azure Cloud Shell, crie a segunda implantação definida no arquivo de manifesto YAML correspondente executando o seguinte comando:

   ```kubectl
   cd ~/aks-w01
   kubectl apply -f aks-deployment-w01.yaml -n=dev-dotnet
   ```

   > **Observação:** prossiga para a próxima etapa sem aguardar a conclusão da implantação. O provisionamento de todos os recursos pode levar alguns minutos.

# Exercício 4: examinar a implantação e desprovisionar todos os recursos
Neste exercício, você examinará os resultados das implantações e desprovisionará todos os recursos.

### Tarefa 1: examinar as implantações e serviços do AKS
Nesta tarefa, você examinará os resultados de ambas as implantações, incluindo os objetos de implantações e serviços.

1. Na sessão Bash do Azure Cloud Shell, exiba o status de ambas as implantações executando os seguintes comandos:

   ```kubectl
   kubectl get deployments -n=dev-node
   kubectl get deployments -n=dev-dotnet
   ```

   > **Observação:** antes de prosseguir para a próxima etapa, verifique se ambas as implantações estão listadas com o status pronto. Se esse não for o caso, aguarde mais um minuto, execute novamente os dois comandos mencionados anteriormente e verifique o status da implantação novamente.

1. Na sessão Bash do Azure Cloud Shell, exiba o status dos dois serviços que foram incluídos nos arquivos de manifesto executando os seguintes comandos:

   ```kubectl
   kubectl get services -n=dev-node
   kubectl get services -n=dev-dotnet
   ```

1. Verifique se a listagem de cada serviço inclui um valor na coluna **EXTERNAL-IP**. 
1. Use um navegador da Web para navegar até os endereços IP que você identificou na etapa anterior e verificar se as páginas da Web resultantes exibem as mensagens **Olá, mundo do Nó** e **Olá, mundo do .NET 7**, respectivamente.

### Tarefa 2: excluir todos os recursos
Nesta tarefa, você excluirá todos os recursos provisionados neste exercício.

1. Na sessão Bash do Azure Cloud Shell, exiba a listagem de recursos nos dois grupos de recursos provisionados neste exercício executando os seguintes comandos:

   ```azurecli
   az resource list --resource-group 'acr-01-RG' --query "[].name" --output tsv
   az resource list --resource-group 'aks-01-RG' --query "[].name" --output tsv
   ```

   > **Observação:** verifique se esses são os recursos que você deseja excluir. Caso positivo, prossiga para a próxima etapa.

1. Na sessão Bash do Azure Cloud Shell, exclua todos os recursos provisionados neste exercício executando os seguintes comandos:

   ```azurecli
   az group delete --name 'acr-01-RG' --no-wait --yes
   az group delete --name 'aks-01-RG' --no-wait --yes
   ```

   > **Observação:** O comando é executado de forma assíncrona (conforme imposto pelo parâmetro --nowait), portanto, mesmo se ambos os comandos retornarem imediatamente ao prompt do shell Bash, levará alguns minutos até que os grupos de recursos e seus recursos sejam realmente removidos.

1. Feche o painel do Azure Cloud Shell.

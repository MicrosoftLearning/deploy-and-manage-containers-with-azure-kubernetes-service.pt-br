---
Exercise:
  title: 'Exercício: implantar imagens de contêiner no Serviço de Kubernetes do Azure'
  module: Guided Project - Deploy applications to Azure Kubernetes Service
---
# Exercício 3: implantar aplicativos no Serviço de Kubernetes do Azure (AKS)

## Objetivos

Este projeto guiado consiste nos seguintes exercícios:

+ Exercício 1: Provisionar Registro de Contêiner do Azure (ACR) e Serviço de Kubernetes do Azure (AKS).
+ Exercício 2: criar imagens de contêiner do Linux e do Windows e armazená-las no Registro de Contêiner do Azure.
+ **Exercício 3: implantar imagens de contêiner no Serviço de Kubernetes do Azure.**
+ Exercício 4: Examinar a implantação e desprovisionar todos os recursos.

Neste exercício, você deve implantar imagens de contêiner no Serviço de Kubernetes do Azure.

## Exercício 3: implantar as imagens de contêiner no AKS 
Neste exercício, você implantará duas imagens de contêiner criadas anteriormente neste exercício no cluster do AKS.
>**Observação**: para concluir este exercício, você precisará de uma [assinatura do Azure](https://azure.microsoft.com/free/).
> Para quaisquer propriedades que não sejam especificadas, use o valor padrão.
> **Observação:** Antes de prosseguir com este exercício, certifique-se de que o provisionamento do cluster do AKS tenha sido concluído com êxito.

### Tarefa 1: criar namespaces personalizados do AKS
Nesta tarefa, você criará dois namespaces no cluster do AKS que você criou anteriormente nesse laboratório.

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


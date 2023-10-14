---
Exercise:
  title: 'Exercício: examinar a implantação e desprovisionar todos os recursos'
  module: Guided Project - Deploy applications to Azure Kubernetes Service
---
# Exercício 4 – implantar aplicativos no Serviço de Kubernetes do Azure (AKS)

## Objetivos

Este projeto guiado consiste nos seguintes exercícios:

+ Exercício 1: provisionar o Registro de Contêiner do Azure (ACR) e o Serviço de Kubernetes do Azure (AKS)
+ Exercício 2: criar imagens de contêiner do Linux e do Windows e armazená-las no ACR
+ Exercício 3: implantar as imagens de contêiner no AKS
+ **Exercício 4: examinar a implantação e desprovisionar todos os recursos**

Neste exercício, você examina a implantação dos serviços nos exercícios anteriores e desprovisiona todos os recursos.

# Exercício 4: examinar a implantação e desprovisionar todos os recursos
Neste exercício, você examinará os resultados das implantações e desprovisionará todos os recursos.

>**Observação**: para concluir este exercício, você precisará de uma [assinatura do Azure](https://azure.microsoft.com/free/).
> Para quaisquer propriedades que não sejam especificadas, use o valor padrão.

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
Nesta tarefa, você excluirá todos os recursos provisionados neste laboratório.

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

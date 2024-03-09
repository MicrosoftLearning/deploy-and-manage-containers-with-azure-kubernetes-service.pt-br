---
Exercise:
  title: 'Exercício: provisionar o Registro de Contêiner do Azure (ACR) e o Serviço de Kubernetes do Azure (AKS)'
  module: Guided Project - Deploy applications to Azure Kubernetes Service
---

# Exercício – implantar aplicativos no Serviço de Kubernetes do Azure (AKS)

## Objetivos

Este projeto guiado consiste nos seguintes exercícios:

+ **Exercício 1: Provisionar Registro de Contêiner do Azure (ACR) e Serviço de Kubernetes do Azure (AKS).**
+ Exercício 2: criar imagens de contêiner do Linux e do Windows e armazená-las no Registro de Contêiner do Azure.
+ Exercício 3: implantar imagens de contêiner no Serviço de Kubernetes do Azure.
+ Exercício 4: examinar a implantação e desprovisionar todos os recursos.

Neste exercício, você provisiona recursos do Registro de Contêiner do Azure e Serviço de Kubernetes do Azure.
## Exercício 1: provisionar o Registro de Contêiner do Azure (ACR) e o Serviço de Kubernetes do Azure (AKS)
Neste exercício, você criará um Registro de Contêiner do Azure e um cluster do AKS.

>**Observação**: para concluir este exercício, você precisará de uma [assinatura do Azure](https://azure.microsoft.com/free/).
> Para quaisquer propriedades que não sejam especificadas, use o valor padrão.

### Tarefa 1: criar um registro de contêiner do Azure
Nesta tarefa, você criará um Registro de Contêiner do Azure

1. No computador, abra uma janela do navegador da Web e navegue até o portal do Azure em https://portal.azure.com.
1. Quando solicitado, entre com uma conta de usuário que tenha a função Proprietário na assinatura do Azure que você usará neste laboratório. 
1. Entre no portal do Azure. 
1. No portal do Azure, na caixa de texto **Pesquisar**, pesquise e selecione **Registros de contêiner**.
1. Na página **Registros de contêiner**, selecione **+ Criar** e especifique as seguintes configurações:

   |Configuração|Valor|
   |---|---|
   |Assinatura|O nome da assinatura do Azure que você usará neste laboratório|
   |Grupo de recursos|O nome de um novo grupo de recursos **acr-01-RG**|
   |Nome do registro|Qualquer nome válido e globalmente exclusivo que contenha entre 5 e 50 caracteres alfanuméricos|
   |Region|Qualquer região do Azure na qual você pode criar um Registro de Contêiner do Azure e um cluster do AKS|
   |Zonas de disponibilidade|**Nenhuma**|
   |SKU|**Basic**|

1. Na página **Registros de contêiner**, selecione **Examinar + criar** e, na guia **Examinar + criar**, selecione **Criar**.

   > **Observação:** prossiga para o próximo exercício sem aguardar a conclusão do provisionamento do Registro de Contêiner do Azure.

### Tarefa 2: criar uma rede virtual do Azure e um cluster do AKS
Nesta tarefa, você criará uma rede virtual do Azure e implantará um cluster do AKS, incluindo um pool de nós do Windows nela.

> **Observação:** Embora você possa criar uma rede virtual ao provisionar um cluster do AKS, implantar clusters do AKS em uma rede virtual existente exige algumas considerações adicionais com as quais você deve estar familiarizado.

1. No portal do Azure, na caixa de texto **Pesquisar**, pesquise e selecione **Redes virtuais**.
1. Na página **Redes virtuais**, selecione **+ Criar** e, na guia **Noções básicas** da página **Criar rede virtual**, especifique as seguintes configurações:

   |Configuração|Valor|
   |---|---|
   |Assinatura|O nome da assinatura do Azure selecionada no primeiro exercício deste laboratório|
   |Grupo de recursos|O nome de um novo grupo de recursos **aks-01-RG**|
   |Nome da rede virtual|**vnet-01**|
   |Region|A mesma região do Azure selecionada no primeiro exercício deste laboratório|

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
1. Na guia **Rede** da página **Criar cluster do Kubernetes**, selecione a opção **CNI do Azure** e marque a caixa de seleção **Traga sua própria rede virtual**. Na lista suspensa **Rede virtual**, selecione **vnet-01** e, abaixo da caixa de texto **Sub-rede do cluster**, selecione **Gerenciar configuração de sub-rede**.
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
1. Na guia **Integração** da página **Criar cluster do Kubernetes**, na lista suspensa **Registro de contêiner**, selecione a entrada que representa o registro de contêiner do Azure que você criou no exercício anterior, desmarque a caixa de seleção **Habilitar regras de alerta recomendadas**, verifique se a opção **Azure Policy** está desabilitada e selecione **Próximo**.
1. Na guia **Monitoramento** da página **Criar cluster do Kubernetes**, desmarque a caixa de seleção **Habilitar métricas do Prometheus** e, em seguida, selecione **Examinar + criar**.
1. Na guia **Revisar + criar** da página **Criar cluster do Kubernetes**, selecione **Criar**.

   > **Observação:** prossiga para o próximo exercício sem aguardar a conclusão do provisionamento do cluster do AKS. O processo de provisionamento pode levar cerca de 5 minutos.

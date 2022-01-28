---
title: Securely integrate with Azure Synapse
titleSuffix: Azure Machine Learning
description: 'How to use a virtual network when integrating Azure Synapse with Azure Machine Learning.'
services: machine-learning
ms.service: machine-learning
ms.subservice: enterprise-readiness
ms.topic: how-to
ms.author: jhirono
author: jhirono
ms.reviewer: larryfr
ms.date: 01/28/2022
ms.custom: devx-track-python, ignite-fall-2021
---

# How to securely integrate Azure Machine Learning and Azure Synapse

In this article, learn how to securely integrate with Azure Machine Learning from Azure Synapse.

## Prerequisites

* An Azure subscription.
* An Azure Machine Learning workspace with a private endpoint connection to a virtual network. The following workspace dependency services must also have a private endpoint connection to the virtual network:

    * Azure Storage Account

        > [!TIP]
        > For the storage account there are three separate private endpoints; one each for blob, file, and dfs.

    * Azure Key Vault
    * Azure Container Registry

    A quick and easy way to build this configuration is to use a [Microsoft Bicep or HashiCorp Terraform template](tutorial-create-secure-workspace-template.md).

    > [!IMPORTANT]
    > The steps in this article assume that you can connect to the Azure Machine Learning studio of your workspace, but does not provide details on how to connect. Depending on your VNet configuration, you might be connecting through a jump box (VM), Azure VPN Gateway, or Azure ExpressRoute.

* An Azure Synapse workspace in a virtual network. For more information, see [Azure Synapse Analytics Managed Virtual Network](/azure/synapse-analytics/security/synapse-workspace-managed-vnet).

    > [!NOTE]
    > The steps in this article assume that the Azure Synapse workspace is in a different resource group and virtual network than the Azure Machine Learning workspace.


## Configure Azure Synapse

1. From Azure Synapse Studio, [Create a new Azure Machine Learning linked service](/azure/synapse-analytics/machine-learning/quickstart-integrate-azure-machine-learning).
1. After creating and publishing the linked service, select __Manage__,  __Managed private endpoints__, and then __+ New__ in Azure Synapse Studio.

    :::image type="content" source="./media/how-to-private-endpoint-integration-synapse/add-managed-private-endpoint.png" alt-text="{alt-text}":::

1. From the __New managed private endpoint__ page, search for __Azure Machine Learning__ and select the tile.

    :::image type="content" source="./media/how-to-private-endpoint-integration-synapse/new-private-endpoint-select-machine-learning.png" alt-text="{alt-text}":::

1. When prompted to select the Azure Machine Learning workspace, use the __Azure subscription__ and __Azure Machine Learning workspace__ you added previously as a linked service. Select __Create__ to create the endpoint.
    
    :::image type="content" source="./media/how-to-private-endpoint-integration-synapse/new-managed-private-endpoint.png" alt-text="{alt-text}":::

1. The endpoint will be listed as __Provisioning__ until it has been created. Once created, the __Approval__ column will list a status of __Pending__. You will approve the endpoint in the [Configure Azure Machine Learning](#configure-azure-machine-learning) section.

    > [!NOTE]
    > In the following screenshot, a managed private endpoint has been created for the Azure Data Lake Storage Gen 2 associated with this Synapse workspace. For information on how to create an Azure Data Lake Storage Gen 2 and enable a private endpoint for it, see [Provision and secure a linked service with Managed VNet](/azure/synapse-analytics/data-integration/linked-service).

    :::image type="content" source="./media/how-to-private-endpoint-integration-synapse/managed-private-endpoint-connections.png" alt-text="{alt-text}":::

### Create a Spark pool

To verify that the integration between Azure Synapse and Azure Machine Learning is working, you will use an Apache Spark pool. For information on creating one, see [Create a Spark pool](/azure/synapse-analytics/quickstart-create-apache-spark-pool-portal).

## Configure Azure Machine Learning

1. From the [Azure Portal](https://portal.azure.com), select your __Azure Machine Learning workspace__, and then select __Networking__.
1. Select __Private endpoints__, and then select the endpoint you created in the previous steps. It should have a status of __pending__. Select __Approve__ to approve the endpoint connection.

    :::image type="content" source="./media/how-to-private-endpoint-integration-synapse/approve-pending-private-endpoint.png" alt-text="{alt-text}":::

1. From the left of the page, select __Access control (IAM)__. Select __+ Add__, and then select __Role assignment__.

    :::image type="content" source="./media/how-to-private-endpoint-integration-synapse/workspace-role-assignment.png" alt-text="{alt-text}":::

1. Select __Contributor__, and then select __Next__.

    :::image type="content" source="./media/how-to-private-endpoint-integration-synapse/contributor-role.png" alt-text="{alt-text}":::

1. Select __User, group, or service principal__, and then __+ Select members__. Enter the name of the identity created earlier, select it, and then use the __Select__ button.

    :::image type="content" source="./media/how-to-private-endpoint-integration-synapse/add-role-assignment.png" alt-text="{alt-text}":::

1. Select __Review + assign__, verify the information, and then select the __Review + assign__ button.

    > [!TIP]
    > It may take several minutes for the Azure Machine Learning workspace to update the credentials cache. Until it has been updated, you may receive errors when trying to access the Azure Machine Learning workspace from Synapse.

<!-- 1. From the left of the page, select __Overview__ and then select __Launch studio__.
1. From the Azure Machine Learning studio, select __Linked Services__ and then select __Add Integration__.

    :::image type="content" source="./media/how-to-private-endpoint-integration-synapse/machine-learning-add-integration.png" alt-text="{alt-text}":::

1. Provide a friendly name for the linked Azure Synapse workspace, select the Azure subscription that contains the Synapse workspace, and then select the Synapse workspace.

    :::image type="content" source="./media/how-to-private-endpoint-integration-synapse/link-workspace.png" alt-text="{alt-text}":::

1. Select __Next__, and then select the Spark pool. Enter a name for the compute resource, and then select __Next__.
1. Select __Create__. -->

## Verify connectivity

1. From Azure Synapse Studio, select __Develop__, and then __+ Notebook__.

    :::image type="content" source="./media/how-to-private-endpoint-integration-synapse/add-synapse-notebook.png" alt-text="{alt-text}":::

1. In the __Attach to__ field, select the Apache Spark pool for your Azure Synapse workspace, and enter the following code in the first cell:

    ```python
    from notebookutils.mssparkutils import azureML

    # getWorkspace() takes the linked service name,
    # not the Azure Machine Learning workspace name.
    ws = azureML.getWorkspace("AzureMLService1")

    print(ws.name)
    ```

    This code snippet connects to the linked workspace, and then prints the workspace info. Note that in the printed output, the value displayed is the name of the Azure Machine Learning workspace, not the linked service name that was used in the `getWorkspace()` call.

---
title: Create and manage registries (preview)
titleSuffix: Azure Machine Learning
description: Learn how create registries with the CLI, Azure portal and AzureML Studio 
services: machine-learning
ms.service: machine-learning
ms.subservice: mlops
ms.author: mabables
author: ManojBableshwar
ms.reviewer: larryfr
ms.date: 9/9/2022
ms.topic: how-to
ms.custom: devx-track-python
---

# Manage Azure Machine Learning registries

Azure Machine Learning entities can be grouped into two broad categories:

* Assets such as __models__, __environments__, __components__, and __datasets__ are durable entities that are _workspace agnostic_. For example, a model can be registered with any workspace and deployed to any endpoint. 
* Resources such as __compute__, __job__, and __endpoints__ are _transient entities that are workspace specific_. For example, an online endpoint has a scoring URI that is unique to a specific instance in a specific workspace. Similarly, a job runs for a known duration and generates logs and metrics each time it's run. 

Assets lend themselves to being stored a central repository and used in different workspaces, possibly in different regions. Resources are workspace specific. 

AzureML registries enable you to create and use those assets in different workspaces. Since the target workspaces in which you can use assets hosted in a registry can be in different Azure regions, registries support multi region replication for low latency access to assets when they to be used. Creating a registry will provision Azure resources required to facilitate replication. First, Azure blob storage accounts in each supported region. Second, a single Azure Container Registry with replication enabled to each supported region. 

![Diagram of the relationship between assets in workspace and registry](./media/how-to-manage-registries/machiene-learning-registry-block-diagram.png)

## Prerequisites

[!INCLUDE [CLI v2 preres](../../includes/machine-learning-cli-v2-prereqs.md)]

## Prepare to create registry

You need to decide the following information carefully before proceeding to create a registry:

### Choosing a name

Consider the following factors before picking a name.
* Registries are meant to facilitate sharing of ML assets across teams within your organization across all workspaces. Choose a name that is reflective of the sharing scope. The name should help identify your group, division or organization. 
* Registry unique with your organization (Azure Active Directory tenant). It's recommended to prefix your team or organization name and avoid generic names. 
* Registry names can't be changed once created because they're used in IDs of models, environments and components that are referenced in code. 
  * Length can be 2-32 characters. 
  * Alphanumerics, underscore, hyphen are allowed. No other special characters. No spaces - registry names are part of model, environment, and component IDs that can be referenced in code.  
  * Name can contain underscore or hyphen but can't start with an underscore or hyphen. Needs to start with an alphanumeric. 

### Choosing Azure regions 

Registries enable sharing of assets across workspaces. To do so, a registry replicates content across multiple Azure regions. You need to define the list of regions that a registry supports when creating the registry. Create a list of all regions in which you have workspaces today and plan to add in near future. This list is a good set of regions to start with. When creating a registry, you define a primary region and a set of additional regions. The primary region can't be changed after registry creation, but the additional regions can be updated at a later point.

### Checking permissions

Make sure you're the "Owner" or "Contributor" of the subscription or resource group in which you plan to create the registry. If you don't have one of these built-in roles, review the section on permissions toward the end of this article. 


## Create a registry

# [Azure CLI](#tab/cli)

Create the YAML definition and name it `registry.yml`.

```YAML
type: registry
description: Platform to share machine learning models, components and environments
name: HelloWorldReg
resource_group: ContosoML
primary_location: eastus
locations:
 - eastus2
 - westus
```

> [!TIP]
> You typically see display names of Azure regions such as 'East US' in the Azure Portal but the registry creation YAML needs names of regions without spaces and lower case letters. Use `az account list-locations -o table` to find the mapping of region display names to the name of the region that can be specified in YAML.

Run the registry create command.

`az ml registry create --file registry.yml`

# [Studio](#tab/studio)

You can create registries in AzureML studio using the following steps:

1. In the [AzureML studio](https://ml.azure.com), select the __Registries__, and then __Manage registries__. Select __+ Create registry__.

    > [!TIP]
    > If you are in a workspace, navigate to the global UI by clicking your organization or tenant name in the navigation pane to find the __Registries__ entry.  You can also go directly there by navigating to [https://ml.azure.com/registries](https://ml.azure.com/registries).

    :::image type="content" source="./media/how-to-manage-registries/studio-create-registry-button.png" alt-text="Screenshot of the create registry screen.":::
	
1. Enter the registry name, select the subscription and resource group and then select __Next__.

    :::image type="content" source="./media/how-to-manage-registries/studio-create-registry-basics.png" alt-text="Screenshot of the registry creation basics tab.":::

1. Select the __Primary region__ and __Additional region__, then select __Next__.

    :::image type="content" source="./media/how-to-manage-registries/studio-registry-select-regions.png" alt-text="Screenshot of the registry region selection":::

1. Review the information you provided, and then select __Create__. You can track the progress of the create operation in the Azure portal. Once the registry is successfully created, you can find it listed in the __Manage Registries__ tab.

    :::image type="content" source="./media/how-to-manage-registries/studio-create-registry-review.png" alt-text="Screenshot of the create + review tab.":::
# [Azure portal](#tab/portal)

1. From the [Azure portal](https://portal.azure.com), navigate to the Azure Machine Learning service. You can get there by searching for __Azure Machine Learning__ in the search bar at the top of the page or going to __All Services__ looking for __Azure Machine Learning__ under the __AI + machine learning__ category. 

1. Select __Create__, and then select __Azure Machine Learning registry__. Enter the registry name, select the subscription, resource group and primary region, then select __Next__.

    :::image type="content" source="./media/how-to-manage-registries/create-registry-basics.png" alt-text="Screenshot of the basics tab.":::
	
1. Select the additional regions the registry must support, then select __Next__ until you arrive at the __Review + Create__ tab.

    :::image type="content" source="./media/how-to-manage-registries/create-registry-review.png" alt-text="Screenshot of the review + create tab.":::

1. Review the information and select __Create__.

---

## Add users to the registry 

Decide if you want to allow users to only use assets (models, environments and components) from the registry or both use and create assets in the registry. 

# [Use assets](#tab/use)

To let a user only read assets, you can grant the user the built-in __Reader__ role. If don't want to use the built-in role, create a custom role with the following permissions

Permission | Description 
--|--
Microsoft.MachineLearningServices/registries/read | Allows the user to list registries and get registry metadata
Microsoft.MachineLearningServices/registries/assets/read | Allows the user to browse assets and use the assets in a workspace

# [Create and use assets](#tab/create-use)

To let the user both read and create or delete assets, grant the following write permission in addition to the above read permissions.

Permission | Description 
--|--
Microsoft.MachineLearningServices/registries/assets/write | Create assets in registries
Microsoft.MachineLearningServices/registries/assets/delete| Delete assets in registries

# [Create and manage registries](#tab/create-registry)

To let users create, update and delete registries, grant them the built-in __Contributor__ or __Owner__ role. If you don't want to use built in roles, create a custom role with the following permissions, in addition to all the above permissions to read, create and delete assets in registry.

> [!WARNING]
> The built-in __Contributor__ and __Owner__ roles allow users to create, update and delete registries. 

Permission | Description 
--|--
Microsoft.MachineLearningServices/registries/write| Allows the user to create or update registries
Microsoft.MachineLearningServices/registries/delete | Allows the user to delete registries

---

## Next steps

* [Learn how to share models, components and environments across workspaces with registries using CLI (preview)](./how-to-share-models-pipelines-across-workspaces-with-registries.md)
* [Learn how to share models, components and environments across workspaces with registries using Python SDK (preview)](./how-to-share-models-pipelines-across-workspaces-with-registries-sdk.md)


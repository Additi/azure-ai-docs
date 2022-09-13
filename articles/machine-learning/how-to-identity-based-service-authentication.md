---
title: Set up service authentication
titleSuffix: Azure Machine Learning
description: Learn how to set up and configure authentication between Azure ML and other Azure services.
services: machine-learning
author: rastala
ms.author: roastala
ms.reviewer: larryfr
ms.service: machine-learning
ms.subservice: enterprise-readiness
ms.date: 09/12/2022
ms.topic: how-to
ms.custom: has-adal-ref, devx-track-js, contperf-fy21q2, subject-rbac-steps, cliv2, sdkv2, event-tier1-build-2022
---

# Set up authentication between Azure ML and other services
	
Azure Machine Learning is composed of multiple Azure services. There are multiple ways that authentication can happen between Azure Machine Learning and the services it relies on.


* The Azure Machine Learning workspace uses a __managed identity__ to communicate with other services. By default, this is a system-assigned managed identity. You can also use a user-assigned managed identity instead.
* Azure Machine Learning uses Azure Container Registry (ACR) to store Docker images used to train and deploy models. If you allow Azure ML to automatically create ACR, it will enable the __admin account__.
* The Azure ML compute cluster uses a __managed identity__ to retrieve connection information for datastores from Azure Key Vault and to pull Docker images from ACR. You can also configure identity-based access to datastores, which will instead use the managed identity of the compute cluster.
* Data access can happen along multiple paths depending on the data storage service and your configuration. For example, authentication to the datastore may use an account key, token, security principal, managed identity, or user identity.

    For information on how data access is authenticated, see the [Data administration](how-to-administrate-data-authentication.md) article.

* Managed online endpoints can use a managed identity to access Azure resources when performing inference. For more information, see [Access Azure resources from an online endpoint](how-to-access-resources-from-endpoints-managed-identities.md).

## Prerequisites

* An Azure Machine Learning workspace. For more information, see [Create workspace resources](quickstart-create-resources.md).
* The [Azure Machine Learning SDK v2](https://aka.ms/sdk-v2-install).

    > [!IMPORTANT]
    > SDK v2 is currently in public preview.
    > The preview version is provided without a service level agreement, and it's not recommended for production workloads. Certain features might not be supported or might have constrained capabilities. 
    > For more information, see [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

* Install the [Azure CLI](/cli/azure/install-azure-cli).
* To assign roles, the login for your Azure subscription must have the [Managed Identity Operator](../role-based-access-control/built-in-roles.md#managed-identity-operator) role, or other role that grants the required actions (such as __Owner__).
* You must be familiar with creating and working with [Managed Identities](../active-directory/managed-identities-azure-resources/overview.md).

## User-assigned managed identity

With the default system-assigned managed identities, you have to grant access to storage and other resources for each workspace. A new system-assigned identity is created for each new workspace. User-assigned managed identities can be re-used, allowing you to grant access to storage and other resources for multiple workspaces. The same applies to Azure RBAC role assignments when granting permissions to the managed identities. 

To use a user-assigned managed identity with your workspace, compute instance, or compute cluster, use the following information:

### Workspace

> [!IMPORTANT]
> When creating workspace with user-assigned managed identity, you must perform the following steps first:
> 1. Create a managed identity.
> 1. Create the Azure Storage Account, Azure Container Registry, and Azure Key Vault resources that will be used by Azure Machine Learning.
> 1. Use the [role assignment ARM template](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.machinelearningservices/machine-learning-dependencies-role-assignment) to grant the managed identity the required role assignments to the services.

Use Azure CLI or Python SDK to create the workspace. When using the CLI, specify the ID using the `--primary-user-assigned-identity` parameter. When using the SDK, use `primary_user_assigned_identity`. The following are examples of using the Azure CLI and Python to create a new workspace using these parameters:

# [Azure CLI](#tab/cli)

[!INCLUDE [cli v2](../../includes/machine-learning-cli-v2.md)]

```azurecli-interactive
az ml workspace create -w <workspace name> -g <resource group> --primary-user-assigned-identity <managed identity ARM ID>
```

# [Python SDK](#tab/pythonsdk)

[TBD v2]

# [Studio](#tab/azure-studio)

Currently there is no option to create a workspace from Azure Machine Learning studio. However, you can create using the Azure portal. Use the following steps while creating the cluster:

1. From the __Basics__ page, select the Azure Storage Account, Azure Container Registry, and Azure Key Vault you want to use with the workspace.
1. From the __Advanced__ page, select __User-assigned identity__ and then select the managed identity to use.

---

You can also use [an ARM template](https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.machinelearningservices/) to create a workspace with user-assigned managed identity.

> [!TIP]
> For a workspace with [customer-managed keys for encryption](concept-data-encryption.md), you can pass in a user-assigned managed identity to authenticate from storage to Key Vault. Use the `user-assigned-identity-for-cmk-encryption` (CLI) or `user_assigned_identity_for_cmk_encryption` (SDK) parameters to pass in the managed identity. This managed identity can be the same or different as the workspace primary user assigned managed identity.

### Compute instance

### Compute cluster

[!INCLUDE [cli v2](../../includes/machine-learning-cli-v2.md)]

# [Azure CLI](#tab/cli)

To create a new managed compute cluster with managed identity, use this command:

```azurecli
az ml compute create -f create-cluster.yml
```

Where the contents of *create-cluster.yml* are as follows: 

:::code language="yaml" source="~/azureml-examples-main/cli/resources/compute/cluster-user-identity.yml":::

For comparison, the following example is from a YAML file that creates a cluster that uses a system-assigned managed identity:

:::code language="yaml" source="~/azureml-examples-main/cli/resources/compute/cluster-system-identity.yml":::

If you have an existing compute cluster, you can change between user-managed and system-managed identity. The following examples demonstrate how to change the configuration:

__User-assigned managed identity__

:::code language="azurecli" source="~/azureml-examples-main/cli/deploy-mlcompute-update-to-user-identity.sh":::

__System-assigned managed identity__

:::code language="azurecli" source="~/azureml-examples-main/cli/deploy-mlcompute-update-to-system-identity.sh":::

# [Studio](#tab/azure-studio)

During cluster creation or when editing compute cluster details, in the **Advanced settings**, toggle **Assign a managed identity** and specify a system-assigned identity or user-assigned identity.

---

## Scenario: Azure Container Registry without admin user

When you disable the admin user for ACR, Azure ML uses a managed identity to build and pull docker images. There are two workflows when configuring Azure ML to use an ACR with the admin user disabled:

* Allow Azure ML to create the ACR instance and then disable the admin user afterwards.
* Bring an existing ACR with the admin user already disabled.

### Azure ML with auto-created ACR instance

1. Create a new Azure Machine Learning workspace.
1. Perform an action that requires Azure Container Registry. For example, the [Tutorial: Train your first model](tutorial-1st-experiment-sdk-train.md).
1. Get the name of teh ACR created by the cluster.

    ```azurecli-interactive
    az ml workspace show -w <my workspace> \
    -g <my resource group>
    --query containerRegistry
    ```

    This command returns a value similar to the following text. You only want the last portion of the text, which is the ACR instance name:

    ```output
    /subscriptions/<subscription id>/resourceGroups/<my resource group>/providers/MicrosoftContainerReggistry/registries/<ACR instance name>
    ```

1. Update the ACR to disable the admin user:

    ```azurecli-interactive
    az acr update --name <ACR instance name> --admin-enabled false
    ```

### Bring your own ACR

If ACR admin user is disallowed by subscription policy, you should first create ACR without admin user, and then associate it with the workspace. Also, if you have existing ACR with admin user disabled, you can attach it to the workspace.

[Create ACR from Azure CLI](../container-registry/container-registry-get-started-azure-cli.md) without setting ```--admin-enabled``` argument, or from Azure portal without enabling admin user. Then, when creating Azure Machine Learning workspace, specify the Azure resource ID of the ACR. The following example demonstrates creating a new Azure ML workspace that uses an existing ACR:

> [!TIP]
> To get the value for the `--container-registry` parameter, use the [az acr show](/cli/azure/acr#az-acr-show) command to show information for your ACR. The `id` field contains the resource ID for your ACR.

[!INCLUDE [cli v2](../../includes/machine-learning-cli-v2.md)]

```azurecli-interactive
az ml workspace create -w <workspace name> \
-g <workspace resource group> \
-l <region> \
--container-registry /subscriptions/<subscription id>/resourceGroups/<acr resource group>/providers/Microsoft.ContainerRegistry/registries/<acr name>
```

### Create compute with managed identity to access Docker images for training

To access the workspace ACR, create machine learning compute cluster with system-assigned managed identity enabled. You can enable the identity from  Azure portal or Studio when creating compute, or from Azure CLI using the below. For more information, see [using managed identity with compute clusters](how-to-create-attach-compute-cluster.md#set-up-managed-identity).

# [Python](#tab/python)

When creating a compute cluster with the [AmlComputeProvisioningConfiguration](/python/api/azureml-core/azureml.core.compute.amlcompute.amlcomputeprovisioningconfiguration), use the `identity_type` parameter to set the managed identity type.

# [Azure CLI](#tab/azure-cli)

[!INCLUDE [cli v2](../../includes/machine-learning-cli-v2.md)]

```azurecli-interaction
az ml compute create --name cpucluster --type <cluster name>  --identity-type systemassigned
```

# [Portal](#tab/azure-portal)

For information on configuring managed identity when creating a compute cluster in studio, see [Set up managed identity](how-to-create-attach-compute-cluster.md#set-up-managed-identity).

---

A managed identity is automatically granted ACRPull role on workspace ACR to enable pulling Docker images for training.

> [!NOTE]
> If you create compute first, before workspace ACR has been created, you have to assign the ACRPull role manually.

### Use Docker images for inference

Once you've configured ACR without admin user as described earlier, you can access Docker images for inference without admin keys from your Azure Kubernetes service (AKS). When you create or attach AKS to workspace, the cluster's service principal is automatically assigned ACRPull access to workspace ACR.

> [!NOTE]
> If you bring your own AKS cluster, the cluster must have service principal enabled instead of managed identity.

## Scenario: Use a private Azure Container Registry

By default, Azure Machine Learning uses Docker base images that come from a public repository managed by Microsoft. It then builds your training or inference environment on those images. For more information, see [What are ML environments?](concept-environments.md).

To use a custom base image internal to your enterprise, you can use managed identities to access your private ACR. There are two use cases:

 * Use base image for training as is.
 * Build Azure Machine Learning managed image with custom image as a base.

### Pull Docker base image to machine learning compute cluster for training as is

Create machine learning compute cluster with system-assigned managed identity enabled as described earlier. Then, determine the principal ID of the managed identity.

[!INCLUDE [cli v2](../../includes/machine-learning-cli-v2.md)]

```azurecli-interactive
az ml compute show --name <cluster name> -w <workspace> -g <resource group>
```

Optionally, you can update the compute cluster to assign a user-assigned managed identity:

[!INCLUDE [cli v2](../../includes/machine-learning-cli-v2.md)]

```azurecli-interactive
az ml compute update --name <cluster name> --user-assigned-identities <my-identity-id>
```

To allow the compute cluster to pull the base images, grant the managed service identity ACRPull role on the private ACR

[!INCLUDE [cli v2](../../includes/machine-learning-cli-v2.md)]

```azurecli-interactive
az role assignment create --assignee <principal ID> \
--role acrpull \
--scope "/subscriptions/<subscription ID>/resourceGroups/<private ACR resource group>/providers/Microsoft.ContainerRegistry/registries/<private ACR name>"
```

Finally, when submitting a training job, specify the base image location in the [environment definition](how-to-use-environments.md#use-existing-environments).

[!INCLUDE [sdk v1](../../includes/machine-learning-sdk-v1.md)]

```python
from azureml.core import Environment
env = Environment(name="private-acr")
env.docker.base_image = "<ACR name>.azurecr.io/<base image repository>/<base image version>"
env.python.user_managed_dependencies = True
```

> [!IMPORTANT]
> To ensure that the base image is pulled directly to the compute resource, set `user_managed_dependencies = True` and do not specify a Dockerfile. Otherwise Azure Machine Learning service will attempt to build a new Docker image and fail, because only the compute cluster has access to pull the base image from ACR.

### Build Azure Machine Learning managed environment into base image from private ACR for training or inference

[!INCLUDE [cli v2](../../includes/machine-learning-cli-v2.md)]

In this scenario, Azure Machine Learning service builds the training or inference environment on top of a base image you supply from a private ACR. Because the image build task happens on the workspace ACR using ACR Tasks, you must perform more steps to allow access.

1. Create __user-assigned managed identity__ and grant the identity ACRPull access to the __private ACR__.  
1. Grant the workspace __system-assigned managed identity__ a Managed Identity Operator role on the __user-assigned managed identity__ from the previous step. This role allows the workspace to assign the user-assigned managed identity to ACR Task for building the managed environment. 

    1. Obtain the principal ID of workspace system-assigned managed identity:

        ```azurecli-interactive
        az ml workspace show -w <workspace name> -g <resource group> --query identityPrincipalId
        ```

    1. Grant the Managed Identity Operator role:

        ```azurecli-interactive
        az role assignment create --assignee <principal ID> --role managedidentityoperator --scope <user-assigned managed identity resource ID>
        ```

        The user-assigned managed identity resource ID is Azure resource ID of the user assigned identity, in the format `/subscriptions/<subscription ID>/resourceGroups/<resource group>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<user-assigned managed identity name>`.

1. Specify the external ACR and client ID of the __user-assigned managed identity__ in workspace connections by using [Workspace.set_connection method](/python/api/azureml-core/azureml.core.workspace.workspace#set-connection-name--category--target--authtype--value-):

    [!INCLUDE [sdk v1](../../includes/machine-learning-sdk-v1.md)]

    ```python
    workspace.set_connection(
        name="privateAcr", 
        category="ACR", 
        target = "<acr url>", 
        authType = "ManagedIdentity", 
        value={"ResourceId": "<user-assigned managed identity resource id>", "ClientId": "<user-assigned managed identity client ID>"})
    ```

Once the configuration is complete, you can use the base images from private ACR when building environments for training or inference. The following code snippet demonstrates how to specify the base image ACR and image name in an environment definition:

[!INCLUDE [sdk v1](../../includes/machine-learning-sdk-v1.md)]

```python
from azureml.core import Environment

env = Environment(name="my-env")
env.docker.base_image = "<acr url>/my-repo/my-image:latest"
```

Optionally, you can specify the managed identity resource URL and client ID in the environment definition itself by using [RegistryIdentity](/python/api/azureml-core/azureml.core.container_registry.registryidentity). If you use registry identity explicitly, it overrides any workspace connections specified earlier:

[!INCLUDE [sdk v1](../../includes/machine-learning-sdk-v1.md)]

```python
from azureml.core.container_registry import RegistryIdentity

identity = RegistryIdentity()
identity.resource_id= "<user-assigned managed identity resource ID>"
identity.client_id="<user-assigned managed identity client ID>"
env.docker.base_image_registry.registry_identity=identity
env.docker.base_image = "my-acr.azurecr.io/my-repo/my-image:latest"
```

## Next steps

* Learn more about [enterprise security in Azure Machine Learning](concept-enterprise-security.md)
* Learn about [identity-based data access](how-to-identity-based-data-access.md)
* Learn about [managed identities on compute cluster](how-to-create-attach-compute-cluster.md).
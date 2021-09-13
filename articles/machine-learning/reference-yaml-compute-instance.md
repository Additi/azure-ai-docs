---
title: 'CLI (v2) compute instance YAML schema'
titleSuffix: Azure Machine Learning
description: Reference documentation for the CLI (v2) compute instance YAML schema.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference

author: lostmygithubaccount
ms.author: copeters
ms.date: 09/20/2021
ms.reviewer: laobri
---

# CLI (v2) compute instance YAML schema

[!INCLUDE [preview disclaimer](../../includes/machine-learning-preview-generic-disclaimer.md)]

## YAML syntax

| Key | Type | Description | Allowed values | Default value |
| --- | ---- | ----------- | -------------- | ------- |
| `$schema` | string | The YAML schema. If you use the Azure Machine Learning VS Code extension to author the YAML file, including `$schema` at the top of your file enables you to invoke schema and resource completions. | | |
| `type` | string | **Required.** The type of compute. | `computeinstance` | |
| `name` | string | **Required.** Name of the compute. | | |
| `description` | string | Description of the compute. | | |
| `tags` | object | Dictionary of tags for the compute. | | |
| `size` | string | The VM size to use for the compute instance. For more information, see [Supported VM series and sizes](concept-compute-target.md#supported-vm-series-and-sizes). Note that not all sizes are available in all regions. | For the list of supported sizes in a given region, please use the `az ml compute list-sizes` command.  | `Standard_NC6` |
| `create_on_behalf_of` | object | Settings for creating the compute instance on behalf of another user. Please ensure that the assigned user has correct RBAC permissions. |  |  |
| `create_on_behalf_of.user_tenant_id` | string | The AAD Tenant ID of the assigned user. |  |  |
| `create_on_behalf_of.user_object_id` | string | The AAD Object ID of the assigned user. |  |  |
| `ssh_public_access_enabled` | boolean | Whether to enable public SSH access on the compute instance. | | `false` |
| `ssh_settings` | object | SSH settings for connecting to the compute instance. | | |
| `ssh_settings.admin_username` | string | The name of the administrator user account that can be used to SSH into the compute instance. | | |
| `ssh_settings.ssh_key_value` | string | The SSH public key of the administrator user account. | | |
| `network_settings` | object | Network security settings. | | |
| `network_settings.vnet_name` | string | Name of the virtual network (VNet) when creating a new one or referencing an existing one. | | |
| `network_settings.subnet` | string | Either the name of the subnet when creating a new VNet or referencing an existing one, or the fully qualified resource ID of a subnet in an existing VNet. Do not specify `network_settings.vnet_name` if the subnet ID is specified. The subnet ID can refer to a VNet/subnet in another resource group. | | |
| `network_settings.private_ip_only` | boolean | Whether to remove the public IP address from the compute instance. | | `false` |

## Remarks

The `az ml compute` command can be used for managing Azure Machine Learning compute.

## Schema

The source JSON schema can be found at https://azuremlschemas.azureedge.net/latest/workspace.schema.json. The schema is provided below in JSON and YAML formats for convenience.

# [JSON](#tab/json)

:::code language="json" source="~/azureml-examples-main/cli/.schemas/jsons/latest/workspace.schema.json":::

# [YAML](#tab/yaml)

:::code language="yaml" source="~/azureml-examples-main/cli/.schemas/yamls/latest/workspace.schema.yml":::

---

## Next steps

- [Install and use the CLI (v2)](how-to-configure-cli.md)

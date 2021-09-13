---
title: 'CLI (v2) attached Virtual Machine YAML schema'
titleSuffix: Azure Machine Learning
description: Reference documentation for the CLI (v2) attached Virtual Machine schema.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference

author: lostmygithubaccount
ms.author: copeters
ms.date: 09/20/2021
ms.reviewer: laobri
---

# CLI (v2) attached Virtual Machine YAML schema

[!INCLUDE [preview disclaimer](../../includes/machine-learning-preview-generic-disclaimer.md)]

## YAML syntax

| Key | Type | Description | Allowed values | Default value |
| --- | ---- | ----------- | -------------- | ------- |
| `$schema` | string | The YAML schema. If you use the Azure Machine Learning VS Code extension to author the YAML file, including `$schema` at the top of your file enables you to invoke schema and resource completions. | | |
| `type` | string | **Required.** The type of compute. | `virtualmachine` | |
| `name` | string | **Required.** Name of the compute. | | |
| `description` | string | Description of the compute. | | |
| `tags` | object | Dictionary of tags for the compute. | | |
| `resource_id` | string | **Required.** Fully qualified resource ID of the Azure Virtual Machine to attach to the workspace as a compute target. | | |

## Remarks

The `az ml compute` command can be used for managing Virtual Machines (VM) attached to an Azure Machine Learning workspace.

## Examples

[TODO]

## Schema

The source JSON schema can be found at https://azuremlschemas.azureedge.net/latest/vmCompute.schema.json. The schema is provided below in JSON and YAML formats for convenience.

# [JSON](#tab/json)

:::code language="json" source="~/azureml-examples-cli-preview/cli/.schemas/jsons/latest/vmCompute.schema.json":::

# [YAML](#tab/yaml)

:::code language="yaml" source="~/azureml-examples-cli-preview/cli/.schemas/yamls/latest/vmCompute.schema.yml":::

---

## Next steps

- [Install and use the CLI (v2)](how-to-configure-cli.md)

---
title: 'CLI (v2) environment YAML schema'
titleSuffix: Azure Machine Learning
description: Reference documentation for the CLI (v2) environment YAML schema.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference

author: lostmygithubaccount
ms.author: copeters
ms.date: 09/20/2021
ms.reviewer: laobri
---

# CLI (v2) environment YAML schema

[!INCLUDE [preview disclaimer](../../includes/machine-learning-preview-generic-disclaimer.md)]

## YAML syntax

| Key | Type | Description | Allowed values | Default value |
| --- | ---- | ----------- | -------------- | ------- |
| `$schema` | string | The YAML schema. If you use the Azure Machine Learning VS Code extension to author the YAML file, including `$schema` at the top of your file enables you to invoke schema and resource completions. | | |
| `name` | string | **Required.** Name of the environment. | | |
| `version` | string | Version of the environment. If omitted, Azure ML will autogenerate a version. | | |
| `description` | string | Description of the environment. | | |
| `tags` | object | Dictionary of tags for the environment. | | |
| `image` | string | The Docker image to use for the environment. **One of `image` or `build` is required.** | | |
| `conda_file` | string or object | The standard conda YAML configuration file of the dependencies for a conda environment. See https://conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html#creating-an-environment-file-manually. <br> <br> If specified, `image` must be specified as well. Azure ML will build the conda environment on top of the Docker image provided. | | |
| `build` | object | The Docker build context configuration to use for the environment. **One of `image` or `build` is required.** | | |
| `build.local_path` | string | Local path to the directory to use as the build context. | | |
| `build.dockerfile_path` | string | Relative path to the Dockerfile within the build context. | | `Dockerfile` |
| `os_type` | string | The type of operating system. | `linux`, `windows` | `linux` |  
| `inference_config` | object | Inference container configurations. Only applicable if the environment is used to build a serving container for online deployments. See [Attributes of the `inference_config` key](#attributes-of-the-inference_config-key). | | |

### Attributes of the `inference_config` key

| Key | Type | Description |
| --- | ---- | ----------- |
| `liveness_route` | object | The liveness route for the serving container. |
| `liveness_route.path` | string | The path to route liveness requests to. |
| `liveness_route.port` | integer | The port to route liveness requests to. |
| `readiness_route` | object | The readiness route for the serving container. |
| `readiness_route.path` | string | The path to route readiness requests to. |
| `readiness_route.port` | integer | The port to route readiness requests to. |
| `scoring_route` | object | The scoring route for the serving container. |
| `scoring_route.path` | string | The path to route scoring requests to. |
| `scoring_route.port` | integer | The port to route scoring requests to. |

## Remarks

The `az ml environment` command can be used for managing Azure Machine Learning environments.

## Schema

The source JSON schema can be found at https://azuremlschemas.azureedge.net/latest/environment.schema.json. The schema is provided below in JSON and YAML formats for convenience.

# [JSON](#tab/json)

:::code language="json" source="~/azureml-examples-cli-preview/cli/.schemas/jsons/latest/environment.schema.json":::

# [YAML](#tab/yaml)

:::code language="yaml" source="~/azureml-examples-cli-preview/cli/.schemas/yamls/latest/environment.schema.yml":::

---

## Next steps

- [Install and use the CLI (v2)](how-to-configure-cli.md)

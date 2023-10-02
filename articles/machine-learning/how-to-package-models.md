---
title: Package a model for online deployment
titleSuffix: Azure Machine Learning
description:  Learn how you can package a model for online serving using model packages
author: santiagxf
ms.author: fasantia
ms.reviewer: mopeakande
ms.service: machine-learning
ms.subservice: mlops
ms.date: 10/04/2023
ms.topic: how-to
---

# Package a model for online deployment

Model packages is a capability in Azure Machine Learning that allows you to collect all the dependencies required to deploy a machine learning model to a serving platform. Creating packages before deploying models provides robust and reliable deployment and a more efficient MLOps practice. Packages can be moved across workspaces and even outside of Azure Machine Learning. 

In this article, you learn how package a model and deploy it to an Online Endpoint in Azure Machine Learning.

## Package models before deployments

The simplest way to deploy models with packages is by indicating Azure Machine Learning to do so before executing the deployment. When creating a deployment in an Online Endpoints, just indicate to prepackage the model. This is supported in the Azure CLI, Azure Machine Learning SDK, and Azure Machine Learning studio.

# [Azure CLI](#tab/cli)

```azurecli
az ml online-deployment create  -f deployment.yml --package-model
```

# [Python](#tab/sdk)

The workspace is the top-level resource for Azure Machine Learning, providing a centralized place to work with all the artifacts you create when you use Azure Machine Learning. In this section, we connect to the workspace in which you perform deployment tasks.

```python
ml_client.batch_deployment.create(
    ManagedOnlineDeployment(
        name="default",
        endpoint_name=endpoint_name,
        model=model,
        instance_count=1,
        package_model=True,
    )
).result()
```

# [Studio](#tab/studio)

In the model detail page in [Azure Machine Learning studio](https://ml.azure.com), select the option **Deploy** and then click on **Online Endpoints**. In the creation wizard, you see an option to package the model before deployment.

:::image type="content" source="./media/model-packaging/model-package-ux.png" alt-text="An screenshot of the model deployment wizard to Online Endpoints highlighting the Package model option.":::

---

Azure Machine Learning packages the model first and then execute the deployment. Notice that when using packages, if you indicate a base environment with conda or pip dependencies, you don't need to include the dependencies of the inference server (`azureml-inference-server-http`). it's automatically added for you.

> [!TIP] 
> Packaging MLflow models before deployment is highly advisable and required for endpoints without outbound networking connectivity. MLflow models indicate their dependencies in the model itself, which requires dynamic installation of packages. When an MLflow model is packaged, this dynamic installation is performed at packaging time, avoiding the installation of software at deployment time.

## Create model packages

You can create model packages explicitly, allowing you to control how the operation is done. Use this workflow when:

> [!div class="checklist"]
> * You want to customize how the package is created.
> * You want to deploy the package outside of Azure Machine Learning.
> * You want to use packages in an MLOps workflow.

Model packages can be creating by indicating the model to package, the serving technology that you want to use to run the model, and the base image that's going to be used to package the model. Base images are indicated using **Environments** in Azure Machine Learning. For MLflow models, base environments and scoring scripts are automatically generated.

> [!NOTE]
> Each model package can contain only 1 single model. Packaging multiple models under the same package isn't supported by the moment.


### Example: Packaging a custom model

In the following example, we package a model of type **custom** to finally deploy it to an online endpoint for online inference.

#### Connect to your workspace

First, let's connect to Azure Machine Learning workspace where we're going to work on.

# [Azure CLI](#tab/cli)

```azurecli
az account set --subscription <subscription>
az configure --defaults workspace=<workspace> group=<resource-group> location=<location>
```

# [Python](#tab/sdk)

The workspace is the top-level resource for Azure Machine Learning, providing a centralized place to work with all the artifacts you create when you use Azure Machine Learning. In this section, we connect to the workspace in which you perform deployment tasks.

1. Import the required libraries:

    ```python
    from azure.ai.ml import MLClient, Input
    from azure.ai.ml.entities import ManagedOnlineEndpoint, ManagedOnlineDeployment, Model
    from azure.ai.ml.constants import AssetTypes
    from azure.identity import DefaultAzureCredential
    ```

2. If you're running in a Compute Instance in Azure Machine Learning, create an `MLClient` as follows:

    ```python
    ml_client = MLClient.from_config(DefaultAzureCredential())
    ```

    Otherwise, configure workspace details and get a handle to the workspace:

    ```python
    subscription_id = "<subscription>"
    resource_group = "<resource-group>"
    workspace = "<workspace>"
    
    ml_client = MLClient(DefaultAzureCredential(), subscription_id, resource_group, workspace)
    ```

# [Studio](#tab/studio)

Navigate to [Azure Machine Learning studio](https://ml.azure.com).

---


#### Registering the model

In this case, we already have a local copy of the model in the repository, so we only need to publish the model to the registry in the workspace. You can skip this step if the model you're trying to deploy is already registered.
   
# [Azure CLI](#tab/cli)

```azurecli
MODEL_NAME='sklearn-diabetes'
MODEL_PATH='model'
az ml model create --name $MODEL_NAME --path $MODEL_PATH --type custom_model
```

# [Python](#tab/sdk)

```python
model_name = "sklearn-regression"
model_path = "model"
model = ml_client.models.create_or_update(
    Model(name=model_name, path=model_path, type=AssetTypes.CUSTOM_MODEL)
)
```

# [Studio](#tab/studio)

To create a model in Azure Machine Learning, open the Models page in Azure Machine Learning. Select **Register model** and select where your model is located. Fill out the required fields, and then select __Register__.

:::image type="content" source="./media/how-to-manage-models/register-model-as-asset.png" alt-text="Screenshot of the UI to register a model." lightbox="./media/how-to-manage-models/register-model-as-asset.png":::

---



#### The base environment

Models require an environment indicating all the packages and software they need to run. In packages, such environment is called the *base environment*. For MLflow models, the base environment is automatically detected by Azure Machine Learning and hence they're optional. For custom models, they're required. 

In this example we create an environment using a base image and the following conda file:

__conda.yaml__

```yaml
name: model-env
channels:
  - conda-forge
dependencies:
  - python=3.9
  - numpy=1.23.5
  - pip=23.0.1
  - scikit-learn=1.2.2
  - scipy=1.10.1
```

> [!TIP]
> **What's the difference with the environments you use to deploy to Online or Batch Endpoints?** When you deployed to Online Endpoints, you need to include the dependencies of the model + the packages required by Managed Online Endpoints to work. This brings a manual process into the deployment where you have to combine the requirements of your model with the requirements of the serving platform. Packages remove that friction since the **required packages for the inference server will be injected at packaging time automatically**.

We can create the environment as follows:

# [Azure CLI](#tab/cli)

```azurecli
az ml environment create -f environment/sklearn-regression-env.yml
```

# [Python](#tab/sdk)

```python
base_environment = ml_client.environments.create_or_update(
    Environment(
        name=f"{model_name}-env",
        image="mcr.microsoft.com/azureml/openmpi4.1.0-ubuntu22.04",
        conda_file="environment/conda.yaml"
    )
)
```

# [Studio](#tab/studio)

1. Navigate to the section **Environments** in Azure Machine Learning studio.

1. Select the tab **Custom environments**.

1. Click on **Create**.

1. Configure the wizard as follows:

    1. Idicate *sklearn-regression-env* as **Name**.

    1. Indicate *use existing docker image with optional conda file* as **Select environment source**.

    1. Leave **Container registry image path** as defaults.

1. Click on **Next**.

1. On **Customize** paste the following conda YAML definition:

    ```yml
    name: model-env
    channels:
      - conda-forge
    dependencies:
      - python=3.9
      - numpy=1.23.5
      - pip=23.0.1
      - scikit-learn=1.2.2
      - scipy=1.10.1
      - xgboost==1.3.3
    ``` 

1. Finish the wizard.

1. Your environment is ready to be used.

---

#### Creating a package for a custom model

Packages can be created in Azure Machine Learning using the Azure CLI or the Azure Machine Learning SDK for Python:

# [Azure CLI](#tab/cli)

To create a model package, we need to create a package specification. The package supports the following attributes:

| Attribute                               | Type      | Description | Required |
|-----------------------------------------|-----------|-------------|----------|
| `target_environment_name`               | `str`     | The name of the package to create. Packages are materialized as environments is Azure Machine Learning | Yes |
| `base_environment_source`               | `object`  | The base image to use to create the package where dependencies for the model are indicated. | Yes, unless model is MLflow. |
| `base_environment_source.type`          | `str`     | The type of the base image. Only using another environment as the base image is supported (`type: environment_asset`) is supported.  |  |
| `base_environment_source.resource_id`   | `str`     | The resource ID of the base environment to use. Use format `azureml:<name>:<version>` or a long resource id.   |  |
| `inferencing_server`                    | `object`  | The inferencing server to use. | Yes |
| `inferencing_server.type`               | `azureml_online` <br /> `custom` | Use `azureml_online` to Azure Machine Learning inferencing server, or `custom` for a custom online server like TensorFlow serving, Torch Serve, etc. | Yes |
| `inferencing_server.code_configuration` | `object`  | The code configuration with the inference routine. It should contain at least one Python file with methods `init` and `run`. | Yes, unless model is MLflow. |
| `model_configuration`                   | `object`  | The model configuration. Use this attribute to control how the model is packaged in the resulting image. | No |
| `model_configuration.mode`              | `download` <br /> `copy` | Indicate how the model would be placed in the package. Possible values are `download` and `copy`. Defaults to `download`. | No  |

__package-moe.yml__

```yml
$schema: http://azureml/sdk-2-0/ModelVersionPackage.json
base_environment_source:
    type: environment_asset
    resource_id: azureml:sklearn-regression-env@latest
target_environment_name: sklearn-regression-online-pkg
inferencing_server: 
    type: azureml_online
    code_configuration:
      code: src
      entry_script: score.py
```

# [Python](#tab/sdk)

To create a model package, we need to create a package specification. The package supports the following attributes:

| Attribute                             | Type                  | Description | Required |
|---------------------------------------|-----------------------|-------------|----------|
| `target_environment_name`             | `str`                 | The name of the package to create. Packages are materialized as environments is Azure Machine Learning | Yes |
| `base_environment_source`             | `BaseEnvironment`     | The base image to use to create the package where dependencies for the model are indicated. | Yes, unless model type is MLflow. |
| `base_environment_source.type`        | `BaseEnvironmentType` | The type of the base image. Only using another environment (`EnvironmentAsset`) as the base image is supported.  |  |
| `base_environment_source.resource_id` | `str`                 | The resource ID of the base environment to use. Use format `azureml:<name>:<version>` or a long-format resource id. |  |
| `inferencing_server`                  | `AzureMLOnlineInferencingServer` <br /> `CustomOnlineInferenceServer` | The inferencing server to use. Use `AzureMLOnlineInferencingServer` to Azure Machine Learning inferencing server, or `CustomOnlineInferenceServer` for a custom online server like TensorFlow serving, Torch Serve, etc. <br /><br />If set to `AzureMLInferencingServer` and the model type isn't Mlflow, a code configuration section should be indicated containing at least one Python file with methods `init` and `run`. <br /><br />If set to `CustomOnlineInferenceServer`, an online server configuration section should be indicated.  | Yes |
| `model_configuration`                 | `ModelConfiguration`  | The model configuration. Use this attribute to control how the model is packaged in the resulting image. | No |
| `model_configuration.mode`            | `ModelInputMode`      | Indicate how the model would be placed in the package. Possible values are `ModelInputMode.DOWNLOAD` (default) and `ModelInputMode.COPY`. Downloading the model helps to make packages more lighweighted, specially for large models. However, it requires packages to be deployed to Azure Machine Learning. Copying, on the other hand, generated bigger packages as all the artifacts are copied on it, but they can be deployed anywhere. | No |


```python
package_config = ModelPackage(
    target_environment_name="sklearn-regression-online-pkg",
    base_environment_source=BaseEnvironment(
        type=BaseEnvironmentType.EnvironmentAsset, 
        resource_id=base_environment.id
    ),
    inferencing_server=AzureMLOnlineInferencingServer(
        code_configuration=CodeConfiguration(
            code='src', 
            scoring_script='score.py'
        )
    ),
)
```

# [Studio](#tab/studio)

Creating packages in studio isn't supported by the moment. Use the Azure CLI or Azure Machine Learning SDK for Python.

---

Let's start the package operation:

# [Azure CLI](#tab/cli)

```azurecli
az ml model package -n $MODEL_NAME -l latest --file package-moe.yml
```

# [Python](#tab/sdk)

```python
model_package = ml_client.models.begin_package(model_name, model.version, package_config)
```

# [Studio](#tab/studio)

Creating packages in studio isn't supported by the moment. Use the Azure CLI or Azure Machine Learning SDK for Python.

---

The result of the package operation is an environment.

#### Deploy model packages to online endpoints

Model packages can be deployed directly to Online Endpoint in Azure Machine Learning. Follow these steps to deploy a package to an online endpoint:

1. Let's create an endpoint to host the deployment of the package. We need to pick a name for it:

    # [Azure CLI](#tab/cli)

    ```azurecli
    ENDPOINT_NAME = "sklearn-regression-online"
    ```

    # [Python](#tab/sdk)
    
    ```python
    endpoint_name = "sklearn-regression-online"
    ```
    
    # [Studio](#tab/studio)
    
    Deploying custom model packages isn't supported in studio by the moment. Use the option [Package models before deployments](#package-models-before-deployments) to deploy standard model packages from studio or use the Azure CLI or Azure Machine Learning SDK for Python.
    
1. Create the endpoint:

    # [Azure CLI](#tab/cli)

    ```azurecli
    az ml online-endpoint create -n $ENDPOINT_NAME
    ```

    # [Python](#tab/sdk)
    
    ```python
    endpoint = ManagedOnlineEndpoint(name=endpoint_name)
    endpoint = ml_client.online_endpoints.begin_create_or_update(endpoint).result()
    ```

    # [Studio](#tab/studio)
    
    Deploying custom model packages isn't supported in studio by the moment. Use the option [Package models before deployments](#package-models-before-deployments) to deploy standard model packages from studio.

1. Crate the deployment using the package. Notice how `environment` is configured with the package we've just created.

    # [Azure CLI](#tab/cli)

    __deployment.yml__
    
    ```yml
    $schema: https://azuremlschemas.azureedge.net/latest/managedOnlineDeployment.schema.json
    name: with-package
    endpoint_name: hello-packages
    environment: azureml:sklearn-regression-online-pkg@latest
    instance_type: Standard_DS3_v2
    instance_count: 1
    ```

    # [Python](#tab/sdk)
    
    ```python
    deployment_package = ManagedOnlineDeployment(
        name="with-package",
        endpoint_name=endpoint_name,
        environment=model_package,
        instance_count=1
    )
    ```

    # [Studio](#tab/studio)

    Deploying custom model packages isn't supported in studio by the moment. Use the option [Package models before deployments](#package-models-before-deployments) to deploy standard model packages from studio or use the Azure CLI or Azure Machine Learning SDK for Python.

    ---

    > [!TIP]
    > Notice how model or scoring script are not being indicated in this example. All of them are part of the package.

1. Start the deployment:

    # [Azure CLI](#tab/cli)
    
    ```azurecli
    az ml online-deployment create -f model-deployment.yml
    ```

    # [Python](#tab/sdk)
    
    ```python
    ml_client.online_deployments.begin_create_or_update(deployment_package).result()
    ```

    # [Studio](#tab/studio)

    Deploying custom model packages isn't supported in studio by the moment. Use the option [Package models before deployments](#package-models-before-deployments) to deploy standard model packages from studio or use the Azure CLI or Azure Machine Learning SDK for Python.

1. At this point, the deployment is ready to be consumed. We can test how it's working by creating a sample request file:

    __sample-request.json__

    ```json
    {
        "data": [
            [1,2,3,4,5,6,7,8,9,10], 
            [10,9,8,7,6,5,4,3,2,1]
        ]
    }
    ```

1. Let's send the request to the endpoint: 

    # [Azure CLI](#tab/cli)

    ```azurecli
    az ml online-endpoint invoke -n $ENDPOINT_NAME -d with-package -f sample-request.json
    ```

    # [Python](#tab/sdk)
    
    ```python
    ml_client.online_endpoints.invoke(
        endpoint_name=endpoint_name, 
        deployment_name="with-package", 
        request_file="sample-request.json"
    )
    ```

    # [Studio](#tab/studio)

    1. Go to the section **Endpoints**.

    1. Select the endpoint you've just created.

    1. Select the tab **Test**.

    1. In the section **Input data to test endpoint**, paste the following content:

        ```json
        {
            "data": [
                [1,2,3,4,5,6,7,8,9,10], 
                [10,9,8,7,6,5,4,3,2,1]
            ]
        }
        ```

    1. Click on **Test**.

1. You should see two numeric predictions being generated.

## Package models with dependencies in private Python feeds 

Model packages can resolve Python dependencies that are available in private feeds. To use this capability, you need to create a connection from your workspace to the feed and specify the credentials. The following Python code shows how you can configure the workspace where you're running the package operation.

```python
from azure.ai.ml.entities import WorkspaceConnection
from azure.ai.ml.entities import SasTokenConfiguration

# fetching secrets from env var to secure access, these secrets can be set outside or source code
python_feed_sas = os.environ["PYTHON_FEED_SAS"]

credentials = SasTokenConfiguration(sas_token=python_feed_sas)

ws_connection = WorkspaceConnection(
    name="<connection_name>",
    target="<python_feed_url>",
    type="python_feed",
    credentials=credentials,
)

ml_client.connections.create_or_update(ws_connection)
```

Once the connection is created, build the model package as usual. In the following example, the **base environment** of the package is using a private feed for the Python dependency `bar` indicated in a conda file:

__conda.yml__

```yml
name: foo
channels:
  - defaults
dependencies:
  - python
  - pip
  - pip:
    - --extra-index-url <python_feed_url>
    - bar
```

If your model is MLflow, model dependencies are indicated inside the model itself and hence a **base environment** isn't needed. In that case, indicate private feed dependencies when logging the model as explained at [Logging models with a custom signature, environment or samples](how-to-log-mlflow-models.md#logging-models-with-a-custom-signature-environment-or-samples).

## Package models in registries

Model packages provide a convenient way to collect dependencies before deployment. However, when models are hosted in registries, the deployment target is usually another workspace. When creating packages in this setup, use the property `target_environment_name` to indicate the full location where you want the package to be created instead of just its name.

The following example creates a package of the `t5-base` model:

1. Connect to the registry where the model is placed and the workspace you need the package to be created to:

    # [Azure CLI](#tab/cli)

    ```azurecli
    az login
    ```

    # [Python](#tab/sdk)
    
    ```python
    workspace_client = MLClient.from_config(
        credential=DefaultAzureCredential(), 
    )
    registry_client = MLClient(
        credential=DefaultAzureCredential(), 
        registry_name="azureml"
    )
    ```
    
    # [Studio](#tab/studio)
    
    Creating packages in studio isn't supported by the moment. Use the Azure CLI or Azure Machine Learning SDK for Python.
    
1. Get a reference to the model you want to package:

    # [Azure CLI](#tab/cli)

    ```azurecli
    MODEL_NAME="t5-base"
    MODEL_VERSION=$(az ml model show --name $MODEL_NAME --label latest --registry-name azureml | jq .version -r)
    ```

    # [Python](#tab/sdk)
    
    ```python
    model_name = "t5-base"
    model = registry_client.models.get(name=model_name, label="latest")
    ```
    
    # [Studio](#tab/studio)
    
    Creating packages in studio isn't supported by the moment. Use the Azure CLI or Azure Machine Learning SDK for Python.
    
1. Create a reference to the workspace you want to target the package to. Package uses the model in the registry but produce an environment (the packaged model) in the target workspace:

    # [Azure CLI](#tab/cli)

    The following section gets the subscription, resource group name, and workspace name, from the defaults values in the console. Replace them with specific values if needed.

    ```azurecli
    SUBSCRIPTION_ID=$(az account show | jq .tenantId -r)
    RESOURCE_GROUP_NAME=$(az configure --list-defaults | jq '.[] | select(.name == "group") | .value' -r)
    WORKSPACE_NAME=$(az configure --list-defaults | jq '.[] | select(.name == "workspace") | .value' -r)
    MODEL_PACKAGE_NAME="pkg-$MODEL_NAME-$MODEL_VERSION"
    MODEL_PACKAGE_VERSION=$(date +%s)

    TARGET_ENVIRONMENT="/subscriptions/$SUBSCRIPTION_ID/resourceGroups/$RESOURCE_GROUP_NAME/providers/Microsoft.MachineLearningServices/workspaces/$WORKSPACE_NAME/environments/$MODEL_PACKAGE_NAME/versions/$MODEL_PACKAGE_VERSION"
    ```

    # [Python](#tab/sdk)

    The following sections gets the subscription, resource group name, and workspace name, from the workspace `MLClient`. Replace them with specific values if needed.
    
    ```python
    import time

    model_package_name = f"pkg-{model.name}-{model.version}"
    model_package_version = str(int(time.time()))

    target_environment = f"/subscriptions/{ml_client.subscription_id}/resourceGroups/{ml_client.resource_group_name}/providers/Microsoft.MachineLearningServices/workspaces/{ml_client.workspace_name}/environments/{model_package_name}/versions/{model_package_version}"
    ```
    
    # [Studio](#tab/studio)
    
    Creating packages in studio isn't supported by the moment. Use the Azure CLI or Azure Machine Learning SDK for Python.
    
1. Create a package configuration:

    # [Azure CLI](#tab/cli)
    
    __package.yml__
    
    ```yml
    $schema: http://azureml/sdk-2-0/ModelVersionPackage.json
    target_environment_name: $TARGET_ENVIRONMENT
    inferencing_server: 
        type: azureml_online
    ```
    
    # [Python](#tab/sdk)
    
    ```python
    package_config = ModelPackage(
        target_environment_name=target_environment,
        inferencing_server=AzureMLOnlineInferencingServer(
            code_configuration=CodeConfiguration()
        ),
    )
    ```
    
    # [Studio](#tab/studio)
    
    Creating packages in studio isn't supported by the moment. Use the Azure CLI or Azure Machine Learning SDK for Python.
    
    ---

    > [!NOTE]
    > See how `taget_environment_name` now contains the name of a resource.
    
2. Let's start the package operation:

    # [Azure CLI](#tab/cli)
    
    ```azurecli
    az ml model package --name $MODEL_NAME \
                        --version $MODEL_VERSION \
                        --registry-name azureml \
                        --file package.yml \
                        --set target_environment_name=$TARGET_ENVIRONMENT
    ```

    > [!TIP]
    > Notice the command `--set target_environment_name` to dynamically change the property value from the YAML definition.
    
    # [Python](#tab/sdk)
    
    ```python
    model_package = ml_client.models.begin_package(model_name, model.version, package_config)
    ```
    
    # [Studio](#tab/studio)
    
    Creating packages in studio isn't supported by the moment. Use the Azure CLI or Azure Machine Learning SDK for Python.
    
1. The package is created in the target workspace and ready to be deployed.


## Create packages to deploy outside of Azure Machine Learning

Model packages can be deployed outside of Azure Machine Learning if needed.  To guarantee portability, you only need to ensure that the **model configuration** in your package has the mode set to **copy**. That means that the model itself is copied inside of the generated docker image instead of referenced from the model registry in Azure Machine Learning.

The following example shows how to configure copy in a package:

# [Azure CLI](#tab/cli)

__package-azure-app.yml__

```yml
$schema: http://azureml/sdk-2-0/ModelVersionPackage.json
base_environment_source:
    type: environment_asset
    resource_id: azureml:sklearn-regression-env@latest
target_environment_name: sklearn-regression-docker-pkg
inferencing_server: 
    type: azureml_online
    code_configuration:
        code: src
        entry_script: score.py
model_configuration:
    mode: copy
```

# [Python](#tab/sdk)

```python
package_config = ModelPackage(
    target_environment_name="sklearn-regression-online-pkg",
    base_environment_source=BaseEnvironment(
        type=BaseEnvironmentType.EnvironmentAsset, 
        resource_id=base_environment.id
    ),
    inferencing_server=AzureMLOnlineInferencingServer(
        code_configuration=CodeConfiguration(
            code='src', 
            scoring_script='score.py'
        )
    ),
    model_configuration=ModelConfiguration(mode="copy")
)
```

# [Studio](#tab/studio)

TODO

---
    
See an example about [how to deploy Azure Machine Learning models to Azure App Service](how-to-package-models-app-service.md).

## Next steps

> [!div class="nextstepaction"]
> [Package and deploy a model to App Service](how-to-package-models-app-service.md)

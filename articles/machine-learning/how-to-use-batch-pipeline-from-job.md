---
title: How to deploy existing pipeline jobs to a batch endpoint
description: Learn how to create pipeline component deployment for Batch Endpoints
author: santiagxf
ms.author: fasantia
ms.service: machine-learning
ms.topic: how-to
ms.date: 05/01/2023
ms.custom: how-to, devplatv2
---

# Deploy existing pipeline jobs to batch endpoints

Batch endpoints allow the deployment of pipeline components, providing a convenient way to operationalize pipelines in Azure Machine Learning. Batch endpoints can accept pipeline components to be deployed. If you already have a pipeline job that run successfully, we can take that job as input and create the pipeline component automatically for you. In this article, you'll learn how to achieve so.

[!INCLUDE [machine-learning-preview-generic-disclaimer](../../includes/machine-learning-preview-generic-disclaimer.md)]

## About this example

In this example, we're going to deploy a pipeline consisting of a simple command job that prints "hello world!". Instead of registering the component before the deployment, we are indicating an existing pipeline job for deployment. When done, batch endpoints will create the pipeline component automatically for us and deploy it under a batch endpoint pipeline component deployment.

[!INCLUDE [machine-learning-batch-clone](../../includes/machine-learning/azureml-batch-clone-samples.md)]

The files for this example are in:

```azurecli
cd endpoints/batch/deploy-pipelines/hello-batch
```

## Prerequisites

[!INCLUDE [machine-learning-batch-prereqs](../../includes/machine-learning/azureml-batch-prereqs.md)]

## Run the pipeline job you want to deploy

To demostrate it, let's first run a pipeline job:

# [Azure CLI](#tab/cli)

The following `pipeline-job.yml` file contains the configuration for the pipeline job:

__pipeline-job.yml__

:::code language="yaml" source="~/azureml-examples-batch-pup/cli/endpoints/batch/deploy-pipelines/hello-batch/pipeline-job.yml" :::

# [Python](#tab/python)

Load the pipeline component and instanciate it:

```python
hello_batch = load_component(source="hello-component/hello.yml")
pipeline_job = hello_batch()
```

Now, we'll configure some run settings to run the test:

```python
pipeline_job.settings.default_compute = "batch-cluster"
pipeline_job.settings.default_datastore = "workspaceblobstore"
```
---

Create the pipeline job:

# [Azure CLI](#tab/cli)

:::code language="azurecli" source="~/azureml-examples-batch-pup/cli/endpoints/batch/deploy-pipelines/hello-batch/cli-deploy.sh" ID="run_pipeline_job_deployment" :::

# [Python](#tab/python)

```python
pipeline_job_run = ml_client.jobs.create_or_update(
    pipeline_job, experiment_name="hello-batch-pipeline"
)
pipeline_job_run
```

---


## Create a batch endpoint

Now, let being with the deployment of the pipeline job. First, we need to deploy a batch endpoint to host the deployment.

1. Provide a name for the endpoint. A batch endpoint's name needs to be unique in each region since the name is used to construct the invocation URI. To ensure uniqueness, append any trailing characters to the name specified in the following code.

    # [Azure CLI](#tab/cli)

    :::code language="azurecli" source="~/azureml-examples-batch-pup/cli/endpoints/batch/deploy-pipelines/hello-batch/cli-deploy.sh" ID="name_endpoint" :::

    # [Python](#tab/python)

    ```python
    endpoint_name="hello-batch-component"
    ```

1. Configure the endpoint:

    # [Azure CLI](#tab/cli)
    
    The `endpoint.yml` file contains the endpoint's configuration.

    __endpoint.yml__
    
    :::code language="yaml" source="~/azureml-examples-batch-pup/cli/endpoints/batch/deploy-pipelines/hello-batch/endpoint.yml" :::

    # [Python](#tab/python)

    ```python
    endpoint = BatchEndpoint(
        name=endpoint_name,
        description="A hello world endpoint for component deployments",
    )
    ```

1. Create the endpoint:

    # [Azure CLI](#tab/cli)

    :::code language="azurecli" source="~/azureml-examples-batch-pup/cli/endpoints/batch/deploy-pipelines/hello-batch/cli-deploy.sh" ID="create_endpoint" :::

    # [Python](#tab/python)

    ```python
    ml_client.batch_endpoints.begin_create_or_update(endpoint).result()
    ```

1. Query the endpoint URI:

    # [Azure CLI](#tab/cli)

    :::code language="azurecli" source="~/azureml-examples-batch-pup/cli/endpoints/batch/deploy-pipelines/hello-batch/cli-deploy.sh" ID="query_endpoint" :::

    # [Python](#tab/python)

    ```python
    endpoint = ml_client.batch_endpoints.get(name=endpoint_name)
    print(endpoint)
    ```

## Deploy the pipeline job

To deploy the pipeline component, we have to create a batch deployment from the existing job.

1. We need to indicate to Azure Machine Learning the name of the job we want to deploy. In our case, that job is indicated in the following variable:

    # [Azure CLI](#tab/cli)
    
    ```azurecli
    echo $JOB_NAME
    ```
    
    # [Python](#tab/python)

    ```python
    print(job.name)
    ```

1. Configure the deployment:

    # [Azure CLI](#tab/cli)
    
    The `deployment-from-job.yml` file contains the deployment's configuration. Notice now how we use the key `job_definition` instead of `component`:

    __deployment-from-job.yml__

    :::code language="yaml" source="~/azureml-examples-batch-pup/cli/endpoints/batch/deploy-pipelines/hello-batch/deployment-from-job.yml" :::
    
    # [Python](#tab/python)

    Notice now how we use the property `job_definition` instead of `component`:
    
    ```python
    deployment = BatchPipelineComponentDeployment(
        name="hello-batch-from-job,
        description="A hello world deployment with a single step. This deployment is created from a pipeline job.",
        endpoint_name=endpoint.name,
        job_definition=pipeline_job_run,
        settings={
            "default_comput": "batch-cluster",
            "continue_on_step_failure": False
        }
    )
    ```
    
1. Create the deployment:

    # [Azure CLI](#tab/cli)
    
    Run the following code to create a batch deployment under the batch endpoint and set it as the default deployment.
    
    :::code language="azurecli" source="~/azureml-examples-batch-pup/cli/endpoints/batch/deploy-pipelines/hello-batch/cli-deploy.sh" ID="create_deployment_from_job" :::
    
    > [!TIP]
    > Notice the use of `--set job_definition=azureml:$JOB_NAME`. Since job names are unique, the command `--set` is used here to change the name of the job when your run it in your workspace.

    # [Python](#tab/python)

    This command will start the deployment creation and return a confirmation response while the deployment creation continues.

    ```python
    ml_client.batch_deployments.begin_create_or_update(deployment).result()
    ```
    
    Once created, let's configure this new deployment as the default one:

    ```python
    endpoint = ml_client.batch_endpoints.get(endpoint.name)
    endpoint.defaults.deployment_name = deployment.name
    ml_client.batch_endpoints.begin_create_or_update(endpoint).result()
    ```

1. Your deployment is ready for use.

### Test the deployment

Once the deployment is created, it's ready to receive jobs. You can invoke the default deployment as follows:

# [Azure CLI](#tab/cli)

:::code language="azurecli" source="~/azureml-examples-batch-pup/cli/endpoints/batch/deploy-pipelines/hello-batch/cli-deploy.sh" ID="invoke_deployment_inline" :::

# [Python](#tab/python)

```python
job = ml_client.batch_endpoints.invoke(
    endpoint_name=endpoint.name, 
)
```

---

You can monitor the progress of the show and stream the logs using:

# [Azure CLI](#tab/cli)

:::code language="azurecli" source="~/azureml-examples-batch-pup/cli/endpoints/batch/deploy-pipelines/hello-batch/cli-deploy.sh" ID="stream_job_logs" :::

# [Python](#tab/python)

```python
ml_client.jobs.get(name=job.name)
```

To wait for the job to finish, run the following code:

```python
ml_client.jobs.get(name=job.name).stream()
```
---

## Clean up resources

Once you're done, delete the associated resources from the workspace:

# [Azure CLI](#tab/cli)

Run the following code to delete the batch endpoint and its underlying deployment. `--yes` is used to confirm the deletion.

:::code language="azurecli" source="~/azureml-examples-batch-pup/cli/endpoints/batch/deploy-pipelines/hello-batch/cli-deploy.sh" ID="delete_endpoint" :::

# [Python](#tab/python)

Delete the endpoint:

```python
ml_client.batch_endpoints.begin_delete(endpoint.name).result()
```
---

(Optional) Delete compute, unless you plan to reuse your compute cluster with later deployments.

# [Azure CLI](#tab/cli)

```azurecli
az ml compute delete -n batch-cluster
```

# [Python](#tab/python)

```python
ml_client.compute.begin_delete(name="batch-cluster")
```
---



## Next steps
---
title: 'Train models (create jobs) with the CLI (v2)'
titleSuffix: Azure Machine Learning
description: Learn how to train models (create jobs) using Azure CLI extension for Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: how-to
author: lostmygithubaccount
ms.author: copeters
ms.date: 10/18/2021
ms.reviewer: laobri
ms.custom: devx-track-azurecli, devplatv2
---

# Train models with the CLI (v2)

The Azure Machine Learning CLI (v2) is an Azure CLI extension enabling you to accelerate the model training process while scaling up and out on Azure compute, with the model lifecycle tracked and auditable.

Training a machine learning model is typically an iterative process. Modern tooling makes it easier than ever to train larger models on more data faster. Previously tedious manual processes like hyperparameter tuning and even algorithm selection are often automated. With the Azure Machine Learning CLI you can track your jobs (and models) in a [workspace](concept-workspace.md) with hyperparameter sweeps, scale-up on high-performance Azure compute, and scale-out utilizing distributed training.

[!INCLUDE [preview disclaimer](../../includes/machine-learning-preview-generic-disclaimer.md)]

## Prerequisites

- To use the CLI, you must have an Azure subscription. If you don't have an Azure subscription, create a free account before you begin. Try the [free or paid version of Azure Machine Learning](https://azure.microsoft.com/free/) today.
- [Install and set up the Azure CLI extension for Machine Learning](how-to-configure-cli.md)

> [!TIP]
> For a full-featured development environment, use Visual Studio Code and the [Azure Machine Learning extension](how-to-setup-vs-code.md) to [manage Azure Machine Learning resources](how-to-manage-resources-vscode.md) and [train machine learning models](tutorial-train-deploy-image-classification-model-vscode.md).

### Clone examples repository

To run the training examples, first clone the examples repository and change into the `cli` directory:

:::code language="azurecli" source="~/azureml-examples-cli-preview/cli/misc.sh" id="git_clone":::

Note that `--depth 1` clones only the latest commit to the repository which reduces time to complete the operation.

### Create compute

You can create an Azure Machine Learning compute cluster from the command line. For instance, the following commands will create one cluster named `cpu-cluster` and one named `gpu-cluster`.

:::code language="azurecli" source="~/azureml-examples-cli-preview/repo-setup/create-compute.sh" id="create_computes":::

Note that you are not charged for compute at this point as `cpu-cluster` and `gpu-cluster` will remain at 0 nodes until a job is submitted. Learn more about how to [manage and optimize cost for AmlCompute](how-to-manage-optimize-cost.md#use-azure-machine-learning-compute-cluster-amlcompute).

The following example jobs in this article use one of `cpu-cluster` or `gpu-cluster`. Adjust these as needed to the name of your cluster(s). Use `az ml compute create -h` for more details on compute create options.

[!INCLUDE [arc-enabled-kubernetes](../../includes/machine-learning-create-arc-enabled-training-computer-target.md)]

## Hello world

For the Azure Machine Learning CLI (v2), jobs are authored in YAML format. A job aggregates:

- What to run
- How to run it
- Where to run it

The "hello world" job has all three:

:::code language="yaml" source="~/azureml-examples-cli-preview/cli/jobs/basics/hello-world.yml":::

>[!WARNING] Python must be installed in the environment used for jobs. Run `apt-get update -y && apt-get install python3 -y` in your Dockerfile to install if needed, or derive from a base image with Python installed already. This limitation will be removed in a future release.

Which you can run:

:::code language="azurecli" source="~/azureml-examples-cli-preview/cli/train.sh" id="hello_world":::

>[!TIP] The `--web` parameter will attempt to open your job in the Azure Machine Learning studio using your default web browser. The `--stream` parameter can be used to stream logs to the console and block further commands.

## Overriding values with `--set`

YAML job specification values can be overridden using `--set` when creating or updating a job. For instance:

:::code language="azurecli" source="~/azureml-examples-cli-preview/cli/train.sh" id="hello_world_set":::

>[!TIP] `--set` is useful for changing the compute target or training parameters when experimenting. It can also be used with `az ml job update`, which is shown below.

## Job names

Most `az ml job` commands other than `create` and `list` require `--name/-n`, which is a job's name or "Run ID" in the studio. It is strongly discouraged to directly set a job's `name` property during creation as it must be unique per workspace. Azure Machine Learning generates a random GUID for the job name if it is not set which can be obtained from the output of job creation in the CLI or by copying the "Run ID" property in the studio.

For organization of jobs set a `display_name` instead which does not have to be unique.

For automation of jobs you can capture a job's name when it is created by querying and stripping the output by adding `--query name -o tsv`. The specifics will vary by shell, but for Bash:

:::code language="azurecli" source="~/azureml-examples-cli-preview/cli/train.sh" id="hello_world_name":::

Then use `$run_id` in subsequent commands like `update`, `show`, or `stream`:

:::code language="azurecli" source="~/azureml-examples-cli-preview/cli/train.sh" id="hello_world_show":::

## Organizing with display name, description, and tags

To organize jobs, you can set a display name, description, and tags. Descriptions support markdown syntax in the studio. These properties are mutable after a job is created. A full example:

:::code language="yaml" source="~/azureml-examples-cli-preview/cli/jobs/basics/hello-world-org.yml":::

You can run this job, where these properties will be immediately visible in the studio:

:::code language="azurecli" source="~/azureml-examples-cli-preview/cli/train.sh" id="hello_world_org":::

Using `--set` we can update the mutable values after the job is created:

:::code language="azurecli" source="~/azureml-examples-cli-preview/cli/train.sh" id="hello_world_org_set":::

## Environment variables

You can set environment variables for use in your job:

:::code language="yaml" source="~/azureml-examples-cli-preview/cli/jobs/basics/hello-world-env-var.yml":::

You can run this job:

:::code language="azurecli" source="~/azureml-examples-cli-preview/cli/train.sh" id="hello_world_env_var":::

>[!WARNING] It is recommended to use `inputs` for passing arguments in the `command`. See [inputs and outputs](##inputs-and-outputs).

## Tracking models and their source code

Production machine learning models need to be auditable if not reproducible. It is crucial to keep track of the source code for a given model. Azure Machine Learning takes a snapshot of your source code and keeps it with the job. Additionally, the source repository and commit are kept if you are running jobs from a Git repository.

>[!TIP] If you're following along and running from the examples repository, you can see the source repository and commit in the studio on any of the jobs run so far.

You can specify the `code.local_path` key in a job with the value as the path to a source code directory. A snapshot of the directory is taken and uploaded with the job. The contents of the directory are directly available from the working directory of the job.

>[!TIP] The source code should not include large data inputs for model training. Instead, [use data inputs](###data-inputs). You can use a `.gitignore` file in the source code directory to exclude files from the snapshot.

Let's look at a job which specifies code:

:::code language="yaml" source="~/azureml-examples-cli-preview/cli/jobs/basics/hello-mlflow.yml":::

The Python script is in the local source code directory. The command then invokes `python` to run the script. The same pattern can be applied for other programming languages.

>[!WARNING] The "hello" family of jobs shown in this article are for demonstration purposes do not necessarily follow recommended best practices. Using `&&` or similar to run many jobs in a sequence is not recommended - instead, consider writing the commands to a script file in the source code directory and invoking the script in your `command`. Installing dependencies in the `command`, as shown above via `pip install`, is not recommended - instead, all job dependencies should be specified as part of your environment. See [how to manage environments with the CLI (v2)](TODO) for details.

### Model tracking with MLflow

While iterating on models, data scientists need to be able to keep track of model parameters and training metrics. Azure Machine Learning integrates with MLflow tracking to enable the logging of models, artifacts, metrics, and parameters to a job. To use MLflow in your Python scripts just `import mlflow` and call `mlflow.log_*` or `mlflow.autolog()` APIs in your training code.

>[!WARNING] The `mlflow` and `azureml-mlflow` packages must be installed in your Python environment for MLflow tracking features.

>[!TIP] `mlflow.autolog()` is supported for many popular frameworks and takes care of the majority of logging for you.

Let's take a look at Python script invoked in the job above which uses `mlflow` to log a parameter, a metric, and an artifact:

:::code language="python" source="~/azureml-examples-cli-preview/cli/jobs/basics/src/hello-mlflow.py":::

We can run this job in the cloud via Azure Machine Learning, where it is tracked and auditable:

:::code language="azurecli" source="~/azureml-examples-cli-preview/cli/train.sh" id="hello_mlflow":::

### Query metrics with MLflow

After running jobs, you might want to query runs and their resulting metrics. Python is better suited for this task than a CLI. You can query runs and their metrics via `mlflow` and load into familiar objects like Pandas dataframes for analysis.

First, retrieve the MLflow tracking URI for you Azure Machine Learning workspace:

:::code language="azurecli" source="~/azureml-examples-cli-preview/cli/train.sh" id="mlflow_uri":::

Then use this in `mlflow.set_tracking_uri(<YOUR_TRACKING_URI>)`. MLflow calls will now corresponding to jobs in your Azure Machine Learning workspace.

## Inputs and outputs

Jobs typically have inputs, including model parameters and training data, and generate artifacts. This section overviews the different types of inputs which can then be used in training models.

### Literal inputs

Literal inputs are directly inferred in the command. We can modify our "hello world" job to use literal inputs:

:::code language="yaml" source="~/azureml-examples-cli-preview/cli/jobs/basics/hello-world-input.yml":::

You can run this job:

:::code language="azurecli" source="~/azureml-examples-cli-preview/cli/train.sh" id="hello_world_input":::

You can use `--set` to override inputs:

:::code language="azurecli" source="~/azureml-examples-cli-preview/cli/train.sh" id="hello_world_input_set":::

Literal inputs to jobs can easily be [converted to search space inputs](###search-space-inputs) for hyperparameter sweeps on model training.

### Search space inputs

For a sweep job, you can use specify a search space for literal inputs to be chosen from. For the full range of options for search space inputs, see the [search space YAML syntax reference documentation](TODO).

>[!WARNING] Sweep jobs are not currently supported in pipeline jobs. This limitation will be removed in a future release.

We can demonstrate the concept with a simple Python script which takes in arguments and logs a random metric:

:::code language="python" source="~/azureml-examples-cli-preview/cli/jobs/basics/src/hello-sweep.py":::

And create a corresponding sweep job:

:::code language="yaml" source="~/azureml-examples-cli-preview/cli/jobs/basics/hello-world.yml":::

And run it:

:::code language="azurecli" source="~/azureml-examples-cli-preview/cli/train.sh" id="hello_sweep":::

### Default outputs

The `./outputs` and `./logs` directories receive special treatment by Azure Machine Learning. If you write any files to these directories during your job, these files will get uploaded to the job so that you can still access them once the it is complete. The `./outputs` folder is uploaded at the end of the job, while the files written to `./logs` are uploaded in real time. Use the latter if you want to stream logs during the job, such as TensorBoard logs.

With this, we can easily modify the "hello world" job to output to a file in the default outputs directory instead of printing to `stdout`:

:::code language="yaml" source="~/azureml-examples-cli-preview/cli/jobs/basics/hello-world-output.yml":::

You can run this job:

:::code language="azurecli" source="~/azureml-examples-cli-preview/cli/train.sh" id="hello_world_output":::

And download the logs, where `helloworld.txt` will be present in the `<RUN_ID>/outputs/` directory:

:::code language="azurecli" source="~/azureml-examples-cli-preview/cli/train.sh" id="hello_world_output_download":::

### Data inputs

Data inputs are inferred as a path on the job compute's local filesystem. Let's demonstrate with the classic Iris dataset, which is hosted publicly in a blob container at `https://azuremlexamples.blob.core.windows.net/datasets/iris.csv`.

We can take a simple Python script which takes the path to the Iris CSV file as an argument, reads it into a dataframe, prints out the first 5 lines, and saves it to the `outputs` directory.

:::code language="python" source="~/azureml-examples-cli-preview/cli/jobs/basics/src/hello-iris.py":::

Azure storage URI inputs can be specified which will mount or download data to the local filesystem. You can specify a single file:

:::code language="yaml" source="~/azureml-examples-cli-preview/cli/jobs/basics/hello-iris-file.yml":::

And run:

:::code language="azurecli" source="~/azureml-examples-cli-preview/cli/train.sh" id="iris_file":::

Or specify an entire folder:

:::code language="yaml" source="~/azureml-examples-cli-preview/cli/jobs/basics/hello-iris-folder.yml":::

And run:

:::code language="azurecli" source="~/azureml-examples-cli-preview/cli/train.sh" id="iris_folder":::

For private data in Azure Blob or Azure Data Lake Storage connected to Azure Machine Learning through a datastore, you can leverage Azure Machine Learning URIs which take the format `azureml://datastores/<DATASTORE_NAME>/paths/<PATH_TO_DATA>` for input data. For instance, if we upload the Iris CSV to a directory named `"example-data"` in the Blob container corresponding to the datastore named `workspaceblobstore` we can modify our job to use a file:

>[!WARNING] Running these jobs will fail for you if you have not copied the Iris CSV to the same location.

:::code language="yaml" source="~/azureml-examples-cli-preview/cli/jobs/basics/hello-iris-datastore-file.yml":::

Or the entire directory:

:::code language="yaml" source="~/azureml-examples-cli-preview/cli/jobs/basics/hello-iris-datastore-folder.yml":::

### Data outputs

You can specify named data outputs. This will create a directory in the default datastore which will be read/write mounted by default.

We can modify the earlier "hello world" job to write to a named data output:

:::code language="yaml" source="~/azureml-examples-cli-preview/cli/jobs/basics/hello-world-output-data.yml":::

>[!TIP] You will be able to specify locations other than the default datastore for data outputs in a future release. These non-default outputs are useful in pipelines.

## Hello pipelines

Pipeline jobs can run multiple jobs in parallel. If there are input/output dependencies between steps in a pipeline, the dependent step will run after the other completes.

We can split a "hello world" job into two jobs:

:::code language="yaml" source="~/azureml-examples-cli-preview/cli/jobs/basics/hello-pipeline.yml":::

And run it:

:::code language="azurecli" source="~/azureml-examples-cli-preview/cli/train.sh" id="hello_pipeline":::

The "hello" and "world" jobs respectively will run in parallel if the compute target has the available resources to do so.

To pass data between steps in a pipeline, we define a data output in the "hello" job. We define a corresponding input in the "world" job, which refers to the prior's output:

:::code language="yaml" source="~/azureml-examples-cli-preview/cli/jobs/basics/hello-pipeline-io.yml":::

And run it:

:::code language="azurecli" source="~/azureml-examples-cli-preview/cli/train.sh" id="hello_pipeline_io":::

This time, the "world" job will run after the "hello" job completes.

To avoid duplicating common settings across jobs in a pipeline, you can set them outside the jobs:

:::code language="yaml" source="~/azureml-examples-cli-preview/cli/jobs/basics/hello-pipeline-settings.yml":::

You can run this:

:::code language="azurecli" source="~/azureml-examples-cli-preview/cli/train.sh" id="hello_pipeline_settings":::

The corresponding setting on an individual job will override the common settings for a pipeline job. We can combine the concepts so far into a three-step jobs. The "C" job has a data dependency on the "B" job, while the "A" job can run independently. The "A" job will also use an individually set environment and bind one of its inputs to a top-level input:

:::code language="yaml" source="~/azureml-examples-cli-preview/cli/jobs/basics/hello-pipeline-abc.yml":::

You can run this:

:::code language="azurecli" source="~/azureml-examples-cli-preview/cli/train.sh" id="hello_pipeline_abc":::

## Train a model

At this point, we still haven't trained a model. Let's add some `sklearn` code into a Python script with MLflow tracking to train a model on the Iris CSV:

:::code language="python" source="~/azureml-examples-cli-preview/cli/jobs/single-step/scikit-learn/iris/src/main.py":::

The scikit-learn framework is supported by MLflow for autologging, so a single `mlflow.autolog()` call in the script will log all model parameters, training metrics, model artifacts, and some additional artifacts (in this case a confusion matrix).

To run this in the cloud, specify as a job:

:::code language="yaml" source="~/azureml-examples-cli-preview/cli/jobs/single-step/single-step/scikit-learn/iris/job.yml":::

And run it:

:::code language="azurecli" source="~/azureml-examples-cli-preview/cli/train.sh" id="sklearn_iris":::

To register a model, you can download the outputs and create a model from the local directory:

:::code language="azurecli" source="~/azureml-examples-cli-preview/cli/train.sh" id="sklearn_download_register_model":::

### Sweep hyperparameters

We can modify the previous job to sweep over hyperparameters:

:::code language="yaml" source="~/azureml-examples-cli-preview/cli/jobs/single-step/scikit-learn/iris/job-sweep.yml":::

And run it:

:::code language="azurecli" source="~/azureml-examples-cli-preview/cli/train.sh" id="sklearn_sweep":::

> [!TIP] Check the "Child runs" tab in the studio to monitor progress and view parameter charts..

For more sweep options, see the [sweep job YAML reference](TODO).

## Distributed training

Azure Machine Learning supports PyTorch, TensorFlow, and MPI-based distributed training. See the [distributed section of a command job](TODO) for details.

As an example, we'll train a simple convolutional neural network (CNN) on the CIFAR-10 dataset using distributed PyTorch. The full script is [available in the examples repository](https://github.com/Azure/azureml-examples/tree/cli-preview/jobs/single-step/pytorch/cifar-distributed/).

The CIFAR-10 dataset in `torchvision` expects as input a directory which contains the `cifar-10-batches-py` directory. We can download the zipped source and extract into a local directory:

:::code language="azurecli" source="~/azureml-examples-cli-preview/setup-repo/create-datasets.sh" id="download_untar_cifar":::

Then create an Azure Machine Learning dataset from the local directory, which will be uploaded to the default datastore:

:::code language="azurecli" source="~/azureml-examples-cli-preview/setup-repo/create-datasets.sh" id="create_cifar":::

Optionally, remove the local file and directory:

:::code language="azurecli" source="~/azureml-examples-cli-preview/setup-repo/create-datasets.sh" id="cleanup_cifar":::

Datasets (File only) can be referred to in a job using the `dataset` key of a data input. The format is `azureml:<DATASET_NAME>:<DATASET_VERSION>`, so for the CIFAR-10 dataset we just created `azureml:cifar-10-example:1`.

With the dataset in place, we can author a distributed PyTorch job to train our model:

:::code language="yaml" source="~/azureml-examples-cli-preview/cli/jobs/single-step/pytorch/cifar-distributed/job.yml":::

And run it:

:::code language="azurecli" source="~/azureml-examples-cli-preview/cli/train.sh" id="pytorch_cifar":::

## Build a training pipeline

The CIFAR-10 example above translates well to a pipeline job. We will split the previous job into three jobs for orchestration in a pipeline:

- `get-data` to run a Bash script to download and extract `cifar-10-batches-py`
- `train-model` to take the data and train a model with distributed PyTorch
- `eval-model` to take the data and the trained model and evaluate accuracy

Both `train-model` and `eval-model` will have a dependency on the `get-data` job's output. Additionally, `eval-model` will have a dependency on the `train-model` job's output. Thus the three jobs will run sequentially.

This can be authored as a pipeline job:

:::code language="yaml" source="~/azureml-examples-cli-preview/cli/jobs/pipelines/cifar-10/job.yml":::

And run:

:::code language="azurecli" source="~/azureml-examples-cli-preview/cli/train.sh" id="cifar_10_pipeline":::

## Next steps

- [Deploy and score a machine learning model with a managed online endpoint (preview)](how-to-deploy-managed-online-endpoints.md)
- [Train models with REST (preview)](how-to-train-with-rest.md)

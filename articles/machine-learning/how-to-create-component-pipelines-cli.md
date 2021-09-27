---
title: Create and run component-based ML pipelines (CLI)
titleSuffix: Azure Machine Learning
description: Create and run machine learning pipelines using the Azure Machine Learning CLI. 
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.author: laobri
author: lobrien
ms.date: 09/05/2021
ms.topic: how-to

---

# Create and run machine learning pipelines using components with the Azure Machine Learning CLI (Preview)

In this article, you learn how to create and run [machine learning pipelines](concept-ml-pipelines.md) by using the Azure CLI and Components {>>TODO<<}. You can create pipelines without using components (for more, see TODO), but components offer the greatest amount of flexibility and reuse. AzureML Pipelines may be defined in YAML and run from the CLI, authored in Python, or composed in AzureML Studio Designer with a drag-and-drop UI. This document focuses on the CLI.

[!INCLUDE [preview disclaimer](../../includes/machine-learning-preview-generic-disclaimer.md)]

## Prerequisites

* If you don't have an Azure subscription, create a free account before you begin. Try the [free or paid version of Azure Machine Learning](https://azure.microsoft.com/free/).

* You'll need an [Azure Machine Learning workspace](how-to-manage-workspace.md) for your pipelines and associated resources

* [Install and set up the Azure CLI extension for Machine Learning](how-to-configure-cli.md)

* Clone the examples repository:

    ```azurecli-interactive
    git clone https://github.com/Azure/azureml-examples --depth 1
    cd azureml-examples/cli/jobs/pipelines-with-components/
    ```

## Introducing machine learning pipelines

Pipelines in AzureML let you sequence a collection of machine learning tasks into a workflow. Data Scientists typically iterate with scripts focusing on individual tasks such as data preparation, training, scoring, and so forth. When all these scripts are ready, pipelines help connect a collection of such scripts into production-quality processes that are:

| Benefit | Description |
| --- | --- |
| Self-contained | Pipelines may run in a self-contained way for hours, or even days, taking upstream data, processing it, and passing it to subsequent scripts without any manual intervention. |
| Powerful | Pipelines may run on large compute clusters hosted in the cloud that have the processing power to crunch large datasets or to perform thousands of sweeps to find the best models. | 
| Repeatable & Automatable | Pipelines can be scheduled to run and process new data and update ML models, making ML workflows repeatable. | 
| Reproducible | Pipelines can generate reproducible results by logging all activity and persisting all outputs including intermediate data to the cloud, helping meet compliance and audit requirements. |

Azure has other types of pipelines: Azure Data Factory pipelines have strong support for data-to-data pipelines, while Azure Pipelines are the best choice for CI/CD automation. [Compare Machine Learning pipelines with these different pipelines](concept-ml-pipelines.md#which-azure-pipeline-technology-should-i-use).

## Create your first pipeline

From the `cli/jobs/pipelines-with-components/basics` directory of the`azureml-examples` repository, navigate to the `3a_basic_pipeline` subdirectory. List your available compute resources with the following command: 

```azurecli
az ml compute list
```

If you do not have it, create a cluster called `cpu-cluster` by running:

```azurecli
az ml compute create -n cpu-cluster --type amlcompute --min-instances 0 --max-instances 10
```

Now, create a pipeline job with the following command:

```azurecli
az ml job create --file pipeline.yml
```

You should receive a JSON dictionary with information about the pipeline job, including:

| Key | Description | 
| --- | --- | 
| `name` | The GUID-based name of the job. | 
| `experiment_name` | The name under which jobs will be organized in Studio. | 
| `interaction_endpoints.Studio.endpoint` | A URL for monitoring and reviewing the pipeline job. | 
| `status` | The status of the job. This will likely be `Preparing` at this point. |

### Review a component 

 Take a quick look at the Python source code in `componentA_src`, `componentB_src`, and `componentC_src`. As you can see, each of these directories contains a slightly different "Hello World" program in Python. 

Open `ComponentA.yaml` to see how the first component is defined: 

:::code language="yaml" source="~/azureml-examples-cli-preview/cli/jobs/pipelines-with-components/basics/3a_basic_pipeline/componentA.yml":::

In the current preview, only components of type `command_component` are supported {>> TODO Is this now `command`? <<}. The `name` and `display_name` values are the unique identifier and the name used in Studio to describe the component. The `version` key-value pair allows you to evolve your pipeline components while maintaining reproducibility with older versions. 

All files in the `code.local_path` value will be uploaded to Azure for processing. 

The `environment` section allows you to specify the software environment in which the component runs. In this case, the component uses a base Docker image, as specified in `environment.docker.image`. For more, see [Create & use software environments in Azure Machine Learning](how-to-use-environments.md). 

Finally, the `command` key is used to specify the command to be run.

> [!NOTE]
> The value of `command` begin with `>-` which is YAML "folding style with block-chomping." This allows you to write your command over multiple lines of text for clarity.

For more information on components and their specification, see "What is an Azure Machine Learning component (preview)?".

### Review the pipeline specification

In the example directory, the `pipeline.yaml` file looks like the following code:

:::code language="yaml" source="~/azureml-examples-cli-preview/cli/jobs/pipelines-with-components/basics/3a_basic_pipeline/pipeline.yml":::

If you open the job's URL in Studio (the value of `interaction_endpoints.Studio.endpoint` from the `job create` command), you'll see a graph representation of your pipeline:

:::image type="content" source="media/how-to-create-component-pipelines-cli/pipeline-graph.png" lightbox="media/how-to-create-component-pipelines-cli/pipeline-graph.png" alt-text="The pipeline's graph representation in Studio":::

There are no dependencies between the components in this pipeline. Generally, pipelines will have dependencies and this page will show them visually. Since these components are not dependent upon each other, and since the `cpu-cluster` had sufficient nodes, they ran concurrently. 

If you double-click on a component in the pipeline graph, you can see details of the component's child run. 

:::image type="content" source="media/how-to-create-component-pipelines-cli/component-details.png" alt-text="Screenshot showing the details and outputs of a component's child run" lightbox="media/how-to-create-component-pipelines-cli/component-details.png"::: 

## Upload and use data

The example `3b_pipeline_with_data` demonstrates how you define input and output data flow and storage in pipelines. 

You define input data directories for your pipeline in the pipeline YAML file using the `inputs` path. You define output and intermediate data directories using the `outputs` path. You use these definitions in the `jobs.{component_name}.inputs` and `jobs.{component_name}.outputs` paths, as shown in the following image:

:::image type="content" source="media/how-to-create-component-pipelines-cli/inputs-and-outputs.png" alt-text="Image showing how the inputs and outputs paths map to the jobs inputs and outputs paths" lightbox="media/how-to-create-component-pipelines-cli/inputs-and-outputs.png":::

1. The `inputs.pipeline_sample_input_data` path creates a key identifier and uploads the input data from the `local_path` directory. This key `inputs.pipeline_sample_input_data` is then used as the value of the `jobs.componentA_job.inputs.componentA_input` key. 
1. The `outputs.pipeline_sample_output_data_A` path creates a key identifier and specifies that this output data should be stored in the default workspace blobstore, at the `simple_pipeline_A` path. This key `outputs.pipeline_sample_output_data_A` is then used as the value of the `jobs.componentA_job.outputs.componentA_output` key. 
1. The `jobs.componentA_job.outputs.componentA_output` path is a key identifier that is used as the value for the subsequent step's `jobs.componentB_job.inputs.componentB_input` key. Similarly, the output of Component B is used as the input to Component C.

Studio's visualization of this pipeline looks like this: 

:::image type="content" source="media/how-to-create-component-pipelines-cli/pipeline-graph-dependencies.png" alt-text="Screenshot showing Studio's graph view of a pipeline with data dependencies" lightbox="media/how-to-create-component-pipelines-cli/pipeline-graph-dependencies.png":::

You can see that `inputs.pipeline_sample_input_data` is represented as a `Dataset`. The keys of the `jobs.{component_name}.inputs` and `outputs` paths are shown as data flows between the pipeline components.

You can run this example by switching to the `3b_pipeline_with_data` subdirectory of the samples repository and running:

`az ml job create --file pipeline.yaml`

### Access data in your script

Input and output directory paths for a component are passed to your script as arguments. The name of the argument will be the key you specified in the YAML file in the `inputs` or `outputs` path. For instance:

```python
import argparse

parser = argparse.ArgumentParser()
parser.add_argument("--componentA_input", type=str)
parser.add_argument("--componentA_output", type=str)

args = parser.parse_args()

print("componentA_input path: %s" % args.componentA_input)
print("componentA_output path: %s" % args.componentA_output)

```

For inputs, the pipeline orchestrator downloads (or mounts) the data from the cloud store and makes it available as a local folder to read from for the script that runs in each job. This behavior means the script does not need any modification between running locally and running on cloud compute. Similarly, for outputs, the script writes to a local folder that is mounted and synced to the cloud store or is uploaded after script is complete. You can use the `mode` keyword to specify download vs mount for inputs and upload vs mount for outputs. 

## Create a preparation-train-evaluate pipeline

One of the common scenarios for machine learning pipelines has three major phases:

1. Data preparation
1. Training
1. Evaluating the model

Each of these phases may have multiple components. For instance, the data preparation step may have separate steps for loading and transforming the training data. The examples repository contains an end-to-end example pipeline in the `cli/jobs/pipelines-with-components/nyc_taxi_data_regression` directory. 

The `pipeline.yml` begins with the `name` of the job and the mandatory `type: pipeline_job` key-value pair {>> TODO Nope <<}. Then, it defines inputs and outputs as follows:

:::code language="yaml" source="~/azureml-examples-cli-preview/cli/jobs/pipelines-with-components/nyc_taxi_data_regression/job.yml" id="inputs_and_outputs":::

As described previously, these entries specify the input data to the pipeline, in this case the dataset in `./data`, and the intermediate and final outputs of the pipeline, which are stored in separate paths. The names within these input and output entries become values in the `inputs` and `outputs` entries of the individual jobs: 

:::code language="yaml" source="~/azureml-examples-cli-preview/cli/jobs/pipelines-with-components/nyc_taxi_data_regression/job.yml" id="jobs":::

Notice how `jobs.train_job.outputs.model_output` is used as an input to both the prediction job and the scoring job, as shown in the following diagram: 

:::image type="content" source="media/how-to-create-component-pipelines-cli/regression-graph.png" alt-text="pipeline graph of the NYC taxi-fare prediction task" lightbox="media/how-to-create-component-pipelines-cli/regression-graph.png":::

## Caching & reuse  

By default, only those components whose inputs have changed are rerun. You can change this behavior by setting the `is_deterministic` key of the component specification YAML to `False`. A common need for this is a component that loads data that may have been updated from a fixed location or URL. 

## Next steps

- To share your pipeline with colleagues or customers, see [Publish machine learning pipelines](how-to-deploy-pipelines.md)
- Use [these Jupyter notebooks on GitHub](https://aka.ms/aml-pipeline-readme) to explore machine learning pipelines further
- See the SDK reference help for the [azureml-pipelines-core](/python/api/azureml-pipeline-core/) package and the [azureml-pipelines-steps](/python/api/azureml-pipeline-steps/) package
- See the [how-to](how-to-debug-pipelines.md) for tips on debugging and troubleshooting pipelines=
- Learn how to run notebooks by following the article [Use Jupyter notebooks to explore this service](samples-notebooks.md).

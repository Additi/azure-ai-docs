---
title: "Tutorial: Train a model"
titleSuffix: Azure Machine Learning
description: Dive in to the process of training a model  
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: tutorial
ms.reviewer: ssalgado
author: ssalgadodev
ms.author: ssalgado
ms.date: 02/21/2023
#Customer intent: As a data scientist, I want to know how to prototype and develop machine learning models on a cloud workstation.
---

# Tutorial: Train a model in Azure Machine Learning

Learn how a data scientist uses Azure Machine Learning (Azure ML) to train a model.  In this example, we use the associated credit card dataset to show how you can use AutoML for a classification problem. The goal is to predict if a customer has a high likelihood of defaulting on a credit card payment.

The training script handles the data preparation, then trains and registers a model. This tutorial takes you through steps to submit a cloud-based training job (command job). If you would like to learn more about how to load your data into Azure, see [Create data assets](how-to-create-data-assets.md). 


The steps are:

> [!div class="checklist"]
> * Connect to your Azure ML workspace
> * Create your compute resource and job environment
> * Create your training script
> * Create and run your command job to run the training script on the compute resource, configured with the appropriate job environment and the data source
> * View the output of your training script
> * Deploy the newly-trained model as an endpoint
> * Call the Azure ML endpoint for inferencing

## Prerequisites

* Azure account with an active subscription. [Create an account for free](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Complete the [Create resources you need to get started](quickstart-create-resources.md) if you need help to:
    * Create a workspace.
    * Create a cloud-based compute instance to use for your development environment.

* Complete [Run notebooks on your cloud workstation](quickstart-run-notebooks.md) if you need help to:
    * Create a new notebook, if you want to copy/paste code into cells.
    * Or, open the notebook version of this tutorial by opening **Put notebook location here** from the **Samples** section of studio.  Then select **Clone** to add the notebook to your **Files**.

## Set your notebook kernel

1. On the top bar, you see the compute instance you created during [Create resources you need to get started](quickstart-create-resources.md) to use for running the notebook.

1. If the compute instance is stopped, select **Start compute** and wait until it's running.

    :::image type="content" source="media/tutorial-azure-ml-in-a-day/start-compute.png" alt-text="Screenshot shows how to start compute if it's stopped.":::

2. Make sure that the kernel, found on the top right, is `Python 3.10 - SDK v2`.  If not, use the dropdown to select this kernel.

<!-- nbstart https://raw.githubusercontent.com/Azure/azureml-examples/new-tutorial-series/tutorials/get-started-notebooks/train-model.ipynb -->


## Use a command job to train a model in Azure Machine Learning

To train a model, you need to submit a *job*. The type of job  you'll submit in this tutorial is a *command job*. Azure Machine Learning offers several different types of jobs to train models. Users can select their method of training based on complexity of the model, data size, and training speed requirements.  In this tutorial, you'll learn how to submit a *command job* to run a *training script*. 

A command job is a function that allows you to submit a custom training script to train your model. This can also be defined as a custom training job. A command job in Azure Machine Learning is a type of job that runs a script or command in a specified environment. You can use command jobs to train models, process data, or any other custom code you want to execute in the cloud. 

In this tutorial, we'll focus on using a command job to create a custom training job that we'll use to train a model. For any custom training job, the below items are required:

* compute resource (either a compute instance or a compute cluster)
* environment
* data
* command job 
* training script


In this tutorial we'll provide all these items for our example: creating a classifier to predict customers who have a high likelihood of defaulting on credit card payments.


## Connect to the workspace

Before you dive in the code, you need to connect to your Azure ML workspace.

We're using DefaultAzureCredential to get access to workspace. DefaultAzureCredential handles most Azure SDK authentication scenarios.

In this cell, enter your Subscription ID, Resource Group name and Workspace name. To find these values:

1. In the upper right Azure Machine Learning studio toolbar, select your workspace name.
1. Copy the value for workspace, resource group and subscription ID into the code.
1. You'll need to copy one value, close the area and paste, then come back for the next one.


```python
from azure.ai.ml import MLClient
from azure.identity import DefaultAzureCredential
# authenticate
credential = DefaultAzureCredential()
# # Get a handle to the workspace
ml_client = MLClient(
     credential=credential, 
     subscription_id="<SUBSCRIPTION_ID>",
     resource_group_name="<RESOURCE_GROUP>",
     workspace_name="<AML_WORKSPACE_NAME>",
    )
```

## Create a compute cluster to run your job

In Azure, a job can refer to several tasks that Azure allows its users to do: training, pipeline creation, deployment, etc. For this tutorial and our purpose of training a machine learning model, we'll use *job* as a reference to running training computations (*training job*).

You need a compute resource for running any job in Azure Machine Learning. It can be single or multi-node machines with Linux or Windows OS, or a specific compute fabric like Spark. In Azure, there are two compute resources that you can choose from: instance and cluster. A compute instance contains one node of computation resources while a *compute cluster* contains several. A *compute cluster* contains more memory for the computation task. For training, we recommend using a compute cluster because it allows the user to distribute calculations on multiple nodes of computation, which results in a faster training experience. 

You provision a Linux compute cluster. See the [full list on VM sizes and prices](https://azure.microsoft.com/pricing/details/machine-learning/) .

For this example, you only need a basic cluster, so you use a Standard_DS3_v2 model with 2 vCPU cores, 7-GB RAM.


```python
from azure.ai.ml.entities import AmlCompute

# Name assigned to the compute cluster
cpu_compute_target = "cpu-cluster"

try:
    # let's see if the compute target already exists
    cpu_cluster = ml_client.compute.get(cpu_compute_target)
    print(
        f"You already have a cluster named {cpu_compute_target}, we'll reuse it as is."
    )

except Exception:
    print("Creating a new cpu compute target...")

    # Let's create the Azure ML compute object with the intended parameters
    cpu_cluster = AmlCompute(
        name=cpu_compute_target,
        # Azure ML Compute is the on-demand VM service
        type="amlcompute",
        # VM Family
        size="STANDARD_DS3_V2",
        # Minimum running nodes when there is no job running
        min_instances=0,
        # Nodes in cluster
        max_instances=4,
        # How many seconds will the node running after the job termination
        idle_time_before_scale_down=180,
        # Dedicated or LowPriority. The latter is cheaper but there is a chance of job termination
        tier="Dedicated",
    )
    print(
        f"AMLCompute with name {cpu_cluster.name} will be created, with compute size {cpu_cluster.size}"
    )
    # Now, we pass the object to MLClient's create_or_update method
    cpu_cluster = ml_client.compute.begin_create_or_update(cpu_cluster)
```

## Create a job environment

To run your AzureML job on your compute resource, you need an [environment](concept-environments.md). An environment lists the software runtime and libraries that you want installed on the compute where you’ll be training. It's similar to your python environment on your local machine.

AzureML provides many curated or ready-made environments, which are useful for common training and inference scenarios. You can also create your own custom environments using a docker image, or a conda configuration.

In this example, you'll create a custom conda environment for your jobs, using a conda yaml file.

First, create a directory to store the file in.


```python
import os

dependencies_dir = "./dependencies"
os.makedirs(dependencies_dir, exist_ok=True)
```

Now, create the file in the dependencies directory.


```python
%%writefile {dependencies_dir}/conda.yml
name: model-env
channels:
  - conda-forge
dependencies:
  - python=3.8
  - numpy=1.21.2
  - pip=21.2.4
  - scikit-learn=0.24.2
  - scipy=1.7.1
  - pandas>=1.1,<1.2
  - pip:
    - inference-schema[numpy-support]==1.3.0
    - xlrd==2.0.1
    - mlflow== 1.26.1
    - azureml-mlflow==1.42.0
    - psutil>=5.8,<5.9
    - tqdm>=4.59,<4.60
    - ipykernel~=6.0
    - matplotlib
```


The specification contains some usual packages, that you'll use in your job (numpy, pip).

Reference this *yaml* file to create and register this custom environment in your workspace:


```python
from azure.ai.ml.entities import Environment

custom_env_name = "aml-scikit-learn"

custom_job_env = Environment(
    name=custom_env_name,
    description="Custom environment for Credit Card Defaults job",
    tags={"scikit-learn": "0.24.2"},
    conda_file=os.path.join(dependencies_dir, "conda.yml"),
    image="mcr.microsoft.com/azureml/openmpi3.1.2-ubuntu18.04:latest",
)
custom_job_env = ml_client.environments.create_or_update(custom_job_env)

print(
    f"Environment with name {custom_job_env.name} is registered to workspace, the environment version is {custom_job_env.version}"
)
```

## Configure a training job using the command function

You create an Azure ML *command job* to train a model for credit default prediction. The command job runs a *training script* in a specified environment on a specified compute resource.  You've already created the environment and the compute cluster.  Next you'll create the training script. In our specific case, we're training our dataset to produce a classifier using the `GradientBoostingClassifier` model. 

The *training script* handles the data preparation, training and registering of the trained model. The method `train_test_split` handles splitting the dataset into test and training data. In this tutorial, you'll create a Python training script. 

Command jobs can be run from CLI, Python SDK, or studio interface. In this tutorial, you'll use the Azure ML Python SDK v2 to create and run the command job.

After running the training job, if you need to learn more about deploying the model you can follow the next tutorial in this series.

## Create training script

Let's start by creating the training script - the *main.py* python file.

First create a source folder for the script:


```python
import os

train_src_dir = "./src"
os.makedirs(train_src_dir, exist_ok=True)
```

This script handles the preprocessing of the data, splitting it into test and train data. It then consumes this data to train a tree based model and return the output model. 

[MLFlow](https://mlflow.org/docs/latest/tracking.html) is used to log the parameters and metrics during our job. The MLFlow package allows you to keep track of metrics and results for each model Azure trains. We'll be using MLFlow to first get the best model for our data, then we'll view the model's metrics on the Azure studio. If you would like to learn more about how MLFLow works with Azure Machine Learning [visit this link](concept-mlflow.md). 


```python
%%writefile {train_src_dir}/main.py
import os
import argparse
import pandas as pd
import mlflow
import mlflow.sklearn
from sklearn.ensemble import GradientBoostingClassifier
from sklearn.metrics import classification_report
from sklearn.model_selection import train_test_split

def main():
    """Main function of the script."""

    # input and output arguments
    parser = argparse.ArgumentParser()
    parser.add_argument("--data", type=str, help="path to input data")
    parser.add_argument("--test_train_ratio", type=float, required=False, default=0.25)
    parser.add_argument("--n_estimators", required=False, default=100, type=int)
    parser.add_argument("--learning_rate", required=False, default=0.1, type=float)
    parser.add_argument("--registered_model_name", type=str, help="model name")
    args = parser.parse_args()
   
    # Start Logging
    mlflow.start_run()

    # enable autologging
    mlflow.sklearn.autolog()

    ###################
    #<prepare the data>
    ###################
    print(" ".join(f"{k}={v}" for k, v in vars(args).items()))

    print("input data:", args.data)
    
    credit_df = pd.read_cvs(args.data, header=1, index_col=0)

    mlflow.log_metric("num_samples", credit_df.shape[0])
    mlflow.log_metric("num_features", credit_df.shape[1] - 1)

    #Split train and test datasets
    train_df, test_df = train_test_split(
        credit_df,
        test_size=args.test_train_ratio,
    )
    ####################
    #</prepare the data>
    ####################

    ##################
    #<train the model>
    ##################
    # Extracting the label column
    y_train = train_df.pop("default payment next month")

    # convert the dataframe values to array
    X_train = train_df.values

    # Extracting the label column
    y_test = test_df.pop("default payment next month")

    # convert the dataframe values to array
    X_test = test_df.values

    print(f"Training with data of shape {X_train.shape}")

    clf = GradientBoostingClassifier(
        n_estimators=args.n_estimators, learning_rate=args.learning_rate
    )
    clf.fit(X_train, y_train)

    y_pred = clf.predict(X_test)

    print(classification_report(y_test, y_pred))
    ###################
    #</train the model>
    ###################

    ##########################
    #<save and register model>
    ##########################
    # Registering the model to the workspace
    print("Registering the model via MLFlow")
    mlflow.sklearn.log_model(
        sk_model=clf,
        registered_model_name=args.registered_model_name,
        artifact_path=args.registered_model_name,
    )

    # Saving the model to a file
    mlflow.sklearn.save_model(
        sk_model=clf,
        path=os.path.join(args.registered_model_name, "trained_model"),
    )
    ###########################
    #</save and register model>
    ###########################
    
    # Stop Logging
    mlflow.end_run()

if __name__ == "__main__":
    main()
```

In this script, once the model is trained, the model file is saved and registered to the workspace. Registering your model allows you to store and version your models in the Azure cloud, in your workspace. Once you register a model, you can find all other registered model in one place in the Azure Studio called the model registry. The model registry helps you organize and keep track of your trained models. 

## Configure the command

Now that you have a script that can perform the classification task, use the general purpose **command** that can run command line actions. This command line action can be directly calling system commands or by running a script. 

Here, create input variables to specify the input data, split ratio, learning rate and registered model name.  The command script will:
* Use the compute created earlier to run this command.
* Use the environment created earlier - you can use the `@latest` notation to indicate the latest version of the environment when the command is run.
* Configure some metadata like display name, experiment name etc. An *experiment* is a container for all the iterations you do on a certain project. All the jobs submitted under the same experiment name are next to each other in Azure ML studio.
* Configure the command line action itself - `python main.py` in this case. The inputs/outputs are accessible in the command via the `${{ ... }}` notation.


```python
from azure.ai.ml import command
from azure.ai.ml import Input

registered_model_name = "credit_defaults_model"

job = command(
    inputs=dict(
        data=Input(
            type="uri_file",
            path="https://azuremlexamples.blob.core.windows.net/datasets/credit_card/default_of_credit_card_clients.csv",
        ),
        test_train_ratio=0.2,
        learning_rate=0.25,
        registered_model_name=registered_model_name,
    ),
    code="./src/",  # location of source code
    command="python main.py --data ${{inputs.data}} --test_train_ratio ${{inputs.test_train_ratio}} --learning_rate ${{inputs.learning_rate}} --registered_model_name ${{inputs.registered_model_name}}",
    environment="aml-scikit-learn@latest",
    compute="cpu-cluster",
    experiment_name="train_model_credit_default_prediction",
    display_name="credit_default_prediction",
)
```

## Submit the job 

It's now time to submit the job to run in Azure Machine Learning studio. This time you'll use `create_or_update`  on `ml_client.jobs`. 


```python
ml_client.create_or_update(job)
```

## View job output and wait for job completion

View the job in Azure ML studio by selecting the link in the output of the previous cell. The output of this job looks like this in Azure ML studio. Explore the tabs for various details like metrics, outputs etc. Once the model completes, this training script registers a model in your workspace. 

:::image type="content" source="media/tutorial-azure-ml-in-a-day/view-job.gif" alt-text="Screenshot shows the overview page for the job.":::

> [!IMPORTANT]
> Wait until the status of the job is complete before returning to this notebook to continue. The job will take 2 to 3 minutes to run. It could take longer (up to 10 minutes) if the compute cluster has been scaled down to zero nodes and custom environment is still building.

When you run the cell, the notebook output shows a link to the job's details page on Azure Studio. Alternatively, you can also select Jobs on the left navigation menu. A job is a grouping of many runs from a specified script or piece of code. Information for the run is stored under that job. The details page gives an overview of the job, the time it took to run, when it was created, etc. The page also has tabs to other information about the job such as metrics, Outputs + logs, and code. Listed below are the tabs available in the job's details page:

* Overview: The overview section provides basic information about the job, including its status, start and end times, and the type of job that was run
* Inputs: The input section lists the data and code that were used as inputs for the job. This section can include datasets, scripts, environment configurations, and other resources that were used during training. 
* Outputs + logs: The Outputs + logs tab contains logs generated while the job was running. This tab assists in troubleshooting if anything goes wrong with your training script or model creation.
* Metrics: The metrics tab showcases key performance metrics from your model such as training score, f1 score, and precision score. 

## Clean up resources

If you plan to continue now to other tutorials, skip to [Next steps](#next-steps).

### Stop compute instance

If you're not going to use it now, stop the compute instance:

1. In the studio, in the left navigation area, select **Compute**.
1. In the top tabs, select **Compute instances**
1. Select the compute instance in the list.
1. On the top toolbar, select **Stop**.

### Delete all resources

[!INCLUDE [aml-delete-resource-group](../../includes/aml-delete-resource-group.md)]





<!-- nbend -->



## Clean up resources

If you plan to continue now to other tutorials, skip to [Next steps](#next-steps).

### Stop compute instance

If you're not going to use it now, stop the compute instance:

1. In the studio, in the left navigation area, select **Compute**.
1. In the top tabs, select **Compute instances**
1. Select the compute instance in the list.
1. On the top toolbar, select **Stop**.

### Delete all resources

[!INCLUDE [aml-delete-resource-group](../../includes/aml-delete-resource-group.md)]


## Next Steps

Learn about deploying a model 

> [!div class="nextstepaction"]
[Deploy a model](tutorial-deploy-model.md).

This tutorial used an online data file.  To learn more about other ways to access data, see [Tutorial: Upload, access and explore your data in Azure Machine Learning](tutorial-explore-data.md).

If you would like to learn more about different ways to train models in Azure Machine Learning, see [What is automated machine learning (AutoML)?](concept-automated-ml.md). Automated ML is a supplemental tool to reduce the amount of time a data scientist spends finding a model that works best with their data.

If you would like more examples similar to this tutorial, see [**Samples**](quickstart-create-resources.md#learn-from-sample-notebooks) section of studio. These same samples are available at our [GitHub examples page.](https://github.com/Azure/azureml-examples) The examples include complete Python Notebooks that you can run code and learn to train a model. You can modify and run  existing scripts from the samples, containing scenarios including classification, natural language processing, and anomaly detection. 

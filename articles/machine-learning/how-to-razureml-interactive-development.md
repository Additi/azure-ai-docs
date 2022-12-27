---
title: Use R interactively on Azure Machine Learning
titleSuffix: Azure Machine Learning
description: 'Learn how to use Azure Machine Learning for interactive R work'
ms.service: machine-learning
ms.date: 12/21/2022
ms.topic: how-to
author: wahalulu
ms.author: mavaisma
ms.reviewer: sgilley
ms.devlang: r
---

# Interactive R development

[!INCLUDE [dev v2](../../includes/machine-learning-dev-v2.md)]

This article will show you how to use R on a compute instance in Azure Machine Learning studio, running an R kernel in a Jupyter notebook.

Many R users also use RStudio, a popular IDE. You can install RStudio or Posit Workbench in a custom container on a compute instance.  However, there are limitations with the container in reading and writing to your Azure Machine Learning workspace.  

> [!IMPORTANT]
> The code shown in this article works on an Azure Machine Learning compute instance.  The compute instance has environment and configuration file necessary for the code to run successfully.  

## Prerequisites

- If you don't have an Azure subscription, create a free account before you begin. Try the [free or paid version of Azure Machine Learning](https://azure.microsoft.com/free/) today
- An [Azure Machine Learning workspace and a compute instance](quickstart-create-resources.md)
- A basic understand of using Jupyter notebooks in Azure Machine Learning studio.  For more information, see [Quickstart: Run Jupyter notebooks in studio](quickstart-run-notebooks.md)

## Run R in a notebook in studio

You'll use a notebook in your Azure Machine Learning workspace, on a compute instance.  

1. Sign in to [Azure Machine Learning studio](https://ml.azure.com)
1. Open your workspace it if isn't already open
1. On the left navigation, select **Notebooks**
1. Create a new notebook, named **RunR.ipynb**

    > [!TIP]
    > If you're not sure how to create and work with notebooks in studio, review [Quickstart: Run Jupyter notebooks in studio](quickstart-run-notebooks.md).

1. Select the notebook
1. On the notebook toolbar, make sure your compute instance is running.  If not, start it now.
1. On the notebook toolbar, switch the kernel to **R**.

    :::image type="content" source="media/how-to-razureml-interactive-development/r-kernel.png" alt-text="Screenshot: Switch the notebook kernel to use R." lightbox="media/how-to-razureml-interactive-development/r-kernel.png":::

Your notebook is now ready for you to run R commands.

## Access data

You can upload files to your workspace file storage and access them in R.  But for larger files, stored in Azure [_data assets_ or data from _datastores_](concept-data.md), you first need to install a few packages.

This section describes how to use Python and the `reticulate` package to load your data assets and datastores into R from an interactive session. You'll read tabular data as Pandas DataFrames using the [`azureml-fsspec`](/python/api/azureml-fsspec/?view=azure-ml-py&preserve-view=true) Python package and the `reticulate` R package. 

To install these packages:

1. Create a new file on the compute instance, called **setup.sh**.  
1. Copy this code into the file:

    :::code language="bash" source="~/azureml-examples-mavaisma-r-azureml/tutorials/using-r-with-azureml/01-setup-env-for-r-azureml/ci-setup-interactive-r.sh":::


1. Select  **Save and run script in terminal** to run the script

The install script performs the following steps:

* `pip` installs `azureml-fsspec` in the default conda environment for the compute instance
* Installs the R `reticulate` package if necessary (version must be 1.26 or greater)


### Read tabular data from registered _data assets_ or _datastores_

Use these steps to read a tabular file data asset [created in Azure Machine Learning](how-to-create-data-assets.md?tabs=cli#create-a-uri_file-data-asset) into an R `data.frame`:

1. Ensure you have the correct version of `reticulate`.  If the version is less than 1.26, try to run on a newer compute instance.

    ```r
    packageVersion("reticulate")
    ``` 

1. Load `reticulate` and set the conda environment where `azureml-fsspec` was installed

    [!Notebook-python[] (~/azureml-examples-mavaisma-azureml/tutorials/using-r-with-azureml/02-develop-in-interactive-r/work-with-data-assets.ipynb?name=reticulate)]


1. Find the URI path to the data file. In the code below, replace `<DATA_NAME>` and `<VERSION_NUMBER>` with the name and number of your data asset.

    > [!TIP]
    > In studio, select **Data** in the left navigation to find your data asset's name and version number.
    
    ```r
    py_code <- "from azure.identity import DefaultAzureCredential
    from azure.ai.ml import MLClient
    credential = DefaultAzureCredential()
    ml_client = MLClient.from_config(credential=credential)
        
    # get a handle to the data asset, then get the uri
    data_asset = ml_client.data.get(name='<DATA_NAME>', version='<VERSION_NUMBER>')
    data_uri = data_asset.path"
    
    py_run_string(py_code)
    # your uri is now available in the variable py$data_uri
    ```
    
1. Use Pandas read functions to read in the file(s) into the R environment

    [!Notebook-python[] (~/azureml-examples-mavaisma-azureml/tutorials/using-r-with-azureml/02-develop-in-interactive-r/work-with-data-assets.ipynb?name=read-uri)]

## Install R packages

There are many R packages pre-installed on the compute instance.

> [!TIP]
> When you create or use a different compute instance, you'll need to again install any packages you've installed.

When you want to install other packages, you'll need to explicitly state the location and dependencies.

For example, to install the `tsibble` package:

```r
install.packages("tsibble", 
                 dependencies = TRUE,
                 lib = "/home/azureuser")
```

> [!NOTE]
> Since you are installing packages within an R session running in a Jupyter notebook, `dependencies = TRUE` is required. Otherwise, dependent packages will not be automatically installed. The lib location is also required to install in the correct compute instance location.

## Load R libraries

Add `/home/azureuser` to the R library path. 

```r
.libPaths("/home/azureuser")
```

> [!TIP]
> You need to update the `.libPaths` in each interactive R script to access user installed libraries.  Add this code to the top of each interactive R script or notebook.  

Once the libPath is updated, load libraries as usual

```r
library('tsibble')
```

## Use R in the notebook

Other than the above issues, use R as you would in any other environment, such as your local workstation.  In your notebook or script, you can read and write to the path where the notebook/script is stored.

## Known limitations

@@Verify this: 
- Reading a file with `reticulate` only works with tabular data.
- From an interactive R session, you can only write to the workspace file system.
- From an interactive R session, you can't interact with MLflow (such as, log model or query registry).


## Next steps

* [How to train R models in Azure Machine Learning](how-to-razureml-train-model.md)
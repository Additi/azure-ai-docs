---
title: Connect to Azure storage services
titleSuffix: Azure Machine Learning
description: Learn how to create datastores and datasets to securely connect to Azure storage services during training 
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: nibaccam
author: nibaccam
ms.reviewer: nibaccam
ms.date: 09/22/2020
ms.custom: how-to, seodec18, devx-track-python

# Customer intent: As low code experience data scientist, I need to make my data in Azure storage available to my remote compute to train my machine learning models.
---

# Connect to data with the Azure Machine Learning studio

In this article, learn how to access and use your data with the [Azure Machine Learning studio](overview-what-is-machine-learning-studio.md). Connect to your data in Azure storage services with [Azure Machine Learning datastores](how-to-access-data.md), and then use that data in your machine learning workflows with [Azure Machine Learning datasets](how-to-create-register-datasets.md).

**Azure Machine Learning datastores** securely connect to your Azure storage service without putting your authentication credentials and your original data sources at risk. They store connection information, like your subscription ID and token authorization in the [Key Vault](https://azure.microsoft.com/services/key-vault/) associated with your workspace, so you can securely access your storage without having to hard code them in your scripts.

**Azure Machine Learning datasets** make it easier to access and work with your data. By creating a dataset, you create a reference to the data source location, along with a copy of its metadata. Because datasets are lazily-evaluated, and the data remains in its existing location, you

* Incur no extra storage cost
* Don't risk unintentionally changing your original data sources.
* Improve ML workflow performance speeds

To understand where datastores and datasets fit in Azure Machine Learning's overall data access workflow, see  the [Securely access data](concept-data.md#data-workflow) article.

For a code first experience, see the following articles to use the [Azure Machine Learning Python SDK](https://docs.microsoft.com/python/api/overview/azure/ml/?view=azure-ml-py) to:
* [Connect to Azure storage services with datastores](how-to-access-data.md). 
* [Create Azure Machine Learning datasets](how-to-create-register-datasets.md). 

## Prerequisites

You'll need:
- An Azure subscription. If you don't have an Azure subscription, create a free account before you begin. Try the [free or paid version of Azure Machine Learning](https://aka.ms/AMLFree).

- An Azure storage account with a [supported storage type](how-to-access-data.md#matrix).

- Access to [Azure Machine Learning studio](https://ml.azure.com/).

- An Azure Machine Learning workspace. [Create an Azure Machine Learning workspace](how-to-manage-workspace.md).

## Storage access and permissions

To ensure you securely connect to your Azure storage service, Azure Machine Learning  requires that you  have permission to access the corresponding data storage container. This access depends on the authentication credentials used to register the datastore.

### Virtual network

If your data storage account is in a **virtual network**, additional configuration steps are required to ensure Azure Machine Learning has access to your data. See [Network isolation & privacy](how-to-enable-virtual-network.md#machine-learning-studio) to ensure the appropriate configuration steps are applied when you create and register your datastore.  

### Access validation

**As part of the initial datastore create and register process**, Azure Machine Learning automatically validates that the underlying storage service exists and the user provided principal (username, service principal, or SAS token) has access to the specified storage.

**After datastore creation**, this validation is only performed for methods that require access to the underlying storage container, **not** each time datastore objects are retrieved. For example, validation happens if you want to download files from your datastore; but if you just want to change your default datastore, then validation does not happen.

To authenticate your access to the underlying storage service, you can provide either your account key, shared access signatures (SAS) tokens, or service principal according to the datastore type you want to create. The [storage type matrix](how-to-access-data.md#matrix) lists the supported authentication types that correspond to each datastore type.

You can find account key, SAS token, and service principal information on your [Azure portal](https://portal.azure.com).

* If you plan to use an account key or SAS token for authentication, select **Storage Accounts** on the left pane, and choose the storage account that you want to register.
  * The **Overview** page provides information such as the account name, container, and file share name.
      1. For account keys, go to **Access keys** on the **Settings** pane.
      1. For SAS tokens, go to **Shared access signatures** on the **Settings** pane.

* If you plan to use a service principal for authentication, go to your **App registrations** and select which app you want to use.
    * Its corresponding **Overview** page will contain required information like tenant ID and client ID.

> [!IMPORTANT]
> For security reasons, you may need to change your access keys for an Azure Storage account (account key or SAS token). When doing so, be sure to sync the new credentials with your workspace and the datastores connected to it. Learn how to sync your updated credentials with [these steps](how-to-change-storage-access-key.md).

### Permissions

For Azure blob container and Azure Data Lake Gen 2 storage, make sure your authentication credentials  has **Storage Blob Data Reader** access. Learn more about [Storage Blob Data Reader](https://docs.microsoft.com/azure/role-based-access-control/built-in-roles#storage-blob-data-reader). 

## Create datastores

You can create datastores from [these Azure storage solutions](#matrix). **For unsupported storage solutions**, and to save data egress cost during ML experiments, [move your data](how-to-access-data.md#move) to a supported Azure storage solution. [Learn more about datastores](how-to-access-data.md). 

Create a new datastore in a few steps with the Azure Machine Learning studio.

> [!IMPORTANT]
> If your data storage account is in a virtual network, additional configuration steps are required to ensure the studio has access to your data. See [Network isolation & privacy](how-to-enable-virtual-network.md#machine-learning-studio) to ensure the appropriate configuration steps are applied.

1. Sign in to [Azure Machine Learning studio](https://ml.azure.com/).
1. Select **Datastores** on the left pane under **Manage**.
1. Select **+ New datastore**.
1. Complete the form for a new datastore. The form intelligently updates itself based on your selections for Azure storage type and authentication type. See the [storage access and permissions section](#access-validation) to understand where to find the authentication credentials you need to populate this form.

The following example demonstrates what the form looks like when you create an **Azure blob datastore**:

![Form for a new datastore](media/how-to-connect-data-ui/new-datastore-form.png)

## Create datasets

After you create a datastore, create a dataset to interact with your data. Datasets package your data into a lazily evaluated consumable object for machine learning tasks, like training. [Learn more about datasets](how-to-create-register-datasets.md). 

With datasets you can,

* Keep a single copy of data in your storage.
* Share data and collaborate with other users.
* Seamlessly access data during model training without worrying about connection strings or data paths.
* Leverage open source libraries for data exploration like pandas.
* Create references to single or multiple files or public URLs with [FileDatasets](how-to-create-register-datasetsmd#filedataset).
     
    **OR**
* Represent your data in a tabular format with [TabularDatasets](how-to-create-register-datasets.md#tabulardataset).



The following steps and animation show how to create a dataset in [Azure Machine Learning studio](https://ml.azure.com).

> [!Note]
> Datasets created through Azure Machine Learning studio are automatically registered to the workspace.

![Create a dataset with the UI](./media/how-to-connect-data-ui/create-dataset-ui.gif)

To create a dataset in the studio:
1. Sign in at https://ml.azure.com.
1. Select **Datasets** in the **Assets** section of the left pane.
1. Select **Create Dataset** to choose the source of your dataset. This source can be local files, a datastore, public URLs or [Azure Open Datasets](../open-datasets/how-to-create-dataset-from-open-dataset.md).
1. Select **Tabular** or **File** for Dataset type.
1. Select **Next** to open the **Datastore and file selection** form. On this form you select where to keep your dataset after creation, as well as select what data files to use for your dataset.
    1. Enable skip validation if your data is in a virtual network. Learn more about [virtual network isolation and privacy](how-to-enable-virtual-network.md#machine-learning-studio).
1. Select **Next** to populate the **Settings and preview** and **Schema** forms; they are intelligently populated based on file type and you can further configure your dataset prior to creation on these forms. 
1. Select **Next** to review the **Confirm details** form. Check your selections and create an optional data profile for your dataset. Learn more about [data profiling](how-to-use-automated-ml-for-ml-models.md#profile).
1. Select **Create** to complete your dataset creation.

## Train with datasets

Use your datasets in your machine learning experiments for training ML models. [Learn more about how to train with datasets](how-to-train-with-datasets.md)

## Next steps

* [Train a model](how-to-train-ml-models.md)
* Use automated machine learning to [train with TabularDatasets](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/automated-machine-learning/forecasting-energy-demand/auto-ml-forecasting-energy-demand.ipynb).
* For more dataset training examples, see the [sample notebooks](https://github.com/Azure/MachineLearningNotebooks/tree/master/how-to-use-azureml/work-with-data/).

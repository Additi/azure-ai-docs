---
title: Text data labeling
titleSuffix: Azure Machine Learning
description: Use our data labeling tool to label text. Apply either a single label or multiple labels to a piece of text.
author: sdgilley
ms.author: sgilley
ms.service: machine-learning
ms.subservice: core
ms.topic: how-to
ms.date: 09/23/2021
ms.custom: data4ml
---

# Create a text data labeling project and export labels (preview)

Learn how to create and run data labeling projects to label text data in Azure Machine Learning.  Use machine-learning-assisted data labeling, or human-in-the-loop labeling, to aid with the task.

> [!IMPORTANT]
> Text labeling is currently in public preview.
> The preview version is provided without a service level agreement, and it's not recommended for production workloads. Certain features might not be supported or might have constrained capabilities.
> For more information, see [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## Data labeling capabilities

> [!Important]
> Text must be available in an Azure blob datastore. (If you do not have an existing datastore, you may upload files during project creation.)
 
Text data can be either ".txt" or ".csv" files.

* For ".txt" files, each file represents one item to be labeled.
* For ".csv" files, each row of the file is one item to be labeled.

Azure Machine Learning data labeling is a central place to create, manage, and monitor labeling projects:

- Coordinate data, labels, and team members to efficiently manage labeling tasks. 
- Tracks progress and maintains the queue of incomplete labeling tasks.
- Start and stop the project and control the labeling progress.
- Review the labeled data and export labeled as an Azure Machine Learning dataset.

## Prerequisites

[!INCLUDE [prerequisites](../../includes/machine-learning-data-labeling-prerequisites.md)]

## Create a data labeling project

[!INCLUDE [start](../../includes/machine-learning-data-labeling-start.md)]

* Select **Text** to create a text labeling project.

    :::image type="content" source="media/how-to-create-labeling-projects/text-labeling-creation-wizard.png" alt-text="Labeling project creation for text labeling":::

    * Choose **Text Classification Multi-class (Preview)** for projects when you want to apply only a *single label* from a set of labels to each piece of text.
    * Choose **Text Classification Multi-label (Preview)** for projects when you want to apply *one or more* labels from a set of labels to each piece of text.

* Select **Next** when you're ready to continue.

## Specify the data to label

If you already created a dataset that contains your data, select it from the **Select an existing dataset** drop-down list. Or, select **Create a dataset** to use an existing Azure datastore or to upload local files.

> [!NOTE]
> A project cannot contain more than 500,000 files.  If your dataset has more, only the first 500,000 files will be loaded.  

### Create a dataset from an Azure datastore

In many cases, it's fine to just upload local files. But [Azure Storage Explorer](https://azure.microsoft.com/features/storage-explorer/) provides a faster and more robust way to transfer a large amount of data. We recommend Storage Explorer as the default way to move files.

To create a dataset from data that you've already stored in Azure Blob storage:

1. Select **Create a dataset** > **From datastore**.
1. Assign a **Name** to your dataset.
1. Choose the **Dataset type**:
    * Select **Tabular** if you are using a .csv file, where each row is a response.
    * Select **File** if you are using separate .txt files for each response.
1. Select the datastore.
1. If your data is in a subfolder within your blob storage, choose **Browse** to select the path.
    * Append "/**" to the path to include all the files in subfolders of the selected path.
    * Append "**/*.*" to include all the data in the current container and its subfolders.
1. Provide a description for your dataset.
1. Select **Next**.
1. Confirm the details. Select **Back** to modify the settings or **Create** to create the dataset.

### Create a dataset from uploaded data

To directly upload your data:

1. Select **Create a dataset** > **From local files**.
1. Assign a **Name** to your dataset.
1. Choose the **Dataset type**.
    * Select **Tabular** if you are using a .csv file, where each row is a response.
    * Select **File** if you are using separate .txt files for each response.
1. *Optional:* Select **Advanced settings** to customize the datastore, container, and path to your data.
1. Select **Browse** to select the local files to upload.
1. Provide a description of your dataset.
1. Select **Next**.
1. Confirm the details. Select **Back** to modify the settings or **Create** to create the dataset.

The data gets uploaded to the default blob store ("workspaceblobstore") of your Machine Learning workspace.

## <a name="incremental-refresh"> </a> Configure incremental refresh

[!INCLUDE [refresh](../../includes/machine-learning-data-labeling-refresh.md)]

## Specify label classes

[!INCLUDE [classes](../../includes/machine-learning-data-labeling-classes.md)]

## Describe the data labeling task

[!INCLUDE [classes](../../includes/machine-learning-data-labeling-classes.md)]

>[!NOTE]
> Be sure to note that the labelers will be able to select the first 9 labels by using number keys 1-9.


## Initialize the data labeling project

[!INCLUDE [initialize](../../includes/machine-learning-data-labeling-initialize.md)]

## Run and monitor the project

[!INCLUDE [run](../../includes/machine-learning-data-labeling-initialize.md)]

### Dashboard

[!INCLUDE [dashboard](../../includes/machine-learning-data-labeling-dashboard.md)]

### Data tab

On the **Data** tab, you can see your dataset and review labeled data. If you see incorrectly labeled data, select it and choose **Reject**, which will remove the labels and put the data back into the unlabeled queue.

### Details tab

View details of your project.  In this tab you can:

* View project details and input datasets
* Enable incremental refresh
* View details of the storage container used to store labeled outputs in your project
* Add labels to your project
* Edit instructions you give to your labels

### Access for labelers

[!INCLUDE [access](../../includes/machine-learning-data-labeling-access.md)]

## Add new label class to a project

[!INCLUDE [add-label](../../includes/machine-learning-data-labeling-add-label.md)]

## Export the labels
 
Use the **Export** button on the **Project details** page of your labeling project. You can export the label data for Machine Learning experimentation at any time.

You can export:

    * A CSV file. The CSV file is created in the default blob store of the Azure Machine Learning workspace in a folder within *Labeling/export/csv*. 
    * An [Azure Machine Learning dataset with labels](how-to-use-labeled-dataset.md). 

Access exported Azure Machine Learning datasets in the **Datasets** section of Machine Learning. The dataset details page also provides sample code to access your labels from Python.

![Exported dataset](./media/how-to-create-labeling-projects/exported-dataset.png)

## Troubleshooting

[!INCLUDE [troubleshooting](../../includes/machine-learning-data-labeling-troubleshooting.md)]


## Next steps

* [Tutorial: Create your first image classification labeling project](tutorial-labeling.md).
* Label images for [image classification or object detection](how-to-label-data.md)
* Learn more about [Azure Machine Learning and Machine Learning Studio (classic)](./overview-what-is-machine-learning-studio.md#ml-studio-classic-vs-azure-machine-learning-studio)

---
title: Transform data
titleSuffix: Azure Machine Learning
description: Learn how to transform data in Azure Machine Learning designer to create your own datasets.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: how-to

author: peterclu
ms.author: peterlu
ms.date: 05/04/2020
---

# Transform data in Azure Machine Learning designer (preview)
[!INCLUDE [applies-to-skus](../../includes/aml-applies-to-enterprise-sku.md)]

In this article, you will learn how to transform and save datasets in the designer so that you can prepare your own data for machine learning. Use the provided [Adult Census Income Binary Classification dataset](sample-designer-datasets.md) to prepare two datasets: one dataset that includes adult census information from only the United States and another dataset that only includes census information from non-US adults.

In this article, you learn how to:

1. Transform a dataset to prepare it for training.
1. Export the resulting datasets to a datastore.

This how-to is a prerequisite for the [how to retrain designer models](how-to-retrain-designer.md) article. In that article, you will learn how to use the transformed datasets to train multiple models.

## Transform a dataset

In this section, you learn how to import the sample dataset and split the data into US and non-US datasets using the **Split Data** module. For this how to, use **Adult Census Income Binary classification** as your starting point. For more information on how to import your own data into the designer, see [how to import data](how-to-designer-import-data.md).

### Import data

Use the following steps to import the sample dataset.

1. Sign in to <a href="https://ml.azure.com?tabs=jre" target="_blank">ml.azure.com</a>, and select the workspace you want to work with.

1. Go to the designer and create a new pipeline by selecting **Easy-to-use prebuilt modules**.

1. Select a default compute target to run the pipeline.

1. To the left of the pipeline canvas is a palette of datasets and modules. Select **Datasets**, and then view the **Samples** section.

1. Drag and drop the **Adult Census Income Binary classification** dataset onto the canvas.

1. Select the **Adult Census Income** dataset module.

1. In the details pane that appears to the right of the canvas, select **Outputs**. Then select the visualize icon ![visualize icon](media/how-to-designer-transform-data/visualize-icon.png).

1. Use the data preview window to explore the dataset. Take note of the "native-country" column values.

### Split the data

In this section, you use the [Split Data module](algorithm-module-reference/split-data.md) to identify rows that contain "United-States" in the  "native-country" column. 

1. In the module palette to the left of the canvas, expand the **Data Transformation** section and find the **Split Data** module.

1. Drag the **Split Data** module onto the canvas, and drop the module below the dataset module.

1. Connect the **Adult Census Income Binary classification** dataset to the **Split Data** module.

1. Select the **Split Data** module.

1. In the module details pane to the right of the canvas, set **Splitting mode** to **Regular Expression**.

1. Enter the **Regular Expression**: `\"native-country" United-States`.

    The **Regular expression** mode tests a single column for a value. For more information on the Split Data module, see the related [algorithm reference page](algorithm-module-reference/split-data.md).

Your pipeline should look like this:

![Screenshot showing how to configure the pipeline and the Split Data module](media/how-to-designer-transform-data/split-data.png).


## Save the datasets

Now that your pipeline is set up to split the data, you need to specify where to persist the datasets to access them later. For this example, use the **Export Data** module to save your dataset to a datastore.

1. In the module palette to the left of the canvas, expand the **Data Input and Output** section and find the **Export Data** module.

1. Drag and drop two **Export Data** modules below the **Split Data** module.

1. Connect each output port of the **Split Data** module to a different **Export Data** module.

    Your pipeline should look something like this:

    ![Screenshot showing how to connect the Export Data modules](media/how-to-designer-transform-data/export-data-pipeline.png).

1. Select the **Export Data** module connected to the *left*-most port of the **Split Data** module.

    The order of the output ports matter. The first output port (left) contains the rows where the Split Data regular expression is true. In this case, the first port contains rows for the US-based income, and the second port contains rows for the non-US based income.

1. In the module details pane to the right of the canvas, set the following options:
    
    **Datastore type**: Azure Blob Storage

    **Datastore**: Select an existing datastore or select "New datastore" to create one now.

    **Path**: `/data/us-income`

    **File format**: csv

    > [!NOTE]
    > This article assumes that you have access to a datastore registered to the current Azure Machine Learning workspace. For instructions on how to setup a datastore, see [Connect to Azure storage services](how-to-access-data.md#azure-machine-learning-studio).

    If you don't have a datastore, you can create one now. For example purposes, this article will save the datasets to the default blob storage account associated with the workspace. It will save the datasets into a new folder called `data`.

1.  Select the **Export Data** module connected to the *right*-most port of the **Split Data** module.

1. In the module details pane to the right of the canvas, set the following options:
    
    **Datastore type**: Azure Blob Storage

    **Datastore**: Select an existing datastore or select "New datastore" to create one now.

    **Path**: `/data/non-us-income`

    **File format**: csv

1. Confirm the **Export Data** module connected to the left port of the **Split Data** has the **Path** `/data/us-income`.

1. Confirm the **Export Data** module connected to the right port has the **Path** `/data/non-us-income`.

    Your pipeline and settings should look like this:
    
    ![Screenshot showing how to configure the Export Data modules](media/how-to-designer-transform-data/us-income-export-data.png).

1. At the top of the canvas, select **Submit** to submit the run.

1. In the **Set up pipeline run** dialog, select **Create new**.

1. Provide a descriptive experiment name like "split-census-data".

1. Select **Submit**.

### View results

After the pipeline finishes running, you can view your results by navigating to your blob storage in the Azure portal. You can also view the intermediary results for the **Split Data** module to confirm that your data split correctly.

1. Select the **Split Data** module.

1. In the module details pane to the right of the canvas, select **Outputs + logs**. 

1. Select the visualize icon ![visualize icon](media/how-to-designer-transform-data/visualize-icon.png) next to **Results dataset1**. 

1. Verify that the "native-country" column only contains the value "United-States".

1. Select the visualize icon ![visualize icon](media/how-to-designer-transform-data/visualize-icon.png) next to **Results dataset2**. 

1. Verify that the "native-country" column does not contain the value "United-States".

## Clean up resources

Skip this section if you want to continue on with part 2 of this how to, [Retrain models with Azure Machine Learning designer](how-to-retrain-designer.md).

[!INCLUDE [aml-ui-cleanup](../../includes/aml-ui-cleanup.md)]

## Next steps

In this article, you learned how to transform a dataset and save it to a registered datastore. Continue on with [Retrain models with Azure Machine Learning designer](how-to-retrain-designer.md) to use your transformed datasets and pipeline parameters to train machine learning models.
---
title: Create and explore labeled datasets
titleSuffix: Azure Machine Learning
description: Learn how to export data labels from your Azure Machine Learning labeling projects and use them for machine learning tasks.  
author: nibaccam
ms.author: nibaccam
ms.service: machine-learning
ms.topic: how-to
ms.date: 01/21/2020

# Customer intent: As an experienced Python developer, I need to export my data labels and use them for machine learning tasks.
---

# Export and utilize Azure Machine Learning dataset with labels

In this article, you'll learn how to export the data labels from a labeling project
and load them into popular formats such as, a pandas dataframe for data exploration.

## What are datasets with labels 

Azure Machine Learning datasets with labels are [TabularDatasets](how-to-create-register-datasets.md#dtaset-types) with a label property, we will refer to them as labeled datasets. These specific types of TabularDatasets are only created as an output of Azure Machine Learning data labeling projects. Create a data labeling project with [these steps](how-to-create-labeling-projects.md).

## Prerequisites

* An Azure subscription. If you don’t have an Azure subscription, create a [free account](https://aka.ms/AMLFree) before you begin.
* The [Azure Machine Learning SDK for Python](https://docs.microsoft.com/python/api/overview/azure/ml/intro?view=azure-ml-py), or access to [Azure Machine Learning studio](https://ml.azure.com/).
    * Install the [azure-contrib-dataset](https://docs.microsoft.com/python/api/azureml-contrib-dataset/?view=azure-ml-py) package
* A Machine Learning workspace. See [Create an Azure Machine Learning workspace](how-to-manage-workspace.md).
* Access to an Azure Machine Learning data labeling project

## Export data labels 

When you complete a data labeling project, you can then export the label data from a labeling project. Doing so, allows you to capture both the reference to the data and its labels, and export them in [COCO format](http://cocodataset.org/#format-data) or as an Azure Machine Learning dataset. 

### COCO 

Use the **Export** button on the **Project details** page of your labeling project. The COCO file is created in the default blob store of the Azure Machine Learning workspace in a folder within *export/coco*. 

### Azure Machine Learning studio

You can access the exported Azure Machine Learning dataset in the **Datasets** section of your Azure Machine Learning studio. The dataset **Details** page also provides sample code to access your labels from Python.

![Exported dataset](./media/how-to-create-labeling-projects/exported-dataset.png)

## Explore labeled datasets

Load your labeled datasets into a pandas dataframe or Torchvision dataset to leverage popular open-source libraries for data exploration, as well as PyTorch provided libraries for image transformation and training.

### Pandas dataframe

You can load labeled datasets into a pandas dataframe with the [`to_pandas_dataframe()`](https://docs.microsoft.com/python/api/azureml-core/azureml.data.tabulardataset?view=azure-ml-py#to-pandas-dataframe-on-error--null---out-of-range-datetime--null--) method from the `azureml-contrib-dataset` class. We offer the following file handling options for file streams when converting to a pandas dataframe. 

>[!NOTE]
>The azureml.contrib namespace changes frequently, as we work to improve the service. As such, anything in this namespace should be considered as a preview, and not fully supported by Microsoft.

* Download: Download your data files to a local path.
* Mount: Mount your data files to a mount point. Mount only works for Linux-based compute, including Azure Machine Learning notebook VM and Azure Machine Learning Compute.


```Python
animal_pd = animal_labels.to_pandas_dataframe(file_handling_option=FileHandlingOption.DOWNLOAD, target_path='./download/', overwrite_download=True)

import matplotlib.pyplot as plt
import matplotlib.image as mpimg

#read images from downloaded path
img = mpimg.imread(animal_pd.loc[0,'image_url'])
imgplot = plt.imshow(img)
```

### Torchvision datasets

You can load labeled datasets into Torchvision dataset with the [to_torchvision()](https://docs.microsoft.com/python/api/azureml-contrib-dataset/azureml.contrib.dataset.tabulardataset?view=azure-ml-py#to-torchvision--) method also from the `azureml-contrib-dataset` class.

```python
from torchvision.transforms import functional as F

# load animal_labels dataset into torchvision dataset
pytorch_dataset = animal_labels.to_torchvision()
img = pytorch_dataset[0][0]
print(type(img))

# use methods from torchvision to transform the img into grayscale
pil_image = F.to_pil_image(img)
gray_image = F.to_grayscale(pil_image, num_output_channels=3)

imgplot = plt.imshow(gray_image)
```

## Next steps

* [Create a data labeling project](how-to-create-labeling-projects.md).
* [How to train with datasets](how-to-train-with-datasets.md).

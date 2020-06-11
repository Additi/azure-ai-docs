---
title: Configure cross-validation and data splits in automated machine learning experiments
titleSuffix: Azure Machine Learning
description: Learn how to configure cross-validation and dataset splits for automated machine learning experiments
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: how-to
ms.author: cesardl
author: CESARDELATORRE
ms.reviewer: nibaccam
ms.date: 06/16/2020

---

# Configure data splits and cross-validation in automated machine learning

In this article, you learn how to configure training and validation data splits and cross-validation for your automated machine learning, AutoML, experiments.

This article assumes you have an understanding of cross-validation and train/validation data splits as ML concepts. For a high-level explanation, take a look at these articles, 

 * [About Train, Validation and Test Sets in Machine Learning](https://towardsdatascience.com/train-validation-and-test-sets-72cb40cba9e7)
 * [Understanding Cross Validation](https://towardsdatascience.com/understanding-cross-validation-419dbd47e9bd)

In Azure Machine Learning, when you use AutoML to build multiple ML models, each child run needs to validate the related model by calculating the quality metrics for that model, such as accuracy  or AUC weighted. These metrics are calculated by comparing the predictions made with each model with real labels from past observations in the validation data. 

Automated machine learning experiments perform model validation automatically. However, the following sections describe how you can further customize these validation settings for your AutoML experiments. 

## Prerequisites

For this article you need,

* An Azure Machine Learning workspace. To create the workspace, see [Create an Azure Machine Learning workspace](how-to-manage-workspace.md).

* A basic familiarity with setting up an automated machine learning experiment with the Azure Machine Learning SDK.
    * For a code-first experience, follow the [tutorial](tutorial-auto-train-models.md) or [how-to](how-to-configure-auto-train.md) to see the basic automated machine learning experiment design patterns.

    * For a low-code or no-code experience, see [Create your automated machine learning experiments in Azure Machine Learning studio](how-to-use-automated-ml-for-ml-models.md).


    
## Default cross-validation and data splits

For automated machine learning experiments, you use an [AutoMLConfig](https://docs.microsoft.com/python/api/azureml-train-automl-client/azureml.train.automl.automlconfig.automlconfig?view=azure-ml-py) object to define your experiment and training settings. In the following code snippet notice that only the basic parameters are defined, that is no parameters for `n_cross_validation` or `validation_ data` are included.

```python
data = "https://automlsamplenotebookdata.blob.core.windows.net/automl-sample-notebook-data/creditcard.csv"

dataset = Dataset.Tabular.from_delimited_files(data)

automl_config = AutoMLConfig(compute_target = aml_remote_compute,
                             task = 'classification',
                             primary_metric = 'AUC_weighted',
                             training_data = dataset,
                             label_column_name = 'Class'
                            )
```

If you do not explicitly specify either a `validation_data` or `n_cross_validation` parameter, AutoML will apply default techniques depending on the number of rows in the single dataset `training_data` provided:

 * **If the training data set is larger than 20,000 rows**, a **train/validation data split** is applied. The default is to take 10% of the initial training data set as the validation set. In turn, that validation set is used for metrics calculation.

* **If the training dataset is smaller than 20,000 rows**, a **cross-validation** approach is applied. The default number of folds depends on the number of rows.
    * **If the dataset is less than 1,000 rows**, 10 folds are used. 
    * **If the rows are between 1,000 and 20,000**, then three folds are used.

## Provide validation data

In this case, in addition to the training data, you explicitly provide a validation dataset using the `validation_data` parameter.

With validation data provided in the form of an [Azure Machine Learning dataset](how-to-create-register-dataset.md) or pandas dataframe, `training_data` defines what data to use for training purposes, and `validation_data` specifies what data to use for model validation and metrics calculation. 

The following code example explicitly defines which portion of the provided data is to be used for training and validation.

```python
data = "https://automlsamplenotebookdata.blob.core.windows.net/automl-sample-notebook-data/creditcard.csv"

dataset = Dataset.Tabular.from_delimited_files(data)

training_data, validation_data = dataset.random_split(percentage=0.8, seed=1)

automl_config = AutoMLConfig(compute_target = aml_remote_compute,
                             task = 'classification',
                             primary_metric = 'AUC_weighted',
                             training_data = training_data,
                             validation_data = validation_data,
                             label_column_name = 'Class'
                            )
```

## Provide validation set size

In addition to assigning a single dataset with the `training_data` parameter, you can provide the `validation_size` parameter, which is the fraction of the data to hold out for validation when the `validation_data` parameter is **not** specified. That is,  the validation set will be split by AutoML from the initial `training_data` provided. This value should be between 0.0 and 1.0 non-inclusive (that is, 0.2 means 20% of the data hold out for validation data).

See the following code example

```python
data = "https://automlsamplenotebookdata.blob.core.windows.net/automl-sample-notebook-data/creditcard.csv"

dataset = Dataset.Tabular.from_delimited_files(data)

automl_config = AutoMLConfig(compute_target = aml_remote_compute,
                             task = 'classification',
                             primary_metric = 'AUC_weighted',
                             training_data = dataset,
                             validation_size = 0.2,
                             label_column_name = 'Class'
                            )
```

## Set the number of cross-validations

To perform cross-validation, include the `n_cross_validations` parameter and set it to a value. This parameter sets how many cross validations to perform, based on the same number of folds.

In the following code, five folds for cross- validation are defined. Hence, five different trainings, each training using 4/5 of the data, and each validation using 1/5 of the data with a different holdout fold each time.

As a result, metrics are calculated with the average of the 5 validation metrics.

```python
data = "https://automlsamplenotebookdata.blob.core.windows.net/automl-sample-notebook-data/creditcard.csv"

dataset = Dataset.Tabular.from_delimited_files(data)

automl_config = AutoMLConfig(compute_target = aml_remote_compute,
                             task = 'classification',
                             primary_metric = 'AUC_weighted',
                             training_data = dataset,
                             n_cross_validations = 5
                             label_column_name = 'Class'
                            )
```

## Specify custom cross-validation data folds

You can also provide your own cross-validation (CV) data folds. This is considered a more advanced scenario because you are specifying which columns to split and use for validation.  Include custom CV split columns in your training data, and specify which columns by populating the column names in the `cv_split_column_names` parameter. Each column represents one cross- validation split, and is filled with integer values 1 or 0 --where 1 indicates the row should be used for training and 0 indicates the row should be used for validation.

The following code snippet contains bank marketing data with two CV split columns 'cv1' and 'cv2'.

```python
data = "https://automlsamplenotebookdata.blob.core.windows.net/automl-sample-notebook-data/bankmarketing_with_cv.csv"

dataset = Dataset.Tabular.from_delimited_files(data)

automl_config = AutoMLConfig(compute_target = aml_remote_compute,
                             task = 'classification',
                             primary_metric = 'AUC_weighted',
                             training_data = dataset,
                             label_column_name = 'y',
                             cv_split_column_names = ['cv1', 'cv2']
                            )
```

> [!NOTE]
> To use `cv_split_column_names` with `training_data` and `label_column_name`, please upgrade your Azure Machine Learning Python SDK version 1.6.0 or later. For previous SDK versions, please refer to using `cv_splits_indices`, but note that it is used with `X` and `y` dataset input only. 

## Next steps

* Prevent class imbalance and overfitting.
* [Tutorial: Use automated machine learning to predict taxi fares - Split data section](tutorial-auto-train-models.md#split-the-data-into-train-and-test-sets).
---
title: Iris Tutorial for Machine Learning Server | Microsoft Docs
description: This full-length tutorial shows how to use Azure Machine Learning end-to-end. This is part 1 on data preparation.
services: machine-learning
author: hning86
ms.author: haining
manager: mwinkle
ms.reviewer: garyericson, jasonwhowell, mldocs
ms.service: machine-learning
ms.workload: data-services
ms.custom: mvc
ms.topic: hero-article
ms.date: 09/06/2017
---

# Classifying Iris Part 1: Prepare Data

In this tutorial, we show you the basics of Azure ML preview features by creating a data preparation package, building a model and operationalizing it as a real-time web service. To make things simple, we use the timeless [Iris flower dataset](https://en.wikipedia.org/wiki/Iris_flower_data_set). The instructions and screenshots are created for Windows, but they are similar, if not identical, for macOS.

This is part 1 of a 3-part tutorial, convering project setup and data preparation.

## Step 1. Launch Azure ML Workbench
Follow the [installation guide](Installation.md) to install Azure ML Workbench desktop application, which also includes command-line interface (CLI). Launch the Azure ML Workbench desktop app and log in if needed.

## Step 2. Create a new project
Click on _File_ --> _New Project_ (or click on the "+" sign in the project list pane). You can also create a new Workspace first from this drop down menu.

![new ws](media/tutorial-classifying-iris/new_ws.png)

Fill in the project name (this tutorial assumes you use `myIris`). Choose the directory the project is going to be created in (this tutorial assumes you choose `C:\Temp`). Enter an optional description. Choose a Workspace (this tutorial uses `IrisGarden`). And then select the `Classifying Iris` template from the project template list. 

![New Project](media/tutorial-classifying-iris/new_project.png)
>Optionally, you can fill in the Git repo field with an existing empty (with no master branch) Git repo on VSTS. Doing so allows you to enable roaming and sharing scenarios later. For more information, please reference the [Using Git repo](UsingGit.md) article and the [Roaming and Sharing](collab.md) article.

Click on _Create_ button to create the project. The project is now created and opened.

## Step 3. Create a Data Preparation package
Open the `iris.csv` file from the File View, observe that the file is a simple table with 5 columns and 150 rows. It has four numerical feature columns and a string target column. Also notice it doesn't have column headers.

![iris.csv](media/tutorial-classifying-iris/show_iris_csv.png)

>Note it is not recommended to include data files in your project folder, particularly when the file size is large. We include `iris.csv` in this template for demonstration purposes because it is tiny. For more information, please reference the [How to Deal with Large Data Files](PersistChanges.md) article.

Under Data Explorer view, click on "+" to add a new data source. This launches the _Add Data Source_ wizard. 

![data view](media/tutorial-classifying-iris/data_view.png)

Select the _File(s)/Directory_ option, and choose the `iris.csv` local file. Accept all the default settings for each screen and finally click on _Finish_. 

![select iris](media/tutorial-classifying-iris/select_iris_csv.png)

>Make sure you select the `iris.csv` file from within the current project directory for this exercise, otherwise latter steps may fail. 

This creates an `iris-1.dsource` file (because the sample project already comes with an `iris.dsource` file) and opens it in the _Data_ view. A series of column headers, from `Column1` to `Column5`, are automatically added to this dataset. Also notice the last row of the dataset is empty, probably because of an extra line break in the csv file.

![iris data view](media/tutorial-classifying-iris/iris_data_view.png)

Now click on the _Metrics_ button. Observe the histograms and a complete set of statistics that are calculated for you for each column. You can also switch over to the _Data_ view to see the data itself. 

![iris data view](media/tutorial-classifying-iris/iris_metrics_view.png)

Now click on the _Prepare_ button next to the _Metrics_ button, and this creates a new file named `iris-1.dprep`. Again, this is because the sample project already comes with an `iris.dprep` file. It opens in Data Prep editor. Now let's do some simple data wrangling.

Rename the column names by clicking on each column header and make the text editable. Enter `Sepal Length`, `Sepal Width`, `Petal Length`, `Petal Width`, and `Species` for the five columns respectively.

![rename columns](media/tutorial-classifying-iris/rename_column.png)

Select the `Species` column, and right-click on it and choose _Value Counts_. 

![value count](media/tutorial-classifying-iris/value_count.png)

This creates a histogram with four bars. Notice our target column has three distinct values, `Iris_virginica`, `Iris_versicolor`, `Iris-setosa`. And there is also one row with a `(null)` value. Let's get rid of this row by selecting the bar representing the null value, and click on the "-" filter button to remove it. 

![value count](media/tutorial-classifying-iris/filter_out.png)

Also notice as you are working on column renaming and filtering out the null value row, each action is recorded as a dataprep step in the _Steps_ pane. You can edit them (to adjust their settings), reorder them, or even remove them.

![steps](media/tutorial-classifying-iris/steps.png)

## Step 4. Generate Python/PySpark Code to Invoke Data Prep Package

Now close the DataPrep editor. Don't worry, it is auto-saved. Right click on the `iris-1.dprep` file, and choose _Generate Data Access Code File_. 

![generate code](media/tutorial-classifying-iris/generate_code.png)

This creates an `iris-1.py` file with following two lines of code prepopulated (along with some comments):

```python
# This code snippet will load the referenced package and return a DataFrame.
# If the code is run in a PySpark environment, the code will return a
# Spark DataFrame. If not, the code will return a Pandas DataFrame.

from azureml.dataprep.package import run
df = run('iris.dprep', dataflow_idx=0)
```
This code snippet shows how you can invoke the data wrangling logic you have created as a Data Prep package. Depending on the context in which this code runs, `df` can be a Python Pandas DataFrame if executed in Python runtime, or a Spark DataFrame if executed in a Spark context. For more information on how to use DataPrep in Azure ML Workbench, reference the [Getting Started with Data Preparation](DataPrep_GettingStarted.md) guide.

Now we have a data prep package that can be invoked from Python code, we are ready to move on to the next to build a machine learning model.

## Next Steps
- Part 1: Project setup and data preparation
- [Part 2: Model building](tutorial-classifying-iris-part-2.md)
- [Part 3: Model deployment](tutorial-classifying-iris-part-3.md)
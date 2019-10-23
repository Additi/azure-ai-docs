---
title: What is Azure Machine Learning
description: Overview of Azure Machine Learning - An integrated, end-to-end data science solution for professional data scientists to develop, experiment, and deploy advanced analytics applications at cloud scale.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: overview
author: j-martens
ms.author: jmartens
ms.date: 10/21/2019
ms.custom: seodec18
---

# What is Azure Machine Learning?

Azure Machine Learning is a cloud service that you use to train, deploy, automate, and manage machine learning models, all at the broad scale that the cloud provides.

> [!VIDEO https://channel9.msdn.com/Events/Connect/Microsoft-Connect--2018/D240/player]

> [!Tip]
> **Free trial!**  If you don’t have an Azure subscription, create a free account before you begin. Try the [free or paid version of Azure Machine Learning](https://aka.ms/AMLFree) today. You get credits to spend on Azure services. After they're used up, you can keep the account and use [free Azure services](https://azure.microsoft.com/free/). Your credit card is never charged unless you explicitly change your settings and ask to be charged.


## What is machine learning?

Machine learning is a data science technique that allows computers to use existing data to forecast future behaviors, outcomes, and trends. By using machine learning, computers learn without being explicitly programmed.

Forecasts or predictions from machine learning can make apps and devices smarter. For example, when you shop online, machine learning helps recommend other products you might want based on what you've bought. Or when your credit card is swiped, machine learning compares the transaction to a database of transactions and helps detect fraud. And when your robot vacuum cleaner vacuums a room, machine learning helps it decide whether the job is done.

## About Azure Machine Learning

Azure Machine Learning provides a cloud-based environment you can use to train, deploy, automate, manage, and track ML models. Start training on your local machine and then scale out to the cloud.


The service fully supports open-source technologies such as PyTorch, TensorFlow, and scikit-learn and can be used for any kind of machine learning, from classical ml to deep learning, supervised and unsupervised learning.

Train, test, and deploy your models with rich tools such as:
+ The [Azure Machine Learning designer](ui-tutorial-automobile-price-train-score.md) (preview): drag-n-drop modules to build your experiments and then deploy models
+ Jupyter notebooks: use our [example notebooks](https://aka.ms/aml-notebooks) or create your own notebooks to leverage our Python SDK samples for your machine learning. 
+ R scripts or notebooks in which you use the [R SDK](https://azure.github.io/azureml-sdk-for-r/reference/index.html) to write your own code, or use the R modules in the designer.
+ [Visual Studio Code extension](how-to-vscode-tools.md)
+ [Machine learning CLI](reference-azure-machine-learning-cli.md)

## What can I do with the service?

Use the <a href="https://docs.microsoft.com/python/api/overview/azure/ml/intro?view=azure-ml-py" target="_blank">Azure Machine Learning Python SDK</a> with open-source Python packages, or use the [designer to build and train highly accurate machine learning and deep-learning models yourself in an Azure Machine Learning Workspace.

Azure Machine Learning provides all the tools you need for your machine learning workflow such as Azure Machine Learning <a href="https://docs.microsoft.com/python/api/overview/azure/ml/intro?view=azure-ml-py" target="_blank">SDK for Python</a> and <a href="https://azure.github.io/azureml-sdk-for-r/reference/index.html on" target="_blank">SDK for R</a>. The service also interoperates with popular open-source tools, such as <a href="https://scikit-learn.org/stable/" target="_blank">Scikit-learn</a>, <a href="https://www.tensorflow.org" target="_blank">Tensorflow</a>, and <a href="https://pytorch.org" target="_blank">PyTorch</a>.  You can even use [MLflow to track metrics and deploy models](how-to-use-mlflow.md) or Kubeflow to [build end-to-end workflow pipelines](https://www.kubeflow.org/docs/azure/).

Whether you write code or use the [designer](ui-tutorial-automobile-price-train-score.md), you can build, train and track highly accurate machine learning and deep-learning models in an Azure Machine Learning Workspace.

### Code-first experience

Start training on your local machine using the <a href="https://docs.microsoft.com/python/api/overview/azure/ml/intro?view=azure-ml-py" target="_blank">Azure Machine Learning Python SDK</a> and then scale out to the cloud. With many available [compute targets](how-to-set-up-training-targets.md), like Azure Machine Learning Compute and [Azure Databricks](/azure/azure-databricks/what-is-azure-databricks), and with [advanced hyperparameter tuning services](how-to-tune-hyperparameters.md), you can build better models faster by using the power of the cloud.

You can also [automate model training and tuning](tutorial-auto-train-models.md) using the SDK.

### UI-based, low-code experience

For code-free training, try:

+ **Azure Machine Learning designer (preview)**

  Use the designer to prep data, train, test, deploy, manage, and track machine learning models without writing any code. There is no programming required, you visually connect datasets and modules to construct your model.   Try out the [designer tutorial](ui-tutorial-automobile-price-train-score.md).

  Learn more in [the Azure Machine Learning designer overview article](ui-concept-visual-interface.md). 

  ![Azure Machine Learning designer](media/overview-what-is-azure-ml/designer.png)

+ **Automated machine learning UI**

  Learn how to create [automated ML experiments](tutorial-first-experiment-automated-ml.md) in the easy-to-use interface. 

  [![Azure Machine Learning studio navigation pane](media/overview-what-is-azure-ml/azure-machine-learning-automated-ml-ui.jpg)](media/overview-what-is-azure-ml/azure-machine-learning-automated-ml-ui.jpg)


### Deploy & operationalize (MLOps)

When you have the right model, you can easily use it in a web service, on an IoT device, or from Power BI. For more information, see the article on [how to deploy and where](how-to-deploy-and-where.md).

Then you can manage your deployed models by using the [Azure Machine Learning SDK for Python](https://aka.ms/aml-sdk), [Azure Machine Learning studio](https://ml.azure.com), or the [machine learning CLI](reference-azure-machine-learning-cli.md).

These models can be consumed and return predictions in [real time](how-to-consume-web-service.md) or [asynchronously](how-to-run-batch-predictions.md) on large quantities of data.

And with advanced [machine learning pipelines](concept-ml-pipelines.md), you can collaborate on each step from data preparation, model training and evaluation, through deployment. Pipelines allow you to:

* Automate the end-to-end machine learning process in the cloud
* Reuse components and only re-run steps when needed
* Use different compute resources in each step
* Run batch scoring tasks

If you want to use scripts to automate your machine learning workflow, the [machine learning CLI](reference-azure-machine-learning-cli.md) provides command-line tools that perform common tasks, such as submitting a training run or deploying a model.

To get started using Azure Machine Learning, see [Next steps](#next-steps).

## <a name="sku"></a>Basic & Enterprise editions

Azure Machine Learning offers two editions tailored for your machine learning needs:
+ Basic (generally available)
+ Enterprise (preview)

These editions determine which machine learning tools are available to developers and data scientists from their workspace.   

Basic workspaces allow you to continue using Azure Machine Learning and pay for only the Azure resources consumed during the machine learning process. Enterprise edition workspaces will be charged only for their Azure consumption while the edition is in preview. Learn more about what's available in the Azure Machine Learning [edition overview & pricing page](https://azure.microsoft.com/pricing/details/machine-learning/). 

You assign the edition whenever you create a workspace. And, pre-existing workspaces have been converted to the Basic edition for you. Basic edition includes all features that were already generally available as of October 2019. Any experiments in those workspaces that were built using Enterprise edition features will continue to be available to you in read-only until you upgrade to Enterprise. Learn how to [upgrade a Basic workspace to Enterprise edition](how-to-manage-workspace.md#upgrade). 

Customers are responsible for costs incurred on compute and other Azure resources during this time.

## Next steps

- Create your first experiment with your preferred method:
  + [Use Python notebooks to train & deploy ML models](tutorial-1st-experiment-sdk-setup.md)
  + [Use R Markdown to train & deploy ML models]( tutorial-1st-r-experiment.md) 
  + [Use automated machine learning to train & deploy ML models](ui-tutorial-automobile-price-train-score.md) 
  + [Use the designer's drag & drop capabilities to train & deploy](tutorial-first-experiment-automated-ml.md) 
  + [Use the machine learning CLI to train and deploy a model](tutorial-train-deploy-model-cli.md)

- Learn about [machine learning pipelines](/azure/machine-learning/service/concept-ml-pipelines) to build, optimize, and manage your machine learning scenarios.

- Read the in-depth [Azure Machine Learning architecture and concepts](concept-azure-machine-learning-architecture.md) article.

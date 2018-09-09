---
title: Build machine learning pipelines with Azure Machine Learning service
description: Learn about machine learning pipelines for your workflow in Azure Machine Learning service. 
services: machine-learning
ms.service: machine-learning
ms.component: core
ms.topic: conceptual
ms.author: sanpil
author: sanpil
ms.date: 09/24/2018
---

# Pipelines and Azure Machine Learning

Machine learning (ML) pipelines are used by data scientists to build, optimize, and manage their machine learning workflows. A typical pipeline involves a sequence of steps that cover the following areas:
+ Data preparation, such as normalizations and transformations
+ Model training, such as hyper parameter tuning and validation
+ Model deployment and evaluation  

The following diagram shows an example pipeline:

[ ![Machine learning pipelines](./media/concept-ml-pipelines/pipelines.png) ]
(./media/concept-ml-pipelines/pipelines.png#lightbox)

## Why build pipelines with Azure Machine Learning?

With pipelines, you can optimize your workflow with simplicity, speed, portability, and reuse. When building pipelines with Azure Machine Learning, you can focus on what you know best &mdash; machine learning &mdash; rather than infrastructure.

Using distinct steps makes it possible to rerun only the steps you need as you tweak and test your workflow. Once the pipeline is designed, there is often more fine-tuning around the training loop of the pipeline. When you rerun a pipeline, the execution jumps to the steps that need to be rerun, such as an updated training script, and skips what hasn't changed. The same paradigm applies to unchanged scripts and metadata. 

With Azure Machine Learning, you can use distinct toolkits and frameworks for each step in your pipeline. Azure coordinates between the various compute targets you use so that your intermediate data can be shared with the downstream compute targets easily. 

## Key advantages

The key advantages to building pipelines for your machine learning workflows is:

|Key advantage|Description|
|:-------:|-----------|
|**Unattended&nbsp;execution**|Schedule a few scripts to run in parallel or in sequence in a reliable and unattended manner. Since data prep and modeling can last days or weeks, you can now focus on other tasks while your pipeline is running. |
|**Mixed and diverse compute**|Use multiple pipelines that are reliably coordinated across heterogeneous and scalable computes and storages. Individual pipeline steps can be run on different compute targets, such as HDInsight, GPU Data Science VMs, and Databricks, to make efficient use of available compute options.|
|**Reusability**|Pipelines can be templatized for specific scenarios such as retraining and batch scoring.  They can be triggered from external systems via simple REST calls.|
|**Tracking and versioning**|Instead of manually tracking data and result paths as you iterate, use the pipelines SDK to explicitly name and version your data sources, inputs, and outputs as well as manage scripts and data separately for increased productivity.|

## The Python SDK for pipelines

Use Python to create your ML pipelines. The Azure Machine Learning SDK offers imperative constructs for sequencing and parallelizing the steps in your pipelines. You can interact with it in Jupyter notebooks or in another preferred IDE. 

Using declarative data dependencies, you can optimize your tasks. The SDK includes a framework of pre-built modules for common tasks such as data transfer, compute target creation, and model publishing. The framework can be extended to model your own conventions.

Pipelines can be saved as templates so you can schedule batch-scoring or retraining jobs.

Check out the Python reference docs for pipelines:
+ https://docs.microsoft.com/python/api/azureml_pipeline_core/?view=azure-ml-py
+ https://docs.microsoft.com/python/api/azureml_pipeline_steps/?view=azure-ml-py
+ https://docs.microsoft.com/python/api/azureml_pipeline_viz/?view=azure-ml-py

## Next steps

Download [this Jupyter notebook](https://aka.ms/aml-notebook-train) to try out a pipeline for yourself. 

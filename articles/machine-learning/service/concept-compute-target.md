---
title: Compute targets
titleSuffix: Azure Machine Learning service
description: Learn about the compute resources used to train and deploy machine learning models for Azure Machine Learning service.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: sgilley
author: sdgilley
ms.date: 05/30/2019
# As a data scientist, I want to understand what a compute target is and why I need it.
---

#  What is an Azure Machine Learning compute target?

A compute target is the compute resource that you use to run your training script or to host your service deployment.  

Compute targets let you quickly change your compute environment without changing your code.  

* Start experimenting locally 
* Move to a distributed environment to train in parallel
* Deploy to several web hosting environments, or to IoT devices

Compute resources are attached to a workspace. Compute resources other than the local machine are shared by users of the workspace.

## Training targets

Train your models using these compute resources:

| Compute resource | SDK  |  Visual interface  
| ---- |:----:|:----:| 
| Your local computer | ✓ | &nbsp; |  
| Azure Machine Learning compute | ✓ | ✓ | 
| A Linux VM in Azure</br>(such as the Data Science Virtual Machine) | ✓ | &nbsp; | 
| Azure Databricks | ✓ | &nbsp; |  
| Azure Data Lake Analytics | ✓ | &nbsp; |  
| Apache Spark for HDInsight | ✓ | &nbsp; |  


## Deployment targets

Deploy your models on these compute resources:


| Compute resource | SDK  |  Visual interface 
| ---- |:----:|:----:|
| Azure Container Instances | ✓ |  ✓ | 
| Azure Kubernetes Service |  ✓ |   &nbsp; | 
| Azure IoT Edge |  ✓ |    &nbsp; | 
| Field-programmable gate array (FPGA) |  ✓ |   &nbsp; | 


## Managed compute

A managed compute resource is created and managed by Azure Machine Learning service. This compute is optimized for machine learning workloads. Azure Machine Learning compute is the only managed compute as of May 30, 2019. Additional managed compute resources may be added in the future.

You can create machine learning compute instances with any of the following:

* The Azure portal
* The Azure Machine Learning SDK
* The Azure CLI

All other compute resources must be created outside the workspace and then attached to it.

## Unmanaged compute

An unmanaged compute resource is *not* managed by Azure Machine Learning service. You create this type of compute outside Azure Machine Learning and then attach it to your workspace before use. Unmanaged compute resources can require additional steps for you to maintain or to improve performance for machine learning workloads.

## Next steps

* For information about selecting a compute target for training, see [Select and use a compute target to train your model](how-to-set-up-training-targets.md).

* For information about selecting a compute target for deployment, see the [Deploy models with Azure Machine Learning service](how-to-deploy-and-where.md).
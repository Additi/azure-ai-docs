---
title: High performance, cross platform inference with ONNX
titleSuffix: Azure Machine Learning service
description: Learn about ONNX and the ONNX Runtime for accelerating models
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual

ms.reviewer: jmartens
ms.author: prasantp
author: prasanthpul
ms.date: 4/8/2019
ms.custom: seodec18
---

# ONNX and Azure Machine Learning: Create and accelerate ML models

Optimizing machine learning models for inference is difficult since you need to tune the model and the inference library to make the most of the hardware capabilities. The problem becomes extremely hard if you want to get optimal performance on a variety of platforms (cloud/edge, CPU/GPU, etc), since each one has different capabilities and characteristics. The complexity explodes if you have models from a variety frameworks that need to run on a variety of platforms. It's very time consuming to optimize all the different combinations of frameworks and hardware. A solution to train once in your preferred framework and run anywhere on the cloud or edge is needed. This is where ONNX comes in.

Microsoft has partnered with a community of partners to create [Open Neural Network Exchange](https://onnx.ai) (ONNX), an open standard for representing machine learning models. Models from [many frameworks](https://onnx.ai/supported-tools) including TensorFlow, PyTorch, SciKit-Learn, Keras, Chainer, MXNet, MATLAB, and more can be exported or converted to the standard ONNX format. Once the models are in the ONNX format, they can be run on a variety of platforms and devices.

[ONNX Runtime](https://github.com/Microsoft/onnxruntime) is a high-performance inference engine for deploying ONNX models to production. It is optimized for both cloud and edge and works on Linux, Windows, and Mac. Written in C++, it also has C, Python, and C# APIs. ONNX Runtime provides support for all of the ONNX-ML specification and also integrates with the best accelerators on different hardware such as TensorRT on NVidia GPUs.

ONNX Runtime is not only open sourced, it's also used in high scale Microsoft services such as Bing, Office, and Cognitive Services where they have seen an average 2x performance gain. ONNX Runtime is also used as part of Windows ML on hundreds of millions of devices. By using ONNX Runtime, you can benefit from the extensive production-grade optimizations, testing, and ongoing improvements.

[![ONNX flow diagram showing training, converters, and deployment](media/concept-onnx/onnx.png) ](./media/concept-onnx/onnx.png#lightbox)

## Get ONNX models

You can obtain ONNX models in several ways:
+ Get a pre-trained ONNX model from the [ONNX Model Zoo](https://github.com/onnx/models) (see examples at the bottom of this article)
+ Generate a customized ONNX model from [Azure Custom Vision service](https://docs.microsoft.com/azure/cognitive-services/Custom-Vision-Service/) 
+ Convert existing model from another format to ONNX (see the [tutorials](https://github.com/onnx/tutorials)) 
+ Train a new ONNX model in Azure Machine Learning service (see examples at the bottom of this article)

## Deploy ONNX models in Azure

With Azure Machine Learning service, you can deploy, manage, and monitor your ONNX models. Using the standard [deployment workflow](concept-model-management-and-deployment.md) and ONNX Runtime, you can create a REST endpoint hosted in the cloud. See example Jupyter notebooks at the end of this article to try it out for yourself. 

## Examples

See [how-to-use-azureml/deployment/onnx](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/deployment/onnx) for example notebooks that create and deploy ONNX models.

[!INCLUDE [aml-clone-in-azure-notebook](../../../includes/aml-clone-for-examples.md)]

## More info

Learn more about ONNX or contribute to the project:
+ [ONNX project website](https://onnx.ai)
+ [ONNX code on GitHub](https://github.com/onnx/onnx)

Learn more about ONNX Runtime or contribute to the project:
+ [ONNX Runtime GitHub Repo](https://github.com/Microsoft/onnxruntime)



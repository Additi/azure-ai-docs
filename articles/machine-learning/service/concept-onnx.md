---
title: ONNX for AI models with Azure Machine Learning service
description: Learn how to use ONNX and Azure Machine Learning together. 
services: machine-learning
ms.service: machine-learning
ms.component: core
ms.topic: conceptual
ms.author: prasantp
author: prasanthpul
ms.date: 09/24/2018
---

# ONNX and Azure Machine Learning: Open and Interoperable AI

The [Open Neural Network Exchange](http://onnx.ai) (ONNX) format is an open standard for representing machine learning models. ONNX is supported by a community of partners, including Microsoft, who create compatible frameworks and tools. 

Microsoft is committed to open and interoperable AI so that data scientists and developers can:

+ Use the framework of their choice to create and train models
+ Deploy models cross-platform with minimal integration work

Microsoft supports ONNX across its products including Azure and Windows to help you achieve these goals.  

## Benefits of ONNX

Interoperability makes it possible to get great ideas into production faster. With ONNX, data scientists can choose their preferred framework for the job. Similarly, developers can spend less time getting models ready for production, and deploy across the cloud and edge.  

You can export ONNX models from many frameworks, including:
+ PyTorch
+ Chainer
+ Microsoft Cognitive Toolkit (CNTK)
+ MXNet

Converters exist other frameworks such as TensorFlow, Keras, and SciKit-Learn.

There is also an ecosystem of tools for visualizing and accelerating ONNX models. A number of pre-trained ONNX models are also available for common scenarios.

[ ![ONNX flow diagram showing training, converters, and deployment](media/concept-onnx/onnx.png) ]
(./media/concept-onnx/onnx.png#lightbox)


## Create ONNX models in Azure

You can create ONNX models in several ways:
+ Train a model in Azure Machine Learning service and convert or export it to ONNX - learn more with this [sample notebook](http://aka.ms/aml-onnx-training-notebook)

+ Get a pre-trained ONNX model from [Azure AI Model Gallery](https://gallery.azure.ai/models)

+ Generate a customized ONNX model from [Azure Custom Vision service](https://docs.microsoft.com/en-us/azure/cognitive-services/Custom-Vision-Service/)

+ Convert a model you got from somewhere else

Once you have an ONNX model, you can deploy it to Azure Machine Learning. You can also deploy the same ONNX model to Windows 10 devices using [Windows ML](https://docs.microsoft.com/en-us/windows/ai/).

## Convert to ONNX

For **TensorFlow** models, you can convert to the ONNX format with the [tensorflow-onnx converter](https://github.com/onnx/tensorflow-onnx).

For **Keras**, **ScitKit-Learn**, and other models, convert to ONNX using the [WinMLTools](https://docs.microsoft.com/en-us/windows/ai/convert-model-winmltools) package if you have a model in one of the following formats:
* Keras
* SciKit-Learn
* xgboost
* libSVM
* CoreML

## Prepare to deploy

With Azure Machine Learning service, you can deploy, manage, and monitor your ONNX models. Using the standard [deployment workflow](https://docs.microsoft.com/en-us/azure/machine-learning/service/concept-model-management-and-deployment) and the ONNX Runtime, you can create a REST endpoint hosted in the cloud.

Below is an example for deploying an ONNX model:

1. Initialize your Azure Machine Learning Workspace.

   ```python
   from azureml.core import Workspace

   ws = Workspace.from_config()
   print(ws.name, ws.resource_group, ws.location, ws.subscription_id, sep = '\n')
   ```

2. Register the model with Azure Machine Learning.

   ```python
   from azureml.core.model import Model

   model = Model.register(model_path = "model.onnx",
                          model_name = "MyONNXmodel",
                          tags = ["onnx"],
                          description = "test",
                          workspace = ws)
   ```

3. Create an image with the model and any dependencies.

   ```python
   from azureml.core.image import ContainerImage
   
   image_config = ContainerImage.image_configuration(execution_script = "score.py",
                                                     runtime = "python",
                                                     conda_file = "myenv.yml",
                                                     description = "test",
                                                     tags = ["onnx"]
                                                    )

   image = ContainerImage.create(name = "myonnxmodelimage",
                                 # this is the model object
                                 models = [model],
                                 image_config = image_config,
                                 workspace = ws)

   image.wait_for_creation(show_output = True)
   ```

   The file `score.py` contains the scoring logic and is included in the image. This file is used to run the model in the image. An example file for an ONNX model is shown below:

   ```python
   import json
   import numpy as np
   import onnxruntime
   import sys
   from azureml.core.model import Model

   def init():
       global model_path
       model_path = Model.get_model_path(model_name = 'MyONNXmodel')

   # note you can pass in multiple rows for scoring
   def run(raw_data):
       try:
           data = json.loads(raw_data)['data']
           data = np.array(data)
        
           sess = onnxruntime.InferenceSession(model_path)
           result = sess.run(["outY"], {"inX": data})
        
           return json.dumps({"result": result.tolist()})
       except Exception as e:
           result = str(e)
           return json.dumps({"error": result})
   ```

### Deploy your web service

You can deploy your ONNX model with Azure Machine Learning to:
* Azure Container Instances (ACI): [Learn how...](how-to-deploy-to-aci.md)

* Azure Kubernetes Service (AKS): [Learn how...](how-to-deploy-to-aks.md)

Once deployed, your ONNX model is hosted in the cloud and ready to be called!

Download [this Jupyter notebook](https://aka.ms/aml-onnx-notebook) to try it out for yourself. 

## Next steps

Learn more about ONNX or contribute to the project:
+ [ONNX project website](http://onnx.ai)

+ [ONNX code on GitHub](https://github.com/onnx/onnx)
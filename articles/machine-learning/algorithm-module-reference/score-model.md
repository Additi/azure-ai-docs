---
title:  "Decision Forest Regression: Module Reference"
titleSuffix: Azure Machine Learning service
description: Learn how to use the Decision Forest Regression module in Azure Machine Learning to create a regression model based on an ensemble of decision trees.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference

author: xiaoharper
ms.author: amlstudiodocs
ms.date: 04/22/2019
ROBOTS: NOINDEX
---
# Score Model

*Scores predictions for a trained classification or regression model*

Category: Machine Learning / Score 

## Module overview

This article describes how to use the [Score Model](./score-model.md) module in Azure Machine Learning to generate predictions using a trained classification or regression model.

## How to use Score Model

1. Add the **Score Model** module to your experiment.

2. Attach a trained model and a dataset containing new input data. 

    The data should be in a format compatible with the type of trained model you are using. The schema of the input dataset should also generally match the schema of the data used to train the model.

3. Run the experiment.

### Results

After you have generated a set of scores using [Score Model](./score-model.md):

+ To generate a set of metrics used for evaluating the model’s accuracy (performance).  you can connect the scored dataset to [Evaluate Model](./evaluate-model.md), 
+ Right-click the module and select **Visualize** to see a sample of the results.
+ Save the results to a dataset.

The score, or predicted value, can be in many different formats, depending on the model and your input data:

- For classification models, [Score Model](./score-model.md) outputs a predicted value for the class, as well as the probability of the predicted value.
- For regression models, [Score Model](./score-model.md) generates just the predicted numeric value.
- For image classification models, the score might be the class of object in the image, or a Boolean indicating whether a particular feature was found.

### Publish scores as a web service

A common use of scoring is to return the output as part of a predictive web service. For more information, see this tutorial on how to create a web service based on an experiment in Azure Machine Learning:


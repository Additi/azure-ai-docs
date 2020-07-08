---
title: Curated environments
titleSuffix: Azure Machine Learning
description: Collection of curated environments available in Azure Machine Learning
services: machine-learning
author: luisquintanilla
ms.author: luquinta
ms.reviewer: luquinta
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.date: 06/23/2020
---

# Azure Machine Learning Curated Environments

> [!NOTE]
> This list is updated as of June 2020. To get the most recent list, see the [environments article](./how-to-use-environments.md#)

## Azure AutoML

### AzureML AutoML

packages channels:
- anaconda
- conda-forge
- pytorch

dependencies:
- python=3.6.2
- pip:
  - azureml-core==1.8.0
  - azureml-pipeline-core==1.8.0
  - azureml-telemetry==1.8.0
  - azureml-defaults==1.8.0
  - azureml-interpret==1.8.0
  - azureml-explain-model==1.8.0
  - azureml-automl-core==1.8.0
  - azureml-automl-runtime==1.8.0
  - azureml-train-automl-client==1.8.0
  - azureml-train-automl-runtime==1.8.0.post1
  - inference-schema
  - pyarrow==0.17.0
  - py-cpuinfo==5.0.0
- numpy>=1.16.0,<=1.16.2
- pandas>=0.21.0,<=0.23.4
- py-xgboost<=0.90
- fbprophet==0.5
- setuptools-git
- psutil>5.0.0,<6.0.0

### AzureML AutoML GPU

packages channels:
- anaconda
- conda-forge
- pytorch

dependencies:
- python=3.6.2
- pip:
  - azureml-core==1.8.0
  - azureml-pipeline-core==1.8.0
  - azureml-telemetry==1.8.0
  - azureml-defaults==1.8.0
  - azureml-interpret==1.8.0
  - azureml-explain-model==1.8.0
  - azureml-automl-core==1.8.0
  - azureml-automl-runtime==1.8.0
  - azureml-train-automl-client==1.8.0
  - azureml-train-automl-runtime==1.8.0.post1
  - inference-schema
  - pyarrow==0.17.0
  - py-cpuinfo==5.0.0
- numpy>=1.16.0,<=1.16.2
- pandas>=0.21.0,<=0.23.4
- fbprophet==0.5
- setuptools-git
- psutil>5.0.0,<6.0.0

### AzureML AutoML DNN

packages channels:
- anaconda
- conda-forge
- pytorch

dependencies:
- python=3.6.2
- pip:
  - azureml-core==1.8.0
  - azureml-pipeline-core==1.8.0
  - azureml-telemetry==1.8.0
  - azureml-defaults==1.8.0
  - azureml-interpret==1.8.0
  - azureml-explain-model==1.8.0
  - azureml-automl-core==1.8.0
  - azureml-automl-runtime==1.8.0
  - azureml-train-automl-client==1.8.0
  - azureml-train-automl-runtime==1.8.0.post1
  - inference-schema
  - pytorch-transformers==1.0.0
  - spacy==2.1.8
  - https://aka.ms/automl-resources/packages/en_core_web_sm-2.1.0.tar.gz
  - pyarrow==0.17.0
  - py-cpuinfo==5.0.0
- pandas>=0.21.0,<=0.23.4
- numpy>=1.16.0,<=1.16.2
- py-xgboost<=0.90
- fbprophet==0.5
- setuptools-git
- pytorch=1.4.0
- cudatoolkit=10.0.130
- psutil>5.0.0,<6.0.0

### AzureML AutoML DNN GPU

packages channels:
- anaconda
- conda-forge
- pytorch

dependencies:
- python=3.6.2
- pip:
  - azureml-core==1.8.0
  - azureml-pipeline-core==1.8.0
  - azureml-telemetry==1.8.0
  - azureml-defaults==1.8.0
  - azureml-interpret==1.8.0
  - azureml-explain-model==1.8.0
  - azureml-automl-core==1.8.0
  - azureml-automl-runtime==1.8.0
  - azureml-train-automl-client==1.8.0
  - azureml-train-automl-runtime==1.8.0.post1
  - inference-schema
  - horovod==0.19.4
  - pytorch-transformers==1.0.0
  - spacy==2.1.8
  - https://aka.ms/automl-resources/packages/en_core_web_sm-2.1.0.tar.gz
  - pyarrow==0.17.0
  - py-cpuinfo==5.0.0
- pandas>=0.21.0,<=0.23.4
- numpy>=1.16.0,<=1.16.2
- fbprophet==0.5
- setuptools-git
- pytorch=1.4.0
- cudatoolkit=10.0.130
- psutil>5.0.0,<6.0.0

### AzureML AutoML DNN Vision GPU

dependencies:
- python=3.7
- pip:
  - azureml-automl-core==1.8.0
  - azureml-core==1.8.0
  - azureml-telemetry==1.8.0
  - azureml-train-automl-client==1.8.0
  - azureml-automl-runtime==1.8.0
  - azureml-interpret==1.8.0
  - azureml-defaults==1.8.0
  - azureml-explain-model==1.8.0
  - azureml-train-automl-runtime==1.8.0.post1
  - azureml-train-automl==1.8.0
  - azureml-contrib-dataset==1.8.0
  - azureml-pipeline-core==1.8.0
  - azureml-train-restclients-hyperdrive==1.8.0
  - azureml-train-core==1.8.0
  - azureml-pipeline-steps==1.8.0
  - azureml-pipeline==1.8.0
  - azureml-train==1.8.0
  - azureml-sdk==1.8.0
  - azureml-contrib-automl-dnn-vision==1.8.0

## Azure ML Designer

### AzureML Designer

packages channels:
- conda-forge

dependencies:
- python=3.6.8
- scikit-surprise=1.0.6
- pip:
  - azureml-designer-classic-modules==0.0.124
  - https://github.com/explosion/spacy-models/releases/download/en_core_web_sm-2.1.0/en_core_web_sm-2.1.0.tar.gz#egg=en_core_web_sm
  - spacy==2.1.7

### AzureML-Designer-CV

packages channels:
- defaults

dependencies:
- python=3.6.8
- pip:
  - azureml-designer-cv-modules==0.0.6

### AzureML Designer CV Transform

packages channels:
- defaults

dependencies:
- python=3.6.8
- pip:
  - azureml-designer-cv-modules[pytorch]==0.0.6

### AzureML Designer IO

packages channels:
- defaults

dependencies:
- python=3.6.8
- pip:
  - azureml-dataprep>=1.6
  - azureml-designer-dataio-modules==0.0.30

### AzureML Designer NLP

packages channels:
- defaults

dependencies:
- python=3.6.8
- pip:
  - azureml-designer-classic-modules==0.0.121
  - https://github.com/explosion/spacy-models/releases/download/en_core_web_sm-2.1.0/en_core_web_sm-2.1.0.tar.gz#egg=en_core_web_sm
  - spacy==2.1.7

### AzureML Designer PyTorch

packages channels:
- defaults

dependencies:
- python=3.6.8
- pip:
  - azureml-designer-pytorch-modules==0.0.8

### AzureML Designer PyTorch Train

packages channels:
- defaults

dependencies:
- python=3.6.8
- pip:
  - azureml-designer-pytorch-modules==0.0.8

### AzureML Designer R

packages channels:
- conda-forge

dependencies:
- python=3.6.8
- r-caret=6.0
- r-catools=1.17.1
- r-cluster=2.1.0
- r-dplyr=0.8.5
- r-e1071=1.7
- r-forcats=0.5.0
- r-forecast=8.12
- r-glmnet=2.0
- r-igraph=1.2.4
- r-matrix=1.2
- r-mclust=5.4.6
- r-mgcv=1.8
- r-nlme=3.1
- r-nnet=7.3
- r-plyr=1.8.6
- r-randomforest=4.6
- r-reticulate=1.12
- r-rocr=1.0
- r-rodbc=1.3
- r-rpart=4.1
- r-stringr=1.4.0
- r-tidyverse=1.2.1
- r-timedate=3043.102
- r-tseries=0.10
- r=3.5.1
- pip:
  - azureml-designer-classic-modules==0.0.124

### AzureML Designer Recommender

packages channels:
- defaults

dependencies:
- python=3.6.8
- pip:
  - azureml-designer-recommender-modules==0.0.5

### AzureML Designer Score

packages channels:
- defaults

dependencies:
- conda
- python=3.6.8
- pip:
  - azureml-designer-score-modules==0.0.5

### AzureML Designer Transform

packages channels:
- defaults

dependencies:
- python=3.6.8
- pip:
  - azureml-designer-datatransform-modules==0.0.49

## AzureML Hyperdrive ForecastDNN

dependencies:
- python=3.7
- pip:
  - azureml-core==1.8.0
  - azureml-pipeline-core==1.8.0
  - azureml-telemetry==1.8.0
  - azureml-defaults==1.8.0
  - azureml-automl-core==1.8.0
  - azureml-automl-runtime==1.8.0
  - azureml-train-automl-client==1.8.0
  - azureml-train-automl-runtime==1.8.0.post1
  - azureml-contrib-automl-dnn-forecasting==1.8.0

## AzureML Minimal

packages channels:
- conda-forge

dependencies:
- python=3.6.2
- pip:
  - azureml-core==1.8.0
  - azureml-defaults==1.8.0

## AzureML Sidecar

packages channels:
- conda-forge

dependencies:
- python=3.6.2

## AzureML Tutorial

packages channels:
- anaconda
- conda-forge

dependencies:
- python=3.6.2
- pip:
  - azureml-core==1.8.0
  - azureml-defaults==1.8.0
  - azureml-telemetry==1.8.0
  - azureml-train-restclients-hyperdrive==1.8.0
  - azureml-train-core==1.8.0
  - azureml-widgets==1.8.0
  - azureml-pipeline-core==1.8.0
  - azureml-pipeline-steps==1.8.0
  - azureml-opendatasets==1.8.0
  - azureml-automl-core==1.8.0
  - azureml-automl-runtime==1.8.0
  - azureml-train-automl-client==1.8.0
  - azureml-train-automl-runtime==1.8.0.post1  
  - azureml-train-automl==1.8.0
  - azureml-train==1.8.0
  - azureml-sdk==1.8.0
  - azureml-interpret==1.8.0
  - azureml-tensorboard==1.8.0
  - azureml-mlflow==1.8.0
  - mlflow
  - sklearn-pandas
- pandas
- numpy
- tqdm
- scikit-learn
- matplotlib

## Dask

### AzureML Dask CPU

packages channels:
- conda-forge
- pytorch
- defaults

dependencies:
- python=3.6.9
- pip:
  - adlfs
  - azureml-core==1.8.0
  - azureml-dataprep
  - dask[complete]
  - dask-ml[complete]
  - distributed
  - fastparquet
  - fsspec
  - joblib
  - jupyterlab
  - lz4
  - mpi4py
  - notebook
  - pyarrow

### AzureML Dask GPU

packages channels:
- conda-forge

dependencies:
- python=3.6.9
- pip:
  - azureml-defaults==1.8.0
  - adlfs
  - azureml-core==1.8.0
  - azureml-dataprep
  - dask[complete]
  - dask-ml[complete]
  - distributed
  - fastparquet
  - fsspec
  - joblib
  - jupyterlab
  - lz4
  - mpi4py
  - notebook
  - pyarrow
- matplotlib

## Chainer

### AzureML Chainer 5.1.0 CPU

packages channels:
- conda-forge

dependencies:
- python=3.6.2
- pip:
  - azureml-core==1.8.0
  - azureml-defaults==1.8.0
  - azureml-telemetry==1.8.0
  - azureml-train-restclients-hyperdrive==1.8.0
  - azureml-train-core==1.8.0
  - chainer==5.1.0
  - mpi4py==3.0.0

### AzureML Chainer 5.1.0 GPU

packages channels:
- conda-forge

dependencies:
- python=3.6.2
- pip:
  - azureml-core==1.8.0
  - azureml-defaults==1.8.0
  - azureml-telemetry==1.8.0
  - azureml-train-restclients-hyperdrive==1.8.0
  - azureml-train-core==1.8.0
  - chainer==5.1.0
  - cupy-cuda90==5.1.0
  - mpi4py==3.0.0

## PySpark

### AzureML PySpark MmlSpark 0.15

packages channels:
- conda-forge

dependencies:
- python=3.6.2
- pip:
  - azureml-core==1.8.0
  - azureml-defaults==1.8.0
  - azureml-telemetry==1.8.0
  - azureml-train-restclients-hyperdrive==1.8.0
  - azureml-train-core==1.8.0

## PyTorch

### AzureML PyTorch 1.0 CPU

packages channels:
- conda-forge

dependencies:
- python=3.6.2
- pip:
  - azureml-core==1.8.0
  - azureml-defaults==1.8.0
  - azureml-telemetry==1.8.0
  - azureml-train-restclients-hyperdrive==1.8.0
  - azureml-train-core==1.8.0
  - torch==1.0
  - torchvision==0.2.1
  - mkl==2018.0.3
  - horovod==0.16.1

### AzureML PyTorch 1.0 GPU

packages channels:
- conda-forge

dependencies:
- python=3.6.2
- pip:
  - azureml-core==1.8.0
  - azureml-defaults==1.8.0
  - azureml-telemetry==1.8.0
  - azureml-train-restclients-hyperdrive==1.8.0
  - azureml-train-core==1.8.0
  - torch==1.0
  - torchvision==0.2.1
  - mkl==2018.0.3
  - horovod==0.16.1

### AzureML-PyTorch-1.1-CPU

packages channels:
- conda-forge

dependencies:
- python=3.6.2
- pip:
  - azureml-core==1.8.0
  - azureml-defaults==1.8.0
  - azureml-telemetry==1.8.0
  - azureml-train-restclients-hyperdrive==1.8.0
  - azureml-train-core==1.8.0
  - torch==1.1
  - torchvision==0.2.1
  - mkl==2018.0.3
  - horovod==0.16.1
  - tensorboard==1.14.0
  - future==0.17.1

### AzureML PyTorch 1.1 GPU

packages channels:
- conda-forge

dependencies:
- python=3.6.2
- pip:
  - azureml-core==1.8.0
  - azureml-defaults==1.8.0
  - azureml-telemetry==1.8.0
  - azureml-train-restclients-hyperdrive==1.8.0
  - azureml-train-core==1.8.0
  - torch==1.1
  - torchvision==0.2.1
  - mkl==2018.0.3
  - horovod==0.16.1
  - tensorboard==1.14.0
  - future==0.17.1

### AzureML PyTorch 1.2 CPU

packages channels:
- conda-forge

dependencies:
- python=3.6.2
- pip:
  - azureml-core==1.8.0
  - azureml-defaults==1.8.0
  - azureml-telemetry==1.8.0
  - azureml-train-restclients-hyperdrive==1.8.0
  - azureml-train-core==1.8.0
  - torch==1.2
  - torchvision==0.4.0
  - mkl==2018.0.3
  - horovod==0.16.1
  - tensorboard==1.14.0
  - future==0.17.1

### AzureML PyTorch 1.2 GPU

packages channels:
- conda-forge

dependencies:
- python=3.6.2
- pip:
  - azureml-core==1.8.0
  - azureml-defaults==1.8.0
  - azureml-telemetry==1.8.0
  - azureml-train-restclients-hyperdrive==1.8.0
  - azureml-train-core==1.8.0
  - torch==1.2
  - torchvision==0.4.0
  - mkl==2018.0.3
  - horovod==0.16.1
  - tensorboard==1.14.0
  - future==0.17.1

### AzureML PyTorch 1.3 CPU

packages channels:
- conda-forge

dependencies:
- python=3.6.2
- pip:
  - azureml-core==1.8.0
  - azureml-defaults==1.8.0
  - azureml-telemetry==1.8.0
  - azureml-train-restclients-hyperdrive==1.8.0
  - azureml-train-core==1.8.0
  - torch==1.3
  - torchvision==0.4.1
  - mkl==2018.0.3
  - horovod==0.18.1
  - tensorboard==1.14.0
  - future==0.17.1

### AzureML PyTorch 1.3 GPU

packages channels:
- conda-forge

dependencies:
- python=3.6.2
- pip:
  - azureml-core==1.8.0
  - azureml-defaults==1.8.0
  - azureml-telemetry==1.8.0
  - azureml-train-restclients-hyperdrive==1.8.0
  - azureml-train-core==1.8.0
  - torch==1.3
  - torchvision==0.4.1
  - mkl==2018.0.3
  - horovod==0.18.1
  - tensorboard==1.14.0
  - future==0.17.1

### AzureML PyTorch 1.4 CPU

packages channels:
- conda-forge

dependencies:
- python=3.6.2
- pip:
  - azureml-core==1.8.0
  - azureml-defaults==1.8.0
  - azureml-telemetry==1.8.0
  - azureml-train-restclients-hyperdrive==1.8.0
  - azureml-train-core==1.8.0
  - torch==1.4.0
  - torchvision==0.5.0
  - mkl==2018.0.3
  - horovod==0.18.1
  - tensorboard==1.14.0
  - future==0.17.1

### AzureML PyTorch 1.4 GPU

packages channels:
- conda-forge

dependencies:
- python=3.6.2
- pip:
  - azureml-core==1.8.0
  - azureml-defaults==1.8.0
  - azureml-telemetry==1.8.0
  - azureml-train-restclients-hyperdrive==1.8.0
  - azureml-train-core==1.8.0
  - torch==1.4.0
  - torchvision==0.5.0
  - mkl==2018.0.3
  - horovod==0.18.1
  - tensorboard==1.14.0
  - future==0.17.1

### AzureML PyTorch 1.5 CPU

packages channels:
- conda-forge

dependencies:
- python=3.6.2
- pip:
  - azureml-core==1.8.0
  - azureml-defaults==1.8.0
  - azureml-telemetry==1.8.0
  - azureml-train-restclients-hyperdrive==1.8.0
  - azureml-train-core==1.8.0
  - torch==1.5.0
  - torchvision==0.5.0
  - mkl==2018.0.3
  - horovod==0.19.1
  - tensorboard==1.14.0
  - future==0.17.1

### AzureML PyTorch 1.5 GPU

packages channels:
- conda-forge

dependencies:
- python=3.6.2
- pip:
  - azureml-core==1.8.0
  - azureml-defaults==1.8.0
  - azureml-telemetry==1.8.0
  - azureml-train-restclients-hyperdrive==1.8.0
  - azureml-train-core==1.8.0
  - torch==1.5.0
  - torchvision==0.5.0
  - mkl==2018.0.3
  - horovod==0.19.1
  - tensorboard==1.14.0
  - future==0.17.1

## Scikit Learn

### AzureML Scikit-learn 0.20.3

packages channels:
- conda-forge

dependencies:
- python=3.6.2
- pip:
  - azureml-core==1.8.0
  - azureml-defaults==1.8.0
  - azureml-telemetry==1.8.0
  - azureml-train-restclients-hyperdrive==1.8.0
  - azureml-train-core==1.8.0
  - scikit-learn==0.20.3
  - scipy==1.2.1
  - joblib==0.13.2

## TensorFlow

### AzureML TensorFlow 1.10 CPU

packages channels:
- conda-forge

dependencies:
- python=3.6.2
- pip:
  - azureml-core==1.8.0
  - azureml-defaults==1.8.0
  - azureml-telemetry==1.8.0
  - azureml-train-restclients-hyperdrive==1.8.0
  - azureml-train-core==1.8.0
  - tensorflow==1.10
  - horovod==0.15.2

### AzureML TensorFlow 1.10 GPU

packages channels:
- conda-forge

dependencies:
- python=3.6.2
- pip:
  - azureml-core==1.8.0
  - azureml-defaults==1.8.0
  - azureml-telemetry==1.8.0
  - azureml-train-restclients-hyperdrive==1.8.0
  - azureml-train-core==1.8.0
  - tensorflow-gpu==1.10.0
  - horovod==0.15.2

### AzureML TensorFlow 1.12 CPU

packages channels:
- conda-forge

dependencies:
- python=3.6.2
- pip:
  - azureml-core==1.8.0
  - azureml-defaults==1.8.0
  - azureml-telemetry==1.8.0
  - azureml-train-restclients-hyperdrive==1.8.0
  - azureml-train-core==1.8.0
  - tensorflow==1.12
  - horovod==0.15.2

### AzureML TensorFlow 1.12 GPU

packages channels:
- conda-forge

dependencies:
- python=3.6.2
- pip:
  - azureml-core==1.8.0
  - azureml-defaults==1.8.0
  - azureml-telemetry==1.8.0
  - azureml-train-restclients-hyperdrive==1.8.0
  - azureml-train-core==1.8.0
  - tensorflow-gpu==1.12.0
  - horovod==0.15.2

### AzureML-TensorFlow 1.13 CPU

packages channels:
- conda-forge

dependencies:
- python=3.6.2
- pip:
  - azureml-core==1.8.0
  - azureml-defaults==1.8.0
  - azureml-telemetry==1.8.0
  - azureml-train-restclients-hyperdrive==1.8.0
  - azureml-train-core==1.8.0
  - tensorflow==1.13.1
  - horovod==0.16.1

### AzureML TensorFlow 1.13 GPU

packages channels:
- conda-forge

dependencies:
- python=3.6.2
- pip:
  - azureml-core==1.8.0
  - azureml-defaults==1.8.0
  - azureml-telemetry==1.8.0
  - azureml-train-restclients-hyperdrive==1.8.0
  - azureml-train-core==1.8.0
  - tensorflow-gpu==1.13.1
  - horovod==0.16.1

### AzureML TensorFlow 2.0 CPU

packages channels:
- conda-forge

dependencies:
- python=3.6.2
- pip:
  - azureml-core==1.8.0
  - azureml-defaults==1.8.0
  - azureml-telemetry==1.8.0
  - azureml-train-restclients-hyperdrive==1.8.0
  - azureml-train-core==1.8.0
  - tensorflow==2.0
  - horovod==0.18.1

### AzureML TensorFlow 2.0 GPU

packages channels:
- conda-forge

dependencies:
- python=3.6.2
- pip:
  - azureml-core==1.8.0
  - azureml-defaults==1.8.0
  - azureml-telemetry==1.8.0
  - azureml-train-restclients-hyperdrive==1.8.0
  - azureml-train-core==1.8.0
  - tensorflow-gpu==2.0.0
  - horovod==0.18.1

### AzureML TensorFlow 2.1 CPU

packages channels:
- conda-forge

dependencies:
- python=3.6.2
- pip:
  - azureml-core==1.8.0
  - azureml-defaults==1.8.0
  - azureml-telemetry==1.8.0
  - azureml-train-restclients-hyperdrive==1.8.0
  - azureml-train-core==1.8.0
  - tensorflow==2.1.0
  - horovod==0.19.1

### AzureML TensorFlow 2.1 GPU

packages channels:
- conda-forge

dependencies:
- python=3.6.2
- pip:
  - azureml-core==1.8.0
  - azureml-defaults==1.8.0
  - azureml-telemetry==1.8.0
  - azureml-train-restclients-hyperdrive==1.8.0
  - azureml-train-core==1.8.0
  - tensorflow-gpu==2.1.0
  - horovod==0.19.1

## AzureML VowpalWabbit 8.8.0

packages channels:
- conda-forge

dependencies:
- python=3.6.2
- pip:
  - azureml-core==1.8.0
  - azureml-defaults==1.8.0
  - azureml-dataprep[fuse,pandas]
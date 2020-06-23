---
title: 'What is an Azure Machine Learning compute instance?'
titleSuffix: Azure Machine Learning
description: Learn about the Azure Machine Learning compute instance, a fully managed cloud-based workstation. 
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: sgilley
author: sdgilley
ms.date: 06/22/2020
# As a data scientist, I want to know what a compute instance is and how to use it for Azure Machine Learning.
---

# What is an Azure Machine Learning compute instance?

An Azure Machine Learning compute instance is a fully-managed cloud-based workstation for data scientists. 

Compute instances make it easy to get started with Azure Machine Learning development as well as provide management and enterprise readiness capabilities for IT administrators.  

Use a compute instance as your fully configured and managed development environment in the cloud.

Compute instances are typically used as development environments.  They can also be used as a compute target for training and inferencing for development and testing.  

For production grade model training use an [Azure Machine Learning compute cluster](how-to-set-up-training-targets.md#amlcompute) with multi-node scaling capabilities. For production grade model deployment, use [Azure Kubernetes Service cluster](how-to-deploy-azure-kubernetes-service.md).

## Why use a compute instance?

A compute instance is a fully-managed cloud-based workstation optimized for your machine learning development environment. It provides the following benefits:

|Key benefits||
|----|----|
|Productivity|Data scientists can build and deploy models using integrated notebooks and the following tools in their web browser:<br/>-  Jupyter<br/>-  JupyterLab<br/>-  RStudio|
|Managed & secure|Reduce your security footprint and add compliance with enterprise security requirements. Compute instances  provide robust management policies and secure networking configurations such as:<br/><br/>- Auto-provisioning from Resource Manager templates or Azure Machine Learning SDK<br/>- [Role-based access control (RBAC)](/azure/role-based-access-control/overview)<br/>- [Virtual network support](how-to-enable-virtual-network.md#compute-instance)<br/>- SSH policy to enable/disable SSH access|
|Preconfigured&nbsp;for&nbsp;ML|Save time on setup tasks with pre-configured and up-to-date ML packages, deep learning frameworks, GPU drivers.|
|Fully customizable|Broad support for Azure VM types including GPUs and persisted low-level customization such as installing packages and drivers makes advanced scenarios a breeze. |

## <a name="contents"></a>Tools and environments

Azure Machine Learning compute instance enables you to author, train, and deploy models in a fully integrated notebook experience in your workspace.


These tools and environments are installed on the compute instance: 

|General tools & environments|Details|
|----|:----:|
|Drivers|`CUDA`</br>`cuDNN`</br>`NVIDIA`</br>`Blob FUSE` |
|Intel MPI library||
|Azure CLI ||
|Azure Machine Learning samples ||
|Docker||
|Nginx||
|NCCL 2.0 ||
|Protobuf|| 

|**R** tools & environments|Details|
|----|:----:|
|RStudio Server Open Source Edition||
|R kernel||
|Azure Machine Learning SDK for R|[azuremlsdk](https://azure.github.io/azureml-sdk-for-r/reference/index.html)</br>SDK samples|

|**PYTHON** tools & environments|Details|
|----|----|
|Anaconda Python||
|Jupyter and extensions||
|Jupyterlab and extensions||
[Azure Machine Learning SDK for Python](https://docs.microsoft.com/python/api/overview/azure/ml/intro?view=azure-ml-py)</br>from PyPI|Includes most of the azureml extra packages.  To see the full list, [open a terminal window on your compute instance](how-to-run-jupyter-notebooks.md#terminal) and run <br/> `conda list -n azureml_py36 azureml*` |
|Other PyPI packages|`jupytext`</br>`tensorboard`</br>`nbconvert`</br>`notebook`</br>`Pillow`|
|Conda packages|`cython`</br>`numpy`</br>`ipykernel`</br>`scikit-learn`</br>`matplotlib`</br>`tqdm`</br>`joblib`</br>`nodejs`</br>`nb_conda_kernels`|
|Deep learning packages|`PyTorch`</br>`TensorFlow`</br>`Keras`</br>`Horovod`</br>`MLFlow`</br>`pandas-ml`</br>`scrapbook`|
|ONNX packages|`keras2onnx`</br>`onnx`</br>`onnxconverter-common`</br>`skl2onnx`</br>`onnxmltools`|
|Azure Machine Learning Python & R SDK samples||

Python packages are all installed in the **Python 3.6 - AzureML** environment.  

Compute instances are typically used as development environments.  They can also be used as a compute target for training and inferencing for development and testing.  For large tasks, an [Azure Machine Learning compute cluster](how-to-set-up-training-targets.md#amlcompute) with multi-node scaling capabilities is a better compute target choice.

### Installing packages

You can install packages directly in a Jupyter notebook or Rstudio:

* RStudio Use the **Packages** tab on the bottom right, or the **Console** tab on the top left.  
* Python: Add install code and execute in a Jupyter notebook cell.

Or you can access a terminal window in any of these ways:

* RStudio: Select the **Terminal** tab on top left.
* Jupyter Lab:  Select the **Terminal** tile under the **Other** heading in the Launcher tab.
* Jupyter:  Select **New>Terminal** on top right in the Files tab.
* SSH to the machine.  Then install Python packages into the **Python 3.6 - AzureML** environment.  Install R packages into the **R** environment.

## Accessing files

Notebooks and R scripts are stored in the default storage account of your workspace in Azure file share.  These files are located under your “User files” directory. This storage makes it easy to share notebooks between compute instances. The storage account also keeps your notebooks safely preserved when you stop or delete a compute instance.

The Azure file share account of your workspace is mounted as a drive on the compute instance. This drive is the default working directory for Jupyter, Jupyter Labs, and RStudio. This means that the notebooks and other files you create in Jupyter, JupyterLab, or RStudio are automatically stored on the file share and available to use in other compute instances as well.

The files in the file share are accessible from all compute instances in the same workspace. Any changes to these files on the compute instance will be reliably persisted back to the file share.

You can also clone the latest Azure Machine Learning samples to your folder under the user files directory in the workspace file share.

Writing small files can be slower on network drives than writing to the VM itself.  If you are writing many small files, try using a directory directly on the compute instance, such as a `/tmp` directory. Please note these files will not be accessible from other compute instances. 

You can use the `\tmp` directory on the compute instance for your temporary data.  However, do not write large files of data on the OS disk of the compute instance.  Use [datastores](concept-azure-machine-learning-architecture.md#datasets-and-datastores) instead.

## Managing a compute instance

In your workspace in Azure Machine Learning studio, select **Compute**, then select **Compute Instance** on the top.

![Manage a compute instance](./media/concept-compute-instance/manage-compute-instance.png)

You can perform the following actions:

* [Create a compute instance](#create). 
* Refresh the compute instances tab.
* Start, stop and restart a compute instance.  You do pay for the instance whenever it is running. Stop the VM when you are not using it to reduce cost. Then start it again when you need it. 
* Delete a compute instance.
* Filter the list of compute instances based on all compute instance available to you, all that you create, or all in the workspace.

For each compute instance in your workspace that you have access to, you can:

* Access Jupyter, JupyterLab, RStudio on the compute instance
* SSH into compute instance. SSH access is disabled by default but can be enabled at compute instance creation time. SSH access is through public/private key mechanism. The tab will give you details for SSH connection such as IP address, username, and port number.
* Get details about a specific compute instance such as IP address, and region.

[RBAC](/azure/role-based-access-control/overview) allows you to control which users in the workspace can create, delete, start, stop, restart a compute instance. All users in the workspace contributor and owner role can create, delete, start, stop, and restart compute instances across the workspace. However, only the creator of a specific compute instance is allowed to access Jupyter, JupyterLab, and RStudio on that compute instance. The creator of the compute instance has the compute instance dedicated to them, have root access, and can terminal in through Jupyter. Compute instance will have single-user login of creator user and all actions will use that user’s identity for RBAC and attribution of experiment runs. SSH access is controlled through public/private key mechanism.

These actions can be controlled by RBAC:
* *Microsoft.MachineLearningServices/workspaces/computes/read*
* *Microsoft.MachineLearningServices/workspaces/computes/write*
* *Microsoft.MachineLearningServices/workspaces/computes/delete*
* *Microsoft.MachineLearningServices/workspaces/computes/start/action*
* *Microsoft.MachineLearningServices/workspaces/computes/stop/action*
* *Microsoft.MachineLearningServices/workspaces/computes/restart/action*

### <a name="create"></a>Create a compute instance

In your workspace in Azure Machine Learning studio, create a new compute instance from either the **Compute** section or in the **Notebooks** section when you are ready to run one of your notebooks.

:::image type="content" source="media/concept-compute-instance/create-compute-instance.png" alt-text="Create a new compute instance":::

 * Use these rules to name the compute instance:
    * Name is required and must be between 3 to 24 characters long.
    * Valid characters are upper and lower case letters, digits, and the  **-** character.
    * Name must start with a letter.
    * Name needs to be unique across all existing computes within an Azure region. You will see an alert if the name you choose is not unique.
    * If **-**  character is used, then it needs to be followed by at least one letter later in the name.

* Select the Azure VM type including GPUs. The VM type can not be changed after creation.
* Select whether the instance is for yourself or created for another user of the workspace.
* Enable/disable SSH access. SSH access is disabled by default but can be enabled at compute instance creation time.
* Optionally, configure virtual network settings. 

To create the compute instance within an Azure Virtual Network, ensure all the [network requirements](how-to-enable-virtual-network.md#compute-instance) are met. You can create also create compute instances in workspaces which has [Azure Private Link](how-to-configure-private-link.md) enabled.

You can also create an instance
* Directly from the [integrated notebooks experience](tutorial-1st-experiment-sdk-setup.md#azure)
* In Azure portal
* From Azure Resource Manager template
* With [Azure Machine Learning SDK](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/training/train-on-computeinstance/train-on-computeinstance.ipynb)
* From the [CLI extension for Azure Machine Learning](reference-azure-machine-learning-cli.md#computeinstance)

The dedicated cores per region quota, which applies to compute instance creation is unified and shared with Azure Machine Learning training cluster quota.

## Compute Target

Compute instances can be used as a [training compute target](concept-compute-target.md#train) similar to Azure Machine Learning compute training clusters. Provision a multi-GPU VM to run distributed training jobs using TensorFlow/PyTorch estimators. You can also create a run configuration and use it to run your experiment on compute instance. 
You can use compute instance as a local inferencing deployment target for testing/debugging scenarios.

## <a name="notebookvm"></a>What happened to Notebook VM?

Compute instances are replacing the Notebook VM.  

Any notebook files stored in the workspace file share and data in workspace data stores will be accessible from a compute instance. However, any custom packages previously installed on a Notebook VM will need to be re-installed on the compute instance. Quota limitations which apply to compute clusters creation will apply to compute instance creation as well. 

New Notebook VMs cannot be created. However, you can still access and use Notebook VMs you have created, with full functionality. Compute instances can be created in same workspace as the existing Notebook VMs. 


## Next steps

 * [Tutorial: Train your first ML model](tutorial-1st-experiment-sdk-train.md) shows how to use a compute instance with an integrated notebook.

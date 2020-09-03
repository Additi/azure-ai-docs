---
title: 'Visual Studio Code - Connect to compute instance  (preview)'
titleSuffix: Azure Machine Learning
description: Learn how to connect to an Azure Machine Learning compute instance in Visual Studio Code
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.custom: how-to
ms.author: jmartens
author: j-martens
ms.date: 08/27/2020
# As a data scientist, I want to connect to an Azure Machine Learning compute instance in Visual Studio Code to access my resources and run my code.
---

# Connect to an Azure Machine Learning compute instance with VS Code  (preview)

In this article, you'll learn how to connect to an Azure Machine Learning compute instance using Visual Studio Code.

An [Azure Machine Learning Compute Instance](concept-compute-instance.md) is a fully managed cloud-based workstation for data scientists and provides management and enterprise readiness capabilities for IT administrators.

There are two ways you can connect to a compute instance from Visual Studio Code:

* Remote Jupyter Notebook server. This option allows you to run Jupyter notebooks on a remote Jupyter Notebook server hosted by a compute instance.
* [Visual Studio Code Remote](https://code.visualstudio.com/docs/remote/remote-overview). Visual Studio Code Remote development allows you to use a container, remote machine, or the Windows Subsystem for Linux (WSL) as a full-featured development environment.

## Prerequisite  

* [Visual Studio Code](https://code.visualstudio.com/).
* SSH-enabled compute instance. For more information, [see the Create a compute instance guide.](https://docs.microsoft.com/azure/machine-learning/concept-compute-instance#create)
* On Windows platforms, you must [install an OpenSSH compatible SSH client](https://code.visualstudio.com/docs/remote/troubleshooting#_installing-a-supported-ssh-client) if one is not already present.

> [!Note]
> PuTTY is not supported on Windows since the ssh command must be in the path. 

## Configure compute instance as remote Jupyter Notebook server

In order to configure a compute instance as a remote Jupyter Notebook server, there are a few additional things you'll need:

* Azure Account Visual Studio Code Extension
* Azure Machine Learning Visual Studio Code Extension. For more information, see the [Azure Machine Learning Visual Studio Code Extension setup guide](tutorial-setup-vscode-extension.md).

To connect to a compute instance:

1. Open the command palette.
1. Enter into the text box `Python: Specify local or remote Jupyter Server for connections`.
1. Choose `Azure ML compute instance` from the list of Jupyter server options.
1. Select your subscription the list of subscriptions.
1. Select the workspace that contains the compute instance you want to use.
1. Select your compute instance from the list.
1. For the changes to take effect, you have to reload Visual Studio Code.
1. Open a Jupyter Notebook and run a cell to establish the connection with the compute instance

At this point, you can continue to run cells in your Jupyter notebook.

> [!TIP]
> You can also work with Python script files containing Jupyter-like code cells. For more information, visit the [Visual Studio Code Python interactive documentation](https://code.visualstudio.com/docs/python/jupyter-support-py).

## Configure Visual Studio Code Remote

For a full-featured development experience, you'll need to have the [Remote SSH Extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-ssh) installed:

### Get the IP and SSH port for your compute instance

1. Go to the Azure Machine Learning studio at https://ml.azure.com/.

2. Select your [workspace](concept-workspace.md).
1. Click the **Compute Instances** tab.
1. In the **Application URI** column, click the **SSH** link of the compute instance you want to use as a remote compute. 
1. In the dialog, take note of the IP Address and SSH port. 
1. Save your private key to the ~/.ssh/ directory on your local computer; for instance, open an editor for a new file and paste the key in: 

   **Linux**:

   ```sh
   vi ~/.ssh/id_azmlcitest_rsa  
   ```

   **Windows**:

   ```cmd
   notepad C:\Users\<username>\.ssh\id_azmlcitest_rsa
   ```

   The private key will look somewhat like this:

   ```text
   -----BEGIN RSA PRIVATE KEY-----

   MIIEpAIBAAKCAQEAr99EPm0P4CaTPT2KtBt+kpN3rmsNNE5dS0vmGWxIXq4vAWXD
   ..... 
   ewMtLnDgXWYJo0IyQ91ynOdxbFoVOuuGNdDoBykUZPQfeHDONy2Raw==

   -----END RSA PRIVATE KEY-----
   ```

1. Change permissions on file to make sure only you can read the file.  

   ```sh
   chmod 600 ~/.ssh/id_azmlcitest_rsa
   ```

### Add instance as a host

Open the file `~/.ssh/config` (Linux) or `C:\Users<username>.ssh\config` (Windows) in an editor and add a new entry similar to this:

```
Host azmlci1 

    HostName 13.69.56.51 

    Port 50000 

    User azureuser 

    IdentityFile ~/.ssh/id_azmlcitest_rsa
```

Here some details on the fields:

|Field|Description|
|----|---------|
|Host|Use whatever shorthand you like for the compute instance |
|HostName|This is the IP address of the compute instance |
|Port|This is the port shown on the SSH dialog above |
|User|This needs to be `azureuser` |
|IdentityFile|Should point to the file where you saved the private key |

Now, you should be able to ssh to your compute instance using the shorthand you used above, `ssh azmlci1`.

### Connect VS Code to the instance

1. [Install Visual Studio Code]().

1. [Install the 

1. Click the Remote-SSH icon on the left to show your SSH configurations.

1. Right-click the SSH host configuration you just created.

1. Select **Connect to Host in Current Window**. 

From here on, you are entirely working on the compute instance and you can now edit, debug, use git, use extensions, etc. -- just like you can with your local Visual Studio Code.

## Next steps

Now that you've set up Visual Studio Code Remote, you can use a compute instance as remote compute from Visual Studio Code to [interactively debug your code](how-to-debug-visual-studio-code.md).

[Tutorial: Train your first ML model](tutorial-1st-experiment-sdk-train.md) shows how to use a compute instance with an integrated notebook.
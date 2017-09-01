# Configuring Execution Targets

>Azure ML Execution Service currently supports Python 3.5.2 and Spark 2.1.x as Python and Spark runtime versions, respectively. 

You can choose to execute a Python/PySpark script in a Azure ML Workbench project either locally or at scale in the cloud. The options of execution target are:

* A Python (3.5.2) environment on your local computer installed by Azure ML Workbench.
* A conda Python environment inside of a Docker container on local computer
* A conda Python environment inside of a Docker container on a remote Linux machine such as an [Ubuntu-based DSVM on Azure](https://azuremarketplace.microsoft.com/marketplace/apps/microsoft-ads.linux-data-science-vm-ubuntu)
* [HDInsight for Spark](https://azure.microsoft.com/services/hdinsight/apache-spark/) on Azure

This article will focus on using the CLI (Command-line interface) command _**az ml experiment submit**_ and its **_--runconfig_** (or **_-c_** for short) switch to execute a Python or PySpark script. You can also start and manage executions (runs) from the Workbench desktop app, but you can get more immediate feedback from executing through CLI currently, so this guide focuses on the CLI experience. That being said, you can do everything stated here also from the Workbench desktop app.

Azure ML Execution Service leverages [conda](https://conda.io/docs/) to manage Python environments for execution when you execute against Docker container (local or remote). Note that you don't need to install conda yourself, Azure ML Workbench will provision it for you as part of the [installation process](Installation.md).

## Configuration files
All the configuration is managed through files in the **_aml_config_** folder inside a Azure ML Workbench project. Please refer to the [Configuration Files](aml_config.md) document for more details.

## Executing Python/PySpark scripts using Azure ML Workbench CLI
To launch the Azure ML command line experience, open a project in the Workbench app, and go to the menu _File_ --> _Open Command-Line Interface_. This will launch a terminal window in which you can enter commands to execute scripts in the current project folder. This terminal window is configured with the Python 3.5.2 environment installed by Workbench as the default Python path.

## CLI Execution Authentication
Please note when you execute any _az ml_ command from the command window, you must have authenticated against Azure already. CLI uses an independent authentication cache than than the desktop app. So just because you have logged in to Workbench desktop app, it doesn't mean you have authenticated your CLI environment.

To authenticate, follow the below steps. You only need to do this once in a while as the authenticate token will be cached locally for some periods of time. But if you find the token expired (seeing authentication errors), repeat the following steps.

```
# to authenticate 
$ az login

# to list subscriptions
$ az account list -o table

# to set current subscription to a particular subscription ID 
$ az account set -s <subscription_id>

# to verify your current Azure subscription
$ az account show
```

When you run _az ml_ command within a project folder, make sure that the project belongs to a Azure ML Experimentation ccount on the _current_ Azure subscription. Otherwise you will see execution errors.

### Local Computer
Executing a Python script in a local Python runtime is easy:
```
$ az ml experiment submit -c local my_script.py
```

When executing against _local_, your script is run in the Python environment installed by Azure ML. You can find the path to this default Python environment by typing the following command in the Azure ML Workbench CLI window.

```
$ conda env list
```
This conda command return the path to the Python interpreter of the _root_ conda environment. You will need to manually _pip install_ (or _conda install_) any required dependencies needed to run your script into this environment. 

>Note unlike the per-project conda environment set up by Azure ML Workbench when you execute against Docker container as explained below, Workbench will NOT automatically install any packages in this root conda Python environment for you. 

#### How to configure other Python IDEs to use the Azure ML Python environment

To use the _root_ conda Python environment created for you by Azure ML, you can simply copy the path of the _root_ conda Python interpreter (discoverable by using the above _conda env list_) command, and use the path to configure your favorite Python IDE. 

Below is an example using PyCharm. Note the Python interpreter configuration dialog box as well as the current debugging session. 

![PyCharm Config](../Images/py_charm.png)

You can also run the Python script in the PyCharm terminal window.

![PyCharm terminal window](../Images/py_charm_terminal.png)

> Please note that running az ml CLI is not supported yet from the terminal window of external IDEs.

Here is in Spyder.
![Spder](../Images/spyder_window.png)

And in VS Code.
![VS Code](../Images/vscode.png)

### Docker container on Local Computer
You can execute a Python or PySpark script in a Docker container running on your local computer:
```
$ az ml experiment submit -c docker my_script.py
```
Needless to say, you must have Docker engine already installed and running locally. When running against this target for the first time, a base Docker image is downloaded to your local machine, and a new image is constructed from the base image with added frameworks specified in the _conda_dependencies.yml_ file. Then a Docker container is started based on this new image. This is why the first run will take some time, but subsequent runs will reuse cached Docker layers and hence be significantly faster.

The _.compute_ file controls the type of base image pulled down. By default, it uses a Docker image with Spark installed along with [MMLSpark package](http://github.com/azure/mmlspar). This means you can execute both Python script as well as PySpark script in containers constructed from the images based on this base image. 

Running your script on a Docker image gives you the following benefits:

1. It ensures that your script can be reliably executed in other execution environments, such as your team member's machine for collaboration, or a lager machine for scale-up computing. Running on a Docker container helps you discover and avoid any local references that will impact such portability. 

2. It allows you to quickly test code on runtimes and frameworks that are complex to install and configure, such as Apache Spark, without having to install them yourself.

### Executing in a Docker Container in a Remote Linux Machine
You can also execute a Python or PySpark script in a Docker container running on a remote machine. The targeted remote machine must be a Linux machine accessible over SSH, and it must have Docker engine installed and started. [Ubuntu-flavored Data Science Virtual Machine (DSVM)](https://azuremarketplace.microsoft.com/marketplace/apps/microsoft-ads.linux-data-science-vm-ubuntu) on Azure is a good choice. You can also provision your own Linux VM on Azure (or anywhere else) and install Docker on it yourself. 

In order to run against a remote machine, you need to create a compute context using the following command.
```
$ az ml computetarget attach --name "remotevm" --address "192.168.1.11" --username "sshuser" --password "sshpassword"
```
The name _remotevm_ is arbitrary. This command creates a _remotevm.compute_ file in aml_config folder of your project which contains the following lines.
```
address: <IP address or addressable FQDN>
username: <ssh user name>
password: <hashed password>
```
You can also do this in the Workbench by directly creating a file named _remotevm.compute_ and copying/pasting and modifying the contents of another compute context definition file.

Once you have your configured compute context you want to run against, you can use the following command to run your script.
```
$ az ml experiment submit -c remotevm myscript.py
```
The Docker construction process is exactly the same as when you target a local Docker container so you should expect a similar execution experience.

### Executing in an Azure HDInsight Spark Cluster
You can also execute a PySpark script on a Spark cluster such as Apache Spark for Azure HDInsight . You need to create a compute context using the following command in order to do that.
```
$ az ml computetarget attach --cluster --name "azhdi" --address "<ip address>" --username "sshuser" --password "sshpassword" 
```
This creates a new file named _azhdi.compute_ in the _aml_config_ folder with the following content:
```
address: <IP address or addressable FQDN>
username: <cluster user name>
password: <password>
cluster: True
native: True
```
Please note that if want to use FQDN, and your HDI Spark cluster is named _foo_, the SSH endpoint is on the driver node named _foo-ssh.azurehdinsight.net_. Don't forget the **-ssh** postfix in the sever name.

Once you have the compute context, you can run the following command to execute your script.

```
$ az ml experiment submit -c azhdi myscript.py
```
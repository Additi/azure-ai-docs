---
title: Azure Machine Learning Enterprise & Basic Editions
description: Learn about the differences between the editions of Azure Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
author: j-martens
ms.author: jmartens
ms.date: 06/11/2020
---

# Enterprise & Basic Editions of Azure Machine Learning 

Azure Machine Learning offers two editions tailored for your machine learning needs. These editions determine which machine learning tools are available to developers and data scientists from their workspace.   

<br/>
<br/>

| Basic edition | Enterprise edition                 |
|------------------------------------------------------------------------------------|-----------|
|Great for: <br/>+ open-source development <br/>+ at cloud scale with a<br/>+ code-first experience <br/><br/>Basic workspaces allow you to continue using Azure Machine Learning and [pay for only the Azure resources consumed during the machine learning process](concept-plan-manage-cost.md). |All of Basic edition, plus:<br/>+ the studio web interface <br/>+ secure, comprehensive ML lifecycle management <br/>+ for all skill levels<br/><br/>Enterprise edition workspaces are charged only for their Azure consumption while the edition is in preview. |

## How to choose an edition

You assign the edition whenever you create a workspace. And, pre-existing workspaces have been converted to the Basic edition for you. 

Customers are responsible for costs incurred on compute and other Azure resources during this time. Learn how to [manage costs for Azure Machine Learning](concept-plan-manage-cost.md).

Learn how to [upgrade a Basic workspace to Enterprise edition](how-to-manage-workspace.md#upgrade). 


## What's in each edition

### Data for Machine Learning capabilities  

| Capabilities                     | Edition                 |
|------------------------------------------------------------------------------------|:-----------:|
| Create & manage labeling projects in studio (Web)                                                | All                     |
| Labeler in studio (Web)                                    | All                     |
| Labeling using private workforce                                                  | All                     |
| ML assisted labeling (Image classification and Object detection)                   | Enterprise edition only |
| Create & manage datasets + datastores in Python                       | All                     |
| Create & manage datasets + datastores in studio (Web)                         | All                     |
| Drift: View & manage dataset monitors in Python                           | All                     |
| Drift: View & manage dataset monitors in studio (Web)                            | Enterprise edition only |

<br/>
<br/>

### Build & train capabilities

| Capabilities    | Edition                 |
|------------------------------------------------------------------------------------|:-----------:|
| Automated Machine Learning - Create & run experiments in notebooks               | All                     |
| Automated Machine Learning - Create & run experiments in studio web experience   | Enterprise edition only |
| Automated Machine Learning - Industry-leading forecasting capabilities             | Enterprise edition only |
| Automated Machine Learning - Support for deep learning & other advanced learners | Enterprise edition only |
| Automated Machine Learning - Large data support (up to 100 GB)                     | Enterprise edition only |
| Responsible ML - Model Explainability                                              | All                     |
| Responsible ML - Differential privacy WhiteNoise toolkit                           | All                     |
| Responsible ML - custom tags in Azure Machine Learning to implement datasheets     | All                     |
| Responsible ML - Fairness AzureML Integration                                      | All                     |
| Visual Studio Code integration                                                     | All                     |
| Reinforcement Learning                                                             | All                     |
| Experimentation UI                                                                 | All                     |
| Pipelines: Create, run and publish  in Python                           | All                     |
| Pipelines: Create, edit and delete scheduled runs of pipelines in Python| All                     |
| Pipelines: Create pipeline endpoints in Python SDK                                   | All                     |
| Pipelines: View run details in studio (web)                                              | All                     |
| Pipelines: Create, run, visualize & publish in designer                  | Enterprise edition only |
| Pipelines: Create pipeline endpoints in designer | Enterprise edition only |
| Managed compute instances for integrated notebooks                                 | All                     |
| Jupyter, JupyterLab Integration                                                    | All                     |
| R SDK support                                                                      | All                     |
| Python SDK support                                                                 | All                     |


<br/>
<br/>

### Deploy & model management capabilities

| Capabilities                            | Edition                 |
|------------------------------------------------------------------------------------|:-----------:|
| The Azure DevOps extension for Machine Learning & the Azure ML CLI                 | All                     |
| [Event Grid integration](how-to-use-event-grid.md)                                                             | All                     |
| Integrate Azure Stream Analytics with Azure Machine Learning                       | All                     |
| Create ML pipelines in SDK                                                         | All                     |
| Batch inferencing                                                                  | All                     |
| FPGA based Hardware Accelerated Models                                             | All                     |
| Model profiling                                                                    | All                     |
| Explainability in UI                                                               | Enterprise edition only |

<br/>
<br/>

### Security, governance, and control capabilities

| Capabilities     | Edition                 |
|------------------------------------------------------------------------------------|:-----------:|
| [Role-based Access Control](how-to-assign-roles.md) (RBAC) support                                           | All                     |
| [Virtual Network (VNet)](how-to-enable-virtual-network.md) support for compute                                         | All                     |
| Scoring endpoint authentication                                                    | All                     |
| [Workplace Private link](how-to-configure-private-link.md)                                                            | All                     |
| Managed Identity for AML Compute                                                   | All                     |
| [Quota management](how-to-manage-quotas.md) across workspaces                                                 | Enterprise edition only |

## Next steps

Learn more about what's available in the Azure Machine Learning [edition overview & pricing page](https://azure.microsoft.com/pricing/details/machine-learning/). 

Learn how to [upgrade a Basic workspace to Enterprise edition](how-to-manage-workspace.md#upgrade). 



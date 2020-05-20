---
title: How to enable high availability
titleSuffix: Azure Machine Learning
description: Learn how to make your Azure Machine Learning workspace and associated resources more resilient to outages by using a high availability configuration.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: jhirono
author: 
ms.reviewer: larryfr
ms.date: 05/14/2020
---

# Enable high availability for Azure Machine Learning
[!INCLUDE [aml-applies-to-basic-enterprise-sku](../../includes/aml-applies-to-basic-enterprise-sku.md)]

Learn how to make your Azure Machine Learning workspace and associated resources more resilient by using high availability configurations.

## Understand Azure Services for Azure Machine Learning

Azure Machine Learning depends on multiple Azure services and has several layers. Some of them are provisioned in your (customer) subscription and you are responsible for their high availability setting. Some are created in a Microsoft subscription, and are managed by Microsoft.

* **Azure Machine Learning Infrastructure**: It is inside Microsoft subscription and it has Azure Machine Learning workspace and Azure Cosmos DB to store meta data. It is a high availability service with full disaster recovery capability and fully managed by Microsoft.

* **Associated Resources**: They are the resources provisioned in your subscription at Azure Machine Learning deployment. They includes default storage, key vault, Azure Container Registry(ACR) and App Insights. You are responsible for high availability setting.
  * Default storage has data such as model, training log data and dataset.
  * Key Valult has credentials for storage, ACR, data stores.
  * ACR has docker image for training and inferencing environment.
  * App Insights is for monitoring Azure Machine Learning.

* **Compute Resources**: They are the resources can be created after Azure Machine Learning workspace deployment.
  * Compute Instance: It is a single VM and does not have high availability setting.
  * Training Cluster: It is a high availability service with full disaster recovery capability and fully managed by Microsoft.
  * Other Resources: They are the computing resources can be attached to Azure Machine Learning such as AKS, ADB, ACI, HDI. You are responsible for high availability setting.

* **Additional Data Stores**: Azure Machine Learning can mount additional data stores such as storage, data lake storage, SQL for training data.  They are within your subscription and you are responsible for high availability setting.

The following table shows which, managed by Microsoft, which are managed by you, and which are highly available by default:

> [!IMPORTANT]
> If you provide your own key (customer-managed key) to deploy Azure Machine Learning workspace, Cosmos DB is also provisioned within your subscription. In that case, you are responsible for its high availability.

| Service | Managed by | HA by default | 
| ----- | ----- | ----- |
| **Azure Machine Learning Infrastructure** |
| Azure Machine Learning workspace | Microsoft | ✓ |
| Cosmos DB | Microsoft(*) | ✓ |
| **Associated Resources** |
| Azure Storage | You | |
| Azure Key Vault | You | ✓ |
| Azure Container Registry | You | |
| Application Insights | You | NA |
| **Compute Resources** |
| Compute Instance | Microsoft | NA |
| Compute Cluster | Microsoft | ✓ |
| Other resources such as Azure Kubernetes Service, <br>Azure Databricks, Azure Container Instance, Azure HDInsight | You | ✓ |
| **Additional Data Stores** such as Azure Storage, Azure SQL Database,<br> Azure Database for PostgreSQL, Azure Database for MySQL, <br>Azure Databricks file system | You | |

Use the information in the rest of this document to learn the actions You need to take to make each of these services highly available.

## Associated Resources(Storage, Key Vault, Container Registry, Application Insights)

> [!IMPORTANT]
> Azure Machine Learning does not support default storage account failover using Geo-redundant storage (GRS) or geo-zone-redundant storage (GZRS) or Read-access geo-redundant storage (RA-GRS) or read-access geo-zone-redundant storage (RA-GZRS).

Make sure high availability setting of each resource with below information.

* **Azure Storage**: To configure high availability setting, see [Azure Storage redundancy](https://docs.microsoft.com/azure/storage/common/storage-redundancy).
* **Azure Key Vault**: It provides default high availability service and no user action required.  See [Azure Key Vault availability and redundancy](https://docs.microsoft.com/azure/key-vault/general/disaster-recovery-guidance).
* **Azure Container Registry**: Choose Premium SKU for geo-replication. See [Geo-replication in Azure Container Registry](https://docs.microsoft.com/azure/container-registry/container-registry-geo-replication).
* **Application Insights**: It does not provide high availability setting. You can tweak data retention period and details in [Data collection, retention, and storage in Application Insights](https://docs.microsoft.com/azure/azure-monitor/app/data-retention-privacy#how-long-is-the-data-kept).

## Compute Resources(AKS, ADB, ACI, HDI)

Make sure high availability setting of each resource with below documentation.

* **Azure Kubernetes Service**: See [Best practices for business continuity and disaster recovery in Azure Kubernetes Service (AKS)](https://docs.microsoft.com/azure/aks/operator-best-practices-multi-region) and [Create an Azure Kubernetes Service (AKS) cluster that uses availability zones](https://docs.microsoft.com/azure/aks/availability-zones). In the case of AKS cluster Azure Machine Learning provisions on behalf of customer, corss-region high availability is not supported.
* **Azure Databricks**: See [Regional disaster recovery for Azure Databricks clusters](https://docs.microsoft.com/azure/azure-databricks/howto-regional-disaster-recovery).
* **Azure Container Instance**: ACI orchestrator is responsible for failover. See [Azure Container Instances and container orchestrators](https://docs.microsoft.com/azure/container-instances/container-instances-orchestrator-relationship).
* **Azure HDInsight**: See [High availability services supported by Azure HDInsight](https://docs.microsoft.com/azure/hdinsight/hdinsight-high-availability-components).

## Additional Data Stores(Storage, Data Lake, SQL, PostgreSQL, MySQL, ADB)

Make sure high availability setting of each resource with below documentation.

* **Azure Blob Container / Azure File Share / Azure Data Lake Gen2**: Same as default storage.
* **Azure Data Lake Gen1**: See[Disaster recovery guidance for data in Azure Data Lake Storage Gen1](https://docs.microsoft.com/azure/data-lake-store/data-lake-store-disaster-recovery-guidance).
* **Azure SQL Database**: See [High-availability and Azure SQL Database](https://docs.microsoft.com/azure/sql-database/sql-database-high-availability).
* **Azure Database for PostgreSQL**: See [High availability concepts in Azure Database for PostgreSQL - Single Server](https://docs.microsoft.com/azure/postgresql/concepts-high-availability).
* **Azure Database for MySQL**: See [Understand business continuity in Azure Database for MySQL](https://docs.microsoft.com/azure/mysql/concepts-business-continuity).
* **Databricks File System**: See [Regional disaster recovery for Azure Databricks clusters](https://docs.microsoft.com/azure/azure-databricks/howto-regional-disaster-recovery).

## Deploy Azure Machine Learning with high availability setting

Use [ARM template](https://github.com/Azure/azure-quickstart-templates/tree/master/101-machine-learning-create) with your preferred SKU parameters for default storage and Azure Container Registry.
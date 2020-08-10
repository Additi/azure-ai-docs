---
title: Allow access to indexer IP ranges
titleSuffix: Azure Cognitive Search
description: How to guide that describes how to setup IP firewall rules so that indexers can have access.

manager: nitinme
author: arv100kri
ms.author: arjagann
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 03/13/2020
---

# Setting up IP firewall rules to enable indexer access

Azure resources such as storage accounts, CosmosDB accounts and Azure SQL servers allow configuring a set of IP addresses as part of firewall rules. Only requests originating from these specified IP addresses will be allowed access to data in those resources.

This guide will describe how to configure the IP rules for a storage account so that indexers can have access to the data in the storage account.
While specific to storage account, this example can be directly translated for CosmosDB, Azure SQL server and other such services which also utilize IP firewall as a mechanism to offer network level security. Note that when using a storage account, it should be in a different region than the search service, but this restriction does not apply to other resources.

## Obtain the IP address of the search service

Obtain the fully qualified domain name (FQDN) of your search service, which is of the format `<search-service-name>.search.windows.net`. You can easily find this out by looking up your search service on the Azure portal.

   ![Obtain service FQDN](media\search-indexer-howto-secure-access\search-service-portal.PNG "Obtain service FQDN")

With the FQDN, there are several ways to get the IP address associated with it - the simplest option would be to utilize `nslookup`

```azurepowershell

nslookup contosos.search.windows.net
Server:  server.example.org
Address:  10.50.10.50

Non-authoritative answer:
Name:    <name>
Address:  <IP address of search service>
Aliases:  contoso.search.windows.net
```

Note down the IP address obtained above. Hypothetically, say it is 150.0.0.1.

## Get the IP address ranges for "AzureCognitiveSearch" service tag

The IP address ranges for the `AzureCognitiveSearch` service tag can be either obtained via the [discovery API (preview)](https://docs.microsoft.com/azure/virtual-network/service-tags-overview#use-the-service-tag-discovery-api-public-preview) or a [downloadable JSON file](https://docs.microsoft.com/azure/virtual-network/service-tags-overview#discover-service-tags-by-using-downloadable-json-files).

For this walkthrough, assuming the search service is the Azure Public cloud, the [Azure Public JSON file](https://www.microsoft.com/download/details.aspx?id=56519) should be downloaded.

   ![Download JSON file](media\search-indexer-howto-secure-access\service-tag.PNG "Download JSON file")

From that JSON file, assuming the search service is in West Central US, the following is the list of IP address ranges from which requests can originate, if the indexer is running in the multi-tenant environment.

```json
    {
      "name": "AzureCognitiveSearch.WestCentralUS",
      "id": "AzureCognitiveSearch.WestCentralUS",
      "properties": {
        "changeNumber": 1,
        "region": "westcentralus",
        "platform": "Azure",
        "systemService": "AzureCognitiveSearch",
        "addressPrefixes": [
          "52.150.139.0/26",
          "52.253.133.74/32"
        ]
      }
    }
```

For /32 IP addresses, drop the "/32" (52.253.133.74/32 -> 52.253.133.74), others can be used verbatin.

## Add the IP address ranges to IP firewall rules

The easiest way to add IP address ranges to a storage account's firewall rule is via the Azure portal. Locate the storage account on the portal and navigate to the "Firewalls and virtual networks" tab.

   ![Firewall and virtual networks](media\search-indexer-howto-secure-access\storage-firewall.PNG "Firewall and virtual networks")

Add the 3 IP addresses obtained previously (1 for the search service IP, 2 for the `AzureCognitiveSearch` service tag) in the address range and hit "Save"

   ![Firewall IP rules](media\search-indexer-howto-secure-access\storage-firewall-ip.PNG "Firewall IP rules")

The firewall rules take 5-10 minutes to get updated, and after that indexers will be able to access the data in this storage account.

## Next Steps

- [Configure Azure Storage firewalls](https://docs.microsoft.com/azure/storage/common/storage-network-security)
- [Configure IP firewall for CosmosDB](https://docs.microsoft.com/azure/cosmos-db/firewall-support)
- [Configure IP firewall for Azure SQL server](https://docs.microsoft.com/azure/sql-database/sql-database-firewall-configure)
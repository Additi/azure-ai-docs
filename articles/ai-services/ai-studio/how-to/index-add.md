---
title: How to create vector indexes
titleSuffix: Azure AI services
description: Learn how to create and use a vector index for performing Retrieval Augmented Generation (RAG)
services: cognitive-services
author: eric-urban
manager: nitinme
ms.service: azure-ai-services
ms.topic: how-to
ms.date: 10/1/2023
ms.author: eur
---

# How to create a vector index

In this article, you learn how to create and use a vector index for performing [Retrieval Augmented Generation (RAG)](../concepts/retrieval-augmented-generation.md).

## Prerequisites

You must have:
- An Azure AI project
- An Azure AI Search resource

## Create an index

1. Sign in to Azure AI Studio and open the Azure AI project in which you want to create the index.
1. From the collapsable menu on the left, select **Indexes** under **Components**.

    :::image type="content" source="../media/rag/project-left-menu.png" alt-text="Screenshot of Project Left Menu." lightbox="../media/rag/project-left-menu.png":::

1. Select **+ New index**
1. Choose your **Source data**. You can choose source data from a list of your recent data sources, a storage URL on the cloud or even upload files and folders from the local machine. You can also add a connection to another data source such as Azure Blob Storage.

    :::image type="content" source="../media/rag/select-source-data.png" alt-text="Screenshot of select source data." lightbox="../media/rag/select-source-data.png":::

1. Select **Next** after choosing source data
1. Choose the **Index Storage** - the location where you want your index to be stored
1. If you already have a connection created for an Azure AI Search service, you can choose that from the dropdown.

    :::image type="content" source="../media/rag/index-storage.png" alt-text="Screenshot of select index store." lightbox="../media/rag/index-storage.png":::

    1. If you don't have an existing connection choose **Connect other Azure AI Search service**
    1. Select the subscription and the service you wish to use.
    
    :::image type="content" source="../media/rag/index-store-details.png" alt-text="Screenshot of Select index store details." lightbox="../media/rag/index-store-details.png":::

1. Select **Next** after choosing index storage
1. Configure your **Search Settings**
    1. The search type defaults to **Hybrid + Semantic**, which is a combination of keyword search, vector search and semantic search to give the best possible search results.
    1. For the hybrid option to work, you need an embedding model. Choose the Azure OpenAI resource, which has the embedding model
    1. Select the acknowledgment to deploy an embedding model if it doesn't already exist in your resource
    
    :::image type="content" source="../media/rag/search-settings.png" alt-text="Screenshot of configure search settings." lightbox="../media/rag/search-settings.png":::

1. Use the prefilled name or type your own name for New Vector index name
1. Select **Next** after configuring search settings
1. In the **Index settings**
    1. Enter a name for your index or use the autopopulated name
    1. Choose the compute where you want to run the jobs to create the index. You can
        1. Auto select to allow Azure AI to choose an appropriate VM size that is available
        1. Choose a VM size from a list of recommended options
        1. Choose a VM size from a list of all possible options
        
    :::image type="content" source="../media/rag/index-settings.png" alt-text="Screenshot of configure index settings." lightbox="../media/rag/index-settings.png":::

1. Select **Next** after configuring index settings
1. Review the details you have entered and select **Create**
1. You're taken to the index details page where you can see the status of your index creation


## Use an index in prompt flow

1. Open your AI Studio project
1. In Flows, create a new Flow or open an existing flow 
1. On the top menu of the flow designer, select More tools, and then select Vector Index Lookup

    :::image type="content" source="../media/rag/vector-index-lookup.png" alt-text="Screenshot of Vector index Lookup from More Tools." lightbox="../media/rag/vector-index-lookup.png":::

1. Provide a name for your step and select **Add**.
1. The Vector Index Lookup tool is added to the canvas. If you don't see the tool immediately, scroll to the bottom of the canvas
1. Enter the path to your vector index, along with the query that you want to perform against the index.

    :::image type="content" source="../media/rag/configure-index-lookup.png" alt-text="Screenshot of Configure Vector index Lookup." lightbox="../media/rag/configure-index-lookup.png":::

## Next steps

- [Learn about vector stores](../concepts/vector-stores.md)

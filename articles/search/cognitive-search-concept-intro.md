---
title: Cognitive search for data extraction, natural language processing in Azure Search | Microsoft Docs
description: Data extraction, natural language processing (NLP) and image processing to create searchable content in Azure Search indexing using cognitive skills.
manager: cgronlun
author: HeidiSteen

ms.service: search
ms.devlang: NA
ms.topic: conceptual
ms.date: 05/04/2018
ms.author: heidist
---
# What is cognitive search?

Cognitive search, now in public preview, is a new extensible data extraction and enrichment pipeline in Azure Search. It uses AI powered algorithms to find latent information in non-text sources and unstructured text, transforming data into searchable content. 

New data extraction capabilities in cognitive search make content more searchable in the following ways:

+ Natural language processing - in the form of entity recognition, sentiment analysis, key phrase extraction, and language detection - bring AI-powered modeling that extracts information that can amplify a search experience.
+ Image processing can extract data from images, making scanned documents searchable through optical character recognition. You can also analyze photographs to identify faces or automatically create searchable tags.
+ Custom processing – create your own skills and custom classifiers, and plug them into the enrichment pipeline. 

At the heart of cognitive search is an extensible indexing pipeline powered by *cognitive skills* that enrich source documents through these various forms of processing, in route to a search index.

![Cognitive search pipeline diagram](./media/cognitive-search-intro/cogsearch-architecture.png "Cognitive Search pipeline overview")

## Pipeline components

At both ends of the pipeline, you have persisted data - source data stored in an Azure data source, and a searchable index in Azure Search. In between is a run-time process that moves data through a series of transformations, culminating in an index accessed via search requests through all query types supported by Azure Search. 

Underneath it all, the engine driving the pipeline is an Azure Search *indexer*. An indexer pulls data from supported sources, adds field mappings and logic, and pushes it into a search index that you've defined in advance. Transformations and enrichment are added through individual *skills*, combined into a *skillset* attached to an indexer. The remaining sections explore each step in more detail.

### Source data and document cracking phase

At the start of the pipeline, you have unstructured text or non-text content (such as image files, scanned document JPG files, audio files). Data must exist in an Azure data storage service that can be accessed by an indexer. Supported sources include Azure blob storage, Azure table storage, Azure SQL Database, and Azure Cosmos DB. Blobs can be image files, audio files, scanned documents, and so forth. Text-based content can be extracted from the following file types: PDFs, Word, PowerPoint, CSV files. For the full list, see [Supported formats](search-howto-indexing-azure-blob-storage.md#supported-document-formats).

### Cognitive skills and enrichment phase

Enrichment is implemented as *cognitive skills* that invoke atomic transformations. For example, once you have the textual representation of a PDF file, you could apply natural language processing in the form of an entity recognition skill, a language detection skill, or a key phrase extraction skill to break up undifferentiated text into semantically rich parts, consumable in search workloads. Altogether, the entire collection of skills used in your pipeline is called a *skillset*.  

Cognitive search provides [predefined cognitive skills](cognitive-search-predefined-skills.md) that can be consumed out of the box. The pipeline is also extensible. You can build custom skills from the ground up, and connect it as part of the skillset. For more information, see [Example: create a custom skill](cognitive-search-create-custom-skill-example.md) and [How to define a custom interface](cognitive-search-custom-skill-interface.md).

A skillset can be minimal or highly complex. A skillset determines not only the type of processing, but also the order of operations. A skillset plus the field mappings defined as part of an indexer determines the enrichment pipeline. For more information about pulling all of these pieces together, see [How to create a skillset](cognitive-search-defining-skillset.md).

### Enriched documents

Internally, the pipeline generates a collection of enriched documents. You can decide which parts of the enriched documents should be mapped to indexable fields in your search index. For example, if you applied the key phrases extraction and the entity recognition skills, then those new fields would become part of the enriched document, and they can be mapped to fields on your index.

### Search index and query-based access

When processing is finished, you have a search corpus consisting of enriched documents, fully text-searchable in Azure Search. Querying the index is how developers and users utilize the enriched content generated by the pipeline. 

The index is like any other you might create for Azure Search: you can supplement with custom analyzers, invoke fuzzy search queries, add filtered search, or experiment with scoring profiles to reshape the search results.

Indexes are generated from an index schema that defines the fields, attributes, and other constructs attached to a specific index, such as scoring profiles and synonym maps. Once an index is defined and populated, you can refresh it to pick up new and updated source documents. Enrichment steps are seamlessly integrated with the indexing workload; the same operations performed during initial data ingestion also occur in subsequent refresh operations.



<a name="feature-concepts"></a>

## Key features and concepts

| Concept | Description|
|---------|------------|
| Indexer |  A crawler that extracts searchable data and metadata from an external data source and populates an index based on field-to-field mappings between the index and your data source for document cracking. For cognitive search enrichments, the indexer invokes a skillset, and contains the field mappings associating enrichment output to target fields in the index. The indexer definition contains all of the instructions and references for pipeline operations, and the pipeline is invoked when you run the indexer. |
| Data Source  | An object used by an indexer to connect to an external data source of supported types on Azure. |
| Index | A persisted search corpus in Azure Search, built from an index schema that defines field structure and usage. |
| Document cracking | The process of extracting or creating text content from non-text sources. Optical character recognition (OCR) and audio-to-text translation are two examples. The data source and the indexer definition with field mappings are the key factors in document cracking. |
| Cognitive skill | An atomic transformation in an enrichment pipeline. Often, it is a component that extracts or infers structure, and therefore augments an understanding of the input data. Almost always, the output is text-based and the processing is natural language processing. Output can be mapped to a field in an index, or used as an input for a downstream enrichment. |
| Skillset | A top-level named resource containing a collection of skills. A skillset is the enrichment pipeline. |
| Enriched documents | A transitory internal structure, not directly accessible in code. Enriched documents are generated during processing, but only final outputs are persisted in a search index. Field mappings determine which data elements are added to the index. |

## Where do I start?

**Step 1: Create a search service in a region providing the APIs** 

+ South Central US
+ West Europe

**Step 2: Hands-on experience to master the workflow**

+ [Quickstart (portal)](cognitive-search-quickstart-blob.md)
+ [Tutorial (HTTP requests)](cognitive-search-tutorial-blob.md)
+ [Example custom skills (C#)](cognitive-search-create-custom-skill-example.md)

**Step 3: Review the API (REST only)**

Currently, only REST APIs are provided. Use `api-version=2017-11-11-Preview` on all requests. Use the following APIs to build a cognitive search solution. Only two APIs are added or extended for cognitive search. Other APIs have the same syntax as the generally available versions.

| REST API | Description |
|-----|-------------|
| [Create Data Source](https://docs.microsoft.com/rest/api/searchservice/create-data-source)  | A resource identifying an external data source providing source data used to create enriched documents.  |
| [Create Skillset (api-version=2017-11-11-Preview)](ref-create-skillset.md)  | A resource coordinating the use of [predefined skills](cognitive-search-predefined-skills.md) and [custom cognitive skills](cognitive-search-custom-skill-interface.md) used in an enrichment pipeline during indexing. |
| [Create Index](https://docs.microsoft.com/rest/api/searchservice/create-index)  | A schema expressing an Azure Search index. Fields in the index map to fields in source data or to fields manufactured during the enrichment phase (for example, a field for organization names created by entity recognition). |
| [Create Indexer (api-version=2017-11-11-Preview)](ref-create-skillset.md)  | A resource defining components used during indexing: including a data source, a skillset, field associations from source and intermediary data structures to target index, and the index itself. Running the indexer is the trigger for data ingestion and enrichment. The output is a search corpus based on the index schema, populated with source data, enriched through skillsets.  |
| [Reset Indexer](https://docs.microsoft.com/rest/api/searchservice/reset-indexer) | A command for rebuilding an index. Because pipeline development is an iterative process, plan for frequent index rebuilds.

**Checklist: A typical workflow**

1. Subset your Azure source data into a representative sample. Indexing takes time so start with a small, representative data set and then build it up incrementally as your solution matures.

1. Create a data source object in Azure Search to provide a connection string for data retrieval.

1. Create a skillset with enrichment steps.

1. Define the index schema. The *Fields* collection includes fields from source data. You should also stub out additional fields to hold generated values for content created during enrichment.

1. Define the indexer referencing the data source, skillset, and index.

1. Within the indexer, add *outputFieldMappings*. This section maps output from the skillset (in step 3) to the inputs fields in the index schema (in step 4).

1. Send *Create Indexer* (a POST request with an indexer definition in the request body) to create and run the indexer, invoking the pipeline.

1. Evaluate results and modify code to update skillsets, schema, or indexer configuration.

1. Reset the indexer before rebuilding the pipeline.

## Next steps

+ [Quickstart: Try cognitive search](cognitive-search-quickstart-blob.md)
+ [Tutorial: Enriched indexing of Azure blob content](cognitive-search-tutorial-blob.md)
+ [Example: creating a custom skill](cognitive-search-create-custom-skill-example.md)
+ [How to create a skillset or enrichment pipeline](cognitive-search-defining-skillset.md)
+ [How to define a custom interface](cognitive-search-custom-skill-interface.md)
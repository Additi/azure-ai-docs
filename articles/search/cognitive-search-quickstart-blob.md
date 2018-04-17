---
title: Quickstart cognitive search preview in Azure Search | Microsoft Docs
description: Learn how to use artifical intelligence AI-powered pretrained models for transforming unstructured data into searchable content during indexing. Build a cognitive search solution in Azure Search. 
manager: cgronlun
author: HeidiSteen
ms.service: search
ms.topic: quickstart
ms.date: 05/01/2018
ms.author: heidist
---
# Quickstart: Cognitive search preview

This article provides a roadmap for developers creating a cognitive search solution in Azure Search, including how to provision resources, learning the basic workflow, and testing your work.

This exercise creates multiple objects: an Azure blob data source, a skillset, an index. All objects are pulled together in an indexer definition, executing as an end-to-end pipeline that pulls data from its source, enriches the data, and deposits it into an Azure Search index.

## Prerequisites

To perform each step, you can use a REST test tool such as Telerik Fiddler or Postman to formulate HTTP REST calls. For guidance, see [Explore Azure Search REST APIs using Fiddler or Postman](https://docs.microsoft.com/azure/search/search-fiddler).

Use the [Azure portal](https://portal.azure.com/) to create services used in an end-to-end workflow. 

 ![Dashboard portal](./media/cognitive-search-get-start-preview/create-service-full-portal.png)

## 1 - Set up Azure Search

First, sign up for the Azure Search service.

1. Go to the [Azure portal]() and sign in by using your Azure account.

1. Click **Create a resource**, search for Azure Search, and click **Create**. See [Create a service in the portal](search-create-service-portal.md) if you are setting up a search service for the first time.

1. For Resource group, create a resource group for all resources you create today to make cleanup easier.

1. For Location, choose either **South Central US** or **West Europe**. Currently, the preview is available only in those regions.

1. For Pricing tier, you can create a **Free** service to complete tutorials and quickstarts. For deeper investigation using your own data, create a [paid service](https://azure.microsoft.com/pricing/details/search/) such as **Basic** or **Standard**. 

  A Free service is limited to 3 indexes, 16 MB maximum blob size, and 2 minutes of indexing, which is insufficient for exercising the full capabilities of cognitive search. To review limits for different tiers, see [Service Limits](https://docs.microsoft.com/azure/search/search-limits-quotas-capacity).

1. Pin the service to the dashboard for fast access to service information.

  ![Service definition page in the portal](./media/cognitive-search-get-start-preview/create-search-service.png)

After the service is created, collect the following information once the search service is created: "endpoint", "api-key". You can use either the primary or secondary key.

  ![Endpoint and key information in the portal](./media/cognitive-search-get-start-preview/create-search-collect-info.png)

## 2 - Set up Azure Blob service and load sample data

The enrichment pipeline pulls from Azure data sources. Source data must originate from a [supported data source type](https://docs.microsoft.com/azure/search/search-indexer-overview). For this exercise, we use blob storage to showcase multiple content types.

1. [Download sample data](https://1drv.ms/f/s!As7Oy81M_gVPa-LCb5lC_3hbS-4). Sample data consists of a very small file set of different types. 

2. [Sign up for Azure Blob storage](https://docs.microsoft.com/azure/storage/blobs/storage-quickstart-blobs-storage-explorer) and upload sample files. Create a storage account, log in to Storage Explorer, create a collection, and upload the sample files.

3. Still in the portal, get the connection string for your storage account. On **Settings** > **Access keys**, copy the **Connection String**  field.

You should see a URL similar to the following example:

```
DefaultEndpointsProtocol=https;AccountName=cogsrchdemostorage;AccountKey=y5NIlE4wFVBIyrCi392GzZl+JO4TEGdqOerqfbT79C8zrn28Te8DsWlxvKKnjh69P/HM5k50ztz2shOt8vqlbg==;EndpointSuffix=core.windows.net
```

There are several ways to specify the connection string, for instance you could provide a shared access signature instead. To learn more about data source credentials, see [Indexing Azure Blob Storage](https://docs.microsoft.com/azure/search/search-howto-indexing-azure-blob-storage).

## 3 - Create a data source

Now that your services and source files are prepared, create the [data source object](https://docs.microsoft.com/rest/api/searchservice/create-data-source) used by Azure Search to retrieve source data. The data source is a resource in Azure Search service.

Azure Search provides a REST API. Use Postman or Fiddler for formulating and submitting HTTP requests. In the request header, provide the service name and api-key generated for your search service. In the request body, specify the blob container name and shared access signature.

### Sample Request
```http
POST https://[service name].search.windows.net/datasources?api-version=2017-11-11-Preview
Content-Type: application/json  
api-key: [admin key]  
```
#### Request Body Syntax
```json
{   
    "name" : "demodata",  
    "description" : "Demo files to demonstrate cognitive search capabilities.",  
    "type" : "azureblob",
    "credentials" :
    { "connectionString" :
      "DefaultEndpointsProtocol=https;=<your account name>;AccountName=<your account name>;AccountKey=<your account key>;"
    },  
    "container" : { "name" : "basicdemo" }
}  
```

For the reference documentation, see [Create Data Source (REST API)](https://docs.microsoft.com/rest/api/searchservice/create-data-source).


## 4 - Create a skillset

In this step, define a set of enrichment steps that you want to apply to your data. We call each enrichment step a *skill*, and the set of enrichment steps a *skillset*. You can use [built-in cognitive skills](cognitive-search-predefined-skills.md) or create custom skills and hook them up to the enrichment pipeline.

For this exercise, start with predefined skills, defining four enrichment steps to run on your data:

1. Extract the names of organizations from the content of documents in the blob container using the [named entity recognition skill](cognitive-search-skill-named-entity-recognition.md). 

2. Identify the language of a document using the [language detection skill](cognitive-search-skill-language-detection.md).

3. Extract the top key phrases from the document using the [key phrase extraction skill](cognitive-search-skill-keyphrases.md). Use the language detected in the previous step as an input. 

4. Since the key phrase skill only works with text that is at most 5,000 characters, use the [pagination skill](cognitive-search-skill-pagination.md) to break the content into several pages first before calling the key phrase extraction skill.

Each step executes on the content of the document. During processing, Azure Search cracks each document to read content from different file formats. Found text originating in the source file is placed into a generated ```content``` field, one for each document. As such, you can set the input as ```"/document/content"```.

A graphical representation of the skillset is shown below:

![](media/cognitive-search-get-start-preview/skillset.png)

### Sample Request
Make sure to replace the service name and the admin key in the request below. Reference the skillset name ```demoskillset``` for the rest of this demo.

```http
PUT https://[servicename].search.windows.net/skillsets/demoskillset?api-version=2017-11-11-Preview
api-key: [admin key]
Content-Type: application/json
```
#### Request Body Syntax
```json
{
  "description": 
  "Extract entities, detect language and extract key-phrases",
  "skills":
  [
    {
      "@odata.type": "#Microsoft.Skills.Text.NamedEntityRecognitionSkill",
      "categories": [ "Organization" ],
      "defaultLanguageCode": "en",
      "inputs": [
        {
          "name": "text", "source": "/document/content"
        }
      ],
      "outputs": [
        {
          "name": "organizations", "targetName": "organizations"
        }
      ]
    },
    {
      "@odata.type": "#Microsoft.Skills.Text.LanguageDetectionSkill",
      "inputs": [
        {
          "name": "text", "source": "/document/content"
        }
      ],
      "outputs": [
        {
          "name": "languageCode",
          "targetName": "languageCode"
        }
      ]
    },
    {
      "@odata.type": "#Microsoft.Skills.Text.PaginationSkill",
      "maximumPageLength": 4000,
      "inputs": [
      {
        "name": "text",
        "source": "/document/content"
      },
      { 
        "name": "languageCode",
        "source": "/document/languageCode"
      }
    ],
    "outputs": [
      {
        "name": "pages",
        "targetName": "pages"
      }
    ]
  },
  {
      "@odata.type": "#Microsoft.Skills.Text.KeyPhraseExtractionSkill",
      "context": "/document/pages/*",
      "inputs": [
        {
          "name": "text", "source": "/document/pages/*"
        },
        {
          "name":"languageCode", "source": "/document/languageCode"
        }
      ],
      "outputs": [
        {
          "name": "keyPhrases",
          "targetName": "keyPhrases"
        }
      ]
    }
  ]
}
```
Notice how the key phrase extraction skill is applied for each page. By setting the context to ```"document/pages/*"``` you run this enricher for each member of the document/pages array (for each page in the document).

For more information about skillset fundamentals, see [How to define a skillset](cognitive-search-defining-skillset.md).

## 5 - Create an index

Now let's define what fields to include in the searchable index, and the search attributes for each field. Fields have a type and can take attributes that determine how the field is used (searchable, sortable, and so forth). Field names in an index are not required to identically match the field names in the source. In a later step, you add field mappings in an indexer to connect source-destination fields. For this step, define the index using whatever field naming conventions make sense for your search application.

This exercise uses the following fields and field types:

| field-names: | id       | content   | language | keyphrases         | organizations     |
|--------------|----------|-------|----------|--------------------|-------------------|
| field-types: | Edm.String|Edm.String| Edm.String| List<Edm.String>  | List<Edm.String>  |


### Sample Request
Make sure to replace the service name and the admin key in the request below.
Recall that ```demoindex``` is the index name in this exercise.

```http
PUT https://[servicename].search.windows.net/indexes/demoindex?api-version=2017-11-11-Preview
api-key: [api-key]
Content-Type: application/json
```
#### Request Body Syntax

```json
{
  "fields": [
    {
      "name": "id",
      "type": "Edm.String",
      "key": true,
      "searchable": true,
      "filterable": false,
      "facetable": false,
      "sortable": true
    },
    {
      "name": "content",
      "type": "Edm.String",
      "sortable": false,
      "searchable": true,
      "filterable": false,
      "facetable": false
    },
    {
      "name": "language",
      "type": "Edm.String",
      "searchable": true,
      "filterable": false,
      "facetable": false
    },
    {
      "name": "keyphrases",
      "type": "Collection(Edm.String)",
      "searchable": true,
      "filterable": false,
      "facetable": false
    },
    {
      "name": "organizations",
      "type": "Collection(Edm.String)",
      "searchable": true,
      "sortable": false,
      "filterable": false,
      "facetable": false
    }
  ]
}
```

To learn more about defining an index, see [Create an Azure Search Index](ref-create-index.md).


## 6 - Create an indexer, map fields, and execute transformations

So far, you have created a data source, a skillset, and an index. All become part of an [indexer](search-indexer-overview.md) that pulls each piece together into a single multi-phased operation. That said, you need to add a bit of glue between these components before you can run it. In this step, you define field mappings, which are part of the indexer definition, and execute the transformations when you submit the request.

For non-enriched indexing, the indexer definition provides an optional *fieldMappings* section if field names or data types do not precisely match, or if you want to use a function.

For cognitive search workloads having an enrichment pipeline, an indexer requires *outputFieldMappings*. These mappings are used when an internal process (the enrichment pipeline) is the source of field values. Behaviors unique to *outputFieldMappings* include the ability to handle complex types created as part of enrichment (through the shaper skill). Also, there may be many elements per document (for instance, multiple organizations in a document). The *outputFieldMappings* construct can direct the system to "flatten" collections of elements into a single record.

### Sample Request
Make sure to replace the service name and the admin key in the request below.
Also, provide the name of your indexer. You can reference it as ```demoindexer``` for the rest of this exercise.

```
PUT https://[servicename].search.windows.net/indexers/demoindexer?api-version=2017-11-11-Preview
api-key: [api-key]
Content-Type: application/json
```
#### Request Body Syntax

```json
{
  "name":"demoindexer",	
  "dataSourceName" : "demodata",
  "targetIndexName" : "demoindex",
  "skillsetName" : "demoskillset",
  "fieldMappings" : [
        {
          "sourceFieldName" : "metadata_storage_path",
          "targetFieldName" : "id",
          "mappingFunction" : { "name" : "base64Encode" }
        },
        {
          "sourceFieldName" : "content",
          "targetFieldName" : "content"
        }
   ],
  "outputFieldMappings" : 
  [
        {
          "sourceFieldName" : "/document/organizations", 
          "targetFieldName" : "organizations"
        },
        {
          "sourceFieldName" : "/document/pages/*/keyPhrases/*", 
          "targetFieldName" : "keyphrases"
        },
        {
            "sourceFieldName": "/document/languageCode",
            "targetFieldName": "language"
        }      
  ],
  "parameters":
  {
  	"maxFailedItems":-1,
  	"maxFailedItemsPerBatch":-1,
  	"configuration": 
    {
    	"dataToExtract": "contentAndMetadata",
     	"imageAction": "embedTextInContentField"
		}
  }
}

```
Notice that ```"maxFailedItems"``` is set to -1, which instructs the indexing engine to ignore errors during data import. This is useful because there are so few documents in the demo data source. For a larger data source, you would set the value to greater than 0.

Also notice the ```"dataToExtract":"contentAndMetadata"``` statement in the configuration parameters. This statement tells the indexer to automatically extract the content from different file formats as well as metadata related to each file. 

When content is extracted, you can set ```ImageAction``` to extract text from images found in the data source. The ```"ImageAction":"embedTextInContentField"``` tells the indexer to extract text from the images (for example, the word "stop" from a traffic Stop sign), and embed it as part of the content field. This behavior applies to both the images embedded in the documents (think of an image inside a PDF), as well as images found in the data source, for instance a JPG file.

In this preview, ```"embedTextInContentField"``` is the only valid value for ```"ImageAction"```.

### Check indexer status

Once the indexer is defined, it runs automatically when you submit the request. Send the following request to check the indexer status.

```http
GET https://[servicename].search.windows.net/indexers/demoindexer/status?api-version=2017-11-11-Preview
api-key: [api-key]
Content-Type: application/json
```

The response tells you whether the indexer is running. After indexing is finished, GET STATUS reports any errors and warnings that occurred during enrichment.  
 
## 7 - Verify content

To check your work, run queries that return the contents of individual fields. By default, Azure Search returns the top 50 results. The sample data is small so the defaults work fine. However, when working with larger data sets, you might need to include parameters in the query string to return more results. For instructions, see [How to page results in Azure Search](https://docs.microsoft.com/azure/search/search-pagination-page-layout).

As a verification step, query for "*" to return all contents of a single field.

```http
GET https://[servicename].search.windows.net/indexes/demoindex/docs?search=*&api-version=2017-11-11-Preview
api-key: [api-key]
Content-Type: application/json
```

The syntax can also be scoped to a single field: content, language, keyphrases, and organizations in this exercise.

You can use GET or POST, depending on query string complexity and length. For more information, see [Query using the REST API](https://docs.microsoft.com/azure/search/search-query-rest-api).

## Testing the user experience

Run representative queries to determine whether additional features are necessary for improving the user experience. Depending on the change, you might need to rebuild the index by rerunning the indexer. For more information and examples, see [Search Documents](https://docs.microsoft.com/rest/api/searchservice/search-documents).


## Accessing the enriched document

*For the private preview*, we added a mechanism that allows you to see the structure of the enriched document. Enriched documents are temporary structures created during enrichment, and then deleted when the process is complete.

To capture an enriched document created during indexing, you can add a field called ```enriched``` to your index. The indexer automatically dumps into it a string representation of all the enrichments for that document.

The enriched field will contain a string that is a logical representation of the in memory enriched document in json.  The field value is a valid json document, however, quotes are escaped so you'll need to replace \" with " in order to view the document as formatted json.  The enriched field is intended for debugging purposes only to help you understand the logical shape of the content that expressions are being evaluated against.

This implementation is temporary, likely to be replaced by public preview or general release. For now, it can be a useful tool to understand what's going on and help you debug your skillset.

Repeat the previous exercise, including an `enriched` field to capture the contents of an enriched document:

#### Request Body Syntax
```json
{
  "fields": [
    {
      "name": "id",
      "type": "Edm.String",
      "key": true,
      "searchable": true,
      "filterable": false,
      "facetable": false,
      "sortable": true
    },
    {
      "name": "content",
      "type": "Edm.String",
      "sortable": false,
      "searchable": true,
      "filterable": false,
      "facetable": false
    },
    {
      "name": "language",
      "type": "Edm.String",
      "searchable": true,
      "filterable": false,
      "facetable": false
    },
    {
      "name": "keyphrases",
      "type": "Collection(Edm.String)",
      "searchable": true,
      "filterable": false,
      "facetable": false
    },
    {
      "name": "organizations",
      "type": "Collection(Edm.String)",
      "searchable": true,
      "sortable": false,
      "filterable": false,
      "facetable": false
    },
    {
      "name": "enriched",
      "type": "Edm.String",
      "searchable": false,
      "sortable": false,
      "filterable": false,
      "facetable": false
    }
  ]
}
```

## Creating custom skills

This exercise does not walk you through custom skill definition, but if you plan to create a custom skill at some point, or if you want to step through the [custom skill example](cognitive-search-create-custom-skill-example.md), one approach for providing the skill is using an [Azure Function](https://docs.microsoft.com/azure/azure-functions/functions-overview). Use of an Azure Function comes at an additional cost. See the [pricing page](https://docs.microsoft.com/en-us/azure/azure-functions/functions-overview#pricing) for details.

## Clean up

## Next steps

+ [How to map fields into your index](cognitive-search-output-field-mapping.md)
+ [How to create a skillset or augmentation pipeline](cognitive-search-defining-skillset.md)
+ [Predefined skills](cognitive-search-predefined-skills.md)
+ [How to define a custom interface](cognitive-search-custom-skill-interface.md)
+ [Example: creating a custom skill](cognitive-search-create-custom-skill-example.md)
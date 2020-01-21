---
title: Examples of projections in a knowledge store (preview)
titleSuffix: Azure Cognitive Search
description: Examples of common patterns on how to project enriched documents into the knowledge store for use.

manager: eladz
author: vkurpad
ms.author: vikurpad
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 01/15/2020
---

# Examples of projections in a knowledge store in Azure Cognitive Search

> [!IMPORTANT] 
> Knowledge store is currently in public preview. Preview functionality is provided without a service level agreement, and is not recommended for production workloads. For more information, see [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). 
> The [REST API version 2019-05-06-Preview](search-api-preview.md) provides preview features. There is currently limited portal support, and no .NET SDK support.

Projections enable you to use your enriched documents in scenarios other than search. If you are new to the knowledge store or projections, start [here](knowledge-store-concept-intro.md).

Projections allow you to export your enriched document into the knowledge store. Knowledge store projections support three types of projections
1. Tables
2. Objects
3. Files

As the knowledge store is an Azure Storage account, table projections are Azure Storage tables and are governed by the storage limits on tables, for more information, see [table storage limits](https://docs.microsoft.com/en-us/rest/api/storageservices/understanding-the-table-service-data-model). It is useful to know that the entity size cannot exceed 1 MB and a single property can be no bigger than 64 KB. These constraints make tables a good solution for storing a large number of small entities. 

Object and file projections are written to blob storage, object projections are saved as JSON files and can contain content from the document and any enrichments. The enrichment pipeline can also extract binaries like images, these binaries are projected as file projections. When a binary object is projected as an object projection, only the metadata associated with it is saved as a JSON blob. 

To learn how you work with projections, let's start with a few example scenarios. This tutorial assumes you're familiar with the enrichment process. skillsets](cognitive-search-working-with-skillsets.md). Projections are defined in the knowledge store object of the skillset, see [knowledge store](knowledge-store-concept-intro.md] for details. For all the scenarios, we will work with a sample skillset that you can use the `import data wizard` to generate. 

```json
{
    "name": "azureblob-skillset",
    "description": "Skillset created from the portal. skillsetName: azureblob-skillset; contentField: merged_content; enrichmentGranularity: document; knowledgeStoreStorageAccount: confdemo;",
    "skills": [
        {
            "@odata.type": "#Microsoft.Skills.Text.EntityRecognitionSkill",
            "name": "#1",
            "description": null,
            "context": "/document/merged_content",
            "categories": [
                "Person",
                "Quantity",
                "Organization",
                "URL",
                "Email",
                "Location",
                "DateTime"
            ],
            "defaultLanguageCode": "en",
            "minimumPrecision": null,
            "includeTypelessEntities": null,
            "inputs": [
                {
                    "name": "text",
                    "source": "/document/merged_content"
                },
                {
                    "name": "languageCode",
                    "source": "/document/language"
                }
            ],
            "outputs": [
                {
                    "name": "persons",
                    "targetName": "people"
                },
                {
                    "name": "organizations",
                    "targetName": "organizations"
                },
                {
                    "name": "locations",
                    "targetName": "locations"
                },
                {
                    "name": "entities",
                    "targetName": "entities"
                }
            ]
        },
        {
            "@odata.type": "#Microsoft.Skills.Text.KeyPhraseExtractionSkill",
            "name": "#2",
            "description": null,
            "context": "/document/merged_content",
            "defaultLanguageCode": "en",
            "maxKeyPhraseCount": null,
            "inputs": [
                {
                    "name": "text",
                    "source": "/document/merged_content"
                },
                {
                    "name": "languageCode",
                    "source": "/document/language"
                }
            ],
            "outputs": [
                {
                    "name": "keyPhrases",
                    "targetName": "keyphrases"
                }
            ]
        },
        {
            "@odata.type": "#Microsoft.Skills.Text.LanguageDetectionSkill",
            "name": "#3",
            "description": null,
            "context": "/document",
            "inputs": [
                {
                    "name": "text",
                    "source": "/document/merged_content"
                }
            ],
            "outputs": [
                {
                    "name": "languageCode",
                    "targetName": "language"
                }
            ]
        },
        {
            "@odata.type": "#Microsoft.Skills.Text.MergeSkill",
            "name": "#4",
            "description": null,
            "context": "/document",
            "insertPreTag": " ",
            "insertPostTag": " ",
            "inputs": [
                {
                    "name": "text",
                    "source": "/document/content"
                },
                {
                    "name": "itemsToInsert",
                    "source": "/document/normalized_images/*/text"
                },
                {
                    "name": "offsets",
                    "source": "/document/normalized_images/*/contentOffset"
                }
            ],
            "outputs": [
                {
                    "name": "mergedText",
                    "targetName": "merged_content"
                }
            ]
        },
        {
            "@odata.type": "#Microsoft.Skills.Vision.OcrSkill",
            "name": "#5",
            "description": null,
            "context": "/document/normalized_images/*",
            "textExtractionAlgorithm": "printed",
            "lineEnding": "Space",
            "defaultLanguageCode": "en",
            "detectOrientation": true,
            "inputs": [
                {
                    "name": "image",
                    "source": "/document/normalized_images/*"
                }
            ],
            "outputs": [
                {
                    "name": "text",
                    "targetName": "text"
                },
                {
                    "name": "layoutText",
                    "targetName": "layoutText"
                }
            ]
        }
    ],
    "cognitiveServices": {
        "@odata.type": "#Microsoft.Azure.Search.CognitiveServicesByKey",
        "description": "DemosCS",
        "key": "<COGNITIVE SERVICES KEY>"
    },
    "knowledgeStore": null
}
```

We can now add the knowledgeStore object for each of the scenarios as needed. To create a projection, you either use a shaper skill to create a custom object that contains all the data you intend to project, or you use the inline shaping syntax for inputs to define the projections. For this scenario we will demonstrate both options, you can choose to use either of the options for all other projections.

## Projecting to Tables for Power BI

Power BI can read from tables and discover relationships based on the keys that the knowledge store projections create, this makes tables a good option to project data when you're trying to build a dashboard on your enriched data. Assuming we're trying to build a dashboard where we can visualize the key phrases extracted from documents as a word cloud, we can add a shaper skill to the skillset to create a custom shape that has the document-specific details and key phrases. Add the shaper skill to the skillset to create a new enrichment called ```pbiShape``` on the ```document```.

```json
{
            "@odata.type": "#Microsoft.Skills.Util.ShaperSkill",
            "name": "ShaperForTables",
            "description": null,
            "context": "/document",
            "inputs": [
                {
                    "name": "metadata_storage_content_type",
                    "source": "/document/metadata_storage_content_type",
                    "sourceContext": null,
                    "inputs": []
                },
                {
                    "name": "metadata_storage_name",
                    "source": "/document/metadata_storage_name",
                    "sourceContext": null,
                    "inputs": []
                },
                {
                    "name": "metadata_storage_path",
                    "source": "/document/metadata_storage_path",
                    "sourceContext": null,
                    "inputs": []
                },
                {
                    "name": "metadata_content_type",
                    "source": "/document/metadata_content_type",
                    "sourceContext": null,
                    "inputs": []
                },
                {
                    "name": "keyPhrases",
                    "source": "/document/merged_content/keyphrases/*",
                    "sourceContext": null,
                    "inputs": []
                }
            ],
            "outputs": [
                {
                    "name": "output",
                    "targetName": "pbiShape"
                }
            ]
        }
```
Now that we have all the data needed to project to tables we can update the knowledgeStore object with the table definitions.

Start by setting the knowledgeStore property to the object 
```json
{
        "storageConnectionString": "DefaultEndpointsProtocol=https;AccountName=<Acct Name>;AccountKey=<Acct Key>;",
        "projections": [
            {
                "tables": [
                    {
                        "tableName": "pbiDocument",
                        "generatedKeyName": "Documentid",
                        "source": "/document/pbiShape"
                    },
                    {
                        "tableName": "pbiKeyPhrases",
                        "generatedKeyName": "KeyPhraseid",
                         "source": null,
                        "sourceContext": "/document/pbiShape/keyPhrases/*",
                        "inputs": [
                            {
                                "name": "keyphrases",
                                "source": "/document/pbiShape/keyPhrases/*",
                                "sourceContext": null,
                                "inputs": []
                            }
                        ]

                    }
                ],
                "objects": [],
                "files": []
            }
        ]
    }
```

Set the ```storageConnectionString``` property to a valid storage account connection string. Now we define two tables in the projection object, note that each table requires a ```tableName```, ```source``` and ```generatedKeyName``` property, the ```referenceKeyName``` property is optional. 

### Naming relationships
The generatedKeyName and refereceKeyName properties are used to relate data across tables. Each row in the child table has a property pointing back to the parent row. The name of the column in the child table is the referenceKeyName from the parent table. When the referenceKeyNme is not provided, the service will use the  generatedKeyName from the parent table. PowerBI relies on the generated key name and reference key name to be the same to discover relationships within the tables. 

### Slicing 

When starting with a consolidated shape where all the content that needs to projected is in a single shape, slicing provides you with the ability to slice a single node into multiple tables or objects. In this case the ```pbiShape``` object is sliced into multiple tables. The slicing feature enables you to pull out a part of the shape, ```keyPhrases``` here into a separate table. Slicing generates a relationship between the two tables, using the ```generatedKeyName``` in the parent table to create a column with the same name in the child table. If you need the column in the child table named differently, set the ```referenceKeyName``` property on the child table.

You now have a working projection with two tables that when imported into Power BI should auto discover the relationships and allow you to filter.

### Shaping Enrichments

Source paths for enrichments are required to be well formed JSON objects, which is not always the case in the enrichment tree, in this instance enriching a string with key phrases results in the key phrases being parented to the string merged_content. The projection sample uses inline shaping to create a named value that can be projected. 

## Projecting to multiple types

Sometimes you might need to project content across projection types. For example, if you need to save the OCR results of text and layout text in addition to the table projections, object projections would be a better option for this data. Let's now modify the projection object in the knowledge store to add an object projection definition for the text and layout text.

```json
{
        "storageConnectionString": "DefaultEndpointsProtocol=https;AccountName=<Acct Name>;AccountKey=<Acct Key>;",
        "projections": [
            {
                "tables": [
                    {
                        "tableName": "sampleDocument",
                        "generatedKeyName": "Documentid",
                        "source": "/document/pbiShape"
                    },
                    {
                        "tableName": "sampleKeyPhrases",
                        "generatedKeyName": "KeyPhraseid",
                        "source": "/document/pbiShape/keyPhrases"
                    }
                ],
                "objects": [
                    {
                        "storageContainer": "sampleocrtext",
                        "source": "/document/normalized_images/*/text"
                    },
                    {
                        "storageContainer": "sampleocrlayout",
                        "source": "/document/normalized_images/*/layoutText"
                    }
                ],
                "files": []
            }
        ]
    }
```

Object projections require a container name for each projection, multiple object projections or file projections cannot share containers. Notice that the `generatedKeyName` and `referenceKeyName` properties are optional and when not provided they are auto generated.

### Relationships

This also highlights another feature of projections, by defining multiple types of projections within the same projection object, there is a relationship expressed within and acoss the different types (tables, objects, files)of projections, allowing you to start with a table row for a document and find all the OCR text for the images within that document in the object projection. If you do not want the data related, define the projections in different projection objects, for example the following snippet will result in no relationship between the document table and the OCR text projections. Projection groups are useful when you want to project the same data in different shapes for different needs. For example, a projection group for the Power BI dashboard and another projection group for using the data to train a model.

```json
{
        "storageConnectionString": "DefaultEndpointsProtocol=https;AccountName=<Acct Name>;AccountKey=<Acct Key>;",
        "projections": [
            {
                "tables": [
                    {
                        "tableName": "sampleDocument",
                        "generatedKeyName": "Documentid",
                        "source": "/document/pbiShape"
                    },
                    {
                        "tableName": "sampleKeyPhrases",
                        "generatedKeyName": "KeyPhraseid",
                        "source": "/document/pbiShape/keyPhrases"
                    }
                ],
                "objects": [
                    
                ],
                "files": []
            }, 
            {
                "tables": [],
                "objects": [
                    {
                        "storageContainer": "unrelatedocrtext",
                        "source": "/document/normalized_images/*/text"
                    },
                    {
                        "storageContainer": "unrelatedcrlayout",
                        "source": "/document/normalized_images/*/layoutText"
                    }
                ],
                "files": []
            }
        ]
    }
```
### Projections with inline shaping and multiple types

Continuing with the earlier scenario, if we now want to add the images as well to the projections and ensure that all the data is related, the knowledgeStore object will be edited to

```json
{
        "storageConnectionString": "DefaultEndpointsProtocol=https;AccountName=<Acct Name>;AccountKey=<Acct Key>;",
        "projections": [
            {
                "tables": [
                    {
                        "tableName": "inlineDocument",
                        "generatedKeyName": "Id",
                        "source": null,
                        "sourceContext": "/document",
                        "inputs": [
                            {
                                "name": "inlinePath",
                                "source": "/document/metadata_storage_path"
                            },
                            {
                                "name": "inlineContent",
                                "source": "/document/Content"
                            },
                            
                        ]
                    },
                    {
                        "tableName": "inlineKeyPhrases",
                        "generatedKeyName": "Id",
                        "referenceKeyName": "documentId",
                        "source": null,
                        "sourceContext": "/document/merged_content/keyphrases/*",
                        "inputs": [
                            {
                                "name": "keyphrases",
                                "source": "/document/merged_content/keyphrases/*" 
                            }
                        ]
                    }
                ],
                "objects": [
                    {
                        "storageContainer": "inlineocrtext",
                        "referenceKeyName": "documentId",
                        "source": "/document/normalized_images/*/text"
                    },
                    {
                        "storageContainer": "inlineocrlayout",
                        "referenceKeyName": "documentId",
                        "source": "/document/normalized_images/*/layoutText"
                    }
                ],
                "files": [
                    {
                        "storageContainer": "inlineimages",
                        "source": "/document/normalized_images/*"
                    }
                ]
            }
        ]
    }
```

## Common Issues

When defining a projection, there are a few common issues that can cause unanticipated results.

1. Not shaping string enrichments. When strings are enriched, for example ```merged_content``` is a string and enriched with key phrases, the enriched property is represented as a child of merged_content within the enrichment tree. But at projection time, this needs to be transformed to a valid JSON object with a name and a value.
2. Omitting the ```/*``` at the end of a source path. If for example, the source of a projection is ```/document/pbiShape/keyPhrases``` the key phrases array is projected as a single object/row. Setting the source path to ```/document/pbiShape/keyPhrases/*``` yeilds a single row or object for each of the key phrases.


---
title: Image Analysis cognitive search skill (Azure Search) | Microsoft Docs
description: Extract semantic text through image analysis using the ImageAnalysis cognitive skill in an Azure Search augmentation pipeline.
services: search
manager: pablocas
author: luiscabrer
documentationcenter: ''

ms.assetid: 
ms.service: search
ms.devlang: NA
ms.workload: search
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.date: 05/01/2018
ms.author: luisca
---
#	Image Analysis cognitive skill

The **Image Analysis** skill extracts a rich set of visual features based on the image content. For instance, you can generate a caption from an image, generate tags, or identify celebrities and landmarks.

## @odata.type  
Microsoft.Skills.Vision.ImageAnalysisSkill 

## Skill parameters

Parameters are case-sensitive.

| Parameter name	 | Description |
|--------------------|-------------|
| defaultLanguageCode	|  A string indicating which language to return. The service returns recognition results in a specified language. If this parameter is not specified, the default value is "en". <br/><br/>Supported languages: <br/>*en* - English, Default <br/> *zh* - Simplified Chinese|
|visualFeatures |	An array of strings indicating what visual feature types to return.  <br/><br/> Valid visual feature types include:  <br/> *categories* - categorizes image content according to a taxonomy defined in the Cognitive Services [documentation](https://docs.microsoft.com/azure/cognitive-services/computer-vision/category-taxonomy). <br/> *tags* - tags the image with a detailed list of words related to the image content. <br/>	*Description* - describes the image content with a complete English sentence. <br/>	*Faces* - detects if faces are present. If present, generate coordinates, gender and age.<br/>	*ImageType* - detects if image is clipart or a line drawing.<br/>	*Color* - determines the accent color, dominant color, and whether an image is black&white. <br/>*Adult* - detects if the image is pornographic in nature (depicts nudity or a sex act). Sexually suggestive content is also detected.<br/><br/> Names of visual features are case sensitive.|
| details	| An array of strings indicating which domain-specific details to return.  <br/><br/> Valid visual feature types include: <br/> *Celebrities* - identifies celebrities if detected in the image. <br/> *Landmarks* - identifies landmarks if detected in the image.
 |

## Skill inputs

| Inputs	 | Description |
|--------------------|-------------|
| url | Unique locator for the image. It could be a web URL or the location of blob storage.|



##	Sample definition

```json
{
    "@odata.type": "#Microsoft.Skills.Vision.ImageAnalysisSkill",
    "visualFeatures": [
        "Tags",
        "Faces",
        "Categories",
        "Adult",
        "Description",
        "ImageType",
        "Color"
    ],
    "defaultLanguageCode": "en",
    "inputs": [
        {
            "name": "url",
            "source": "/document/metadata_storage_path"
        }
    ],
    "outputs": [
        {
            "name": "categories",
            "targetName": "myCategories"
        },
        {
            "name": "tags",
            "targetName": "myTags"
        },
        {
            "name": "description",
            "targetName": "myDescription"
        },
        {
            "name": "faces",
            "targetName": "myFaces"
        },
        {
            "name": "imageType",
            "targetName": "myImageType"
        },
        {
            "name": "color",
            "targetName": "myColor"
        },
        {
            "name": "adult",
            "targetName": "myAdultCategory"
        }
    ]
}
```

##	Sample input

```json
{
    "values": [
        {
            "recordId": "1",
            "data": {
                "url": "https://storagesample.blob.core.windows.net/sample-container/image.jpg"
            }
        }
    ]
}
```


##	Sample output

```json
{
    "values": [
      {
        "recordId": "1",
            "data": {
                "categories": [
           {
                        "name": "abstract_",
                        "score": 0.00390625
                    },
                    {
                "name": "people_",
                        "score": 0.83984375,
                "detail": {
                            "celebrities": [
                                {
                                    "name": "Satya Nadella",
                                    "faceRectangle": {
                                        "left": 597,
                                        "top": 162,
                                        "width": 248,
                                        "height": 248
                                    },
                                    "confidence": 0.999028444
                                }
                            ],
                            "landmarks": [
                                {
                                    "name": "Forbidden City",
                                    "confidence": 0.9978346
                                }
                            ]
                        }
                    }
                ],
                "adult": {
                    "isAdultContent": false,
                    "isRacyContent": false,
                    "adultScore": 0.0934349000453949,
                    "racyScore": 0.068613491952419281
                },
                "tags": [
                    {
                        "name": "person",
                        "confidence": 0.98979085683822632
                    },
                    {
                        "name": "man",
                        "confidence": 0.94493889808654785
                    },
                    {
                        "name": "outdoor",
                        "confidence": 0.938492476940155
                    },
                    {
                        "name": "window",
                        "confidence": 0.89513939619064331
                    }
                ],
                "description": {
                    "tags": [
                        "person",
                        "man",
                        "outdoor",
                        "window",
                        "glasses"
                    ],
                    "captions": [
                        {
                            "text": "Satya Nadella sitting on a bench",
                            "confidence": 0.48293603002174407
                        }
                    ]
                },
                "requestId": "0dbec5ad-a3d3-4f7e-96b4-dfd57efe967d",
                "metadata": {
                    "width": 1500,
                    "height": 1000,
                    "format": "Jpeg"
                },
                "faces": [
                    {
                        "age": 44,
                        "gender": "Male",
                    "faceRectangle": {
                            "left": 593,
                            "top": 160,
                            "width": 250,
                            "height": 250
                        }
                    }
                ],
                "color": {
                    "dominantColorForeground": "Brown",
                    "dominantColorBackground": "Brown",
                    "dominantColors": [
                        "Brown",
                        "Black"
                    ],
                    "accentColor": "873B59",
                    "isBWImg": false
                    },
                "imageType": {
                    "clipArtType": 0,
                    "lineDrawingType": 0
                }
           }
      }
    ]
}
```


## Error cases
In the following error cases, no elements are extracted.

| Error Code | Description |
|------------|-------------|
| NotSupportedLanguage | The language provided is not supported. |
| InvalidImageUrl | Image URL is badly formatted or not accessible.|
| InvalidImageFormat | Input data is not a valid image. |
| InvalidImageSize | Input image is too large. |
| NotSupportedVisualFeature  | Specified feature type is not valid. |
| NotSupportedImage | Unsupported image, e.g. child pornography. |
| InvalidDetails | Unsupported domain-specific model. |

## See also

+ [Predefined skills](cognitive-search-predefined-skills.md)
+ [How to define a skillset](cognitive-search-defining-skillset.md)
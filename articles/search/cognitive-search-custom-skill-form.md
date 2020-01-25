---
title: 'Form Recognizer custom skill (C#)'
titleSuffix: Azure Cognitive Search
description: Learn how to create a Form Recognizer custom skill using C# and Visual Studio.

manager: nitinme
author: PatrickFarley
ms.author: pafarley
ms.service: cognitive-search
ms.topic: article
ms.date: 01/21/2020
---

# Example: Create a Form Recognizer custom skill

In this Azure Cognitive Search skillset example, you'll learn how to create a Form Recognizer custom skill using C# and Visual Studio. The example implements the [custom skill interface](cognitive-search-custom-skill-interface.md).

## Prerequisites

- [Visual Studio 2019](https://visualstudio.microsoft.com/downloads/) (any edition).
- Access to the Form Recognizer limited-access preview. To get access to the preview, fill out and submit the [Form Recognizer access request](https://aka.ms/FormRecognizerRequestAccess) form.
- At least five forms of the same type. You can use sample data provided with this guide.

## Create a Form Recognizer resource

[!INCLUDE [create resource](../cognitive-services/form-recognizer/includes/create-resource.md)]

## Train your model

You'll need to train a Form Recognizer model with your input forms before you use this skill. Follow the [cURL quickstart](https://docs.microsoft.com/azure/cognitive-services/form-recognizer/quickstarts/curl-train-extract) to learn how to do train a model. You can use the sample forms provided in that quickstart, or you can use your own data. Once the model is trained, copy its ID value to a secure location.

## Set up the custom skill

Clone the [Azure Search Power Skills](https://github.com/Azure-Samples/azure-search-power-skills) repository to your local machine. Then navigate to **Vision/AnalyzeForm/** and open _AnalyzeForm.csproj_ in Visual Studio. This project creates an Azure Function resource that fulfills the [custom skill interface](cognitive-search-custom-skill-interface.md) and can be used for Azure Cognitive Search enrichment. It takes form documents as inputs, and it outputs (as text) the key/value pairs that you specify.

Open _AnalyzeForm.cs_ and make the following changes.
* Set the `formsRecognizerApiEndpoint` variable to your resource endpoint.
* Set `formsRecognizerApiKeySetting` to your resource's key value.
* Set `modelIdSetting` to your saved model ID.

Next, find the `fieldMappings` variable. This variable defines the list of keys you want to extract from your forms and a custom label for each key. For example, a value of `{ "Address:", "address" }, { "Invoice For:", "recipient" }` means the script will only save the values for the `Address:` and `Invoice For:` keys.

Finally, note the `contentType` variable. This script runs the given Form Recognizer model on PDF files, but if you're working with a different file type, you need to change the `contentType` to the correct [MIME type](https://developer.mozilla.org/docs/Web/HTTP/Basics_of_HTTP/MIME_types/Complete_list_of_MIME_types) for your file.

## Test the function from Visual Studio

After you've edited your script, save it and set the **AnalyzeForm** project as the startup project in Visual Studio. Then Press **F5** to run it in your local environment. Use a REST service like Postman to call the function.

```HTTP
POST https://localhost:7071/api/analyze-form
```

### Request body

> [!NOTE]
> This request points to a sample file stored in the [Azure Search Power Skills](https://github.com/Azure-Samples/azure-search-power-skills) repository. If you trained the model with your own forms, replace this URL with the URL to your own sample form. When the skill is integrated in a skillset, the URL and token will be provided by Cognitive Search.

```json
{
    "values": [
        {
            "recordId": "record1",
            "data": { 
                "formUrl": "https://github.com/Azure-Samples/azure-search-power-skills/raw/master/SampleData/Invoice_4.pdf",
                "formSasToken":  "?st=sasTokenThatWillBeGeneratedByCognitiveSearch"
            }
        }
    ]
}
```

### Response

You should see a response similar to the following example:

```json
{
    "values": [
        {
            "recordId": "record1",
            "data": {
                "address": "1111 8th st. Bellevue, WA 99501 ",
                "recipient": "Southridge Video 1060 Main St. Atlanta, GA 65024 "
            },
            "errors": null,
            "warnings": null
        }
    ]
}
```

## Publish the function to Azure
When you're satisfied with the function behavior, you can publish it.

1. In **Solution Explorer**, right-click the project and select **Publish**. Choose **Create New** > **Publish**.

1. If you haven't already connected Visual Studio to your Azure account, select **Add an account....**

1. Follow the on-screen prompts. You're asked to specify a unique name for your app service, the Azure subscription, the resource group, the hosting plan, and the storage account you want to use. You can create a new resource group, a new hosting plan, and a storage account if you don't already have these. When finished, select **Create**

1. After the deployment is complete, notice the Site URL. This is the address of your function app in Azure. Save it to a temporary location.

1. In the [Azure portal](https://portal.azure.com), navigate to the Resource Group, and look for the `AnalyzeForm` Function you published. Under the **Manage** section, you should see Host Keys. Copy the *default* host key and save it to a temporary location.

## Connect to your pipeline

To use this skill in a Cognitive Search pipeline, you'll need to add a skill definition to your skillset. The following JSON block is a sample skill definition (you should update the inputs and outputs to reflect your particular scenario and skillset environment). Replace `AzureFunctionEndpointUrl` with your function URL, and replace `AzureFunctionDefaultHostKey` with your host key.

```json
{
    "@odata.type": "#Microsoft.Skills.Custom.WebApiSkill",
    "name": "formrecognizer", 
    "description": "Extracts fields from a form using a pre-trained form recognition model",
    "uri": "[AzureFunctionEndpointUrl]/api/analyze-form?code=[AzureFunctionDefaultHostKey]",
    "httpMethod": "POST",
    "timeout": "PT30S",
    "context": "/document",
    "batchSize": 1,
    "inputs": [
        {
            "name": "formUrl",
            "source": "/document/metadata_storage_path"
        },
        {
            "name": "formSasToken",
            "source": "/document/metadata_storage_sas_token"
        }
    ],
    "outputs": [
        {
            "name": "address",
            "targetName": "address"
        },
        {
            "name": "recipient",
            "targetName": "recipient"
        }
    ]
}
```

## Next steps

In this guide, you created a custom skill from the Azure Form Recognizer service. To learn more about custom skills, see the following resources. 

+ [Power Skills: a repository of custom skills](https://github.com/Azure-Samples/azure-search-power-skills)
+ [Add a custom skill to an AI enrichment pipeline](cognitive-search-custom-skill-interface.md)
+ [How to define a skillset](cognitive-search-defining-skillset.md)
+ [Create Skillset (REST)](https://docs.microsoft.com/rest/api/searchservice/create-skillset)
+ [How to map enriched fields](cognitive-search-output-field-mapping.md)

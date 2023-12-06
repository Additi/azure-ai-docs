---
title: How to use the GPT-4 Turbo with Vision model
titleSuffix: Azure OpenAI Service
description: Learn about the options for using GPT-4 Turbo with Vision
author: PatrickFarley #dereklegenzoff
ms.author: pafarley #delegenz
ms.service: azure-ai-openai
ms.custom: 
ms.topic: how-to
ms.date: 11/06/2023
manager: nitinme
keywords:
---

# Use GPT-4 Turbo with Vision

The GPT-4 Turbo with Vision model answers general questions about what's present in the images. You can also show it video, if you use Vision enhancements. 

While it does understand the relationship between objects in images, it isn't optimized to answer detailed questions about the locations of certain objects. For example, you can ask it what color a car is or what some ideas for dinner might be based on what's in your fridge, but if you show it an image of a room and ask it where the chair is, it might not answer correctly.

It's important to note the following:
- GPT-4 Turbo with Vision doesn't behave differently from GPT-4, with the small exception of the system prompt OpenAI uses for the model.
- GPT-4 Turbo with Vision doesn't perform worse at text tasks because it has vision, it's simply GPT-4 with vision added.
- GPT-4 Turbo with Vision is an augmentative set of capabilities for the model.

To use GPT-4 Turbo with Vision, you call the Chat Completion API on a GPT-4 Vision model that you have deployed.

## Call the Chat Completion APIs

The following REST command shows the most basic way to use the GPT-4 Turbo with Vision model with code. If this is your first time using these models programmatically, we recommend starting with our [GPT-4 Turbo with Vision quickstart](../gpt-v-quickstart.md).

Send a POST request to `https://{RESOURCE_NAME}.openai.azure.com/openai/deployments/{DEPLOYMENT_NAME}/chat/completions?api-version=2023-12-01-preview` where 

- RESOURCE_NAME is the name of your Azure OpenAI resource 
- DEPLOYMENT_NAME is the name of your GPT-4 Vision model deployment 

**Required headers**: 
- `Content-Type`: application/json 
- `api-key`: {API_KEY} 

**Body**: 
The following is a sample request body. The format is the same as the chat completions API for GPT-4, except that the message content can be an array containing strings and images (either a URL to an image, or a base-64-encoded image). 

```json
{
    "messages": [ 
        {
            "role": "system", 
            "content": "You are a helpful assistant." 
        },
        {
            "role": "user", 
            "content": [ 
                "Describe this picture:", { "image": "URL or base-64-encoded image" } 
            ] 
        }
    ],
    "max_tokens": 100, 
    "stream": false 
} 
```

## Low or high fidelity image understanding

By controlling the _detail_ parameter, which has two options, `low` or `high`, you can control how the model processes the image and generates its textual understanding.
- `low` disables the "high res" mode. The model receives a low-res 512x512 version of the image and represents the image with a budget of 65 tokens. This allows the API to return faster responses and consume fewer input tokens for use cases that don't require high detail.
- `high` enables "high res" mode, which first allows the model to see the low res image and then creates detailed crops of input images as 512x512 squares based on the input image size. Each of the detailed crops uses twice the token budget (65 tokens) for a total of 129 tokens.


## Next steps

* [Learn more about Azure OpenAI](../overview.md).
* [GPT-4 Turbo with Vision quickstart](../gpt-v-quickstart.md)
* [GPT-4 Turbo with Vision frequently asked questions](../faq.yml#gpt-4-turbo-with-vision)
* [GPT-4 Turbo with Vision API reference](https://aka.ms/gpt-v-api-ref)

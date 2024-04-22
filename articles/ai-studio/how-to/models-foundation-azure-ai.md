---
title: Explore Azure AI services in Azure AI Studio
titleSuffix: Azure AI Studio
description: This article introduces Azure AI services in Azure AI Studio.
manager: nitinme
ms.service: azure-ai-studio
ms.custom:
  - ignite-2023
ms.topic: how-to
ms.date: 2/6/2024
ms.reviewer: eur
ms.author: eur
author: eric-urban
---

# Explore Azure AI services in Azure AI Studio

In Azure AI Studio, you can quickly try out Azure AI services such as Speech, Vision, and Language. Go to the **Home** page and then select **AI Services** to explore these services.

## Azure AI foundation models

Azure AI foundation models have been pre-trained on vast amounts of data, and that can be fine-tuned for specific tasks with a relatively small amount of domain-specific data. These models serve as a starting point for custom models and accelerate the model-building process for a variety of tasks, including natural language processing, computer vision, speech, and generative AI tasks. 

In this article, explore where Azure AI Studio lets you try out and integrate these services into your applications.

## Speech

[Azure AI Speech](/azure/ai-services/speech-service/) provides speech to text and text to speech capabilities using a Speech resource. You can transcribe speech to text with high accuracy, produce natural-sounding text to speech voices, translate spoken audio, and use speaker recognition during conversations.

You can try the following capabilities of Azure AI Speech in AI Studio:
- Real-time speech to text: Quickly test live transcription capabilities on your own audio without writing any code.
- Custom Neural Voice: Use your own audio recordings to create a distinct, one-of-a-kind voice for your text to speech apps. For more information, see the [Custom Neural Voice overview](../../ai-services/speech-service/custom-neural-voice.md) in the Azure AI Speech documentation. The steps to create a Custom Neural Voice are similar in Azure AI Studio and the [Speech Studio](https://aka.ms/speechstudio/).

> [!TIP]
> You can also try speech to text and text to speech capabilities in the Azure AI Studio playground. For more information, see [Hear and speak with chat in the playground](../quickstarts/hear-speak-playground.md).

Explore more Speech capabilities in the [Speech Studio](https://aka.ms/speechstudio/) and the [Azure AI Speech documentation](/azure/ai-services/speech-service/).

## Vision and Document Intelligence

[Azure AI Vision](/azure/ai-services/computer-vision/) gives your apps the ability to read text, analyze images, and detect faces with technology like optical character recognition (OCR) and machine learning. 

> [!TIP]
> You can also try GPT-4 Turbo with Vision capabilities in the Azure AI Studio playground. For more information, see [GPT-4 Turbo with Vision on your images and videos in Azure AI Studio playground](../quickstarts/multimodal-vision.md).

Explore more vision capabilities in the [Vision Studio](https://portal.vision.cognitive.azure.com/) and the [Azure AI Vision documentation](/azure/ai-services/computer-vision/).


## Language

[Azure AI Language](/azure/ai-services/language-service/) can interpret natural language, classify documents, get real-time translations, or integrate language into your bot experiences.

Use Natural Language Processing (NLP) features to analyze your textual data using state-of-the-art pre-configured AI models or customize your own models to fit your scenario.

Explore more Language capabilities in the [Language Studio](https://language.cognitive.azure.com/), [Custom Translator Studio](https://portal.customtranslator.azure.ai/), and the [Azure AI Language documentation](/azure/ai-services/language-service/).


### Try more Azure AI services

Azure AI Studio provides a quick way to try out Azure AI capabilities. However, some Azure AI services are not currently available in AI Studio.

To try more Azure AI services, go to the following studio links:

- [Azure OpenAI](https://oai.azure.com/)
- [Speech](https://speech.microsoft.com/)
- [Language](https://language.cognitive.azure.com/)
- [Vision](https://portal.vision.cognitive.azure.com/)
- [Custom Vision](https://www.customvision.ai/)
- [Document Intelligence](https://formrecognizer.appliedai.azure.com/)
- [Content Safety](https://contentsafety.cognitive.azure.com/)
- [Custom Translator](https://portal.customtranslator.azure.ai/)


## Next steps

- [Explore the model catalog in Azure AI Studio](model-catalog.md)

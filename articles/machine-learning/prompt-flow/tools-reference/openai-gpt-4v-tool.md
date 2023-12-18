---
title: OpenAI GPT-4V (preview)
titleSuffix: Azure Machine Learning
description: The prompt flow OpenAI GPT-4V tool enables you to  leverage OpenAI's GPT-4 with vision, also referred to as GPT-4V or gpt-4-vision-preview in the API, to take images as input and answer questions about them.
services: machine-learning
ms.service: machine-learning
ms.subservice: prompt-flow
ms.topic: reference
author: zhongj
ms.author: jinzhong
ms.date: 12/18/2023
---

# OpenAI GPT-4V (preview)

## Introduction
OpenAI GPT-4V tool enables you to leverage OpenAI's GPT-4 with vision, also referred to as GPT-4V or gpt-4-vision-preview in the API, to take images as input and answer questions about them.

> [!IMPORTANT]
> OpenAI GPT-4V tool is currently in public preview. This preview is provided without a service-level agreement, and is not recommended for production workloads. Certain features might not be supported or might have constrained capabilities.
> For more information, see [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## Prerequisites

- Create OpenAI resources

    Sign up account [OpenAI website](https://openai.com/)
    Login and [Find personal API key](https://platform.openai.com/account/api-keys)

- Get Access to GPT-4 API

    To use GPT-4 with vision, you need access to GPT-4 API. Learn more about [How to get access to GPT-4 API](https://help.openai.com/en/articles/7102672-how-can-i-access-gpt-4)

## Connection

Setup connections to provisioned resources in prompt flow.

| Type        | Name     | API KEY  |
|-------------|----------|----------|
| OpenAI      | Required | Required |

## Inputs

| Name                   | Type        | Description                                                                                    | Required |
|------------------------|-------------|------------------------------------------------------------------------------------------------|----------|
| connection             | OpenAI      | the OpenAI connection to be used in the tool                                                   | Yes      |
| model                  | string      | the language model to use, currently only support gpt-4-vision-preview                         | Yes      |
| prompt                 | string      | The text prompt that the language model will use to generate it's response.                    | Yes      |
| max\_tokens            | integer     | the maximum number of tokens to generate in the response. Default is a low value decided by [OpenAI API](https://platform.openai.com/docs/guides/vision).                      | No       |
| temperature            | float       | the randomness of the generated text. Default is 1.                                            | No       |
| stop                   | list        | the stopping sequence for the generated text. Default is null.                                 | No       |
| top_p                  | float       | the probability of using the top choice from the generated tokens. Default is 1.               | No       |
| presence\_penalty      | float       | value that controls the model's behavior with regards to repeating phrases. Default is 0.      | No       |
| frequency\_penalty     | float       | value that controls the model's behavior with regards to generating rare phrases. Default is 0. | No       |

## Outputs

| Return Type | Description                              |
|-------------|------------------------------------------|
| string      | The text of one response of conversation |
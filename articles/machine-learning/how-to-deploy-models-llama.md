---
title: How to deploy Llama 2 family of large language models with Azure Machine Learning studio
titleSuffix: Azure Machine Learning
description: Learn how to deploy Llama 2 family of large language models with Azure Machine Learning studio.
manager: scottpolly
ms.service: machine-learning
ms.subservice: inferencing
ms.topic: how-to
ms.date: 01/17/2024
ms.reviewer: shubhiraj
reviewer: shubhirajMsft
ms.author: mopeakande
author: msakande
ms.custom: [references_regions]

#customer intent: As a <role>, I want <what> so that <why>.

#This functionality is also available in Azure AI Studio
---


# How to deploy Llama 2 family of large language models with Azure Machine Learning studio

The Llama 2 family of large language models (LLMs) is a collection of pretrained and fine-tuned generative text models ranging in scale from 7 billion to 70 billion parameters. The model family also includes fine-tuned versions optimized for dialogue use cases with Reinforcement Learning from Human Feedback (RLHF), called Llama-2-chat.

Llama 2 can be deployed as a service with pay-as-you-go billing or with hosted infrastructure in real-time endpoints.

[!INCLUDE [machine-learning-preview-generic-disclaimer](includes/machine-learning-preview-generic-disclaimer.md)]

## Deploy Llama 2 models with pay-as-you-go

Certain models in the model catalog can be deployed as a service with pay-as-you-go, providing a way to consume them as an API without hosting them on your subscription, while keeping the enterprise security and compliance organizations need. This deployment option doesn't require quota from your subscription.

Llama 2 models deployed as a service with pay-as-you-go are offered by Meta AI through the Azure Marketplace, and they might add more terms of use and pricing.

### Azure Marketplace model offerings

The following models are available in the Azure Marketplace for Llama 2 when deployed as a service with pay-as-you-go:

* Meta Llama-2-7B (preview)
* Meta Llama 2 7B-Chat (preview)
* Meta Llama-2-13B (preview)
* Meta Llama 2 13B-Chat (preview)
* Meta Llama-2-70B (preview)
* Meta Llama 2 70B-Chat (preview)

If you need to deploy a different model, [deploy it to real-time endpoints](#deploy-llama-2-models-to-real-time-endpoints) instead.

### Prerequisites

- An Azure subscription with a valid payment method. Free or trial Azure subscriptions won't work. If you don't have an Azure subscription, create a [paid Azure Machine Learning account](https://azure.microsoft.com/pricing/purchase-options/pay-as-you-go) to begin.
- An Azure Machine Learning workspace and a compute instance. If you don't have these, use the steps in the [Quickstart: Create workspace resources](quickstart-create-resources.md) article to create them.

    > [!IMPORTANT]
    > Pay-as-you-go model deployment offering is only available in workspaces created in **East US 2** and **West US 3** regions.

- Azure role-based access controls (Azure RBAC) are used to grant access to operations in Azure Machine Learning. To perform the steps in this article, your user account must be assigned the __owner__ or __contributor__ role for the Azure Machine Learning workspace. Alternatively, your account can be assigned a custom role that has the following permissions:

    - On the Azure subscription (needed to sign up the workspace for the Azure Marketplace offering, once for each workspace, per offering):
      - `Microsoft.MarketplaceOrdering/agreements/offers/plans/read`
      - `Microsoft.MarketplaceOrdering/agreements/offers/plans/sign/action`
      - `Microsoft.Marketplace/offerTypes/publishers/offers/plans/agreements/read`
      - `Microsoft.SaaS/register/action`
 
    - On the Resource Group:
      - `Microsoft.SaaS/resources/read`
      - `Microsoft.SaaS/resources/write`
 
    - On the Project (the AI developer role contains these permissions already):
      - `Microsoft.MachineLearningServices/workspaces/marketplaceModelSubscriptions/*`  
      - `Microsoft.MachineLearningServices/workspaces/serverlessEndpoints/*`
      
    - For finetuning, these permissions are needed at the project level (the AI developer role contains these permissions already):
      - `Microsoft.MachineLearningServices/workspaces/jobs/*`
      - `Microsoft.MachineLearningServices/workspaces/datastores/*`

    For more information on permissions, see [Manage access to an Azure Machine Learning workspace](how-to-assign-roles.md).

### Create a new deployment

To create a deployment:

1. Go to the [Azure Machine Learning studio](https://ml.azure.com/home).
1. Select the workspace in which you want to deploy your models. To use the pay-as-you-go model deployment offering, your workspace must belong to the **East US 2** or **West US 3** region.
1. Choose the model you want to deploy from the [model catalog](https://ml.azure.com/model/catalog). Alternatively, you can initiate deployment by selecting **Create** from the **Endpoints** option under **Assets** in your workspace.
1. On the model's overview page, select **Deploy** and then **Pay-as-you-go**.

    :::image type="content" source="media/how-to-deploy-models-llama/deploy-pay-as-you-go.png" alt-text="A screenshot showing how to deploy a model with the pay-as-you-go option." lightbox="media/how-to-deploy-models-llama/deploy-pay-as-you-go.png":::

1. On the deployment wizard, select the link to **Azure Marketplace Terms** to learn more about the terms of use. You can also select the **Marketplace offer details** tab to learn about pricing for the selected model.
1. If this is your first time deploying the model in the workspace, you have to sign your workspace up for the particular offering (Llama-2-70b) from the Azure Marketplace. This step requires that your account has the Azure subscription permissions and resource group permissions listed in the prerequisites. Each workspace has its own connection to the marketplace's offering, which allows you to control and monitor spending per workspace. Select **Subscribe and Deploy**.

    > [!NOTE]
    > Subscribing a project to a particular Azure Marketplace offering (in this case, Llama-2-70b) requires that your account has **Contributor** or **Owner** access at the subscription level where the project is created. Alternatively, your user account can be assigned a custom role that has the Azure subscription permissions and resource group permissions listed in the [prerequisites](#prerequisites).

    :::image type="content" source="media/how-to-deploy-models-llama/deploy-marketplace-terms.png" alt-text="A screenshot showing the terms and conditions of a given model." lightbox="media/how-to-deploy-models-llama/deploy-marketplace-terms.png":::

1. Once you've signed up the workspace for the Azure Marketplace offering, subsequent deployments of the _same_ offering in the _same_ workspace won't require signing up. Therefore, you won't need to have the subscription-level permissions for subsequent deployments. If this is your case, select **Continue to deploy**.

    :::image type="content" source="media/how-to-deploy-models-llama/deploy-pay-as-you-go-project.png" alt-text="A screenshot showing a project that is already subscribed to the offering." lightbox="media/how-to-deploy-models-llama/deploy-pay-as-you-go-project.png":::

1. Give the deployment a name. This name becomes part of the deployment API URL. This URL must be unique in each Azure region.

    :::image type="content" source="media/how-to-deploy-models-llama/deployment-name.png" alt-text="A screenshot showing how to indicate the name of the deployment you want to create." lightbox="media/how-to-deploy-models-llama/deployment-name.png":::

1. Select **Deploy**. Wait until the deployment is finished and you're redirected to the serverless endpoints page.
1. Select the endpoint to open its Details page.
1. Select the **Test** tab to start interacting with the model.
1. You can also take note of the **Target** URL and the **Secret Key** to call the deployment and generate completions.   
1. You can always find the endpoint's details, URL, and access keys by navigating to **Workspace** > **Endpoints** > **Serverless endpoints**.

### Consume Llama 2 models as a service

Models deployed as a service can be consumed using either the chat or the completions API, depending on the type of model you've deployed.

1. In the **workspace**, select **Endpoints** > **Serverless endpoints**.
1. Find and select the deployment you created.
1. Copy the **Target** URL and the **Key** token values.
1. Make an API request based on the type of model you deployed. 

    - For completions models, such as `Llama-2-7b`, use the [`<target_url>/v1/completions`](#completions-api) API
    - For chat models, such as `Llama-2-7b-chat`, use the [`<target_url>/v1/chat/completions`](#chat-api) API. 

    See the [reference](#reference-for-llama-2-models-deployed-as-a-service) section for more details with examples.

### Reference for Llama 2 models deployed as a service

#### Completions API

Use the method `POST` to send the request to the `/v1/completions` route:

__Request__

```rest
POST /v1/completions HTTP/1.1
Host: <DEPLOYMENT_URI>
Authorization: Bearer <TOKEN>
Content-type: application/json
```

#### Request schema

Payload is a JSON formatted string containing the following parameters:

| Key           | Type      | Default | Description    |
|---------------|-----------|---------|----------------|
| `prompt`      | `string`  |  No default. This must be specified.  | The prompt to send to the model. |
| `stream`      | `boolean` | `False`  | Streaming allows the generated tokens to be sent as data-only server-sent events whenever they become available. |
| `max_tokens`  | `integer` | `16`    | The maximum number of tokens to generate in the completion. The token count of your prompt plus `max_tokens` can't exceed the model's context length.  |
| `top_p`       | `float`   | `1`     | An alternative to sampling with temperature, called nucleus sampling, where the model considers the results of the tokens with `top_p` probability mass. So 0.1 means only the tokens comprising the top 10% probability mass are considered. We generally recommend altering it or `temperature` but not both. |
| `temperature` | `float`   | `1`     | The sampling temperature to use, between 0 and 2. Higher values mean the model samples more broadly the distribution of tokens. Zero means greedy sampling. It's recommend altering this or `top_p` but not both. |
| `n`           | `integer` | `1`     | How many completions to generate for each prompt. Note: Because this parameter generates many completions, it can quickly consume your token quota. |
| `stop`        | `array`   | `null`  | String or a list of strings containing the word where the API stops generating further tokens. The returned text won't contain the stop sequence.   |
| `best_of`     | `integer` | `1`     | Generates best_of completions server-side and returns the "best" (the one with the lowest log probability per token). Results can't be streamed. When used with n, best_of controls the number of candidate completions and n specifies how many to return – best_of must be greater than n. Note: Because this parameter generates many completions, it can quickly consume your token quota.|
| `logprobs` | `integer` |  `null` | A number indicating to include the log probabilities on the logprobs most likely tokens, as well the chosen tokens. For example, if logprobs is 10, the API returns a list of the 10 most likely tokens. the API always returns the logprob of the sampled token, so there might be up to logprobs+1 elements in the response.  |
| `presence_penalty`    | `float`   | `null`  | Number between -2.0 and 2.0. Positive values penalize new tokens based on whether they appear in the text so far, increasing the model's likelihood to talk about new topics. |
| `ignore_eos`          | `boolean` | `True`  | Whether to ignore the EOS token and continue generating tokens after the EOS token is generated.  |
| `use_beam_search`     | `boolean` | `False` | Whether to use beam search instead of sampling. In such case, `best_of must > 1` and `temperature` must be `0`. |
| `stop_token_ids`      | `array`   | `null`  | List of tokens' ID that stops the generation when they're generated. The returned output contains the stop tokens unless the stop tokens are special tokens. |
| `skip_special_tokens` | `boolean` | `null`  | Whether to skip special tokens in the output. |

#### Example

__Body__

```json
{
    "prompt": "What's the distance to the moon?",
    "temperature": 0.8,
    "max_tokens": 512,
}
```

#### Response schema

The response payload is a dictionary with the following fields.

| Key       | Type      | Description                                                              |
|-----------|-----------|--------------------------------------------------------------------------|
| `id`      | `string`  | A unique identifier for the completion.                                  |
| `choices` | `array`   | The list of completion choices the model generated for the input prompt. |
| `created` | `integer` | The Unix timestamp (in seconds) of when the completion was created.      |
| `model`   | `string`  | The model_id used for completion.                                        |
| `object`  | `string`  | The object type, which is always "text_completion".                      |
| `usage`   | `object`  | Usage statistics for the completion request.                             |

> [!TIP]
> In the streaming mode, for each chunk of response, `finish_reason` is always `null`, except from the last one which is terminated by a payload `[DONE]`. 


The `choice` object is a dictionary with the following fields. 

| Key     | Type      | Description  |
|---------|-----------|------|
| `index` | `integer` | Choice index. When best_of > 1, the index in this array might not be in order and might not be 0 to n-1. |
| `text`  | `string`  | Completion result. |
| `finish_reason` | `string` | The reason the model stopped generating tokens: `stop`, model hit a natural stop point, or a provided stop sequence; `length`, if max number of tokens have been reached; `content_filter`, When RAI moderates and CMP forces moderation; `content_filter_error`, an error during moderation and wasn't able to make decision on the response; `null`, API response still in progress or incomplete. |
| `logprobs` | `object` | The log probabilities of the generated tokens in the output text. |


The `usage` object is a dictionary with the following fields. 

| Key                 | Type      | Value                                         |
|---------------------|-----------|-----------------------------------------------|
| `prompt_tokens`     | `integer` | Number of tokens in the prompt.               |
| `completion_tokens` | `integer` | Number of tokens generated in the completion. |
| `total_tokens`      | `integer` | Total tokens.                                 |
    
The `logprobs` object is a dictionary with the following fields:

| Key              | Type  | Value |
|------------------|-------------------------|----|
| `text_offsets`   | `array` of `integers`   | The position or index of each token in the completion output.  |
| `token_logprobs` | `array` of `float`      | Selected logprobs from dictionary in top_logprobs array  |
| `tokens`         | `array` of `string`     | Selected tokens  |
| `top_logprobs`   | `array` of `dictionary` | Array of dictionary. In each dictionary, the key is the token and the value is the prob. |

#### Example

```json
{
    "id": "12345678-1234-1234-1234-abcdefghijkl",
    "object": "text_completion",
    "created": 217877,
    "choices": [
        {
            "index": 0,
            "text": "The Moon is an average of 238,855 miles away from Earth, which is about 30 Earths away.",
            "logprobs": null,
            "finish_reason": "stop"
        }
    ],
    "usage": {
        "prompt_tokens": 7,
        "total_tokens": 23,
        "completion_tokens": 16
    }
}
```

#### Chat API

Use the method `POST` to send the request to the `/v1/chat/completions` route:

__Request__

```rest
POST /v1/chat/completions HTTP/1.1
Host: <DEPLOYMENT_URI>
Authorization: Bearer <TOKEN>
Content-type: application/json
```

#### Request schema

Payload is a JSON formatted string containing the following parameters:

| Key | Type | Default | Description |
|-----|-----|-----|-----|
| `messages`    | `string`  | No default. This must be specified.  | The message or history of messages to prompt the model with.  |
| `stream`      | `boolean` | `False` | Streaming allows the generated tokens to be sent as data-only server-sent events whenever they become available.  |
| `max_tokens`  | `integer` | `16`    | The maximum number of tokens to generate in the completion. The token count of your prompt plus `max_tokens` can't exceed the model's context length. |
| `top_p`       | `float`   | `1`     | An alternative to sampling with temperature, called nucleus sampling, where the model considers the results of the tokens with `top_p` probability mass. So 0.1 means only the tokens comprising the top 10% probability mass are considered. We generally recommend altering it or `temperature` but not both.  |
| `temperature` | `float`   | `1`     | The sampling temperature to use, between 0 and 2. Higher values mean the model samples more broadly the distribution of tokens. Zero means greedy sampling. It's recommend altering this or `top_p` but not both.  |
| `n`           | `integer` | `1`     | How many completions to generate for each prompt. Note: Because this parameter generates many completions, it can quickly consume your token quota. |
| `stop`        | `array`   | `null`  | String or a list of strings containing the word where the API stops generating further tokens. The returned text won't contain the stop sequence. |
| `best_of`     | `integer` | `1`     | Generates best_of completions server-side and returns the "best" (the one with the lowest log probability per token). Results can't be streamed. When used with n, best_of controls the number of candidate completions and n specifies how many to return – best_of must be greater than n. Note: Because this parameter generates many completions, it can quickly consume your token quota.|
| `logprobs` | `integer` |  `null` | A number indicating to include the log probabilities on the logprobs most likely tokens, as well the chosen tokens. For example, if logprobs is 10, the API returns a list of the 10 most likely tokens. the API will always return the logprob of the sampled token, so there might be up to logprobs+1 elements in the response.  |
| `presence_penalty`    | `float`   | `null`  | Number between -2.0 and 2.0. Positive values penalize new tokens based on whether they appear in the text so far, increasing the model's likelihood to talk about new topics. |
| `ignore_eos`          | `boolean` | `True`  | Whether to ignore the EOS token and continue generating tokens after the EOS token is generated. |
| `use_beam_search`     | `boolean` | `False` | Whether to use beam search instead of sampling. In such case, `best_of must > 1` and `temperature` must be `0`. |
| `stop_token_ids`      | `array`   | `null`  | List of token IDs that stop the generation when they are generated. The returned output contains the stop tokens unless the stop tokens are special tokens. |
| `skip_special_tokens` | `boolean` | `null`  | Whether to skip special tokens in the output. |

The `message` object has the following fields:

| Key       | Type      | Value |
|-----------|-----------|------------|
| `content` | `string` | The contents of the message. Content is required for all messages. |
| `role`    | `string` | The role of the message's author. One of `system`, `user`, or `assistant`. |


#### Example

__Body__

```json
{
    "messages":
    [
        { 
        "role": "system", 
        "content": "You are a helpful assistant that translates English to Italian."},
        {
        "role": "user", 
        "content": "Translate the following sentence from English to Italian: I love programming."
        }
    ],
    "temperature": 0.8,
    "max_tokens": 512,
}
```

#### Response schema

The response payload is a dictionary with the following fields.

| Key       | Type      | Description                                                                |
|-----------|-----------|----------------------------------------------------------------------------|
| `id`      | `string`  | A unique identifier for the completion.                                    |
| `choices` | `array`   | The list of completion choices the model generated for the input messages. |
| `created` | `integer` | The Unix timestamp (in seconds) of when the completion was created.        |
| `model`   | `string`  | The model_id used for completion.                                          |
| `object`  | `string`  | The object type, which is always "chat.completion".                        |
| `usage`   | `object`  | Usage statistics for the completion request.                               |

> [!TIP]
> In the streaming mode, for each chunk of response, `finish_reason` is always `null`, except from the last one which is terminated by a payload `[DONE]`. In each `choice` object, the key for `message` is changed by `delta`. 


The `choice` object is a dictionary with the following fields. 

| Key     | Type      | Description  |
|---------|-----------|--------------|
| `index` | `integer` | Choice index. When best_of > 1, the index in this array might not be in order and might not be 0 to n-1. |
| `message` or `delta`   | `string`  | Chat completion result in `message` object. When streaming mode is used, `delta` key is used.  |
| `finish_reason` | `string` | The reason the model stopped generating tokens: `stop`, model hit a natural stop point, or a provided stop sequence; `length`, if max number of tokens have been reached; `content_filter`, When RAI moderates and CMP forces moderation; `content_filter_error`, an error during moderation and wasn't able to make decision on the response; `null`, API response still in progress or incomplete. |
| `logprobs` | `object` | The log probabilities of the generated tokens in the output text. |


The `usage` object is a dictionary with the following fields. 

| Key                 | Type      | Value                                         |
|---------------------|-----------|-----------------------------------------------|
| `prompt_tokens`     | `integer` | Number of tokens in the prompt.               |
| `completion_tokens` | `integer` | Number of tokens generated in the completion. |
| `total_tokens`      | `integer` | Total tokens.                                 |
    
The `logprobs` object is a dictionary with the following fields:

| Key              | Type                    | Value   |
|------------------|-------------------------|---------|
| `text_offsets`   | `array` of `integers`   | The position or index of each token in the completion output. |
| `token_logprobs` | `array` of `float`      | Selected logprobs from dictionary in top_logprobs array   |
| `tokens`         | `array` of `string`     | Selected tokens   |
| `top_logprobs`   | `array` of `dictionary` | Array of dictionary. In each dictionary, the key is the token and the value is the prob. |

#### Example

The following is an example response:

```json
{
    "id": "12345678-1234-1234-1234-abcdefghijkl",
    "object": "chat.completion",
    "created": 2012359,
    "model": "",
    "choices": [
        {
            "index": 0,
            "finish_reason": "stop",
            "message": {
                "role": "assistant",
                "content": "Sure, I\'d be happy to help! The translation of ""I love programming"" from English to Italian is:\n\n""Amo la programmazione.""\n\nHere\'s a breakdown of the translation:\n\n* ""I love"" in English becomes ""Amo"" in Italian.\n* ""programming"" in English becomes ""la programmazione"" in Italian.\n\nI hope that helps! Let me know if you have any other sentences you\'d like me to translate."
            }
        }
    ],
    "usage": {
        "prompt_tokens": 10,
        "total_tokens": 40,
        "completion_tokens": 30
    }
}
```
   
## Deploy Llama 2 models to real-time endpoints

Llama 2 models can be deployed to real-time endpoints in Azure Machine Learning studio. When deployed to real-time endpoints, you can select all the details about the infrastructure running the model including the virtual machines used to run it and the number of instances to handle the load you're expecting. Models deployed in this modality consume quota from your subscription. All the models in the Llama family can be deployed to real-time endpoints.

### Create a new deployment

# [Studio](#tab/azure-studio)

Follow the steps below to deploy a model such as `Llama-2-7b-chat` to a real-time endpoint in [Azure ML Studio](https://ml.azure.com).

1.  Choose the worspace you want to deploy the model in.
2.  Choose a model you want to deploy from the ML Studio [model catalog](change - add link). Alternatively, you can initiate deployment by selecting **Create** from `your workspace`>`endpoints`>`serverless endpoints`
3.  On the detail page, select **Deploy** and then **Real-time endpoint**.
4.  Select if you want to enable **Azure AI Content Safety (preview)**.

    > [!TIP]
    > Deploying Llama 2 models with Azure AI Content Safety (preview) is currently only supported using the Python SDK. (change - check)
5. Select **Proceed**. .

    > [!TIP]
    > If you don't have enough quota available in the selected project, you can use the option **I want to use shared quota and I acknowledge that this endpoint will be deleted in 168 hours**.
6. Select the **Virtual machine** and the instance count you want to assign to the deployment.

7. Select if you want to create this deployment as part of a new endpoint or an existing one. Endpoints can host multiple deployments while keeping resources configuration exclusive for each of them. Deployments under the same endpoint share the endpoint URI and its access keys.

8. Indicate if you want to enable **Inferencing data collection (preview)** or **Package Model (preview)**.
9. Select **Deploy**.
10. You land on the deployment details page. Select **Consume** to obtain code samples that can be used to consume the deployed model in your application. 

# [Python SDK](#tab/python) (change - check)

### Consuming Llama 2 models deployed to real-time endpoints

For reference about how to invoke Llama 2 models deployed to real-time endpoints, see the model card in the Azure Machine Learning Studio [model catalog](change - add link).

## Cost and quotas

### Considerations for Llama 2 models deployed as a service

Llama models deployed as a service are offered by Meta through the Azure Marketplace and integrated with Azure Machine Learning Studio for use. You can find the Azure Marketplace pricing when deploying or fine tuning models.

Each time a workspace subscribes to a given offer from the Azure Marketplace, a new resource is created to track the costs associated with its consumption. The same resource is used to track costs associated with inference and fine tuning, However, multiple meters are available to track each scenario independently. See (change - check link) [monitor costs for models offered throughout the Azure Marketplace](./costs-plan-manage.md#monitor-costs-for-models-offered-through-the-azure-marketplace) to learn more about how to track costs.

(change - use same image as AI Studio

:::image type="content" source="../media/cost-management/marketplace/costs-model-as-service-cost-details.png" alt-text="A screenshot showing different resources corresponding to different model offers and their associated meters."  lightbox="../media/cost-management/marketplace/costs-model-as-service-cost-details.png":::
)

Quota is managed per deployment. Each deployment has a rate limit of 200,000 tokens per minute and 1,000 API requests per minute. However, we currently limit one deployment per model per project. Contact Microsoft Azure Support if the current rate limits don’t suffice your scenarios.

### Considerations for Llama 2 models deployed as real-time endpoints

Deploying Llama models and inferencing with real-time endpoints can be done by consuming Virtual Machine (VM) core quota that is assigned to your subscription a per-region basis. When you sign up for Azure Machine Learning Studio, you receive a default VM quota for several VM families available in the region. You can continue to create deployments until you reach your quota limit. Once that happens, you can request for quota increase.  

## Content filtering

Models deployed as a service with pay-as-you-go are protected by Azure AI Content Safety. When deployed to real-time endpoints, you can opt out for this capability. Both the prompt and completion are run through an ensemble of classification models aimed at detecting and preventing the output of harmful content. The content filtering system detects and takes action on specific categories of potentially harmful content in both input prompts and output completions. Learn more about [Azure AI Content Safety](../concepts/content-filtering.md)(change - check link).

## Next steps

- Learn more about what you can do in [Azure ML Studio](change - add link to what is azure ML studio kind of page)
- Get answers to frequently asked questions in the [Azure ML Studio FAQ article](change - add link)

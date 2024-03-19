---
title: "Quickstart: Groundedness detection (preview)"
titleSuffix: Azure AI services
description: Learn how to detect whether the text responses of large language models (LLMs) are grounded in the source materials provided by the users.
services: ai-services
author: PatrickFarley
manager: nitinme
ms.service: azure-ai-content-safety
ms.topic: quickstart
ms.date: 03/18/2024
ms.author: pafarley
---

# Quickstart: Groundedness detection 

Follow this guide to use Azure AI Content Safety Groundedness detection to check whether the text responses of large language models (LLMs) are grounded in the source materials provided by the users.

## Prerequisites

* An Azure subscription - [Create one for free](https://azure.microsoft.com/free/cognitive-services/) 
* Once you have your Azure subscription, <a href="https://aka.ms/acs-create"  title="Create a Content Safety resource"  target="_blank">create a Content Safety resource </a> in the Azure portal to get your key and endpoint. Enter a unique name for your resource, select your subscription, and select a resource group, supported region (East US2, West US, Sweden Central), and supported pricing tier. Then select **Create**.
   * The resource takes a few minutes to deploy. After it does, go to the new resource. In the left pane, under **Resource Management**, select **API Keys and Endpoints**. Copy one of the subscription key values and endpoint to a temporary location for later use.
* [cURL](https://curl.haxx.se/) or [Python](https://www.python.org/downloads/) installed.

## Test groundedness

#### [cURL](#tab/curl)

The following section walks through a sample request with cURL. Paste the following command into a text editor, and make the following changes.

1. Replace `<endpoint>` with the endpoint URL associated with your resource.
1. Replace `<your_subscription_key>` with one of the keys for your resource.
1. Optionally, replace the `"query"` or `"text"` fields in the body with your own text you'd like to analyze.


```shell
curl --location --request POST '<endpoint>/contentsafety/text:detectGroundedness?api-version=2024-02-15-preview' \
--header 'Ocp-Apim-Subscription-Key: <your_subscription_key>' \
--header 'Content-Type: application/json' \
--data-raw '{
  "domain": "Generic",
  "task": "QnA",
  "qna": {
    "query": "How much does she currently get paid per hour at the bank?"
  },
  "text": "12/hour",
  "groundingSources": [
    "I'm 21 years old and I need to make a decision about the next two years of my life. Within a week. I currently work for a bank that requires strict sales goals to meet. IF they aren't met three times (three months) you're canned. They pay me 10/hour and it's not unheard of to get a raise in 6ish months. The issue is, **I'm not a salesperson**. That's not my personality. I'm amazing at customer service, I have the most positive customer service \"reports\" done about me in the short time I've worked here. A coworker asked \"do you ask for people to fill these out? you have a ton\". That being said, I have a job opportunity at Chase Bank as a part time teller. What makes this decision so hard is that at my current job, I get 40 hours and Chase could only offer me 20 hours/week. Drive time to my current job is also 21 miles **one way** while Chase is literally 1.8 miles from my house, allowing me to go home for lunch. I do have an apartment and an awesome roommate that I know wont be late on his portion of rent, so paying bills with 20hours a week isn't the issue. It's the spending money and being broke all the time.\n\nI previously worked at Wal-Mart and took home just about 400 dollars every other week. So I know i can survive on this income. I just don't know whether I should go for Chase as I could definitely see myself having a career there. I'm a math major likely going to become an actuary, so Chase could provide excellent opportunities for me **eventually**."
  ],
  "reasoning": False
}'
```

Open a command prompt and run the cURL command.


#### [Python](#tab/python)

Create a new Python file named _quickstart.py_. Open the new file in your preferred editor or IDE.

1. Replace the contents of _quickstart.py_ with the following code. Enter your endpoint URL and key in the appropriate fields. Optionally, replace the `"query"` or `"text"` fields in the body with your own text you'd like to analyze.
    
    ```Python
    
    import http.client
    import json
    
    conn = http.client.HTTPSConnection("<Endpoint>/contentsafety/text:detectGroundedness?api-version=2024-02-15-preview")
    payload = json.dumps({
      "domain": "Generic",
      "task": "QnA",
      "qna": {
        "query": "How much does she currently get paid per hour at the bank?"
      },
      "text": "12/hour",
      "groundingSources": [
        "I'm 21 years old and I need to make a decision about the next two years of my life. Within a week. I currently work for a bank that requires strict sales goals to meet. IF they aren't met three times (three months) you're canned. They pay me 10/hour and it's not unheard of to get a raise in 6ish months. The issue is, **I'm not a salesperson**. That's not my personality. I'm amazing at customer service, I have the most positive customer service \"reports\" done about me in the short time I've worked here. A coworker asked \"do you ask for people to fill these out? you have a ton\". That being said, I have a job opportunity at Chase Bank as a part time teller. What makes this decision so hard is that at my current job, I get 40 hours and Chase could only offer me 20 hours/week. Drive time to my current job is also 21 miles **one way** while Chase is literally 1.8 miles from my house, allowing me to go home for lunch. I do have an apartment and an awesome roommate that I know wont be late on his portion of rent, so paying bills with 20hours a week isn't the issue. It's the spending money and being broke all the time.\n\nI previously worked at Wal-Mart and took home just about 400 dollars every other week. So I know i can survive on this income. I just don't know whether I should go for Chase as I could definitely see myself having a career there. I'm a math major likely going to become an actuary, so Chase could provide excellent opportunities for me **eventually**."
      ],
      "reasoning": False
    })
    headers = {
      'Ocp-Apim-Subscription-Key': '<your_subscription_key>',
      'Content-Type': 'application/json'
    }
    conn.request("POST", "/contentsafety/text:detectGroundedness?api-version=2024-02-15-preview", payload, headers)
    res = conn.getresponse()
    data = res.read()
    print(data.decode("utf-8"))
    ```

    > [!IMPORTANT]
    > Remember to remove the key from your code when you're done, and never post your key publicly. For production, use a secure way of storing and accessing your credentials. For more information, see [Azure Key Vault](/azure/key-vault/general/overview).

1. Run the application with the `python` command:

    ```console
    python quickstart.py
    ````

Wait a few moments to get the response.

---

> [!TIP]
> To test a summarization task instead of QnA, use the following sample JSON body:
>
> ```json
> {
>     "Domain": "Medical",
>     "Task": "Summarization",
>     "Text": "Ms Johnson has been in the hospital after experiencing a stroke.",
>     "GroundingSources": ["Our patient, Ms. Johnson, presented with persistent fatigue, unexplained weight loss, and frequent night sweats. After a series of tests, she was diagnosed with Hodgkin’s lymphoma, a type of cancer that affects the lymphatic system. The diagnosis was confirmed through a lymph node biopsy revealing the presence of Reed-Sternberg cells, a characteristic of this disease. She was further staged using PET-CT scans. Her treatment plan includes chemotherapy and possibly radiation therapy, depending on her response to treatment. The medical team remains optimistic about her prognosis given the high cure rate of Hodgkin’s lymphoma."],
>     "Reasoning": false
> }
> ```


The following fields must be included in the URL:

| Name     | Required? | Description | Type   |
| :-------------- | :-------- | :------ | :----- |
| **API Version** | Required  | This is the API version to be used. The current version is: api-version=2024-02-15-preview. Example: `<endpoint>/contentsafety/text:shieldPrompt?api-version=2024-02-15-preview` | String |

The parameters in the request body are defined in this table:

| Name  | Description     | Type    |
| :----------- | :--------- | ------- |
| **Domain** | (Optional) `MEDICAL` or `GENERIC`. Default value: `GENERIC`. | Enum  |
| **Task** | (Optional) Type of task: `QnA`, `Summarization`. Default value: `Summarization`. | Enum |
| **qna**       | (Optional) This parameter is only used when the task type is QnA.  | String  |
| - `query`       | (Optional) This is used to submit a question or a query in a Questions and Answers task. Character limit: 7,500. | String  |
| **Text**   | (Required) The text that needs to be checked. Character limit: 7500. |  String  |
| **GroundingSources**  | (Required) Uses an array of grounding sources to validate AI-generated text. Restrictions on the total number of grounding sources that can be analyzed in a single request are 55K characters. | String array    |
| **Reasoning**  | (Optional) Specifies whether to use the reasoning feature. The default value is `False`. If `True`, the service uses our default GPT resources to provide an explanation and include the "ungrounded" sentence. Be careful: using reasoning increases the processing time and incur extra fees.| Boolean   |
| **llmResource**  | (Optional) If you want to use your own GPT resources instead of our default GPT resources, add this field manually and include the subfields for the GPT resources used. If you don't want to use your own GPT resources, remove this field from the input. | String   |
| - `resourceType `| Specifies the type of resource being used, for this version, only allows: `AzureOpenAI`. | Enum|
| - `azureOpenAIEndpoint `| Endpoint URL for Azure's OpenAI service.  | String |
| - `azureOpenAIDeploymentName` | Name of the specific deployment to use. | String|


## Use the reasoning feature

The Groundedness detection API provides the option to include _reasoning_ in the API response. When this feature is enabled, the response includes a `"reasoning"` value. This value details specific instances and explanations for any detected ungroundedness. If you opt to use reasoning, you must either provide your own GPT resources or use the service's default GPT resources. Be careful: using reasoning increases the processing time and incur extra fees. 

If you choose not to use reasoning, the API only classifies the ungroundedness of the submitted content as `true` or `false` and provide a confidence score.

### Enable reasoning with your own GPT resource

You can use your existing Azure OpenAI resource to power the reasoning feature. Use Managed Identity to allow your Content Safety resource to access the Azure OpenAI resource:

1. Enable Managed Identity for Azure AI Content Safety.

    Navigate to your Azure AI Content Safety instance in the Azure portal. Find the **Identity** section under the **Settings** category. Enable the system-assigned managed identity. This action grants your Azure AI Content Safety instance an identity that can be recognized and used within Azure for accessing other resources. 
    
    :::image type="content" source="media/content-safety-identity.png" alt-text="Screenshot of a Content Safety identity resource in the Azure portal.":::

1. Assign Role to Managed Identity.

    Navigate to your Azure OpenAI instance, select **Add role assignment** to start the process of assigning an Azure OpenAI role to the Azure AI Content Safety identity. Choose the **User** role.

    :::image type="content" source="media/add-role-assignment.png" alt-text="Screenshot of adding role assignment in Azure portal.":::

    :::image type="content" source="media/assigned-roles-simple.png" alt-text="Screenshot of the Azure portal with the Contributor and User roles displayed in a list.":::

1. In your request to the Groundedness detection API, set the `"Reasoning"` body parameter to `true`, and provide the other needed parameters:

```json
{
    ...
    "Reasoning": true,
    "llmResource": {
        "resourceType": "AzureOpenAI",
        "azureOpenAIEndpoint": "<your_OpenAI_endpoint>",
        "azureOpenAIDeploymentName": "<your_deployment_name>"
    }
}
```

## Interpret the API response

After you submit your request, you'll receive a JSON response reflecting the Groundedness analysis performed. Here’s what a typical output looks like: 

```json
{
    "ungrounded": true,
    "confidenceScore": 1,
    "ungroundedPercentage": 1,
    "ungroundedDetails": [
     {
 "text": "string",
 "offset": {
 "utf8": 0,
 "utf16": 0,
 "codePoint": 0
      },
 "length": {
 "utf8": 0,
 "utf16": 0,
 "codePoint": 0
      },
      "reason": "string"
    }
  ]
}
```

The JSON objects in the output are defined here:

| Name  | Description    | Type    |
| :------------------ | :----------- | ------- |
| **ungrounded** | Indicates whether the text exhibits ungroundedness.  | Boolean    |
| **confidenceScore** | The confidence value of the _ungrounded_ designation. The score ranges from 0 to 1.	 | Float	 |
| **ungroundedPercentage** | Specifies the proportion of the text identified as ungrounded, expressed as a number between 0 and 1, where 0 indicates no ungrounded content and 1 indicates entirely ungrounded content.| Float	 |
| **ungroundedDetails** | Provides insights into ungrounded content with specific examples and percentages.| String |
| -**`Text`**   |  The specific text that is ungrounded.  | String   |
| -**`offset`**   |  An object describing the position of the ungrounded text in various encoding.  | String   |
| - `offset > utf8`       | The offset position of the ungrounded text in UTF-8 encoding.      | number   |
| - `offset > utf16`      | The offset position of the ungrounded text in UTF-16 encoding.       | number |
| - `offset > codePoint`  | The offset position of the ungrounded text in terms of Unicode code points. |number    |
| -**`length`**   |  An object describing the length of the ungrounded text in various encoding. (utf8, utf16, codePoint), similar to the offset. | String   |
| - `length > utf8`       | The length of the ungrounded text in UTF-8 encoding.      | number   |
| - `length > utf16`      | The length of the ungrounded text in UTF-16 encoding.       | number |
| - `length > codePoint`  | The length of the ungrounded text in terms of Unicode code points. |number    |
| -**`Reason`** |  Offers explanations for detected ungroundedness. | String  |

## Clean up resources

If you want to clean up and remove an Azure AI services subscription, you can delete the resource or resource group. Deleting the resource group also deletes any other resources associated with it.

- [Portal](/azure/ai-services/multi-service-resource?pivots=azportal#clean-up-resources)
- [Azure CLI](/azure/ai-services/multi-service-resource?pivots=azcli#clean-up-resources)

## Next steps

Combine Groundedness detection with other LLM safety features like Prompt Shields.

> [!div class="nextstepaction"]
> [Prompt Shields quickstart](./quickstart-jailbreak.md)

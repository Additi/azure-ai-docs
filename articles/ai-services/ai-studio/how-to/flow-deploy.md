---
title: Deploy a flow as a managed online endpoint for real-time inference
titleSuffix: Azure AI services
description: Learn how to deploy a flow as a managed online endpoint for real-time inference with Azure AI Studio.
author: eric-urban
manager: nitinme
ms.service: azure-ai-services
ms.topic: how-to
ms.date: 10/1/2023
ms.author: eur
---

# Deploy a flow as a managed online endpoint for real-time inference

[!INCLUDE [Azure AI Studio preview](../includes/preview-ai-studio.md)]

After you build a flow and test it properly, you may want to deploy it as an endpoint so that you can invoke the endpoint for real-time inference.

In this article, you'll learn how to deploy a flow as a managed online endpoint for real-time inference. The steps you'll take are:

- Test your flow and get it ready for deployment
- Create an online deployment
- Grant permissions to the endpoint
- Test the endpoint
- Consume the endpoint

## Build the flow and get it ready for deployment

By now you've already tested the flow properly by submitting bulk tests and evaluating the results.

If you didn't complete the tutorial, you need to build a flow. Testing the flow properly by bulk tests and evaluation before deployment is a recommended best practice.

We'll use the sample flow **Web Classification** as example to show how to deploy the flow. This sample flow is a standard flow. Deploying chat flows is similar. Evaluation flow doesn't support deployment.

## Define the environment used by deployment

When you deploy prompt flow to managed online endpoint in UI, by default the deployment will use the environment created based on the latest prompt flow image and dependencies specified in the `requirements.txt` of the flow. You can specify extra packages you needed in `requirements.txt`. You can find `requirements.txt` in the root folder of your flow folder.



## Prerequisites

In order to make the chat playground to respond to your query, you must grant permissions to the endpoint entity after the promptflow deployment is created. This is a subscription owner level action, so if needed, ask your subscription owner to do it for you. 

## Create an online deployment

Now that you have built a flow and tested it properly, it's time to create your online endpoint for real-time inference. 

The Prompt flow supports you to deploy endpoints from a flow, or a bulk test run. Testing your flow before deployment is recommended best practice.


### If you're using AI Studio UI:
1. Follow [the promptflow instruction](https://github.com/Azure/azureai-insiders/blob/aistudio-preview/previews/aistudio/how-to/build_with_promptflow.md) to create a promptflow.
2. Select **Deploy** on the flow editor.
3. Once you're redirected to the deployment details page, **look for the endpoint name** in URL (`EndpointName.region.inference.ml.azure.com/score`). You need this for step 9 (enabling access to secrets).
4. Go to Project details page (`Projects` > `Details`).
5. Select the **YourResourceGroupName** link on the Details page.
6. Once you're redirected to the Azure Resource Group page, Select **Access control (IAM)** on the left navigation menu.
7. Select **Add role assignment**.
8. Select **Azure ML Data Scientist** and select **Next**.
9. Select **+ select members** and search for your endpoint name. Tip: use your project name as a search keyword to find the endpoint quickly. 
10. Select **Select**.
11. Select **Review + Assign**.
12. Return to AI Studio and go to the deployment details page (`Deployments` > `YourDeploymentName`).
13. Test the promptflow deployment (`YourDeploymentName` > `Test`)


## Check the status of the endpoint

There will be notifications after you finish the deploy wizard. After the endpoint and deployment are created successfully, you can select **Deploy details** in the notification to endpoint detail page.

You can also directly go to the **Endpoints** page in the studio, and check the status of the endpoint you deployed.


## Test the endpoint with sample data

In the endpoint detail page, switch to the **Test** tab.

If you select **Allow sharing sample input data for testing purpose only** when you deploy the endpoint, you can see the input data values are already preloaded.

If there's no sample value, you'll need to input a URL.

The **Test result** shows as following: 


### Test the endpoint deployed from a chat flow

For endpoints deployed from chat flow, you can test it in an immersive chat window.


The `chat_input` was set during development of the chat flow. You can input the `chat_input` message in the input box. The **Inputs** panel on the right side is for you to specify the values for other inputs besides the `chat_input`. Learn more about [how to develop a chat flow](./flow-develop.md#develop-a-chat-flow).

## Consume the endpoint

In the endpoint detail page, switch to the **Consume** tab. You can find the REST endpoint and key/token to consume your endpoint. There is also sample code for you to consume the endpoint in different languages.


## Clean up resources

If you aren't going use the endpoint after completing this tutorial, you should delete the endpoint.

> [!NOTE]
> The complete deletion may take approximately 20 minutes.

## Next Steps

- Learn more about what you can do in [Azure AI Studio](../what-is-ai-studio.md)
- Get answers to frequently asked questions in the [Azure AI FAQ article](../faq.yml)

---
title: Using Azure AI Studio with a screen reader
titleSuffix: Azure AI services
description: This tutorial guides you through using Azure AI Studio with a screen reader.
author: eric-urban
manager: nitinme
ms.service: azure-ai-services
ms.topic: tutorial
ms.date: 10/1/2023
ms.author: eur
---

# Tutorial: Using Azure AI Studio with a screen reader

[!INCLUDE [Azure AI Studio preview](../includes/preview-ai-studio.md)]

This article is for people who use screen readers such as Microsoft's Narrator, JAWS, NVDA or Apple's Voiceover, and provides guidance on how to use the Azure AI Studio with a screen reader.   

## Getting started in the Azure AI Studio 

Most Azure AI Studio pages are composed of the following structure: 

- Banner (contains Azure AI Studio app title, settings and profile information) 
- Primary navigation (contains Home, Explore, Build, and Manage) 
- Secondary navigation 
- Main page content 
    - Contains a breadcrumb navigation element 
    - Usually also contains a command toolbar 

For efficient navigation, it may be helpful to navigate by landmarks to move between these sections on the page.

## Explore 

In **Explore** you can explore the different capabilities of Azure AI before creating a project. You can find this in the primary navigation landmark.

Within **Explore**, you will be able to explore a number of capabilities found within the secondary navigation. These include model catalog, model leaderboard, and pages for Azure AI services such as Speech, Vision, and Content Safety. 
- Model catalog contains three main areas: Announcements, Models and Filters. You can use Search and Filters to narrow down model selection 
- Azure AI service pages such as Speech consist of a number of cards containing links. These lead you to demo experiences where you can sample our AI capabilities and may link out to another webpage. 

## Projects 

To work within the Azure AI Studio, you must first create a project: 
1. Navigate to the Build tab in the primary navigation.
1. Press the Tab key until you hear *New project* and select this button.  
1. Enter the information requested in the **Create a new project** dialog.  

You'll then get taken to the project details page. 

Within a project, you will be able to explore a number of capabilities found within the secondary navigation. These include playground, prompt flow, evaluation, and deployments. The secondary navigation contains an H2 heading with the project title, which can be used for efficient navigation.

## Using the playground 

The playground is where you can chat with models and experiment with different prompts and parameters.  

From the **Build** tab, navigate to the secondary navigation landmark and press the down arrow until you hear *playground*.  

### Playground structure 

When you first arrive the playground mode dropdown is set to **Chat** by default. In this mode the playground is composed of the command toolbar and three main panes: **Assistant setup**, **Chat session**, and **Configuration**. If you have added your own data in the playground, the **Citations** pane will also appear when selecting a citation as part of the model response. 

You can navigate by heading to move between these panes, as each pane has its own H2 heading. 

### Assistant setup pane 

This is where you can setup the chat assistant according to your organization's needs. 

Note that once you edit the system message or examples, your changes do not save automatically. Press the **Save changes** button to ensure your changes are saved. 

### Chat session pane  

This is where you can chat to the model and test out your assistant 
- After you send a message, the model may take some time to respond, especially if the response is long. You will hear a screen reader announcement "Message received from the chatbot" when the model has finished composing a response.  
- Content in the chatbot will follow this format: 

    ```
    [message from user] [user image] 
    [chatbot image] [message from chatbot] 
    ```


## Using Prompt flow 

Prompt flows is a tool to create executable flows, linking LLMs, prompts and Python tools through a visualized graph. You can use this to prototype, experiment and iterate on your AI applications before deploying.  

With the Build tab selected, navigate to the secondary navigation landmark and press the down arrow until you hear *flows*.  

The Prompt flow UI in Azure AI Studio is composed of the following main sections: Command toolbar, Flow (includes list of the flow nodes), Files and the Graph view. The Flow, Files and Graph sections each have their own H2 headings which can be used for navigation.


### Flow 

- This is the main working area where you can edit your flow, for example adding a new node, editing the prompt, selecting input data 
- You can also choose to work in code instead of the editor by navigating to the **Raw file mode** toggle button to view the flow in code. 
- You can also open your flow in VS Code Web by selecting the **Work in VS Code Web** button.
- Each node has its own H3 heading, which can be used for navigation.  

### Files 

- This section contains the file structure of the flow. Each flow has a folder that contains a flow.dag.yaml file, source code files, and system folders.  
- You can export or import a flow easily for testing, deployment, or collaborative purposes by navigating to the **Add** and **Zip and download all files** buttons.

### Graph view 

- The graph is a visual representation of the flow. This view is not editable or interactive. 
- You will hear the following alt text to describe the graph: “ "Graph view of [flow name] – for visualization only." Note that we do not currently provide a full screen reader description for this graphical chart. To get all equivalent information, you can read and edit the flow by navigating to Flow, or by toggling on the Raw file view.  

 
### Evaluations  

Evaluation is a tool to help you evaluate the performance of your generative AI application. You can use this to prototype, experiment and iterate on your applications before deploying.

### Creating an evaluation 

To review evaluation metrics, you must first create an evaluation.  

1. Navigate to the Build tab in the primary navigation.
1. Navigate to the secondary navigation landmark and press the down arrow until you hear *evaluations*.
1. Press the Tab key until you hear *new evaluation* and select this button.  
1. Enter the information requested in the **Create a new evaluation** dialog. Once complete, your focus will be returned to the evaluations list. 

### Viewing evaluations 

Once you have created an evaluation, you can access it from the list of evaluations.  

Evaluation runs are listed as links within the Evaluations grid. Selecting a link will take you to a dashboard view with information about your specific evaluation run. 

You may prefer to export the data from your evaluation run so that you can view it in an application of your choosing. To do this, select your evaluation run link, then navigate to the **Export results** button and select it. 

There is also a dashboard view provided to allow you to compare evaluation runs. From the main Evaluations list page, navigate to the **Switch to dashboard view** button. You can also export all this data using the **Export table** button. 

 
## Technical support for customers with disabilities 

Microsoft wants to provide the best possible experience for all our customers. If you have a disability or questions related to accessibility, please contact the Microsoft Disability Answer Desk for technical assistance. The Disability Answer Desk support team is trained in using many popular assistive technologies and can offer assistance in English, Spanish, French, and American Sign Language. Please go to the Microsoft Disability Answer Desk site to find out the contact details for your region. 

If you are a government, commercial, or enterprise customer, please contact the enterprise Disability Answer Desk. 

## Next steps
* Learn how you can build generative AI applications in the [Azure AI Studio](../what-is-ai-studio.md).
* Get answers to frequently asked questions in the [Azure AI FAQ article](../what-is-ai-studio.md).


---
title: How to use the Azure Machine Learning data labeling tool
title.suffix: Azure Machine Learning
description: This article teaches you how to use the data tagging tools in an Azure Machine Learning labeling project.
author: lobrien
ms.author: laobri
ms.service: machine-learning
ms.topic: tutorial
ms.date: 11/04/2019

---

# How to tag images in a labeling project

After your project administrator creates a labeling project in Azure Machine Learning Studio, you can use the labeling tool to rapidly prepare data for a machine learning project. This article describes:

> [!div class="checklist"]
> * How to access your labeling projects.
> * The labeling tools.
> * How to use the tools for specific labeling tasks.

## Prerequisites

* The labeling portal URL for a running data labeling project
* A [Microsoft account](https://account.microsoft.com/account) or an Azure Active Directory account for the organization and project

> [!NOTE]
> The project administrator can find the labeling portal URL on the **Details** tab of the **Project details** page.

## Sign in to the project's labeling portal

Go to the labeling portal URL provided by the project administrator. Sign in by using the email account that the administrator used to add you to the team. For most users, logging in will be done with your Microsoft account. If the labeling project uses Azure Active Directory, that's how you'll sign in.

## Understand the labeling task

After you sign in, you'll see the project's overview page.

Go to **View detailed instructions**. These instructions are specific to your project and explain the type of data you're facing, how you should make your decisions, and other relevant information. After you read this information, return to the project page and select **Start labeling**.

## Common features of the labeling task

All image-labeling tasks involve choosing an appropriate tag or tags from a set specified by the project administrator. You can select among the first nine tags by using the number keys on your keyboard.  

You can use image classification tasks to present multiple images simultaneously. Use the icons above the image area to select the layouts. To select all the displayed images simultaneously, use the **Select all** button. To select individual photos, use the circular selection button in the upper-right corner of the image. You must select at least one image to apply a tag. If you select multiple images, any tag that you select will be applied to all the selected photos.

Here, we've chosen a two-by-two layout and are about to apply the tag "Mammal" to the images of the bear and orca. The image of the shark was already tagged as "Cartilaginous fish", and the iguana hasn't yet been tagged.

![Multiple image layouts and selection](media/how-to-label-images/layouts.png)

> [!Important] 
> Only switch layouts when you have a fresh page of unlabeled data. Switching layouts clears the page's in-progress tagging work. 

Azure enables the **Submit** button when you've tagged all the images on the page. Select **Submit** to save your work.

After you submit tags for the data at hand, Azure refreshes the page with a new set of images from the work queue.

## Tag images for multi-class classification

If your project is of type "Image Classification Multi-Class," you'll assign a single tag to the entire image. At any time, go to the **Instructions** page and select **View detailed instructions** to review the guidance.

If you realize you made a mistake after you assign a tag to an image, you can fix it. Select the "**X**" on the label that's displayed below the image to clear the tag. Or,  select the image and choose another class, and the newly selected value will replace the previously applied tag.

## Tag images for multi-label classification

If you're working on a project of type "Image Classification Multi-Label", you'll apply one *or more* tags to an image. At any time, select **Instructions** and go to **View detailed instructions** to see the project-specific guidance.

Select the image that you want to label and then select the tag. The tag is applied it to all the selected images, and the images are deselected. To apply more tags, you must reselect the images. The following animation shows multi-label tagging:

1. **Select all** is used to apply the "Ocean" tag,
1. A single image is selected and tagged "Closeup."
1. Three images are selected and tagged "Wide angle."

![Animation showing multilabel flow](media/how-to-label-images/multilabel.gif)

To correct a mistake, click the "**X**" to clear an individual tag or select the images and then choose the tag, which clears the tag from all the selected images. In this scenario, clicking  "Land" will clear that tag from the two selected images.

![Screenshot showing multiple deselections](media/how-to-label-images/multiple-deselection.png)

Azure will only enable the **Submit** button after you've applied at least one tag to each image. Press **Submit** for your work to be saved.

## Use bounding boxes for object detection

If your project is of type "Object Identification (Bounding Boxes)", you'll specify one or more bounding boxes in the image and apply a tag to that box. Image can have multiple bounding boxes, each with a single tag. Use **View detailed instructions** to determine if adding multiple bounding boxes is appropriate to your project.

1. Select a tag for the bounding box that you plan to create.
1. Select the **Rectangular box** tool ![Rectangular box tool](media/how-to-label-images/rectangular-box-tool.png) or select "R." 
3. Click and drag diagonally across your target to create a rough bounding box. To adjust the bounding, click and drag the edges or corners of the box.

![Screenshot shows basic bounding box creation.](media/how-to-label-images/bounding-box-sequence.png)

To delete a bounding box, click the X-shaped target that appears next to the bounding box after creation.

You may can't change the tag of an existing bounding box. If you make a tag assignment mistake, you have to delete the bounding box and create a new one with the correct tag.

By default, existing bounding boxes can be edited. The **Lock/unlock regions** tool ![Lock/unlock regions tool](media/how-to-label-images/lock-bounding-boxes-tool.png) or "L" toggles that behavior. If regions are locked, you can only change the shape or location of a new bounding box.

Use the **Regions manipulation** tool ![Regions manipulation tool](media/how-to-label-images/regions-tool.png) or "M" to adjust an existing bounding box. Click and drag on edges or corners to adjust the shape. Click in the interior to be able to drag the whole bounding box. If you can't edit a region, you've probably toggled the **Lock/unlock regions** tool.

Use the **Template-based box** tool ![Template-box tool](media/how-to-label-images/template-box-tool.png) or "T" to create multiple bounding boxes of the same size. If the image has no bounding boxes and you activate template-based boxes, the tool will produce 50 x 50 pixel boxes. If you create a bounding box and then activate template-based boxes, new bounding boxes will be the size of the last one you created. Template-based boxes can be resized after placement. Resizing a template-based box only resizes the particular box.

To delete _all_ the bounding boxes in the current image, select the **Delete all regions** tool ![Delete regions tool](media/how-to-label-images/delete-regions-tool.png).

After you created the bounding boxes for an image, select **Submit** to save your work, or you work in progress won't be saved.

## Finish up 

When you submit a page of tagged data, Azure assigns you new unlabeled data from a work queue. If there's no more unlabeled data available, you'll get a message noting this along with a link to the portal home page.

When you're done labeling, select your name in the upper-right corner of the labeling portal and then select **Signout**. If you don't sign out, eventually Azure will "time you out" and assign your data to another labeler.

## Next steps

* Learn to [train image classification models in Azure](https://docs.microsoft.com/azure/machine-learning/service/tutorial-train-models-with-aml)
* Read about [object detection using Azure and the Faster R-CNN technique](https://www.microsoft.com/developerblog/2017/10/24/bird-detection-with-azure-ml-workbench/)
---
title: Add labeler access to your data labeling project
title.suffix: Azure Machine Learning
description: Grant users access to your data labeling project so that they can label data, but not see the rest of your workspace.
author: sdgilley
ms.author: sgilley
ms.reviewer: vkann
ms.service: machine-learning
ms.subservice: mldata
ms.topic: how-to
ms.date: 11/05/2021
---

# Add labeler access to your data labeling project

This article shows how to grant users access to your data labeling project so that they can label data, but not see the rest of your workspace.  You can use these steps for anyone, whether or not they are from [data labeling vendor company](how-to-outsource-data-labeling.md).  
  
## Prerequisites

* An Azure subscription. If you don't have an Azure subscription [create a free account](https://azure.microsoft.com/free) before you begin.
* An Azure Machine Learning workspace. See [Create an Azure Machine Learning workspace](../articles/machine-learning/how-to-manage-workspace.md).

## Add custom role

To add a custom role, you must have `Microsoft.Authorization/roleAssignments/write` permissions for your subscription, such as [User Access Administrator](../../articles/role-based-access-control/built-in-roles.md#user-access-administrator) or [Owner](../../articles/role-based-access-control/built-in-roles.md#owner).

1. Open your workspace in [Azure Machine Learning studio](https://ml.azure.com)
1. Open the menu on the top right and select **View all properties in Azure Portal**. 
1. Select the **Subscription** link in the middle of the page.
1. On the left, select **Access control (IAM)**.
1. At the top, select **+ Add > Add custom role**.
1. For the **Custom role name**, type **Labeler**.
1. In the **Description** box, add **Labeler access for data labeling projects**.
1. Select **Start from JSON**.
1. At the bottom of the page, select **Next**.
1. Don't do anything for the **Permissions** tab, you'll add permissions in a later step.  Select **Next**.
1. The **Assignable scopes** tab shows your subscription information.  Select **Next**.
1. In the **JSON** tab, above the edit box, select **Edit**.
1. Select lines starting with **"actions:"** and **"notActions:"**.

    :::image type="content" source="media/how-to-add-labeler/replace-lines.png" alt-text="Select lines to replace them in the editor.":::

1. Replace these two lines with:
    
    ```json
    "actions": [
        "Microsoft.MachineLearningServices/workspaces/read",
        "Microsoft.MachineLearningServices/workspaces/labeling/projects/read",
        "Microsoft.MachineLearningServices/workspaces/labeling/labels/write"],
    "notActions": [
        "Microsoft.MachineLearningServices/workspaces/labeling/projects/summary/read"],
    ```

1. Select **Save** at the top of the edit box to save your changes.

    > [!IMPORTANT]
    > Don't select **Next** until you've saved your edits.

1. After you save your edits, select **Next**.
1. Select **Create** to create the custom role.
1. Select **OK**.

## Add labelers into Azure

If your labelers are outside of your organization, you'll now add them so that they can access your workspace.  If labelers are already inside your organization, skip this step.  

1. In the top-left corner, expand the menu and select **Azure Active Directory**.

    :::image type="content" source="media/how-to-add-labeler/menu-active-directory.png" alt-text="Select Azure Active Directory from the menu.":::

1. On the left, select **Users**.
1. At the top, select **New guest user**.
1. Fill in the name and email address for the user.
1. Add a message for the new user.
1. At the bottom of the page, select **Invite**.

    :::image type="content" source="media/how-to-add-labeler/invite-user.png" alt-text="Invite user from Azure Active Directory.":::

Repeat for each of your labelers.  Or use the link at the bottom of the **Invite user** box to invite multiple users in bulk.

> [!TIP]
> Inform your labelers that they'll be receiving this email.  They need to accept the invitation in order to gain access to your project.

## Add labelers to your workspace

Now that you have your labelers added to the system, you're ready to add them to your workspace.  You'll use the custom role here.

1. In the top search field, type **Machine Learning**.  
1. Select **Machine Learning**.

   ![Search for Azure Machine Learning workspace](./media/how-to-manage-workspace/find-workspaces.png)

1. Select the workspace that contains your data labeling project.
1. On the left, select **Access control (IAM)**.
1. At the top, select **+ Add > Add role assignment**.

    :::image type="content" source="media/how-to-add-labeler/add-role-assignment.png" alt-text="Add role assignment from your workspace.":::

1. Select the **Labeler** role in the list.  Use **Search** if necessary to find it.
1. Select **Next**.
1. In the middle of the page, next to **Members**, select the **+ Select members** link.
1. Select each of the users you added above. Use **Search** if necessary to find them.
1. At the bottom of the page, select the **Select** button.
1. Select **Next**.
1. Verify that the **Role** is listed as **Labeler**, and that your users appear in the **Members** list.
1. Select **Review + assign**.

## For your labelers

Your labelers are now all set up to begin labeling in your project.  But they'll still need information from you to access the project.  

If you haven't created your labeling project yet, do so before you contact your labelers.

    * [Create an image labeling project](how-to-create-image-labeling-projects.md).
    * [Create a text labeling project (preview)](how-to-create-text-labeling-projects.md)

Sent the following to your labelers, after filling in your workspace and project names:

1. Accept the invite from **Microsoft Invitations (invites@microsoft.com)**.
1. If you don't yet have a Microsoft account, you'll be prompted to create one.  If you do have one, sign in.
1. Open [Azure Machine Learning studio](https://ml.azure.com).
1. Use the dropdown to select the workspace **\<workspace-name\>**.  
1. Select the project **\<project-name\>**.
1. Select **Start labeling** at the bottom of the page.
1. For more information about how to create labels, see [Labeling images and text documents](how-to-label-data.md).

## Next steps

Learn more about [working with a data labeling vendor company](how-to-outsource-data-labeling.md)
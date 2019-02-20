---
title: Manage users and roles in an Azure Machine Learning workspace
titleSuffix: Azure Machine Learning service
description: Learn how to manage users and roles in an Azure Machine Learning service workspace.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.reviewer: jmartens
author: hning86
ms.author: haining
ms.date: 2/20/2019
ms.custom: seodec18

---


# Manage users and roles in an Azure Machine Learning workspace

In this article, you learn how to add users to an Azure Machine Learning workspace, and assign them with different roles. You also learn how to create custom roles.

## Built-in roles
Azure Machine Learning workspace is an Azure resource. And like any Azure resource, when a new Azure Machine Learning workspace is created, it comes with three default roles. You can add users to the workspace and assign them to one of the these built-in roles.

1. Reader

    This role allows users to perform all read-only actions in the workspace. With this role, you can list and view assets in a workspace, such as experiments, runs, compute, models, web services, published pipelines etc. But you cannot create or update these assets.

2. Contributor

    This role allows users to view, create, edit or delete (where applicable) assets in a workspace. For example, with this role you can create an experiment, create or attach a compute cluster, submit a run, register a model and deploy a web service.

3. Owner

    This role gives you full access to the workspace, allowing you to view, create, edit or delete (where applicable) assets in a workspace. In addition, you can also add or remove users, and change their role assignments.

## Add or remove users
If you are an Owner of a workspace, you can add users to or remove users from the workspace. You can follow the standard Azure resource management mechanisms to perform this action, including using [Azure Portal UI](/azure/role-based-access-control/role-assignments-portal), [PowerShell](/azure/role-based-access-control/role-assignments-powershell), [Azure CLI](/azure/role-based-access-control/role-assignments-cli), [REST API](/azure/role-based-access-control/role-assignments-rest), or [Azure Resource Management templates](/azure/role-based-access-control/role-assignments-template).

In addition, if you have installed Azure Machine Learning CLI, you can also use a convenient CLI command to add user to the workspace.

```azure-cli
az ml workspace share -n <workspace_name> -g <resource_group_name> --role <role_name> --user <user_corp_email_address>
```

Note the `user` field is the email address of an existing user in the same Azure Active Directory where the parent Azure subscription of the workspace lives. Below is an example of using this command:

```azure-cli
az ml workspace share -n my_workspace -g my_resource_group --role Contributor --user jdoe@contoson.com
```


## Create custom role
If the built-in roles are not sufficient for your needs, you can also create custom roles and apply them to the workspace, the resource group, or even the subscription. 

To create a custom role, first construct a role definition JSON file specifying the permission you want for the role. For example, below is a role definition file for a custom role named "Data Scientist" scoped at a particular workspace level. 


`data_scientist_role.json` :
```json
{
    "Name":  "Data Scientist",
    "Id":  "<unique GUID>",
    "IsCustom":  true,
    "Description":  "Can run experiment but can't create or delete compute.",
    "Actions":  [""],
    "NotActions":  [
        "Microsoft.MachineLearningServices/workspaces/computes/*/write",
        "Microsoft.MachineLearningServices/workspaces/computes/*/delete", 
        "Microsoft.Authorization/*/Write"
    ],
    "AssignableScopes":  [
        "/subscriptions/<subscription_id>/resourceGroups/<resource_group_name>/providers/Microsoft. MachineLearningServices/workspaces/<workspace_name>"
    ]
}
```

This role allows user to do everything in the workspace, except 3 actions are not allowed:
1. It cannot create or edit a compute resource.
2. It cannot delete a compute resource.
3. It cannot add, delete user, or alter any user's role assignments.

To deploy this role, use the following Azure-CLI command:

```azure-cli
az role definition create --role-definition data_scientist_role.json
```

Once deployed, this role becomes available in the specified workspace, and you can add users and assign this role to them. 

You can also change the `AssignableScopes` field to set the scope of this custom role at the subscription level, the resource group level, or at a specific workspace level (as shown).

## Next steps

Follow the full-length tutorial to learn how to use a workspace to build, train, and deploy models with Azure Machine Learning service.

> [!div class="nextstepaction"]
> [Tutorial: Train models](tutorial-train-models-with-aml.md)

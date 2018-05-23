---
title: View, export, delete your data from Azure AI Gallery | Microsoft Docs
description: You can export and delete your in-product user data from Azure AI Gallery using the interface or AI Gallery Catalog API. This article shows you how.
services: machine-learning
author: heatherbshapiro
ms.author: hshapiro
manager: cgronlun
ms.reviewer: jmartens, mldocs
ms.service: machine-learning
ms.component: studio
ms.topic: conceptual
ms.date: 05/25/2018
---

# View, export, and delete in-product user data from Azure AI Gallery

You can export and delete your in-product user data from Azure AI Gallery using the interface or AI Gallery Catalog API. This article shows you how.

[!INCLUDE [GDPR-related guidance](../../includes/gdpr-intro-sentence.md)]

## View your data in AI Gallery with the UI

You can view items you published through the Azure AI Gallery website UI. Users can view both public and unlisted solutions, projects, experiments, and other published items:

1.	Sign in to the [Azure AI Gallery](https://gallery.azure.ai/).
2.	Click the profile picture in the top-right corner, and then the account name to load your profile page.
3.	The profile page displays all items published to the gallery, including unlisted entries.

## Use the AI Gallery Catalog API to view your data

You can programmatically view data collected through the AI Gallery Catalog API, which is accessible at https://catalog.cortanaanalytics.com/entities. To view data, you need your Author ID. To view unlisted entities through the Catalog API, you need an access token.

Catalog responses are returned in JSON format.

### Get an author ID
The author ID is based on the email address used when publishing to the Azure AI Gallery. It doesn't change:

1.	Sign in to [Azure AI Gallery](https://gallery.azure.ai/).
2.	Click the profile picture in the top-right corner, and then the account name to load your profile page.
3.	The URL in the address bar displays the alphanumeric ID following `authorId=`. For example, for the URL: 
    `https://gallery.cortanaintelligence.com/Home/Author?authorId=CB9AEFCBEB947FD5D25D29A477B9CBC572AAD7BEF3236FF55724282F7C8CE8BE`
        
    Author ID: 
    `CB9AEFCBEB947FD5D25D29A477B9CBC572AAD7BEF3236FF55724282F7C8CE8BE`

### Get your access token

You need an access token to view unlisted entities through the Catalog API. Without an Access Token, users can still view public entities and other user info.

To get an access token, you need to inspect the `DataLabAccessToken` header of an HTTP request the browser makes to the Catalog API while logged in:

1.	Sign in to the [Azure AI Gallery](https://gallery.azure.ai/).
2.	Click the profile picture in the top-right corner, and then the account name to load your profile page.
3.	Open the browser Developer Tools pane by pressing F12, select the Network tab, and refresh the page. 
4. Filter requests on the string *catalog* by typing into the Filter text box.
5.	In requests to the URL `https://catalog.cortanaanalytics.com/entities`, find a GET request and select the *Headers* tab. Scroll down to the *Request Headers* section.
6.	Under the header `DataLabAccessToken` is the alphanumeric token. To help keep your data secure, don't share this token.

### View user information
Using the author ID you got in the previous steps, view information in a user's profile by replacing `S[AuthorId]` in the following URL:

    https://catalog.cortanaanalytics.com/users/[AuthorID]

For example, this URL request:
    
    https://catalog.cortanaanalytics.com/users/CB9AEFCBEB947FD5D25D29A477B9CBC572AAD7BEF3236FF55724282F7C8CE8BE

Returns a response such as:

    {"entities_count":9,"contribution_score":86.351575190956922,"scored_at":"2018-05-07T14:30:25.9305671+00:00","contributed_at":"2018-05-07T14:26:55.0381756+00:00","created_at":"2017-12-15T00:49:15.6733094+00:00","updated_at":"2017-12-15T00:49:15.6733094+00:00","name":"First Last","slugs":["First-Last"],"tenant_id":"14b2744cf8d6418c87ffddc3f3127242","community_id":"9502630827244d60a1214f250e3bbca7","id":"CB9AEFCBEB947FD5D25D29A477B9CBC572AAD7BEF3236FF55724282F7C8CE8BE","_links":{"self":"https://catalog.azureml.net/tenants/14b2744cf8d6418c87ffddc3f3127242/communities/9502630827244d60a1214f250e3bbca7/users/CB9AEFCBEB947FD5D25D29A477B9CBC572AAD7BEF3236FF55724282F7C8CE8BE"},"etag":"\"2100d185-0000-0000-0000-5af063010000\""}


View published entities
The Catalog API stores information about published entities to the Azure AI Gallery. One can also view this information directly on the Gallery website at https://gallery.cortanaintelligence.com. See previous sections for more information.
To view published entities, visit the following URL, replacing [AuthorId] with the Author ID obtained from the previous section.
https://catalog.cortanaanalytics.com/entities?$filter=author/id eq '[AuthorId]'
For example:
https://catalog.cortanaanalytics.com/entities?$filter=author/id eq 'CB9AEFCBEB947FD5D25D29A477B9CBC572AAD7BEF3236FF55724282F7C8CE8BE'

This query will only display public entities. To view all your entities, including unlisted ones, one must provide the Access Token obtained from the previous section.

1.	Using a tool like Postman (https://www.getpostman.com), create an HTTP GET request to the catalog URL as described above.
2.	Create an HTTP request header called “DataLabAccessToken”, with the value set to the Access Token.
3.	Make the HTTP request.

Note: If unlisted entities are not showing up in responses from the Catalog API, the user may have an invalid or expired Access Token. Log out of the Azure AI Gallery, then repeat the steps in “Getting an Access Token” to renew the token.

Links to Public Documentation
Further REST API documentation for Web Services, and Commitment Plan billing is available at: https://docs.microsoft.com/en-us/rest/api/machinelearning/

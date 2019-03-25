---
title: 'Example showing autocomplete and suggested queries for typeahead in a search box - Azure Search'
description: Enable typeahead query actions in Azure Search by creating suggesters and formulating requests that fill in a search box with completed terms or phrases. 
manager: pablocas
author: mrcarter8
services: search
ms.service: search
ms.devlang: NA
ms.topic: conceptual
ms.date: 03/25/2019
ms.author: mcarter
ms.custom: seodec2018
#Customer intent: As a developer, I want to understand autocomplete implementation, benefits, and tradeoffs.
---

# Example: Add Suggestions or Autocomplete query inputs in Azure Search

In this example, learn how to use [suggestions](https://docs.microsoft.com/rest/api/searchservice/suggestions), [autocomplete](https://docs.microsoft.com/rest/api/searchservice/autocomplete) and [.NET SDK](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.documentsoperationsextensions?view=azure-dotnet) to build a powerful search box that supports search-as-you-type behaviors.

+ *Suggestions* provide a list of suggested queries. 

+ *Autocomplete*, [a new preview feature](search-api-preview.md), "finishes" the word or phrase that a user is currently typing. 

You can download and run sample code to evaluate these features. The sample code targets a prebuilt index populated with the [NYCJobs](https://github.com/Azure-Samples/search-dotnet-asp-net-mvc-jobs). The index includes a [Suggester construct](index-add-suggesters.md), which is a requirement for using either suggestions or autocomplete. You can either use the index already provided, or populate your own index using a data loader in the NYCJobs sample solution. 

Code is an ASP.NET MVC-based application that uses C# to call the [Azure Search .NET SDK](https://aka.ms/search-sdk). In HomeController.cs, classes are defined for instantiating suggested and autocompleted queries. 

JavaScript uses the [jQuery UI](https://jqueryui.com/autocomplete/) and [XDSoft](https://xdsoft.net/jqplugins/autocomplete/) libraries to build the search box supporting both suggestions and autocomplete. From JavaScript, you can call the Azure Search REST API directly. Inputs collected in the search box are paired with suggestions and autocomplete actions, as defined in HomeController.cs. 

This exercise walks you through the following tasks:

> [!div class="checklist"]
> * Define suggestions and autocomplete actions
> * Implement a search input box in JavaScript and call suggestions and autocomplete ations.
> * Support client-side caching to improve performance 

## Prerequisites

An Azure Search service is optional for this exercise because the solution uses a live sandbox service and a pre-built index. If you want to run this example on your own search service, see [Configure NYC Jobs index to run on your service](#configure-app) for instructions.

* [Visual Studio 2017](https://visualstudio.microsoft.com/downloads/), any edition. Sample code and instructions were tested on the free Community edition.

* Download the [DotNetHowToAutoComplete sample](https://github.com/Azure-Samples/search-dotnet-getting-started/tree/master/DotNetHowToAutocomplete).

The sample is comprehensive, covering suggestions, autocomplete, faceted navigation, and client-side caching. You should review the readme and comments for a full description of what the sample offers.

## Run the sample

1. Open **AutocompleteTutorial.sln** in Visual Studio. The solution contains an ASP.NET MVC project with a connection to the NYC Jobs demo index.

2. Press F5 to run the project and load the page in your browser of choice.

At the top, you'll see an option to select C# or JavaScript. The C# option calls into the HomeController from the browser and uses the Azure Search .NET SDK to retrieve results. 

The JavaScript option calls the Azure Search REST API directly from the browser. This option will typically have noticeably better performance since it takes the controller out of the flow. You can choose the option that suits your needs and language preferences. There are several autocomplete examples on the page with some guidance for each. Each example has some recommended sample text you can try.  

Try typing in a few letters in each search box to see what happens.

## Search box

For both C# and JavaScript versions, the search box implementation is exactly the same. 

Open the Index.cshtml file under the folder \Views\Home to view the code:

```html
<input class="searchBox" type="text" id="example1a" placeholder="search">
```

This is a simple input text box with a class for styling, an ID to be referenced by JavaScript, and placeholder text.  The magic is in the embedded JavaScript.

The C# language sample uses JavaScript in Index.cshtml to leverage the [jQuery UI Autocomplete library](https://jqueryui.com/autocomplete/). This library adds the autocomplete experience to the search box by making asynchronous calls to the MVC controller to retrieve suggestions. 

Let's look at the JavaScript code for the first example, which provides a list of suggested queries:

```javascript
$(function () {
    $("#example1a").autocomplete({
        source: "/home/suggest?highlights=false&fuzzy=false&",
        minLength: 3,
        position: {
            my: "left top",
            at: "left-23 bottom+10"
        }
    });
});
```

This code runs in the browser on page load to configure jQuery UI autocomplete for the "example1a" input box.  `minLength: 3` ensures that recommendations will only be shown when there are at least three characters in the search box.  The source value is important:

```javascript
source: "/home/suggest?highlights=false&fuzzy=false&",
```

This line tells the jQuery UI autocomplete function where to get the list of items to show under the search box. Since this is an MVC project, it calls the Suggest function in HomeController.cs that contains the logic for returning query suggestions (more about Suggest in the next section). This function also passes a few parameters to control highlights, fuzzy matching, and term. The autocomplete JavaScript API adds the term parameter.

#### Extending the sample to support fuzzy matching

Fuzzy search allows you to get results based on close matches even if the user misspells a word in the search box. Let's try this out by changing the source line to enable fuzzy matching.

Change the following line from this:

```javascript
source: "/home/suggest?highlights=false&fuzzy=false&",
```

to this:

```javascript
source: "/home/suggest?highlights=false&fuzzy=true&",
```

Launch the application by pressing F5.

Try typing something like "execative" and notice how results come back for "executive", even though they are not an exact match to the letters you typed.

## HomeController.cs (C# version)

Now that we have reviewed the JavaScript code for web page, let's look at the C# controller code that actually retrieves the suggested queries using the Azure Search .NET SDK.

Open the **HomeController.cs** file under the Controllers directory. 

The first thing you might notice is a method at the top of the class called `InitSearch`. This creates an authenticated HTTP index client to the Azure Search service. For more information, see [How to use Azure Search from a .NET Application](https://docs.microsoft.com/azure/search/search-howto-dotnet-sdk).

On line 41, notice the Suggest function.

```csharp
public ActionResult Suggest(bool highlights, bool fuzzy, string term)
{
    InitSearch();

    // Call suggest API and return results
    SuggestParameters sp = new SuggestParameters()
    {
        UseFuzzyMatching = fuzzy,
        Top = 5
    };

    if (highlights)
    {
        sp.HighlightPreTag = "<b>";
        sp.HighlightPostTag = "</b>";
    }

    DocumentSuggestResult resp = _indexClient.Documents.Suggest(term, "sg", sp);

    // Convert the suggest query results to a list that can be displayed in the client.
    List<string> suggestions = resp.Results.Select(x => x.Text).ToList();
    return new JsonResult
    {
        JsonRequestBehavior = JsonRequestBehavior.AllowGet,
        Data = suggestions
    };
}
```

The Suggest function takes two parameters that determine whether hit highlights are returned or fuzzy matching is used in addition to the search term input. The method creates a SuggestParameters object, which is then passed to the Suggest API. The result is then converted to JSON so it can be shown in the client.

On line 69, notice the Autocomplete function.

```csharp
public ActionResult AutoComplete(string term)
{
    InitSearch();
    //Call autocomplete API and return results
    AutocompleteParameters ap = new AutocompleteParameters()
    {
        AutocompleteMode = AutocompleteMode.OneTermWithContext,
        UseFuzzyMatching = false,
        Top = 5
    };
    AutocompleteResult autocompleteResult = _indexClient.Documents.Autocomplete(term, "sg", ap);

    // Conver the Suggest results to a list that can be displayed in the client.
    List<string> autocomplete = autocompleteResult.Results.Select(x => x.Text).ToList();
    return new JsonResult
    {
        JsonRequestBehavior = JsonRequestBehavior.AllowGet,
        Data = autocomplete
    };
}
```

The Autocomplete function takes the search term input. The method creates an [AutoCompleteParameters object](https://docs.microsoft.com/rest/api/searchservice/autocomplete). The result is then converted to JSON so it can be shown in the client.

(Optional) Add a breakpoint to the start of the Suggest function and step through the code. Notice the response returned by the SDK and how it is converted to the result returned from the method.

The other examples on the page follow the same pattern to add hit highlighting and facets to support client-side caching of the autocomplete results. Review each of these to understand how they work and how to leverage them in your search experience.

## IndexJavaScript.cshtml (JavaScript version with REST API)

For the JavaScript implementation in IndexJavaScript.cshtml page, jQuery UI Autocomplete is also used for the search box, collecting search term inputs and making asynchronous calls to Azure Search to retrieve suggested queries or completed terms. 

Let's look at the JavaScript code for the first example:

```javascript
$(function () {
    $("#example1a").autocomplete({
        source: function (request, response) {
        $.ajax({
            type: "POST",
            url: suggestUri,
            dataType: "json",
            headers: {
                "api-key": searchServiceApiKey,
                "Content-Type": "application/json"
            },
            data: JSON.stringify({
                top: 5,
                fuzzy: false,
                suggesterName: "sg",
                search: request.term
            }),
                success: function (data) {
                    if (data.value && data.value.length > 0) {
                        response(data.value.map(x => x["@@search.text"]));
                    }
                }
            });
        },
        minLength: 3,
        position: {
            my: "left top",
            at: "left-23 bottom+10"
        }
    });
});
```

If you compare this to the example above that calls the Home controller, you'll notice several similarities.  The autocomplete configuration for `minLength` and `position` are exactly the same. 

The significant change here is the source. Instead of calling the Suggest method in the home controller, a REST request is created in a JavaScript function and executed using Ajax. The response is then processed in "success" and used as the source.

REST calls use URIs to specify whether an [Autocomplete]() or [Suggestions]() API call is being made. The following URIs are on lines 9 and 10, respectively.

```javascript
var suggestUri = "https://" + searchServiceName + ".search.windows.net/indexes/" + indexName + "/docs/suggest?api-version=" + apiVersion;
var autocompleteUri = "https://" + searchServiceName + ".search.windows.net/indexes/" + indexName + "/docs/autocomplete?api-version=" + apiVersion;
```

On line 148, you can find a script that calls the `autocompleteUri`. The first call to `suggestUri` is on line 39.

<a name="configure-app"></a>

## Configure NYC Jobs index to run on your service

Until now, you've been using the hosted NYCJobs demo index. If you want full visibility into all of the code, including the index, follow these instructions to create and load the index in your own search service.

1. [Create an Azure Search service](search-create-service-portal.md) or [find an existing service](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices) under your current subscription. You can use a free service for this example. 

   > [!Note]
   > If you are using the free Azure Search service, you are limited to three indexes. The NYCJobs data loader creates two indexes. Make sure you have room on your service to accept the new indexes.

1. Download [NYCJobs](https://github.com/Azure-Samples/search-dotnet-asp-net-mvc-jobs) sample code.

1. In the DataLoader folder of the NYCJobs sample code, open **DataLoader.sln** in Visual Studio.

1. Add the connection information for your Azure Search service. Open the App.config within the DataLoader project and change the TargetSearchServiceName and TargetSearchServiceApiKey appSettings to reflect your Azure Search service and Azure Search Service API Key. These can be found in the Azure portal.

1. Press F5 to launch the application, creating two indexes and importing the NYCJob sample data.

1. Open **AutocompleteTutorial.sln** and edit the Web.config in the **AutocompleteTutorial** project. Change the SearchServiceName and SearchServiceApiKey values to values that are valid for your search service.

1. Press F5 to run the application. The sample web app opens in the default browser. The experience is identical to the sandbox version, only the index and data are hosted on your service.

## Next steps

This example demonstrates the basic steps for building a search box that supports autocomplete and suggestions. You saw how you could build an ASP.NET MVC application and use either the Azure Search .NET SDK or REST API to retrieve suggestions.

As a next step, trying integrating suggestions and autocomplete into your search experience. The following reference articles should help.

> [!div class="nextstepaction"]
> [Autocomplete REST API](https://docs.microsoft.com/rest/api/searchservice/autocomplete)
> [Suggestions REST API](https://docs.microsoft.com/rest/api/searchservice/suggestions)
> [Facets index attribute on a Create Index REST API](https://docs.microsoft.com/rest/api/searchservice/create-index)


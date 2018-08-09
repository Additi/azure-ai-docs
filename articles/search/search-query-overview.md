---
title: Query types and composition in Azure Search | Microsoft Docs
description: Basics for building a search query in Azure Search, using parameters to filter, select, and sort results.
author: HeidiSteen
manager: cgronlun
ms.author: heidist
services: search
ms.service: search
ms.topic: conceptual
ms.date: 08/03/2018

---
# Query types and composition in Azure Search

Query composition in Azure Search is a full specification of a request: match criteria, plus parameters for directing query execution and shaping the response. A request specifies which fields to include, which fields to return, whether to sort or filter, and so forth. Unspecified, a query runs against all searchable fields as a full text search operation, returning an unscored result set in arbitrary order.

## A first look at query requests

Examples are useful for illustrating key concepts. As a representative query constructed in the [REST API](https://docs.microsoft.com/rest/api/searchservice/search-documents), this example targets the [real estate demo index](search-get-started-portal.md) and demonstrates key elements: parser type, match criteria, and which fields to search and bring back.

```
{  
    "queryType": "simple" 
    "search": "seattle townhouse* +\"lake\"", 
    "searchFields": "description, city",  
    "count": "true", 
    "select": "listingId, street, status, daysOnMarket, description",
    "top": "10",
    "orderby": "daysOnMarket"
 } 
```

**`queryType`** sets the parser, which in Azure Search can be the [default simple query parser](search-query-simple-examples.md) (optimal for full text search), or the [full Lucene query parser](search-query-lucene-examples.md) used for advanced query constructs like regular expressions, proximity search, fuzzy and wildcard search, to name a few.

**`search`** is the match criteria, usually text but often accompanied by boolean operators. Single standalone terms are *term* queries. Quote-enclosed multi-part queries are *key phrase queries. Search could be undefined, as in **`search=*`**, but more likely consists of terms, phrases, and operators similar what appears in the example.

**`searchFields`** is optional, used to limit query execution to specific fields.

Responses are subject to parameters you include in the query. In this case, the result set consists of fields listed in the **`select`** statement. Only the top 10 are returned in this query, but **`count`** tells you how many documents match overall. In this example, rows are sorted by the listing 

In Azure Search, query execution is always against one index, authenticated using an api-key provided in the request. In REST, both are provided in request headers.

### How to run this query

To run this query, use [Search explorer and the real estate demo index](search-get-started-portal.md). 

You can paste this query string into the explorer's search bar: `search=seattle townhouse +lake&searchFields=description, city&$count=true&$select=listingId, street, status, daysOnMarket, description&$top=10&orderby=listingId`

### Enable operations through index attributes

Index design and query design are tightly coupled in Azure Search. One critical point to know up front is that the *index schema*, with attributes on each field, determines the kind of query you can build. Index attributes on a field set the allowed operations - whether a field is *searchable* in the index, *retrievable* in results, *sortable*, *filterable*, and so forth. In the example, `"orderby": "daysOnMarket"` only works because the daysOnMarket field is marked as *sortable* in the index schema. 

![Index definition for the real estate sample](./media/search-query-overview/realestate-sample-index-definition.png "Index definition for the real estate sample")

The above screenshot is a partial list of index attributes for the real estate sample. You can view the entire index schema in the portal. For more information about index attributes, see [Create Index REST API](https://docs.microsoft.com/rest/api/searchservice/create-index).

> [!Note]
> Allowed operations on a per-field basis are just one way that index definition informs query execution. Other capabilities enabled in the index include the following: [synonyms](https://docs.microsoft.com/rest/api/searchservice/synonym-map-operations), [linguistic analysis](https://docs.microsoft.com//rest/api/searchservice/language-support), [custom analysis](https://docs.microsoft.com/rest/api/searchservice/custom-analyzers-in-azure-search), [suggester constructs (for autocomplete)](https://docs.microsoft.com/rest/api/searchservice/suggesters), [scoring logic for ranking results](https://docs.microsoft.com/rest/api/searchservice/add-scoring-profiles-to-a-search-index).

<a name="types-of-queries"></a>

## Types of queries

Azure Search supports a broad range of query types. The following table provides a list, but the ability to build queries using regular expressions means that you can often meet unusual and complex requirements not indicated here.

| Query type | Syntax | Examples and more information |
|------------|--------|-------------------------------|
| **search** parameter for free form text search | simple |  TBD | 
| **filter** parameter for an OData filter expression | simple | 


### Simple search and filter expressions

The default simple syntax serves free form text search as well as OData filter expressions. In the introductory example, the **search** parameter was identified as the means by which search criteria is passed to the engine. Using the default parser, you could augment or even replace search with **filter**. 

+ **`search`** queries scan for one or more terms in all *searchable* fields in your index, and works the way you would expect a search engine like Google or Bing to work. The examples in the introduction use the `search` parameter.

+ **`filter`** queries evaluate a boolean expression over all *filterable* fields in an index. Unlike `search`, a `filter` query matches the exact contents of a field, including case-sensitivity on string fields. Unlike search, filter queries are expressed in OData syntax.

You can use search and filter together or separately. A standalone filter, without a query string, is useful when the filter expression is able to fully qualify documents of interest. Without a query string, there is no lexical or linguistic analysis, no scoring, and no ranking. Notice the search string is empty.

```
POST /indexes/nycjobs/docs/search?api-version=2017-11-11  
    {  
      "search": "",
      "filter": "salary_frequency eq 'Annual' and salary_range_from gt 90000",
      "count": "true"
    }
```

Used together, the filter is applied first to the entire index, and then the search is performed on the results of the filter. Filters can therefore be a useful technique to improve query performance since they reduce the set of documents that the search query needs to process.

The syntax for filter expressions is a subset of the [OData filter language](https://docs.microsoft.com/rest/api/searchservice/OData-Expression-Syntax-for-Azure-Search). For search queries, you can use either the [simplified syntax](https://docs.microsoft.com/rest/api/searchservice/Simple-query-syntax-in-Azure-Search) or the [Lucene query syntax](https://docs.microsoft.com/rest/api/searchservice/Lucene-query-syntax-in-Azure-Search) which are discussed below.


## Choose a syntax: simple|full

Azure Search sits on top of Apache Lucene and gives you a choice between two query parsers for handling typical and specialized queries. Typical search requests are formulated using the default [simple query syntax](https://docs.microsoft.com/rest/api/searchservice/Simple-query-syntax-in-Azure-Search). This syntax supports a number of common search operators including the AND, OR, NOT, phrase, suffix, and precedence operators.

The [Lucene query syntax](https://docs.microsoft.com/rest/api/searchservice/Lucene-query-syntax-in-Azure-Search#bkmk_syntax), enabled when you add `queryType=full` to the request, exposes the widely adopted and expressive query language developed as part of [Apache Lucene](https://lucene.apache.org/core/4_10_2/queryparser/org/apache/lucene/queryparser/classic/package-summary.html). Using this query syntax allows specialized queries:

+ [Field-scoped queries](https://docs.microsoft.com/rest/api/searchservice/Lucene-query-syntax-in-Azure-Search#bkmk_fields)
+ [fuzzy search](https://docs.microsoft.com/rest/api/searchservice/Lucene-query-syntax-in-Azure-Search#bkmk_fuzzy)
+ [proximity search](https://docs.microsoft.com/rest/api/searchservice/Lucene-query-syntax-in-Azure-Search#bkmk_proximity)
+ [term boosting](https://docs.microsoft.com/rest/api/searchservice/Lucene-query-syntax-in-Azure-Search#bkmk_termboost)
+ [regular expression search](https://docs.microsoft.com/rest/api/searchservice/Lucene-query-syntax-in-Azure-Search#bkmk_regex)
+ [wildcard search](https://docs.microsoft.com/rest/api/searchservice/Lucene-query-syntax-in-Azure-Search#bkmk_wildcard)

Boolean operators are mostly the same in both syntax, with additional formats in full Lucene:

+ [Boolean operators in simple syntax](https://docs.microsoft.com/rest/api/searchservice/simple-query-syntax-in-azure-search#operators-in-simple-search)
+ [Boolean operators in  full Lucene syntax](https://docs.microsoft.com/rest/api/searchservice/Lucene-query-syntax-in-Azure-Search#bkmk_boolean)

## Elements of a query

Queries are always directed at a single index. You cannot join indexes or create custom or temporary data structures as a query target. 

When submitting search requests to Azure Search, there are a number of parameters that can be specified alongside the actual words that are typed into the search box of your application. These query parameters allow you to achieve some deeper control of the [full-text search experience](search-lucene-query-architecture.md).

Required elements on a query request include the following components:

+ Service endpoint and index documents collection, expressed here as a URL `https://<your-service-name>.search.windows.net/indexes/<your-index-name>/docs`.
+ API version (REST only), expressed as `api-version`
+ query or admin api-key, expressed as `api-key`
+ query string expressed as `search`, which can be unspecified if you want to perform an empty search. You can also send just a filter expression as `$filter`.
+ `queryType`, either simple or full, which can be omitted if you are using the built-in default simple syntax.

All other search parameters are optional.

## Manage search results 

Query results are streamed as JSON documents in the REST API, although if you use .NET APIs, serialization is built in. You can shape results by setting parameters on the query, selecting specific fields for the result

Parameters on the query can be used to structure the result set in the following ways:

+ Limiting or batching the number of documents in the results (50 by default)
+ Selecting fields to include in the results
+ Setting a sort order
+ Adding hit highlights to draw attention to matching terms in the body of the search results

### Tips for unexpected results

Occasionally, the substance and not the structure of results are unexpected. When query outcomes are not what you expect to see, you can try these query modifications to see if results improve:

+ Change `searchMode=any` (default) to `searchMode=all` to require matches on all criteria instead of any of the criteria. This is especially true when boolean operators are included the query.

+ Change the query technique if text or lexical analysis is necessary, but the query type precludes linguistic processing. In full text search, text or lexical analysis auto-corrects for spelling errors, singular-plural word forms, and even irregular verbs or nouns. For some queries such as fuzzy or wildcard search, text analysis is not part of the query parsing pipeline. For some scenarios, regular expressions have been used as a workaround. 

### Paging results
Azure Search makes it easy to implement paging of search results. By using the `top` and `skip` parameters, you can smoothly issue search requests that allow you to receive the total set of search results in manageable, ordered subsets that easily enable good search UI practices. When receiving these smaller subsets of results, you can also receive the count of documents in the total set of search results.

You can learn more about paging search results in the article [How to page search results in Azure Search](search-pagination-page-layout.md).

### Ordering results
When receiving results for a search query, you can request that Azure Search serves the results ordered by values in a specific field. By default, Azure Search orders the search results based on the rank of each document's search score, which is derived from [TF-IDF](https://en.wikipedia.org/wiki/Tf%E2%80%93idf).

If you want Azure Search to return your results ordered by a value other than the search score, you can use the `orderby` search parameter. You can specify the value of the `orderby` parameter to include field names and calls to the [`geo.distance()` function](https://docs.microsoft.com/rest/api/searchservice/OData-Expression-Syntax-for-Azure-Search) for geospatial values. Each expression can be followed by `asc` to indicate that results are requested in ascending order, and `desc` to indicate that results are requested in descending order. The default ranking ascending order.


### Hit highlighting
In Azure Search, emphasizing the exact portion of search results that match the search query is made easy by using the `highlight`, `highlightPreTag`, and `highlightPostTag` parameters. You can specify which *searchable* fields should have their matched text emphasized as well as specifying the exact string tags to append to the start and end of the matched text that Azure Search returns.

## APIs and tools for testing

The following table lists the APIs and tool-based approaches for submitting queries.

| Methodology | Description |
|-------------|-------------|
| [SearchIndexClient (.NET)](https://docs.microsoft.com/dotnet/api/microsoft.azure.search.searchindexclient?view=azure-dotnet) | Client that can be used to query an Azure Search index.  <br/>[Learn more.](search-howto-dotnet-sdk.md#core-scenarios)  |
| [Search Documents (REST API)](https://docs.microsoft.com/rest/api/searchservice/search-documents) | GET or POST methods on an index, using query parameters for additional input.  |
| [Fiddler, Postman, or other HTTP testing tool](search-fiddler.md) | Explains how to set up a request header and body for sending queries to Azure Search.  |
| [Search explorer in Azure portal](search-explorer.md) | Provides a search bar and options for index and api-version selections. Results are returned as JSON documents. <br/>[Learn more.](search-get-started-portal.md#query-index) | 

## See also

+ [How full text search works in Azure Search (query parsing architecture](search-lucene-query-architecture.md)
+ [Search explorer](search-explorer.md)
+ [How to query in .NET](search-query-dotnet.md)
+ [How to query in REST](search-query-rest-api.md)

---
title: Match patterns and special characters
titleSuffix: Azure Cognitive Search
description: Use wildcard and prefix queries to match on whole or partial terms in an Azure Cognitive Search query request. Hard-to-match patterns that include special characters can be resolved using full query syntax and custom analyzers.

manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 01/14/2020
---
# Match on patterns and special characters (dashes)

For queries that include special characters (`-, *, (, ), /, \, =`),  or for query patterns based on partial terms within a larger term, additional configuration steps are typically needed to ensure that the index contains the expected content, in the right format. 

By default, a phone number like `"+1 (425) 703-6214"` is tokenized as 
`"1"`, `"425"`, `"703"`, `"6214"`. As you can imagine, searching on `"3-62"`, partial terms that include a dash, will fail because that content doesn't exist like that in the index. 

When you need to search on partial strings or special characters, you can override the default analyzer with a custom analyzer that operates under simpler tokenization rules that preserves whole terms. Taking a step back, the approach looks like this:

+ Choose a predefined analyzer or define a custom analyzer that produces the desired output
+ Assign the analyzer to the field
+ Build the index and test

This article walks you through each step. The approach described here can be used in other scenarios. Wildcard and regular expression queries also need whole terms as the basis for pattern matching. 

> [!TIP]
> Evaluating analyers requires frequent index rebuilds. You can make this step easier by using Postman, the REST APIs for [Create Index](https://docs.microsoft.com/rest/api/searchservice/create-index), [Delete Index](https://docs.microsoft.com/rest/api/searchservice/delete-index), and [Load Documents](https://docs.microsoft.com/rest/api/searchservice//addupdate-or-delete-documents) where the request body provides a small representative data set that you want to test.

## Choosing an analyzer

When choosing an analyzer that produces whole-term tokens, the following analyzers are common choices:

| Analyzer | Behaviors |
|----------|-----------|
| [keyword](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/core/KeywordAnalyzer.html) | Content of the entire field is tokenized as a single term. |
| [whitespace](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/core/WhitespaceAnalyzer.html) | Separates on white spaces only. Terms that include dashes or other characters are treated as a single token. |
| [custom analyzer](index-add-custom-analyzers.md) | (recommended) Creating a custom analyzer lets you specify both the tokenizer and token filter. The previous analyzers must be used as-is. A custom analyzer lets you pick which tokenizers and token filters to use. A recommended combination is the keyword tokenizer with a lower-case token filter. By itself, the built-in [keyword](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/core/KeywordAnalyzer.html) does not lower-case any upper-case text, which can cause queries to fail. A custom analyzer gives you a mechanism for adding the lower-case token filter. |

The best way to evaluate analyzer is to use a web API test tool like Postman and the [Test Analyzer REST API](https://docs.microsoft.com/rest/api/searchservice/test-analyzer). Given an existing index and a field containing dashes or partial terms, you can try various analyzers over specific terms to see what tokens are emitted.  

1. Check the Standard analyzer to see how terms are tokenized by default.

   ```json
   {
   "text": "SVP10-NOR-00",
   "analyzer": "standard"
   }
    ```

1. Evaluate the response to see how the text is tokenized within the index. Notice how each term is lower-cased and broken up.

    ```json
    {
        "tokens": [
            {
                "token": "svp10",
                "startOffset": 0,
                "endOffset": 5,
                "position": 0
            },
            {
                "token": "nor",
                "startOffset": 6,
                "endOffset": 9,
                "position": 1
            },
            {
                "token": "00",
                "startOffset": 10,
                "endOffset": 12,
                "position": 2
            }
        ]
    }
    ```
1. Modify the request to use the `whitespace` or `keyword` analyzer:

    ```json
    {
    "text": "SVP10-NOR-00",
    "analyzer": "keyword"
    }
    ```

1. Now the response consists of a single token, upper-cased, with dashes preserved as a part of the string. If you need to search on a pattern or a partial term, the query engine now has the basis for finding a match.


    ```json
    {

        "tokens": [
            {
                "token": "SVP10-NOR-00",
                "startOffset": 0,
                "endOffset": 12,
                "position": 0
            }
        ]
    }
    ```
> [!Important]
> Be aware that query parsers often lower-case terms in a search expression when building the query tree. If you are using an analyzer that does not lower-case text inputs, and you are not getting expected results, this could be the reason.

## Analyzer definitions
 
Whether you are evaluating analyzers or moving forward with a specific configuration, you will need to specify the analyzer on the field definition, and possibly configure the analyzer itself if you are not using a built-in analyzer. When swapping analyzers, you typically need to rebuild the index (drop, recreate, and reload). 

### Use built-in analyzers

Built-in or predefined analyzers can be specified by name on an `analyzer` property of a field definition, with no additional configuration required. The following example demonstrates the use of the `whitespace` analyzer.

```json
    {
      "name": "phoneNumber",
      "type": "Edm.String",
      "key": false,
      "retrievable": true,
      "searchable": true,
      "analyzer": "whitespace"
    }
```
For more information about all available built-in analyzers, see [Predefined analyzers list](https://docs.microsoft.com/azure/search/index-add-custom-analyzers#predefined-analyzers-reference). 

### Use custom analyzers

If you are using a [custom analyzer](index-add-custom-analyzers.md), define it in the index with a user-defined combination of tokenizer, tokenfilter, with possible configuration settings. Next, reference it on a field definition, just as you would a built-in analyzer.

When the objective is whole-term tokenization, a custom analyzer that consists of a **keyword tokenizer** and **lower-case token filter** is recommended.

+ The keyword tokenizer creates a single token for the entire contents of a field.
+ The lowercase token filter transforms upper-case letters into lower-case text. Query parsers typically lowercase any uppercase text inputs. Lowercasing homogenizes the inputs with the tokenized terms.

The following example illustrates a custom analyzer that provides the keyword tokenizer and a lowercase token filter.

```json
{
"fields": [
  {
  "name": "accountNumber",
  "analyzer":"myCustomAnalyzer",
  "type": "Edm.String",
  "searchable": true,
  "filterable": true,
  "retrievable": true,
  "sortable": false,
  "facetable": false
  }
]

"analyzers": [
  {
  "@odata.type":"#Microsoft.Azure.Search.CustomAnalyzer",
  "name":"myCustomAnalyzer",
  "charFilters":[],
  "tokenizer":"keyword_v2",
  "tokenFilters":["lowercase"]
  }
],
"tokenizers":[],
"charFilters": [],
"tokenFilters": []
```

> [!NOTE]
> The `keyword_v2` tokenizer and `lowercase` token filter are known to the system and using their default configurations, which is why you can reference them by name without having to define them first.

## Tips and best practices


### Tune query performance

If you implement the recommended configuration that includes the keyword_v2 tokenizer and lower-case token filter, you might notice a decrease in query performance due to the additional token filter processing over existing tokens in your index. 

The following example adds an [EdgeNGramTokenFilter](https://lucene.apache.org/core/6_6_1/analyzers-common/org/apache/lucene/analysis/ngram/EdgeNGramTokenizer.html) to make prefix matches faster. Additional tokens are generated for in 2-25 character combinations: (not only MS, MSF, MSFT, MSFT/, but also embedded/internal partial strings like SQL, SQL., SQL.2)

```json
{
"fields": [
  {
  "name": "accountNumber",
  "analyzer":"myCustomAnalyzer",
  "type": "Edm.String",
  "searchable": true,
  "filterable": true,
  "retrievable": true,
  "sortable": false,
  "facetable": false
  }
]

"analyzers": [
  {
  "@odata.type":"#Microsoft.Azure.Search.CustomAnalyzer",
  "name":"myCustomAnalyzer",
  "charFilters":[],
  "tokenizer":"keyword_v2",
  "tokenFilters":["lowercase", "my_edgeNGram"]
  }
],
"tokenizers":[],
"charFilters": [],
"tokenFilters": [
  {
  "@odata.type":"#Microsoft.Azure.Search.EdgeNGramTokenFilterV2",
  "name":"my_edgeNGram",
  "minGram": 2,
  "maxGram": 25,
  "side": "front"
  }
]
```

### Use different analyzers for indexing and query processing

Analyzers are called during indexing and during query execution. It's common to use the same analyzer for both but you can configure custom analyzers for each workload. Analyzer overrides are specified in the [index definition](https://docs.microsoft.com/rest/api/searchservice/create-index) in an `analyzers` section, and then referenced on specific fields. 

When custom analysis is only required during indexing, you can apply the custom analyzer to just indexing and continue to use the standard Lucene analyzer (or another analyzer) for queries.

To specify role-specific analysis, you can set properties on the field for each one, setting `indexAnalyzer` and `searchAnalyzer` instead of the default `analyzer` property.

```json
"name": "featureCode",
"indexAnalyzer":"my_customanalyzer",
"searchAnalyzer":"standard",
```

### Duplicate fields for different scenarios

Another option leverages the per-field analyzer assignment to optimize for different scenarios. Specifically, you might define "featureCode" and "featureCodeRegex" to support regular full text search on the first, and advanced pattern matching on the second.

```json
{
  "name": "featureCode",
  "type": "Edm.String",
  "retrievable": true,
  "searchable": true,
  "analyzer": null
},
{
  "name": "featureCodeRegex",
  "type": "Edm.String",
  "retrievable": true,
  "searchable": true,
  "analyzer": "my_customanalyzer"
},
```

## Next steps

This article explains how analyzers both contribute to query problems and solve query problems. As a next step, take a closer look at how analyzers are used to modulate indexing and query processing. In particular, consider using the Analyze Text API to return tokenized output so that you can see exactly what an analyzer is creating for your index.

+ [Language analyzers](search-language-support.md)
+ [Analyzers for text processing in Azure Cognitive Search](search-analyzers.md)
+ [Analyze Text (REST)](https://docs.microsoft.com/rest/api/searchservice/test-analyzer) 
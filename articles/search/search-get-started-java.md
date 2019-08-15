---
title: 'Quickstart: Create, load, and query indexes in Java'
description: Explains how to create an index, load data, and run queries using Java and the Azure Search REST APIs.
author: lisaleib
manager: cgronlun
ms.author: v-lilei
tags: azure-portal
services: search
ms.service: search
ms.devlang: java
ms.topic: quickstart
ms.date: 07/11/2019
---
# Quickstart: Create an Azure Search index in Java
> [!div class="op_single_selector"]
> * [Portal](search-get-started-portal.md)
> * [.NET](search-howto-dotnet-sdk.md)
> 
> 

Create a Java console application that creates, load and queries an Azure search index using [IntelliJ] (https://docs.microsoft.com/java/azure/jdk/?view=azure-java-stable) and the [Azure Search Service REST API](https://msdn.microsoft.com/library/dn798935.aspx).This article explains how to create the application step by step. Alternatively, you can [download and run the complete application](https://github.com/Azure-Samples/azure-search-java-samples/tree/master/Quickstart).

If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before you begin.

## Prerequisites

We used the following software to build and test this sample:

+ [IntelliJ IDEA](https://www.jetbrains.com/idea/)

+ [Java 11 SDK](https://docs.microsoft.com/java/azure/jdk/?view=azure-java-stable)

+[Create an Azure Search service](search-create-service-portal.md) or [find an existing service](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices) under your current subscription. You can use a free service for this quickstart.

<a name="get-service-info"></a>

## Get a key and URL

Calls to the service require a URL endpoint and an access key on every request. A search service is created with both, so if you added Azure Search to your subscription, follow these steps to get the necessary information:

1. [Sign in to the Azure portal](https://portal.azure.com/), and in your search service **Overview** page, get the URL. An example endpoint might look like `https://mydemo.search.windows.net`.

2. In **Settings** > **Keys**, get an admin key for full rights on the service. There are two interchangeable admin keys, provided for business continuity in case you need to roll one over. You can use either the primary or secondary key on requests for adding, modifying, and deleting objects.

   Get the query key as well. It's a best practice to issue query requests with read-only access.

![Get an HTTP endpoint and access key](media/search-get-started-postman/get-url-key.png "Get an HTTP endpoint and access key")

All requests require an api-key on every request sent to your service. Having a valid key establishes trust, on a per request basis, between the application sending the request and the service that handles it.

## Set up your environment

Begin by opening IntelliJ IDEA and creating a new project.

### Create the project

1. Select Maven and the Java 11 SDK.
1. For `GroupId` and `ArtifactId`, enter "AzureSearchSample".
1. Accept the remaining defaults to open the project.
1. In the **Project** panel, expand the main tree. The picture below show how it should look when you're done.
1. In 'main' > `java`, add a package named `com.microsoft.azure.search.samples`.
1. In this new package, add two new packages `app` and `service`.
1. In `main` > `resources`, add a package named `com.microsoft.azure.search.samples`.
1. In this new package, add two new packages `app` and `service`.
1. When you are done, the project tree should look similar to the picture.
<INSERT IMAGE>

### Add Azure Search service information

1. In `main` > `resources.com.microsoft.azure.search.samples` > `app`, add a `config.properties` file.
1. Copy the following settings into the new file and replace `<YOUR-SEARCH-SERVICE-NAME>` and `<YOUR-API-KEY>` with your service name and key. If your service endpoint is `https://mydemo.search.windows.net`, the service name would be "mydemo".

```java
    SearchServiceName=<YOUR-SEARCH-SERVICE-NAME>
    SearchServiceApiKey=<YOUR-API-KEY>
    IndexName=hotels-quickstart
    ApiVersion=2019-05-06
```
1. In `src` > `main` > `java.com.microsoft.azure.search.samples` > `app`, create an `App` class.
1. Open the `App` class and replace the content with the following code. This code contains the `main` method. The commented code will be covered later in the article. The uncommented code reads the search service parameters and uses them to create a instance of the search client.
 
```java
package com.microsoft.azure.search.samples.app;

import com.microsoft.azure.search.samples.service.SearchServiceClient;
import java.io.IOException;
import java.util.Properties;

public class App {

    private static Properties loadPropertiesFromResource(String resourcePath) throws IOException {
        var inputStream = App.class.getResourceAsStream(resourcePath);
        var configProperties = new Properties();
        configProperties.load(inputStream);
        return configProperties;
    }

    public static void main(String[] args) {
        try {
            var config = loadPropertiesFromResource("config.properties");
            var client = new SearchServiceClient(
                    config.getProperty("SearchServiceName"),
                    config.getProperty("SearchServiceApiKey"),
                    config.getProperty("ApiVersion"),
                    config.getProperty("IndexName")
            );
//            if(client.indexExists()){ client.deleteIndex();}
//            client.createIndex("index.json");
//            Thread.sleep(1000L); // wait a second to create the index
//            client.uploadDocuments();
//            Thread.sleep(2000L); // wait 2 seconds for data to upload
//
//            // Query 1
//            client.logMessage("\n*QUERY 1****************************************************************");
//            client.logMessage("Search for: Atlanta'");
//            client.logMessage("Return: All fields'");
//            client.searchPlus("Atlanta");
//
//            // Query 2
//            client.logMessage("\n*QUERY 2****************************************************************");
//            client.logMessage("Search for: Atlanta");
//            client.logMessage("Return: HotelName, Tags, Address");
//            SearchServiceClient.SearchOptions options2 = client.createSearchOptions();
//            options2.select = "HotelName,Tags,Address";
//            client.searchPlus("Atlanta", options2);
//
//            //Query 3
//            client.logMessage("\n*QUERY 3****************************************************************");
//            client.logMessage("Search for: wifi & restaurant");
//            client.logMessage("Return: HotelName, Description, Tags");
//            SearchServiceClient.SearchOptions options3 = client.createSearchOptions();
//            options3.select = "HotelName,Description,Tags";
//            client.searchPlus("wifi,restaurant", options3);
//
//            // Query 4 -filtered query
//            client.logMessage("\n*QUERY 4****************************************************************");
//            client.logMessage("Search for: all");
//            client.logMessage("Filter: Ratings greater than 4");
//            client.logMessage("Return: HotelName, Rating");
//            SearchServiceClient.SearchOptions options4 = client.createSearchOptions();
//            options4.filter="Rating%20gt%204";
//            options4.select = "HotelName,Rating";
//            client.searchPlus("*",options4);
//
//            // Query 5 - top 2 results, ordered by
//            client.logMessage("\n*QUERY 5****************************************************************");
//            client.logMessage("Search for: boutique");
//            client.logMessage("Get: Top 2 results");
//            client.logMessage("Order by: Rating in descending order");
//            client.logMessage("Return: HotelId, HotelName, Category, Rating");
//            SearchServiceClient.SearchOptions options5 = client.createSearchOptions();
//            options5.top=2;
//            options5.orderby = "Rating%20desc";
//            options5.select = "HotelId,HotelName,Category,Rating";
//            client.searchPlus("boutique", options5);

        } catch (Exception e) {
            System.err.println("Exception:" + e.getMessage());
            e.printStackTrace();
        }
    }
}
```

### Add the HTTP operations
1.  In `src` > `main` > `java.com.microsoft.azure.search.samples` > `service`, create a `SearchServiceClient`class.
1. Open the `SearchServiceClient` class, and replace the contents with the following code. This code provides the HTTP operations required to use the Azure Search REST service. Additional methods for creating the index, uploading documents and querying the index will be added later in this quickstart.

```java
import javax.json.Json;
import javax.net.ssl.HttpsURLConnection;
import java.io.IOException;
import java.io.StringReader;
import java.net.HttpURLConnection;
import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
import java.nio.charset.StandardCharsets;
import java.util.Formatter;
import java.util.function.Consumer;

/**
 * This class is responsible for implementing HTTP operations for creating the index, uploading documents and searching the data*/
public class SearchServiceClient {
    private final String _apiKey;
    private final String _apiVersion;
    private final String _serviceName;
    private final String _indexName;
    private final static HttpClient client = HttpClient.newHttpClient();


    public SearchServiceClient(String serviceName, String apiKey, String apiVersion, String indexName) {
        this._serviceName = serviceName;
        this._apiKey = apiKey;
        this._apiVersion = apiVersion;
        this._indexName = indexName;
    }

    private HttpRequest httpRequest(URI uri, String method, String contents)
    {
        return httpRequest(uri, _apiKey, method, contents);
    }

    private static HttpResponse<String> sendRequest(HttpRequest request) throws IOException, InterruptedException {
        logMessage(String.format("%s: %s", request.method(), request.uri()));
        return client.send(request, HttpResponse.BodyHandlers.ofString());
    }

private static URI buildURI(Consumer<Formatter> fmtFn)
    {
        Formatter strFormatter = new Formatter();
        fmtFn.accept(strFormatter);
        String url = strFormatter.out().toString();
        strFormatter.close();
        return URI.create(url);
    }

    public static void logMessage(String message) {
        System.out.println(message);
    }

    public static boolean isSuccessResponse(HttpResponse<String> response) {
        try {
            int responseCode = response.statusCode();

            logMessage("\n Response code = " + responseCode);

            if (responseCode == HttpURLConnection.HTTP_OK || responseCode == HttpURLConnection.HTTP_ACCEPTED
                    || responseCode == HttpURLConnection.HTTP_NO_CONTENT || responseCode == HttpsURLConnection.HTTP_CREATED) {
                return true;
            }

            // We got an error
            var msg = response.body();
            if (msg != null) {
                logMessage(String.format("\n MESSAGE: %s", msg));
            }

        } catch (Exception e) {
            e.printStackTrace();
        }

        return false;
    }

    public static HttpRequest httpRequest(URI uri, String apiKey, String method, String contents) {
        contents = contents == null ? "" : contents;
        var builder = HttpRequest.newBuilder();
        builder.uri(uri);
        builder.setHeader("content-type", "application/json");
        builder.setHeader("api-key", apiKey);

        switch (method) {
            case "GET":
                builder = builder.GET();
                break;
            case "HEAD":
                builder = builder.GET();
                break;
            case "DELETE":
                builder = builder.DELETE();
                break;
            case "PUT":
                builder = builder.PUT(HttpRequest.BodyPublishers.ofString(contents));
                break;
            case "POST":
                builder = builder.POST(HttpRequest.BodyPublishers.ofString(contents));
                break;
            default:
                throw new IllegalArgumentException(String.format("Can't create request for method '%s'", method));
        }
        return builder.build();
    }
}

```

### Specify Maven dependencies
1. Open the pom.xml file, and replace the contents with the following xml code.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>AzureSearchQuickstart</groupId>
    <artifactId>AzureSearchQuickstart</artifactId>
    <version>1.0-SNAPSHOT</version>
    <build>
        <sourceDirectory>src</sourceDirectory>
        <plugins>
            <plugin>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.1</version>
                <configuration>
                    <source>11</source>
                    <target>11</target>
                </configuration>
            </plugin>
            <plugin>
                <groupId>org.codehaus.mojo</groupId>
                <artifactId>exec-maven-plugin</artifactId>
                <version>1.6.0</version>
                <executions>
                    <execution>
                        <goals>
                            <goal>exec</goal>
                        </goals>
                    </execution>
                </executions>
                <configuration>
                    <mainClass>com.microsoft.azure.search.samples.app.App</mainClass>
                    <cleanupDaemonThreads>false</cleanupDaemonThreads>
                </configuration>
            </plugin>
        </plugins>
    </build>
    <dependencies>
        <dependency>
            <groupId>org.glassfish</groupId>
            <artifactId>javax.json</artifactId>
            <version>1.0.2</version>
        </dependency>
    </dependencies>

</project>
```
1. Open the maven panel, and execute the following maven goal: `verify exec:java`
<<<INSERT IMAGE?>>>>>

A BUILD SUCCESS method should appear with a 0 exit code.

## 1 - Create index
The hotels index consists of simple fields and one complex field. Examples of a simple field are "HotelName" or "Description". The "Address" field is a complex field because it has subfields, such as "Street Address" and "City". In this quickstart, the index is specified using JSON.

1. In `main` > `resources.com.microsoft.azure.search.samples` > `service`, add an `index.json` file.

1. Copy the following index definition into the file. The name field defines the index name "hotels-quickstart". Attributes on the field determine how it is used in an application. For example, the `IsSearchable` attribute must be assigned to every field that should be included in a full text search.

```json
{
  "name": "hotels-quickstart",
  "fields": [
    {
      "name": "HotelId",
      "type": "Edm.String",
      "key": true,
      "filterable": true
    },
    {
      "name": "HotelName",
      "type": "Edm.String",
      "searchable": true,
      "filterable": false,
      "sortable": true,
      "facetable": false
    },
    {
      "name": "Description",
      "type": "Edm.String",
      "searchable": true,
      "filterable": false,
      "sortable": false,
      "facetable": false,
      "analyzer": "en.lucene"
    },
    {
      "name": "Description_fr",
      "type": "Edm.String",
      "searchable": true,
      "filterable": false,
      "sortable": false,
      "facetable": false,
      "analyzer": "fr.lucene"
    },
    {
      "name": "Category",
      "type": "Edm.String",
      "searchable": true,
      "filterable": true,
      "sortable": true,
      "facetable": true
    },
    {
      "name": "Tags",
      "type": "Collection(Edm.String)",
      "searchable": true,
      "filterable": true,
      "sortable": false,
      "facetable": true
    },
    {
      "name": "ParkingIncluded",
      "type": "Edm.Boolean",
      "filterable": true,
      "sortable": true,
      "facetable": true
    },
    {
      "name": "LastRenovationDate",
      "type": "Edm.DateTimeOffset",
      "filterable": true,
      "sortable": true,
      "facetable": true
    },
    {
      "name": "Rating",
      "type": "Edm.Double",
      "filterable": true,
      "sortable": true,
      "facetable": true
    },
    {
      "name": "Address",
      "type": "Edm.ComplexType",
      "fields": [
        {
          "name": "StreetAddress",
          "type": "Edm.String",
          "filterable": false,
          "sortable": false,
          "facetable": false,
          "searchable": true
        },
        {
          "name": "City",
          "type": "Edm.String",
          "searchable": true,
          "filterable": true,
          "sortable": true,
          "facetable": true
        },
        {
          "name": "StateProvince",
          "type": "Edm.String",
          "searchable": true,
          "filterable": true,
          "sortable": true,
          "facetable": true
        },
        {
          "name": "PostalCode",
          "type": "Edm.String",
          "searchable": true,
          "filterable": true,
          "sortable": true,
          "facetable": true
        },
        {
          "name": "Country",
          "type": "Edm.String",
          "searchable": true,
          "filterable": true,
          "sortable": true,
          "facetable": true
        }
      ]
    }
  ]
}
```json

In this index, the Description fields the optional `analyzer` property to override the default Lucene language analyzer. The `Description_fr` field is using the French Lucene analyzer `fr.lucene` because it stores French text. The `Description` is using the optional Microsoft language analyzer en.lucene. To learn more about analyzers, see [Analyzers for text processing in Azure Search](https://docs.microsoft.com/en-us/azure/search/search-analyzers).

1. Add the following code to the `SearchServiceClient` class. This code builds Azure Search REST service URLs to delete and create an index and to determine if the index exists.

```java
       public boolean indexExists() throws IOException, InterruptedException {
            logMessage("\n Checking if index exists...");
            var uri = buildURI(strFormatter -> strFormatter.format(
                    "https://%s.search.windows.net/indexes/%s/docs?api-version=%s&search=*",
                    _serviceName,_indexName,_apiVersion));
            var request = httpRequest(uri, "HEAD", "");
            var response = sendRequest(request);
            return isSuccessResponse(response);
        }

        public boolean deleteIndex() throws IOException, InterruptedException {
            logMessage("\n Deleting index...");
            var uri = buildURI(strFormatter -> strFormatter.format(
                    "https://%s.search.windows.net/indexes/%s?api-version=%s",
                    _serviceName,_indexName,_apiVersion));
            var request = httpRequest(uri, "DELETE", "*");
            var response = sendRequest(request);
            return isSuccessResponse(response);
        }


        public boolean createIndex(String indexDefinitionFile) throws IOException, InterruptedException {
            logMessage("\n Creating index...");
            //Build the search service URL
            var uri = buildURI(strFormatter -> strFormatter.format(
                    "https://%s.search.windows.net/indexes/%s?api-version=%s",
                    _serviceName,_indexName,_apiVersion));
            //Read in index definition file
            var inputStream = SearchServiceClient.class.getResourceAsStream("index.json");
            var indexDef = new String(inputStream.readAllBytes(), StandardCharsets.UTF_8);
            //Send HTTP PUT request to create the index in the search service
            var request = httpRequest(uri, "PUT", indexDef);
            var response = sendRequest(request);
            return isSuccessResponse(response);
        }
```

1. Uncomment the following code in the `App` class. This code deletes the index "hotels-quickstart", if it exists,  and creates a new index. To ensure the index is created before you upload any documents, a one-second pause is inserted after you make the REST service request to create the index.

```java
    //Delete the index if it exists and then create a new one
    if(client.indexExists()){ client.deleteIndex();}
    client.createIndex("index.json");
    Thread.sleep(1000L); // wait a second to create the index
```

1. Open the maven panel, and execute the following maven goal: `verify exec:java`
<<<INSERT IMAGE?>>>>>

As the code runs, a message should appear telling you that the index is created and finish with a 0 exit code.

## 2 - Upload documents

1. In `main` > `resources.com.microsoft.azure.search.samples` > `service`, add a `hotels.json` file.
1. Copy the following hotel documents into the file.

```json
{
  "value": [
    {
      "@search.action": "upload",
      "HotelId": "1",
      "HotelName": "Secret Point Motel",
      "Description": "The hotel is ideally located on the main commercial artery of the city in the heart of New York. A few minutes away is Time's Square and the historic centre of the city, as well as other places of interest that make New York one of America's most attractive and cosmopolitan cities.",
      "Description_fr": "L'hôtel est idéalement situé sur la principale artère commerciale de la ville en plein cœur de New York. A quelques minutes se trouve la place du temps et le centre historique de la ville, ainsi que d'autres lieux d'intérêt qui font de New York l'une des villes les plus attractives et cosmopolites de l'Amérique.",
      "Category": "Boutique",
      "Tags": [ "pool", "air conditioning", "concierge" ],
      "ParkingIncluded": "false",
      "LastRenovationDate": "1970-01-18T00:00:00Z",
      "Rating": 3.60,
      "Address": {
        "StreetAddress": "677 5th Ave",
        "City": "New York",
        "StateProvince": "NY",
        "PostalCode": "10022",
        "Country": "USA"
      }
    },
    {
      "@search.action": "upload",
      "HotelId": "2",
      "HotelName": "Twin Dome Motel",
      "Description": "The hotel is situated in a  nineteenth century plaza, which has been expanded and renovated to the highest architectural standards to create a modern, functional and first-class hotel in which art and unique historical elements coexist with the most modern comforts.",
      "Description_fr": "L'hôtel est situé dans une place du XIXe siècle, qui a été agrandie et rénovée aux plus hautes normes architecturales pour créer un hôtel moderne, fonctionnel et de première classe dans lequel l'art et les éléments historiques uniques coexistent avec le confort le plus moderne.",
      "Category": "Boutique",
      "Tags": [ "pool", "free wifi", "concierge" ],
      "ParkingIncluded": "false",
      "LastRenovationDate": "1979-02-18T00:00:00Z",
      "Rating": 3.60,
      "Address": {
        "StreetAddress": "140 University Town Center Dr",
        "City": "Sarasota",
        "StateProvince": "FL",
        "PostalCode": "34243",
        "Country": "USA"
      }
    },
    {
      "@search.action": "upload",
      "HotelId": "3",
      "HotelName": "Triple Landscape Hotel",
      "Description": "The Hotel stands out for its gastronomic excellence under the management of William Dough, who advises on and oversees all of the Hotel’s restaurant services.",
      "Description_fr": "L'hôtel est situé dans une place du XIXe siècle, qui a été agrandie et rénovée aux plus hautes normes architecturales pour créer un hôtel moderne, fonctionnel et de première classe dans lequel l'art et les éléments historiques uniques coexistent avec le confort le plus moderne.",
      "Category": "Resort and Spa",
      "Tags": [ "air conditioning", "bar", "continental breakfast" ],
      "ParkingIncluded": "true",
      "LastRenovationDate": "2015-09-20T00:00:00Z",
      "Rating": 4.80,
      "Address": {
        "StreetAddress": "3393 Peachtree Rd",
        "City": "Atlanta",
        "StateProvince": "GA",
        "PostalCode": "30326",
        "Country": "USA"
      }
    },
    {
      "@search.action": "upload",
      "HotelId": "4",
      "HotelName": "Sublime Cliff Hotel",
      "Description": "Sublime Cliff Hotel is located in the heart of the historic center of Sublime in an extremely vibrant and lively area within short walking distance to the sites and landmarks of the city and is surrounded by the extraordinary beauty of churches, buildings, shops and monuments. Sublime Cliff is part of a lovingly restored 1800 palace.",
      "Description_fr": "Le sublime Cliff Hotel est situé au coeur du centre historique de sublime dans un quartier extrêmement animé et vivant, à courte distance de marche des sites et monuments de la ville et est entouré par l'extraordinaire beauté des églises, des bâtiments, des commerces et Monuments. Sublime Cliff fait partie d'un Palace 1800 restauré avec amour.",
      "Category": "Boutique",
      "Tags": [ "concierge", "view", "24-hour front desk service" ],
      "ParkingIncluded": "true",
      "LastRenovationDate": "1960-02-06T00:00:00Z",
      "Rating": 4.60,
      "Address": {
        "StreetAddress": "7400 San Pedro Ave",
        "City": "San Antonio",
        "StateProvince": "TX",
        "PostalCode": "78216",
        "Country": "USA"
      }
    }
  ]
}
```
1. Insert the following code into the SearchServiceClient class. This code builds REST service URLs to upload the hotel documents to the index.

```java
    public boolean uploadDocuments() throws IOException, InterruptedException {
        logMessage("\n Uploading documents...");
        //Build the search service URL
        var endpoint = buildURI(strFormatter -> strFormatter.format(
                "https://%s.search.windows.net/indexes/%s/docs/index?api-version=%s",
                _serviceName,_indexName,_apiVersion));
        //Read in the data to index
        var inputStream = SearchServiceClient.class.getResourceAsStream("hotels.json");
        var documents = new String(inputStream.readAllBytes(), StandardCharsets.UTF_8);
        //Send HTTP POST request to upload and index the data
        var request = httpRequest(endpoint, "POST", documents);
        var response = sendRequest(request);
        return isSuccessResponse(response);
    }
```
1. Uncomment the following code in the `App` class. This code uploads the documents to the index. To ensure that documents are uploaded before you query the index, a two-second pause is inserted after the documents are uploaded.

```java
    client.uploadDocuments();
    Thread.sleep(2000L); // wait 2 seconds for data to upload

```
1. Open the maven panel, and execute the following maven goal: `verify exec:java`

Because you created a "hotels-quickstart" index in the previous step, the code will now delete it and recreate it again, and then will load the hotel documents. 

The process should exit with a 0 exit code.

## 3 - Search an index

Now that you've loaded the hotels documents, you can create search queries to access the hotels data.

1. Add the following code to the `SearchServiceClient` class. This code builds Azure Search REST service URLs to search the indexed data and prints the search results.

The `SearchOptions` class and `createSearchOptions` let you specify a subset of the available Azure Search REST API query options. For more information on the REST API query options, see [Search Documents (Azure Search Service REST API)](https://docs.microsoft.com/en-us/rest/api/searchservice/search-documents).

The `SearchPlus` method creates the search query URL, makes the search request, and then prints the results to the console. 

```java
    public SearchOptions createSearchOptions() { return new SearchOptions();}

    //Defines available search parameters that can be set
    public static class SearchOptions {

        public String select = "";
        public String filter = "";
        public int top = 0;
        public String orderby= "";
    }

    //Concatenates search parameters to append to the search request
    private String createOptionsString(SearchOptions options)
    {
        String optionsString = "";
        if (options != null) {
            if (options.select != "")
                optionsString = optionsString + "&$select=" + options.select;
            if (options.filter != "")
                optionsString = optionsString + "&$filter=" + options.filter;
            if (options.top != 0)
                optionsString = optionsString + "&$top=" + options.top;
            if (options.orderby != "")
                optionsString = optionsString + "&$orderby=" +options.orderby;
        }
        return optionsString;
    }

    public void searchPlus(String queryString)
    {
        searchPlus( queryString, null);
    }

    public void searchPlus(String queryString, SearchOptions options) {

        try {
            String optionsString = createOptionsString(options);
            var uri = buildURI(strFormatter -> strFormatter.format(
                    "https://%s.search.windows.net/indexes/%s/docs?api-version=%s&search=%s%s",
                    _serviceName, _indexName, _apiVersion, queryString, optionsString));
            var request = httpRequest(uri, "GET", null);
            var response = sendRequest(request);
            var jsonReader = Json.createReader(new StringReader(response.body()));
            var jsonArray = jsonReader.readObject().getJsonArray("value");
            var resultsCount = jsonArray.size();
            logMessage("Results:\nCount: " + resultsCount);
            for (int i = 0; i <= resultsCount - 1; i++) {
                logMessage(jsonArray.get(i).toString());
            }

            jsonReader.close();

        }
        catch (Exception e) {
            e.printStackTrace();
        }

    }
```

1. In the `App` class, uncomment the following code. This code sets up 5 different queries, including the search text, query parameters, and data fields to return. 

```java
        // Query 1
        client.logMessage("\n*QUERY 1****************************************************************");
        client.logMessage("Search for: Atlanta'");
        client.logMessage("Return: All fields'");
        client.searchPlus("Atlanta");

        // Query 2
        client.logMessage("\n*QUERY 2****************************************************************");
        client.logMessage("Search for: Atlanta");
        client.logMessage("Return: HotelName, Tags, Address");
        SearchServiceClient.SearchOptions options2 = client.createSearchOptions();
        options2.select = "HotelName,Tags,Address";
        client.searchPlus("Atlanta", options2);

        //Query 3
        client.logMessage("\n*QUERY 3****************************************************************");
        client.logMessage("Search for: wifi & restaurant");
        client.logMessage("Return: HotelName, Description, Tags");
        SearchServiceClient.SearchOptions options3 = client.createSearchOptions();
        options3.select = "HotelName,Description,Tags";
        client.searchPlus("wifi,restaurant", options3);

        // Query 4 -filtered query
        client.logMessage("\n*QUERY 4****************************************************************");
        client.logMessage("Search for: all");
        client.logMessage("Filter: Ratings greater than 4");
        client.logMessage("Return: HotelName, Rating");
        SearchServiceClient.SearchOptions options4 = client.createSearchOptions();
        options4.filter="Rating%20gt%204";
        options4.select = "HotelName,Rating";
        client.searchPlus("*",options4);

        // Query 5 - top 2 results, ordered by
        client.logMessage("\n*QUERY 5****************************************************************");
        client.logMessage("Search for: boutique");
        client.logMessage("Get: Top 2 results");
        client.logMessage("Order by: Rating in descending order");
        client.logMessage("Return: HotelId, HotelName, Category, Rating");
        SearchServiceClient.SearchOptions options5 = client.createSearchOptions();
        options5.top=2;
        options5.orderby = "Rating%20desc";
        options5.select = "HotelId,HotelName,Category,Rating";
        client.searchPlus("boutique", options5);

```

There are two [ways of matching terms in a query](search-query-overview.md#types-of-queries): full-text search, and filters. A full-text search query searches for one or more terms in `IsSearchable` fields in your index. A filter is a boolean expression that is evaluated over `IsFilterable` fields in an index. You can use full-text search and filters together or separately.

1. Open the maven panel, and execute the following maven goal: `verify exec:java`. 

Each query and it's results should appear in the console window, and the process should exit with a 0 exit code.

When you're working in your own subscription, it's a good idea at the end of a project to identify whether you still need the resources you created. Resources left running can cost you money. You can delete resources individually or delete the resource group to delete the entire set of resources.

You can find and manage resources in the portal, using the **All resources** or **Resource groups** link in the left-navigation pane.

If you are using a free service, remember that you are limited to three indexes, indexers, and data sources. You can delete individual items in the portal to stay under the limit. 

## Next steps

In this Java quickstart, you worked through a series of tasks to create an index, load it with documents, and run queries. If you are comfortable with the basic concepts, we recommend the following articles for an exploration of alternative approaches and concepts that will deepen your knowledge.

[Create a basic index in Azure Search](https://docs.microsoft.com/en-us/azure/search/search-what-is-an-index)
---
title: 'Microsoft Genomics: Common questions | Microsoft Docs'
description: Answers to common questions customers ask about Microsoft Genomics. 
services: microsoft-genomics
documentationcenter: ''
author: grhuynh
manager: geramill
editor: geramill

ms.service: microsoft-genomics
ms.workload: genomics
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/17/2017
ms.author: grhuynh

---
# Microsoft Genomics: Common questions

This article lists the top queries you might have relate to Microsoft Genomics. For more information on the Microsoft Genomics service, see [What is Microsoft Genomics?](what-is-Microsoft-Genomics.md) 


## What is the current version of the Microsoft Genomics command line client?
The current version of the Microsoft Genomics CLI is SQL Database is V12. Version V11 has been retired.

## What is the SLA for Microsoft Genomics?
We guarantee that 99.9% of the time Microsoft Genomics service will be available to receive workflow API requests. For more information, see [SLA](https://azure.microsoft.com/en-in/support/legal/sla/genomics/v1_0/).

## How does the usage of Microsoft Genomics show up on my bill?
Microsoft Genomics bills based on the number of gigabases processed per workflow. The first 10 gigabases are billed at a minimum rate of $2.90. This is the genome rate. All additional gigabases are billed at the incremental gigabase rate of $0.29 per gigabase. A gigabase is 1 billion (10000,00,000) bases. Gigabases are computed by counting all the bases in the submitted files, then rounding up or down to the nearest whole gigabase. For more information, see [Pricing](https://azure.microsoft.com/en-us/pricing/details/genomics/).


## Where can I find a list of all possible commands and arguments for the `msgen` client?
You can get a full list of available commands and arguments by running `msgen help`. If no further arguments are provided, it will show a list of available help sections, one for each of `submit`, `list`, `cancel`, and `status`. To get help for a specific command, type `msgen help command`; for example, `msgen help submit` will list all of the submit options.

## What are the most commonly used commands for the `msgen` client?
The most commonly used commands are arguments for the `msgen` client include: 

 |**Command**          |  **Field description** |
 |:--------------------|:-------------         |
 |`list`               |Returns a list of jobs you have submitted. For arguments, see `msgen help list`.  |
 |`submit`             |Submits a workflow request to the service. For arguments, see `msgen help submit`.|
 |`status`             |Returns the status of the workflow specified by `--workflow-id`. See also `msgen help status`. |
 |`cancel`             |Sends a request to cancel processing of the workflow specified by `--workflow-id`. See also `msgen help cancel`. |

## Where do I get the value for `--api-url-base`?
Go to Azure Portal and open your Genomics account page. Choose the "Access keys" blade. There, you will find both the API URL and your access keys.

## Where do I get the value for `--access-key`?
Go to Azure Portal and open your Genomics account page. Choose the "Access keys" blade. There, you will find both the API URL and your access keys.

## Why do I need 2 access keys?
You need two access keys in case you want to update (regenerate) them without interrupting usage of the service. Say, you want to update the first key. In that case, you will switch all new workflows to using the second key, wait until the already running workflows using the first key are finished, and only then will update the key.

## Do you save my storage account keys?
Your storage account key is used to create short-term access tokens for the Microsoft Genomics service to read your input files and write the output files. The default token duration is 48 hours. The token duration can be changed with the `-sas/--sas-duration` option of the submit command; the value is in hours.

## What genome references can I use?

We currently support these references:
 |Reference              | Value of `-pa/--process-args` |
 |:-------------         |:-------------                 |
 |b37                    | `R=b37m1`                     |
 |hg38                   | `R=hg38m1`                    |      
 |hg38 (no alt analysis) | `R=hg38m1x`                   |  
 |hg19                   | `R=hg19m1`                    |    

## How do I format my command-line arguments as a config file? 

msgen understands configuration files in the following format:
1. All options are provided as key-value pairs with values separated from keys by a colon.
Whitespace is ignored.
2. Lines starting with `#` are ignored.
3. Any command-line argument in the long format can be converted to a key by stripping its leading dashes and replacing dashes between words with underscores. Here are some conversion examples:

 |Command line argument            | Configuration file line |
 |:-------------                   |:-------------                 |
 |`-u/--api-url-base https://url`  | *api_url_base:https://url*    |
 |`-k/--access-key KEY`            | *access_key:KEY*              |      
 |`-pa/--process-args R=B37m1`     | *process_args:R-b37m1*        |  
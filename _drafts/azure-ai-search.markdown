---
layout: post
title:  "Azure AI Search"
date:   2024-02-15 11:26:18 +1000
categories: general

---

## Table of Contents

1. [What is Azure AI Search](#whatis)

## Overview whilst investigating

- add search service
  - add a data source - storage account
- sample data: https://github.com/Azure-Samples/azure-search-sample-data/tree/main
  - https://github.com/Azure-Samples/azure-search-sample-data/tree/main?tab=readme-ov-file#ai-enrichment-mixed-media 
- Service principal name: russ-dev-azure-ai-document-search
- ![Azure OpenAI Approval](/assets/az-doc-intel-ai-enrichment.png)

  
> Error detecting index schema from data source: "Unable to retrieve blob container for account 'russdevazuresearchsa' using your managed identity. Ensure the resource ID is correct and that the managed identity for your search service has been granted permission (e.g., Storage Blob Data Reader) to read blob container for this account. See https://go.microsoft.com/fwlink/?linkid=2193521 for detailed documentation of required permissions for your scenario."

This post outlines my findings from my investigation of Azure AI Search. The reason I am writing this post is to expand my knowledge of Azure AI. I wanted to look at Azure Open AI but this requires a company email to apply (I don't have a company email right now) to use it and, at present, I am taking some time off work to be with my family.

![Azure OpenAI Approval](/assets/az-doc-intel-approval-required.png)

I would like to achieve the following outcomes after completing my investigation of Azure AI Search and this port:

- Understand Azure AI Search at a substantial level.
- Ideally use my own documents.
- Write a small proof of concept application that ties everything together.
- Make all the code available for others to fork and use.

## What is Azure AI Search <a name="whatis"></a>

Azure AI Search, formerly recognized as "Azure Cognitive Search," delivers secure and scalable information retrieval for user-owned content in both traditional and conversational search applications.

Fundamental to applications displaying text and vectors, information retrieval plays a crucial role. This encompasses various scenarios such as catalog or document searches, data exploration, and the emerging trend of chat-style copilot applications utilizing proprietary grounding data. When establishing a search service, you leverage the following capabilities:

1. A search engine supporting vector, full text, and hybrid searches across a search index.
2. Comprehensive indexing featuring integrated data chunking and vectorization (in preview), lexical analysis for text, and the option for AI enrichment to extract and transform content.
3. Robust query syntax enabling vector queries, text searches, hybrid queries, fuzzy searches, autocomplete, geo-search, and more.
4. Utilization of Azure's scale, security, and extensive reach.
5. Seamless integration with Azure at the data layer, machine learning layer, Azure AI services, and Azure OpenAI.

From an architectural perspective, a search service functions as an intermediary positioned between external data stores housing your un-indexed data and the client app responsible for sending query requests to a search index and managing the subsequent response.

![Azure AI Search architecture](/assets/azure-ai-search-architecture.svg)

Within your client app, the search experience is crafted using APIs provided by Azure AI Search. This includes various functionalities such as relevance tuning, semantic ranking, autocomplete, synonym matching, fuzzy matching, pattern matching, filtering, and sorting.

Azure AI Search seamlessly integrates across the Azure platform by incorporating indexers, which automate data ingestion/retrieval from Azure data sources. Additionally, it leverages skillsets that integrate consumable AI from Azure AI services, encompassing capabilities like image and natural language processing. Furthermore, Azure AI Search supports custom AI, allowing you to create or encapsulate it within Azure Machine Learning or wrap it inside Azure Functions.

## Why use Azure AI Search?

Why Choose Azure AI Search?

Azure AI Search proves advantageous for various application scenarios, including:

1. Traditional Full Text and Next-Generation Vector Similarity Search: Employ Azure AI Search for traditional full text searches and advanced vector similarity searches. Empower your generative AI applications with information retrieval capabilities that utilize both keyword and similarity search modalities, ensuring the retrieval of the most relevant results.

2. Consolidation of Heterogeneous Content: Create a user-defined search index comprising vectors and text, allowing you to consolidate diverse content. With Azure AI Search, you have ownership and control over what is searchable.

3. Integration of Data Chunking and Vectorization: Seamlessly integrate data chunking and vectorization to enhance generative AI and RAG (Retrieval-Augmented Generation) applications.

4. Granular Access Control: Implement granular access control at the document level, providing heightened security.

5. Offloading Indexing and Query Workloads: Shift indexing and query workloads to a dedicated search service, optimizing performance.

6. Easy Implementation of Search-Related Features: Leverage Azure AI Search for the effortless implementation of search-related features, including relevance tuning, faceted navigation, filters (including geo-spatial search), synonym mapping, and autocomplete.

7. Transformation of Large Files: Transform large undifferentiated text, image files, or application files stored in Azure Blob Storage or Azure Cosmos DB into searchable chunks during indexing. Cognitive skills add external processing from Azure AI to achieve this.

8. Linguistic or Custom Text Analysis: Add linguistic or custom text analysis with Azure AI Search. Support for Lucene analyzers and Microsoft's natural language processors is available, allowing you to configure analyzers for specialized processing of raw content, such as filtering out diacritics or recognizing and preserving patterns in strings.

For detailed information on specific functionalities, refer to the Features of Azure AI Search documentation.


## The code

### Code Overview

In the code, PowerShell and Azure AI Search REST APIs are employed to establish a data source, index, indexer, and skillset.

The indexer establishes a connection with Azure Blob Storage, fetching pre-loaded content. Subsequently, the indexer triggers a skillset for specialized processing, incorporating the enriched content into a search index.

The skillset is linked to an indexer and employs Microsoft's built-in skills for information extraction. The pipeline encompasses Optical Character Recognition (OCR) for images, language detection, key phrase extraction, and entity recognition (organizations, locations, people). Newly generated information within the pipeline is stored in additional fields within an index. Once the index is populated, these fields become accessible for queries, facets, and filters.

> All the code that relates to this post in my public GitHub repository here: [https://github.com/russellmccloy/azure-ai-search](https://github.com/russellmccloy/azure-ai-search)

- I used Terraform to deploy most if the infrastructure for Azure AI Search.
- I used `PowerShell` to deploy some of the Azure AI Search internal infrastructure such as Datasources, Indexes, Indexers and Skillsets. The PowerShell was just used to wrap the Azure AI Search REST API.

## Useful Links and Documentation I used

- [Tutorial: Use REST and AI to generate searchable content from Azure blobs](https://learn.microsoft.com/en-au/azure/search/cognitive-search-tutorial-blob?source=docs)
- [AI enrichment in Azure AI Search](https://learn.microsoft.com/en-au/azure/search/cognitive-search-concept-intro)
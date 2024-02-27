---
layout: post
title:  "Azure AI Search"
date:   2024-02-15 11:26:18 +1000
categories: general

---

### Overview whilst investigating

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

## The code

> All the code that relates to this post in my public GitHub repository here: [https://github.com/russellmccloy/azure-ai-search](https://github.com/russellmccloy/azure-ai-search)

- I used Terraform to deploy most if the infrastructure for Azure AI Search. 
- I used `PowerShell` to deploy some of the Azure AI Search internal infrastructure such as Datasources, Indexes, Indexers and Skillsets.

## What is Azure AI Search

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

## Useful Links and Documentation I used

- [Tutorial: Use REST and AI to generate searchable content from Azure blobs](https://learn.microsoft.com/en-au/azure/search/cognitive-search-tutorial-blob?source=docs)
- [AI enrichment in Azure AI Search](https://learn.microsoft.com/en-au/azure/search/cognitive-search-concept-intro)
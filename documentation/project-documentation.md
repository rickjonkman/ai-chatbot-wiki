# AI Chatbot Project Documentation

## 1. Project Goal

The goal of this project is to build an AI Chatbot that makes knowledge from Azure DevOps Wikis accessible through Retrieval Augmented Generation (RAG). The chatbot should be able to extract wiki pages, prepare and index them, and make them searchable for end users through a chat interface and later through Microsoft Teams.

## 2. High-Level Architecture

The solution consists of the following chain:

1. Extract wiki data from Azure DevOps through the REST API.
2. Clean and enrich markdown content.
3. Chunk the content into retrieval-friendly units.
4. Store chunks in Azure Blob Storage.
5. Index the chunks in Azure AI Search, including vectors.
6. Answer questions through Azure OpenAI and a RAG prompt.
7. Expose the solution through a web interface and Microsoft Teams.

## 3. Data Scraping from Azure DevOps

### 3.1 Authentication

To read Azure DevOps Wiki data, a Personal Access Token is required with only the minimum necessary read permissions. Prefer storing the token securely in Azure Key Vault rather than hardcoding it in application configuration.

### 3.2 Relevant APIs

The Azure DevOps REST API must support at least these use cases:

- retrieve wiki metadata
- retrieve the full page structure with `recursionLevel=full`
- retrieve page content in markdown

Markdown output is suitable as a source format because Azure OpenAI and Azure AI Search can work with it well, provided that the content is cleaned up first.

### 3.3 Storage for Ingestion

An Azure Storage Account is needed for:

- temporary storage of raw responses
- storage of cleaned and chunked content
- optional failure handling or replay scenarios

Recommended containers:

- `raw-wiki`
- `processed-wiki`
- `failed-wiki`

## 4. Wiki Content Preparation

### 4.1 Markdown Cleanup

Retrieval quality depends heavily on the quality of the input text. For that reason, the markdown content must be cleaned.

Content to remove:

- navigation text
- footers
- repeated UI elements
- boilerplate that does not add value

Content to preserve:

- headings such as `#`, `##`, and `###`
- code blocks
- meaningful lists and tables
- relevant links or references

### 4.2 Metadata Enrichment

Metadata should be available per page and per chunk to support filtering, tracing, and source attribution.

Suggested metadata:

- `sourceSystem`: `azure-devops-wiki`
- `wikiName`
- `pageId`
- `title`
- `path`
- `url`
- `lastModified`
- `chunkId`
- `chunkIndex`
- `headingPath`

### 4.3 Chunking Strategy

Each chunk should be semantically useful and contain enough context without becoming too large.

Recommendations:

- primarily chunk on heading structure
- fall back to token or character limits for large sections
- use limited overlap to reduce context loss
- do not split code blocks when that would break meaning

A good starting point is section-based chunking with a maximum size that works well for both embeddings and retrieval.

## 5. Store Chunks in Blob Storage

### 5.1 Blob Model

Each blob contains exactly 1 chunk plus its metadata. JSON is a practical blob format because it keeps both content and metadata in a consistent model.

Example structure:

```json
{
  "id": "wiki-page-42-chunk-3",
  "title": "Deployments",
  "path": "/platform/release/deployments",
  "url": "https://dev.azure.com/...",
  "content": "... cleaned markdown ...",
  "chunkIndex": 3,
  "metadata": {
    "wikiName": "Engineering Wiki",
    "headingPath": "Platform > Release > Deployments",
    "lastModified": "2026-04-08T10:00:00Z"
  }
}
```

### 5.2 Automation

Synchronization can be automated through:

- Azure Function for event-driven or scheduled processing
- pipeline for controlled batch processing

A combination is often the best fit:

- pipeline for the initial full load
- Azure Function for incremental updates

## 6. Azure AI Search Configuration

### 6.1 Index Schema

The minimum index schema is as follows:

| Field     | Type        | Notes                       |
| --------- | ----------- | --------------------------- |
| id        | key         | unique per chunk            |
| content   | searchable  | main retrieval text         |
| title     | searchable  | title or section title      |
| path      | filterable  | path or document location   |
| url       | retrievable | link to the source          |
| embedding | vector      | vector field for embeddings |

Depending on the chosen architecture, additional fields can be added such as `wikiName`, `lastModified`, `tags`, or `headingPath`.

### 6.2 Search Strategy

The recommended approach is Hybrid Search:

- keyword search for exact terms and abbreviations
- vector search for semantic similarity
- semantic ranking for better result ordering

### 6.3 Integrated Vectorization

Integrated Vectorization is attractive when the indexing pipeline is allowed to generate embeddings directly. This reduces complexity in the application layer. If more control is needed over batching, model choice, or caching, then a separate embedding step is the better option.

## 7. Azure OpenAI Configuration

### 7.1 Model Selection

For cost control, mini models are a sensible first choice for chat. For embeddings, choose a model that works well with Azure AI Search.

### 7.2 RAG Prompt

The prompt should force the chatbot to answer based on the retrieved wiki content. Important elements:

- only answer based on the supplied context
- include sources or links where possible
- explicitly state when the context is insufficient
- summarize without losing the meaning of technical content

Example prompt guideline:

```text
Answer the question using only the information in the provided context.
If the context is insufficient, say so explicitly.
Use short, factual answers and include relevant source references.
```

## 8. Query Flow

A robust query flow includes these steps:

1. The user asks a question.
2. The application enriches or normalizes the query.
3. Azure AI Search performs hybrid retrieval.
4. The most relevant chunks are selected.
5. The context is assembled into a prompt.
6. Azure OpenAI generates an answer.
7. The answer is returned with source references.

Additional points of attention:

- logging of search results and prompt versions
- tracing of response times
- filtering by wiki, team, or domain
- rate limiting and error handling

## 9. Chatbot Interface

The interface should support at least:

- free-text question input
- answer rendering
- source and link rendering
- error messages and loading states
- room for future chat history

Non-functional requirements:

- fast response experience
- clear source references
- straightforward extensibility toward Teams

## 10. Teams Integration

Teams integration is the logical next step after the web interface is stable.

Important decisions:

- how authentication and authorization should work
- whether the chatbot responds through a bot or app identity
- how source references remain usable inside Teams

Teams introduces extra requirements around conversation design, permissions, and packaging. For that reason, it is better to integrate Teams only after the query flow and answer quality are stable enough.

## 11. Non-Functional Requirements

Important quality attributes for this project:

- secure handling of PATs, secrets, and access to Azure resources
- low operating cost through efficient model selection
- scalability of ingestion and indexing
- repeatability of synchronization
- observability for failures, latency, and search quality

## 12. Recommended Next Technical Steps

1. Document the exact Azure DevOps endpoints and response models.
2. Define the chunk and metadata model.
3. Design the AI Search index schema including additional metadata.
4. Choose the automation path: pipeline, Azure Function, or both.
5. Build a first end-to-end proof of concept with 1 wiki.

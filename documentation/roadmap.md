# Roadmap AI Chatbot Project

## Goal

Build an AI Chatbot that retrieves, enriches, indexes, and exposes Azure DevOps Wiki content through a chatbot interface and Microsoft Teams.

## Phase 1: Extract Data from Azure DevOps

### Goal

Set up a reliable source integration for reading wiki content from Azure DevOps.

### Work Packages

1. Create a Personal Access Token (PAT) with read-only permissions for wiki and project resources.
2. Define the required Azure DevOps REST API calls.
3. Provision an Azure Storage Account for raw and processed data.
4. Implement extraction of:
   - wiki metadata
   - page hierarchy with `recursionLevel=full`
   - page content in markdown

### Deliverables

- Working authentication for Azure DevOps.
- Documentation of relevant REST endpoints and request formats.
- Storage Account for ingestion and intermediate storage.
- Initial export of wiki data.

### Dependencies

- Available Azure DevOps organization, project, and wiki.
- Access to an Azure subscription.

## Phase 2: Prepare Wiki Content

### Goal

Prepare the wiki content for indexing and retrieval with high search quality.

### Work Packages

1. Clean up markdown:
   - remove navigation text, footer text, and irrelevant boilerplate
   - preserve heading structure such as `#` and `##`
   - preserve code blocks
2. Add metadata per page and per chunk.
3. Design and implement the chunking strategy.

### Deliverables

- Markdown cleanup rules.
- Chunking strategy with chunk size and overlap.
- Standardized metadata model.

### Decision Points

- Chunk by heading level or by token limits.
- Decide whether to add enrichment such as summaries or tags.

## Phase 3: Store Chunks in Blob Storage

### Goal

Store each chunk durably and repeatably together with metadata for further processing.

### Work Packages

1. Design the blob structure for raw, processed, and optionally failed content.
2. Ensure each blob contains exactly 1 chunk with metadata.
3. Set up automation through:
   - Azure Function
   - pipeline-based scheduled or event-driven synchronization

### Deliverables

- Blob naming convention.
- JSON or markdown blob format with metadata.
- Automated synchronization flow.

## Phase 4: Configure Azure AI Search

### Goal

Set up a search index that supports both keyword search and vector search.

### Work Packages

1. Create an index schema with at least these fields:

| Field     | Type        | Notes                       |
| --------- | ----------- | --------------------------- |
| id        | key         | unique                      |
| content   | searchable  | text                        |
| title     | searchable  | page title or section title |
| path      | filterable  | source path inside the wiki |
| url       | retrievable | direct link to the source   |
| embedding | vector      | OpenAI embeddings           |

2. Configure Hybrid Search:
   - keyword search
   - vector search
   - semantic ranking
3. Evaluate whether Integrated Vectorization is sufficient or whether embeddings should be generated in a separate step.

### Deliverables

- Working AI Search index.
- Ingestion flow from chunks to the index.
- Initial relevance tests.

## Phase 5: Configure Azure OpenAI

### Goal

Set up a cost-efficient and reliable LLM layer for RAG-based answers.

### Work Packages

1. Select suitable mini models for chat and embeddings.
2. Create deployment(s) in Azure OpenAI.
3. Define a RAG prompt including:
   - system instructions
   - source references
   - fallback behavior when context is insufficient

### Deliverables

- Selected model deployments.
- Version 1 of the RAG prompt.
- Cost and performance guidelines.

## Phase 6: Design the Query Flow

### Goal

Define an end-to-end query flow from user question to answer with source references.

### Work Packages

1. Define the request flow:
   - receive the user question
   - generate the search query
   - perform retrieval
   - assemble the context
   - call the LLM
   - return the answer with sources
2. Add logging, tracing, and error handling.
3. Define evaluation criteria for relevance and groundedness.

### Deliverables

- Technical flow description.
- API contract for frontend or Teams.
- Logging and observability approach.

## Phase 7: Build the Chatbot Interface

### Goal

Deliver a user-friendly interface for asking questions about the wiki.

### Work Packages

1. Design the UI for chat, source references, and status feedback.
2. Integrate the backend query flow.
3. Add basic support for chat history and error messages.

### Deliverables

- Working web interface.
- Connection to the backend API.
- Initial user test.

## Phase 8: Integrate with Teams

### Goal

Make the chatbot available inside Microsoft Teams.

### Work Packages

1. Choose between a Teams app, bot-style integration, or another Teams entry point.
2. Define the authentication and authorization model.
3. Adapt the UX to Teams conversations and source references.

### Deliverables

- Working Teams integration.
- Teams publication or test package.
- Acceptance criteria for production use.

## Recommended Milestone Breakdown

### Milestone 1

Azure DevOps extraction works and raw wiki data is available.

### Milestone 2

Content has been cleaned, chunked, and stored in Blob Storage.

### Milestone 3

Azure AI Search and Azure OpenAI deliver useful RAG results.

### Milestone 4

A web chat works end-to-end with source references.

### Milestone 5

The chatbot is available in Teams.

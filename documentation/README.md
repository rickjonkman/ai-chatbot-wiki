# AI Chatbot Project Documentation

This folder contains the project documentation for the AI Chatbot platform that exposes Azure DevOps Wiki content through RAG.

Available documents:

- [roadmap.md](roadmap.md): phased roadmap with milestones, deliverables, and dependencies.
- [project-documentation.md](project-documentation.md): functional and technical design from data ingestion to Teams integration.

Suggested implementation order:

1. Set up Azure DevOps access and data ingestion.
2. Preprocess and chunk wiki content.
3. Store content in Blob Storage and automate synchronization.
4. Configure Azure AI Search and Azure OpenAI.
5. Implement the query flow, chatbot interface, and Teams integration.

## PostGRES SQL
The Open WebUI project uses PostgreSQL for data storage with SQLAlchemy as the ORM. The database is structured around several core components:

User Management

- Users: Stores user profiles, roles (admin, user, pending), API keys, settings, and OAuth info
- Authentication: Stores email/password credentials and account status

Chat System

- Chats: Stores conversation history with titles, timestamps, sharing info, and metadata
- Messages: Individual chat messages
- Tags: For categorizing chats with a many-to-many relationship

Organization & Knowledge

- Folders: For organizing chats in hierarchical structure
- Knowledge: Knowledge base entries with descriptions and access controls
- Files: Tracks uploaded files with metadata and ownership

AI Configuration

- Models: Configuration for AI models with parameters
- Functions: Custom functions for AI use with content and metadata
- Prompts: Template prompts with titles and trigger commands
- Tools: External tool integrations
- Memory: Long-term AI memory storage

The database uses UUID strings as primary keys and JSON fields for flexible data storage. It has comprehensive timestamps and a sophisticated access control system for resource sharing between
users.

## OpenSearch
OpenSearch is integrated into Open WebUI as one of the vector database options for retrieval augmented generation (RAG) functionality. Here's a summary of its structure and usage:

Configuration & Setup

- Connected via environment variables (OPENSEARCH_URI, OPENSEARCH_SSL, etc.)
- Selected through the VECTOR_DB environment variable
- Implemented via OpenSearchClient class that inherits from VectorDBBase

Index Structure

- Uses FAISS engine with HNSW algorithm for KNN vector search
- Index mappings contain:
  - id: Document identifier
  - vector: KNN vector field for embeddings
  - text: Original text content
  - metadata: Document metadata

Data Operations

- Creates indexes with specific KNN configurations
- Stores document embeddings generated from various text sources
- Performs vector similarity searches using cosine similarity
- Supports filtering by metadata
- Handles batch operations for efficiency

RAG Process Flow

1. Documents are chunked into smaller pieces
2. Each chunk is embedded via the configured embedding model
3. Embeddings and metadata are stored in OpenSearch
4. User queries are converted to embeddings
5. Vector similarity search retrieves relevant document chunks
6. Retrieved context is sent to LLMs with user queries

The OpenSearch integration provides efficient vector search capabilities while maintaining compatibility with the application's vector database abstraction layer, allowing it to be interchangeable
with other supported vector databases like Chroma, Qdrant, Milvus, and others.


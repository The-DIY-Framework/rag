# DIY RAG

## Overview

This project implements an AI memory tool that leverages **Retrieval-Augmented Generation (RAG)** to provide a dynamic, searchable “memory” for your AI systems. Rather than relying solely on the model’s pre-trained (and static) parameters, this tool enables the AI to query an external knowledge base, retrieve relevant context, and then combine that with its internal reasoning to produce more accurate and domain-specific outputs.

The architecture is built using FastAPI for asynchronous and scalable API endpoints, integrates with LangChain for document embeddings and vector storage, and uses PostgreSQL with the pgvector extension as the default vector database. While initially designed for integration with [DIY Framework](https://www.diyframework.com/), the system is generic enough for any ID-based retrieval use case.

## Key Concepts

- **Retrieval-Augmented Generation (RAG):**  
  RAG is a two-phase process where an external retriever (e.g., vector database) is queried for context relevant to the input, and the retrieved data is then used to augment the generative process of a large language model (LLM). This method mitigates issues like outdated information and hallucinations inherent in standalone LLMs.

- **ID-based Organization:**  
  Documents and memory entries are grouped and indexed by a unique identifier (`file_id`). This design allows targeted querying and metadata association, ensuring that retrieval is both context-aware and precise.

- **Vector Embeddings:**  
  Each document is transformed into a vector representation (embedding) using state-of-the-art models (e.g., OpenAI’s embeddings or Hugging Face models). These embeddings are stored in a vector database (pgvector or Atlas MongoDB) to facilitate efficient similarity search.

- **Dynamic Memory Updates:**  
  The tool is designed to continuously ingest new data, update embeddings, and refresh the vector index, ensuring that the memory remains current and aligned with evolving external knowledge sources.

## Architecture

The solution comprises several components:

1. **Document Ingestion & Embedding:**

   - Raw documents (e.g., text files, PDFs) are processed and split into manageable chunks.
   - Each chunk is converted into an embedding vector.
   - Chunks are tagged with a unique `file_id` and stored in the vector database.

2. **Asynchronous API with FastAPI:**

   - Exposes endpoints for adding, retrieving, and deleting documents.
   - Supports asynchronous operations to handle concurrent requests efficiently.

3. **Retrieval Module:**

   - When a query is received, it is first converted into an embedding.
   - A similarity search is performed on the vector database to retrieve the top matching documents.
   - The retrieved context is then combined with the original query and forwarded to the LLM for augmented generation.

4. **Integration with DIY Framework:**
   - Although originally intended for integration with DIY Framework, the API’s modular design allows it to be adapted for various platforms and workflows.

## Features

- **Document Management:**  
  Methods to add, view, and remove memory entries, all indexed by a unique `file_id`.

- **Efficient Vector Store:**  
  Uses LangChain’s vector storage capabilities for rapid retrieval of relevant documents.

- **Asynchronous Operations:**  
  Built on FastAPI’s asynchronous architecture to support high-performance, scalable deployments.

- **Extensibility:**  
  Modular design allows future upgrades, such as alternative querying strategies (e.g., re-ranking, hybrid search) and different embedding providers.

## Setup

### Prerequisites

- **Python 3.10+**
- **PostgreSQL with pgvector extension** (or Atlas MongoDB if preferred)
- **Docker (optional)** for containerized deployment
- **Virtual Environment** (recommended for local development)

### Environment Configuration

Create a `.env` file with the following essential variables:

```env
# API Keys & Providers
RAG_OPENAI_API_KEY=your_openai_api_key_here
EMBEDDINGS_PROVIDER=openai  # Options: openai, azure, huggingface, etc.
EMBEDDINGS_MODEL=text-embedding-3-small

# Database Settings for pgvector
VECTOR_DB_TYPE=pgvector
POSTGRES_DB=your_database_name
POSTGRES_USER=your_db_user
POSTGRES_PASSWORD=your_db_password
DB_HOST=your_db_host
DB_PORT=5432

# API Server Settings
RAG_HOST=0.0.0.0
RAG_PORT=8000

# Other Configurations
COLLECTION_NAME=testcollection
CHUNK_SIZE=1500
CHUNK_OVERLAP=100
RAG_UPLOAD_DIR=./uploads/
DEBUG_RAG_API=True
```

For Atlas MongoDB integration, adjust the variables as follows:

```env
VECTOR_DB_TYPE=atlas-mongo
ATLAS_MONGO_DB_URI=<your_mongodb_connection_string>
COLLECTION_NAME=your_vector_collection
ATLAS_SEARCH_INDEX=your_search_index
```

### Database Setup

1. **Local PostgreSQL/pgvector Setup:**

   - Ensure PostgreSQL is installed and running.
   - Create the required database and enable the pgvector extension:
     ```sql
     CREATE DATABASE rag_api;
     \c rag_api
     CREATE EXTENSION IF NOT EXISTS vector;
     ```

2. **Using Docker:**
   - To run both the API and database:
     ```bash
     docker compose up
     ```
   - Alternatively, run only the database:
     ```bash
     docker compose -f ./db-compose.yaml up
     ```

### Running the API

- **Docker-based Execution:**
  ```bash
  docker compose up
  ```
- **Local Execution:**
  ```bash
  pip install -r requirements.txt
  uvicorn main:app --reload
  ```

## Advanced Configuration

- **Embedding and Vector Store Customization:**  
  You can change the embeddings model and vector storage backend (e.g., switching to Atlas MongoDB) by updating the environment variables and adjusting the vector store initialization in the code.

- **Security & JWT Integration:**  
  To secure your API, you may set the `JWT_SECRET` variable. If omitted, the API runs without authentication. This basic mechanism assumes a pre-signed JWT from your authentication provider.

- **Cloud Deployment:**  
  Follow the cloud-specific setup instructions if deploying on AWS RDS, Azure, or similar environments. Ensure your database instance supports the pgvector extension (check version requirements).

## Developer Notes

- **Pre-commit Formatter:**  
  For code quality, install pre-commit and set up the Black formatter:
  ```bash
  pip install pre-commit
  pre-commit install
  ```
- **Extending Query Capabilities:**  
  The API is designed to evolve. Future updates might include advanced retrieval strategies, re-ranking of retrieved documents, or integration with alternative LLMs.

- **Logging and Debugging:**  
  Enable detailed logging by setting `DEBUG_RAG_API=True` and adjust `CONSOLE_JSON` for JSON-based logging suitable for cloud logging aggregations.


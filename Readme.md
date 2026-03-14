# RAG PDF Chatbot Backend

A Node.js backend for a PDF-based Retrieval-Augmented Generation system. Users upload PDFs, the server chunks and embeds the content, stores vectors in Pinecone, and answers questions using Azure OpenAI with document-grounded context.

This repository was built as a 5th semester minor project and is structured as a practical backend for a document chat application.

## Highlights

- PDF ingestion pipeline using Multer and LangChain's PDF loader
- Recursive text chunking for better retrieval quality
- Azure OpenAI embeddings and chat completion integration
- Pinecone vector search with per-document namespaces
- Firebase token verification for protected routes
- Conversational memory per authenticated user session
- Simple Express API that is ready to connect to a frontend

## Tech Stack

| Layer | Tools |
| --- | --- |
| Runtime | Node.js, Express |
| Document Processing | Multer, LangChain PDFLoader, RecursiveCharacterTextSplitter |
| Embeddings + LLM | Azure OpenAI via LangChain |
| Vector Database | Pinecone |
| Authentication | Firebase Admin SDK |
| Development | Nodemon, dotenv |

## How It Works

```mermaid
flowchart LR
    A[Client uploads PDF] --> B[Multer stores file temporarily]
    B --> C[PDFLoader extracts text]
    C --> D[RecursiveCharacterTextSplitter creates chunks]
    D --> E[Azure OpenAI Embeddings]
    E --> F[Pinecone namespace per file]
    G[User asks question] --> H[Embed query]
    H --> I[Similarity search in Pinecone]
    I --> J[Prompt with retrieved context + chat history]
    J --> K[Azure OpenAI Chat Model]
    K --> L[Grounded answer returned]
```

## Core Flow

### 1. Document indexing

When a PDF is uploaded:

1. The file is accepted through Multer.
2. LangChain's PDF loader reads the document.
3. The text is split into chunks with:
   - `chunkSize: 500`
   - `chunkOverlap: 50`
4. Each chunk is embedded using Azure OpenAI.
5. The vectors are stored in Pinecone using the original filename as the namespace.

### 2. Retrieval and answering

When a user asks a question:

1. The query is embedded.
2. Pinecone performs similarity search against the selected namespace.
3. The top matching chunks are assembled into prompt context.
4. Prior chat history for that user is injected into the prompt.
5. Azure OpenAI generates a response constrained to the retrieved context.

## Features in the Current Implementation

- Protected upload, ask, and delete routes using Firebase ID token verification
- A dev-only public upload route controlled by `ALLOW_PUBLIC_UPLOAD=true`
- Health endpoint for uptime checks
- In-memory conversation history using `ChatMessageHistory`

## API Endpoints

### `GET /health`

Returns a basic status payload.

### `POST /file/upload`

Protected route. Accepts a PDF file as `multipart/form-data` with the field name `file`.

Behavior:

- loads the PDF
- splits it into chunks
- embeds and stores chunks in Pinecone
- deletes the local temp file after processing

### `POST /file/upload-public`

Available only when `ALLOW_PUBLIC_UPLOAD=true` is set in the environment.

### `POST /ask/ai`

Protected route. Expects JSON:

```json
{
  "query": "What is the paper about?",
  "namespace": "my-file.pdf"
}
```

Returns a grounded answer based on the most relevant stored chunks.

### `DELETE /file/delete/:filename`

Protected route. Deletes all vectors stored under the given Pinecone namespace.

## Project Structure

```text
.
|-- controller/
|   |-- embedAndStore.js
|   |-- retreiveDocument.js
|-- middleware/
|   |-- multer.js
|   |-- verifyToken.js
|-- temp/
|-- utils/
|   |-- ChatModel.js
|   |-- EmbeddingModel.js
|   |-- firebase.js
|   |-- Pinecone.js
|-- index.js
|-- package.json
```

## Local Setup

### 1. Clone the repository

```bash
git clone https://github.com/adarsh-gupta-01/rag-pdf-chatbot-Backend.git
cd rag-pdf-chatbot-Backend
```

### 2. Install dependencies

```bash
npm install
```

### 3. Create a `.env` file

Use the following variables as a starting point:

```env
PORT=5000
ALLOW_PUBLIC_UPLOAD=false

PINECONE_API_KEY=your_pinecone_api_key
PINECONE_INDEX_NAME=your_pinecone_index_name

AZURE_OPENAI_KEY=your_chat_model_key
AZURE_OPENAI_INSTANCE=your_chat_model_instance
AZURE_OPENAI_DEPLOYMENT_NAME=your_chat_model_deployment
AZURE_OPENAI_VERSION=your_chat_model_api_version

AZURE_OPENAI_API_KEY=your_embeddings_key
AZURE_OPENAI_API_INSTANCE_NAME=your_embeddings_instance
AZURE_OPENAI_API_EMBEDDINGS_DEPLOYMENT_NAME=your_embeddings_deployment
AZURE_OPENAI_API_VERSION=your_embeddings_api_version

FIREBASE_PROJECT_ID=your_firebase_project_id
FIREBASE_CLIENT_EMAIL=your_firebase_client_email
FIREBASE_PRIVATE_KEY="-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----\n"
```

### 4. Start the server

```bash
npm run dev
```

The backend will run on `http://localhost:5000` unless overridden by `PORT`.

## Authentication

Protected routes expect a Firebase ID token in the `Authorization` header:

```http
Authorization: Bearer <firebase_id_token>
```

## Notes and Limitations

- Chat memory is stored in memory, so it resets when the server restarts.
- Pinecone namespaces are based on filenames, so filename collisions should be considered in production.
- The `temp/` folder is used only for transient uploaded files and is excluded from Git.
- There is no test suite yet in the current version.

## Future Improvements

- Persistent conversation memory using Redis or a database
- Better namespace strategy using file IDs instead of raw filenames
- Role-based document ownership and access control
- Streaming responses for a better chat UX
- Automated tests and deployment pipeline

## Why This Project Matters

This backend demonstrates a complete RAG pipeline in a way that is practical, understandable, and extendable. It combines document ingestion, vector storage, authentication, and grounded AI responses into a single service that can power academic assistants, research tools, or internal knowledge bots.

## Author

Adarsh Gupta  
5th Semester Minor Project



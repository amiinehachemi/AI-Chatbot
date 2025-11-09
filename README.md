# AI Chatbot Service

AI Chatbot Service provides a Flask-based API for training and querying a retrieval-augmented generation (RAG) chatbot. The service leverages OpenAI's GPT models for response generation and Pinecone for semantic search across embedded documents. It exposes endpoints to ingest data, query the knowledge base with optional chat history, and monitor service health.

## Features
- **Website and text ingestion**: Embed content from arbitrary URLs or raw text into a Pinecone vector store.
- **Conversational querying**: Generate responses using OpenAI's chat completions augmented with Pinecone-powered retrieval, optionally grounded on previous chat history.
- **REST API**: Simple JSON endpoints for training, querying, and health checks.

## Project Structure
```
.
├── controllers/
│   ├── open_ai_controller.py   # Orchestrates OpenAI chat completions and retrieval chains.
│   └── pinecone_controller.py  # Handles embedding, document loading, and Pinecone vector store interactions.
├── server.py                   # Flask application exposing training and query endpoints.
├── conversational_chain.py     # Example CLI workflow for chat history aware retrieval.
├── utils/
│   └── returner.py             # Helper for CORS-friendly JSON responses.
├── requirements.txt            # Python dependencies.
└── dockerFile                  # Container definition for deployment.
```

## Prerequisites
- Python 3.10+
- An [OpenAI API key](https://platform.openai.com/account/api-keys)
- A [Pinecone](https://www.pinecone.io/) account and API key

## Installation
1. Clone the repository and enter the project directory:
   ```bash
   git clone <repository-url>
   cd AI-Chatbot
   ```
2. (Optional) Create and activate a virtual environment:
   ```bash
   python -m venv .venv
   source .venv/bin/activate
   ```
3. Install the Python dependencies:
   ```bash
   pip install -r requirements.txt
   ```

## Environment Variables
Create a `.env` file or export the following environment variables before running the service:

| Variable | Description |
| --- | --- |
| `OPENAI_API_KEY` | OpenAI API key used by LangChain chat and embedding clients. |
| `PINECONE_API_KEY` | Pinecone API key for index management and queries. |
| `PINECONE_ENVIRONMENT` | Pinecone environment name (e.g., `gcp-starter`). |
| `PINECONE_INDEX_NAME` | *(Optional)* Name of the Pinecone index. Defaults to `rapid` in the code. |
| `SERVER_PORT` | *(Optional)* Port for the Flask server. Defaults to `8080`. |

## Running the Server
Start the Flask application:
```bash
python server.py
```
The API will listen on `http://0.0.0.0:<SERVER_PORT>` (8080 by default).

## API Endpoints
All endpoints accept and return JSON.

### `GET /health`
Returns a simple health status message.

### `POST /train/website`
Embed content fetched from a URL.
- **Body parameters**:
  - `website` (string): URL to crawl.
  - `data_type` (string): Must be `"website"`.
  - `namespace` (string): Pinecone namespace where embeddings are stored.
- **Response**: `200 OK` when ingestion succeeds.

### `POST /train/inputs`
Embed raw text input.
- **Body parameters**:
  - `inputs` (string): Text content to embed.
  - `data_type` (string): Must be `"inputs"`.
  - `namespace` (string): Pinecone namespace.
- **Response**: `200 OK` when ingestion succeeds.

### `POST /query`
Retrieve relevant context from Pinecone and generate an answer with OpenAI.
- **Body parameters**:
  - `query` (string): User question.
  - `namespace` (string): Pinecone namespace that was previously populated.
  - `chat_history` (array, optional): List of objects with `question` and `response` fields to provide conversational memory.
- **Response**: Generated answer text.

## Example Usage
```bash
curl -X POST http://localhost:8080/train/inputs \
  -H "Content-Type: application/json" \
  -d '{
    "inputs": "LangChain connects language models to other sources of data.",
    "data_type": "inputs",
    "namespace": "docs"
  }'

curl -X POST http://localhost:8080/query \
  -H "Content-Type: application/json" \
  -d '{
    "query": "What does LangChain do?",
    "namespace": "docs",
    "chat_history": []
  }'
```

## Development Notes
- `conversational_chain.py` demonstrates how to build a CLI chatbot that reuses the same retrieval pipeline with chat history.
- Update `dockerFile` if you plan to containerize the application.

## License
Add your preferred license information here.

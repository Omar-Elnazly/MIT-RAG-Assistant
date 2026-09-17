# SmartUniversityAssistant

An AI-powered university assistant that answers questions about MIT university information using Retrieval-Augmented Generation (RAG).

The system retrieves relevant information from a processed university document collection and uses a locally running Large Language Model (LLM) through Ollama to generate grounded answers with source citations.

---

## Overview

University information is distributed across many web pages and documents, making it difficult for students to quickly find accurate answers.

SmartUniversityAssistant provides a conversational interface where users can ask questions about university-related information.

The system follows a RAG architecture:

1. User asks a question.
2. Relevant document chunks are retrieved from a persistent ChromaDB vector store.
3. The retrieved context is provided to the LLM.
4. Ollama generates an answer using only the retrieved context.
5. The answer is returned with the supporting sources.

If the required information is not available in the retrieved documents, the system is designed to avoid guessing and return a `NOT_IN_CONTEXT` response.

---

## Architecture

```text
User
  |
  v
Streamlit Frontend
  |
  | HTTP
  v
FastAPI Backend
  |
  v
Retrieval Service
  |
  +--> Sentence Transformer
  |
  +--> ChromaDB
  |
  |    Top 5 chunks
  v
Generation Service
  |
  +--> Ollama
       llama3.2:3b
  |
  v
Grounded Answer + Sources
```

---

## Features

- Retrieval-Augmented Generation (RAG)
- Section-aware document chunking
- Persistent ChromaDB vector store
- Semantic search using Sentence Transformers
- Local LLM inference using Ollama
- Grounded answers based only on retrieved context
- Source citations
- FastAPI REST API
- Streamlit conversational interface
- API health monitoring
- Input validation
- Friendly frontend error handling
- Automated backend tests
- RAG evaluation

---

## Project Structure

```text
SmartUniversityAssistant/
|
├── notebooks/
│   ├── collecting_data.ipynb
│   ├── document_processing.ipynb
│   └── rag_pipeline.ipynb
|
├── data/
│   ├── raw/
│   └── processed/
│       └── documents/
|
├── backend/
│   ├── app/
│   │   ├── __init__.py
│   │   ├── main.py
│   │   ├── api/
│   │   │   ├── __init__.py
│   │   │   └── routes/
│   │   │       ├── __init__.py
│   │   │       └── query.py
│   │   ├── core/
│   │   │   ├── __init__.py
│   │   │   └── config.py
│   │   ├── schemas/
│   │   │   ├── __init__.py
│   │   │   └── query.py
│   │   ├── services/
│   │   │   ├── __init__.py
│   │   │   ├── retrieval.py
│   │   │   └── generation.py
│   │   └── utils/
│   │       ├── __init__.py
│   │       └── logging_config.py
│   ├── data/
│   │   └── vector_store/
│   ├── tests/
│   │   ├── __init__.py
│   │   └── test_query.py
│   ├── requirements.txt
│   ├── .env.example
│   └── Dockerfile
|
├── frontend/
│   ├── app.py
│   ├── api_client.py
│   ├── .env.example
│   └── requirements.txt
|
├── README.md
└── .gitignore
```

---

# Data

The project uses university-related documents collected from MIT university web pages.

The document processing pipeline converts the collected source material into structured JSON documents containing information such as:

- Document title
- Source URL
- Document type
- Pages
- Sections
- Headings
- Paragraphs
- Tables
- Metadata

The document processing stage produced approximately:

- **1,264 processed documents**
- **2,143 pages**
- **1,000 JSON documents**
- **136 PDF documents**
- **2 DOCX documents**
- **126 image documents**
- **128 documents processed using OCR**
- **12,488 headings**
- **41,430 paragraphs**
- **4,943 tables**

After document processing, the RAG pipeline identified **1,142 usable documents** and generated **8,108 semantic chunks**.

## Dataset / Processed Data

The complete collected and processed dataset is available through Google Drive:

**Google Drive:** [Dataset & Processed Data](https://drive.google.com/drive/folders/1AHPjIn3JEx-8EqtkMxOAOkghDm1UJlAE?usp=drive_link)

Replace the placeholder above with the actual Google Drive link.

---

# RAG Pipeline

The RAG pipeline is implemented in:

```text
notebooks/rag_pipeline.ipynb
```

## 1. Document Loading

Processed JSON documents are loaded and inspected.

The pipeline checks:

- Number of documents
- Document types
- Pages
- Content availability
- Parsing failures
- Metadata

## 2. Section-Aware Chunking

The project uses section-aware chunking.

Configuration:

```text
Chunk size:       220 words
Chunk overlap:     40 words
Minimum section:   60 words
```

Each heading starts a section, and chunks are kept within their corresponding sections and pages.

The heading is reattached to each chunk to preserve context during retrieval.

## 3. Embeddings

The embedding model used is:

```text
sentence-transformers/all-MiniLM-L6-v2
```

Embedding dimension:

```text
384
```

Embeddings are normalized and compared using cosine similarity.

## 4. Vector Store

ChromaDB is used as the persistent vector database.

Collection:

```text
rag_assistant_chunks
```

Number of stored chunks:

```text
8,108
```

The vector store is persisted to disk and loaded by the backend.

The backend does not rebuild the vector store during requests.

## 5. Retrieval

For each user question, the system:

1. Converts the question into an embedding.
2. Searches the ChromaDB vector store.
3. Retrieves the top 5 relevant chunks.
4. Passes the retrieved context to the generation service.

## 6. Grounded Generation

The generation service uses:

```text
Ollama
llama3.2:3b
```

The LLM receives:

- Retrieved document context
- The user's question
- Grounding rules

The system instructs the model to:

- Use only retrieved context
- Avoid outside knowledge
- Provide source citations
- Avoid inventing information
- Return `NOT_IN_CONTEXT` when the required information is unavailable

---

# Backend

The backend is implemented using **FastAPI**.

## API Endpoints

### Health Check

```http
GET /health
```

Example:

```bash
curl http://localhost:8000/health
```

### Query

```http
POST /query
```

Request:

```json
{
  "question": "Who is the Political Science representative?"
}
```

Response:

```json
{
  "answer": "....",
  "sources": [
    "...."
  ]
}
```

---

# Frontend

The frontend is implemented using **Streamlit**.

It provides:

- Chat-style interface
- User question input
- AI-generated answers
- Source display
- Loading indicators
- Backend status
- Friendly error messages
- Conversation clearing

The frontend communicates with the FastAPI backend through the `API_BASE_URL` environment variable.

---

# Technologies

| Component | Technology |
|---|---|
| Programming Language | Python |
| Backend | FastAPI |
| Frontend | Streamlit |
| Vector Database | ChromaDB |
| Embeddings | Sentence Transformers |
| Embedding Model | all-MiniLM-L6-v2 |
| LLM Runtime | Ollama |
| LLM | llama3.2:3b |
| API Client | Requests |
| Configuration | Pydantic Settings |
| Testing | Pytest |

---

# Installation

## Requirements

- Python 3.10+
- Ollama
- Git
- GitHub account

## 1. Clone the Repository

```bash
git clone YOUR_GITHUB_REPOSITORY_URL
cd SmartUniversityAssistant
```

---

# 2. Install Ollama

Install Ollama for your operating system.

Then download the required model:

```bash
ollama pull llama3.2:3b
```

Verify:

```bash
ollama list
```

You should see:

```text
llama3.2:3b
```

---

# 3. Backend Setup

Navigate to the backend:

```bash
cd backend
```

Create a virtual environment:

```bash
python -m venv .venv
```

Activate it on Windows:

```powershell
.venv\Scripts\activate
```

Install dependencies:

```bash
pip install -r requirements.txt
```

## Backend Environment Variables

Create:

```text
backend/.env
```

with:

```env
OLLAMA_HOST=http://127.0.0.1:11434
OLLAMA_MODEL=llama3.2:3b
```

The repository contains `backend/.env.example` as a template.

---

# 4. Start Backend

From the `backend` directory:

```bash
uvicorn app.main:app --reload
```

The API will be available at:

```text
http://localhost:8000
```

Swagger documentation:

```text
http://localhost:8000/docs
```

Health check:

```text
http://localhost:8000/health
```

---

# 5. Frontend Setup

Open another terminal.

Navigate to the frontend:

```bash
cd frontend
```

Create a virtual environment:

```bash
python -m venv .venv
```

Activate it:

```powershell
.venv\Scripts\activate
```

Install dependencies:

```bash
pip install -r requirements.txt
```

## Frontend Environment Variables

Create:

```text
frontend/.env
```

with:

```env
API_BASE_URL=http://localhost:8000
```

The repository contains `frontend/.env.example` as a template.

---

# 6. Start Frontend

From the `frontend` directory:

```bash
streamlit run app.py
```

The application will normally be available at:

```text
http://localhost:8501
```

---

# Testing

Backend tests are located in:

```text
backend/tests/test_query.py
```

Run:

```bash
pytest
```

The test suite includes checks for:

- Health endpoint
- Valid query requests
- Missing question validation
- Blank question validation

---

# Evaluation

The RAG pipeline was evaluated using test questions covering retrieval relevance, grounding, and answer correctness.

Current evaluation results:

| Metric | Result |
|---|---:|
| Retrieval Relevance | 92.3% |
| Grounded Answers | 84.6% |
| Answer Correctness | 69.2% |
| In-Domain Correctness | 72.7% |
| Generation Errors | 0 |

The evaluation is a practical sanity check of the current RAG pipeline.

Some retrieval weaknesses remain for highly specific or ambiguous questions, where generic queries may retrieve information from unrelated university pages.

---

# Example Questions

```text
What subjects are offered in the half-term?

Who is the Political Science representative?

What are the prerequisites for 18.03?

What information is available about the summer term?

What is the population of Egypt?
```

The final question is intentionally outside the university corpus and can be used to demonstrate the system's grounding behavior.

---

# Grounding and Hallucination Prevention

A key design goal of SmartUniversityAssistant is to reduce unsupported answers.

The generation prompt instructs the LLM to use only the retrieved context.

When the retrieved documents do not contain sufficient information, the system can return:

```text
NOT_IN_CONTEXT: The retrieved documents do not contain information to answer this question.
```

This prevents the assistant from presenting external knowledge as if it came from the university document collection.

---

# Limitations

- Retrieval quality depends on the quality and coverage of the collected documents.
- Ambiguous questions may retrieve less relevant chunks.
- Some highly specific course queries may require improved retrieval strategies.
- Answer correctness depends on the retrieved context and LLM generation.
- The system is currently focused on text-based RAG.
- The Extended Track vision/YOLO functionality is not included in this version.

---

# Future Improvements

Possible future improvements include:

- Query rewriting
- Hybrid keyword + semantic retrieval
- Reranking retrieved chunks
- Better handling of course-specific queries
- Improved evaluation datasets
- Conversation memory
- Authentication
- Cloud deployment
- Monitoring and logging improvements
- Additional university data sources

---

# Project Deliverables

- Data collection notebook
- Document processing notebook
- RAG pipeline notebook
- Persisted ChromaDB vector store
- FastAPI backend
- Streamlit frontend
- API tests
- Environment configuration templates
- Evaluation results
- Documentation

---

# License

This project was developed as a graduation project.

Add the appropriate license here if the project is released under a specific open-source license.

---

# Authors

**SmartUniversityAssistant Team**

Graduation Project

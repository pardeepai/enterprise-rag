# Enterprise RAG

Enterprise-focused Retrieval-Augmented Generation (RAG) system for querying internal documents using hybrid retrieval, metadata filtering, reranking, and local LLM inference.

This project is designed to help teams answer questions from enterprise documents such as policies, standard operating procedures, HR documents, or internal knowledge bases using a document-grounded search + generation workflow.

## Why this project?

Most enterprise knowledge lives in PDFs and internal documentation, not in structured databases. Building a reliable RAG pipeline requires more than just a large LLM:

- strong document ingestion
- chunking that preserves context
- semantic search for meaning
- keyword matching for exact terms
- reranking for better retrieval quality
- metadata-aware filtering
- evaluation to measure retrieval and answer quality

This repository implements that workflow end-to-end.

## Architecture

```text
PDF Documents
  ↓
Document Loading
  ↓
Chunking
  ↓
Embedding Generation
  ↓
FAISS + BM25 Retrieval
  ↓
Hybrid Retrieval + RRF
  ↓
Candidate Pool
  ↓
Cross-Encoder Reranking
  ↓
Top-k Context Selection
  ↓
Ollama LLM
  ↓
Answer + Source References
```

## Key Features

- PDF ingestion and preprocessing
- Sentence-aware chunking with overlap
- Semantic retrieval using embeddings and FAISS
- Keyword retrieval using BM25
- Hybrid retrieval with Reciprocal Rank Fusion (RRF)
- Metadata filtering by filename, department, and page label
- Cross-encoder reranking for improved candidate ranking
- Local LLM inference with Ollama
- FastAPI service for search and chat endpoints
- Retrieval evaluation with Hit@3 and MRR metrics

## Tech Stack

- Python
- FastAPI
- FAISS
- BM25
- Sentence transformers / embeddings
- Cross-encoder reranking
- Ollama
- LlamaIndex
- PyMuPDF / PyPDF
- Docker-ready project structure

## Repository Structure

```text
enterprise-rag/
├── app/
│   ├── chunking/
│   ├── embeddings/
│   ├── evaluation/
│   ├── faiss_store/
│   ├── llm/
│   ├── loaders/
│   ├── rag/
│   ├── retrieval/
│   ├── __init__.py
│   └── main.py
├── data/
├── tests/
├── Dockerfile
├── docker-compose.yml
├── evaluation_report.json
├── requirements.txt
├── pyproject.toml
├── README.md
├── setup.py
└── .gitignore
```

## Getting Started

### Prerequisites

- Python 3.10+
- pip
- Ollama installed and running locally
- A model pulled in Ollama (for example: `mistral` or another suitable model)

### 1) Clone the repository

```bash
git clone https://github.com/pardeepai/enterprise-rag.git
cd enterprise-rag
```

### 2) Create a virtual environment

```bash
python -m venv .venv
source .venv/bin/activate   # Linux/macOS
# .venv\Scripts\activate   # Windows
```

### 3) Install dependencies

```bash
pip install -r requirements.txt
```

### 4) Start Ollama

Make sure Ollama is running and the model is available.

```bash
ollama pull mistral
ollama serve
```

### 5) Run the API

```bash
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

Then open:

- http://localhost:8000/docs
- http://localhost:8000/redoc

## API Endpoints

| Method | Endpoint | Purpose |
|--------|----------|---------|
| GET | `/` | API status |
| GET | `/health` | Health check |
| GET | `/load-pdfs` | Load documents from the configured PDF source |
| GET | `/chunk-pdfs` | Chunk loaded documents |
| POST | `/search` | Semantic search |
| POST | `/hybrid-search` | Hybrid search combining semantic + BM25 |
| POST | `/hybrid-rerank` | Hybrid search with reranking |
| POST | `/bm25-search` | BM25-only keyword search |
| POST | `/chat` | Ask a question using the RAG pipeline |
| POST | `/ingest` | Build the FAISS index |

## Example Requests

### Health check

```bash
curl http://localhost:8000/health
```

### Semantic search

```bash
curl -X POST "http://localhost:8000/search" \
  -H "Content-Type: application/json" \
  -d '{
    "question": "What is the leave policy?",
    "top_k": 5,
    "department": "hr"
  }'
```

### Hybrid search with reranking

```bash
curl -X POST "http://localhost:8000/hybrid-rerank" \
  -H "Content-Type: application/json" \
  -d '{
    "question": "What are the annual leave rules?",
    "top_k": 3,
    "department": "hr",
    "enable_rerank": true
  }'
```

### RAG chat

```bash
curl -X POST "http://localhost:8000/chat" \
  -H "Content-Type: application/json" \
  -d '{
    "question": "Summarize the key leave policy requirements for employees."
  }'
```

## Retrieval Strategy

### Semantic Search

Uses embedding-based retrieval over a FAISS index to capture meaning, paraphrases, and conceptual similarity.

### BM25 Search

Uses keyword search to surface exact terms and domain-specific phrases that may not be well represented by embeddings alone.

### Hybrid Retrieval

The system combines semantic search and BM25 using reciprocal rank fusion (RRF), which helps reduce blind spots in either retrieval method.

### Reranking

After candidate retrieval, a cross-encoder reranker reorders the retrieved chunks to improve final context quality before the LLM is asked to answer.

## Chunking and Metadata

The project uses sentence-aware chunking with overlap to preserve context between adjacent chunks. Metadata such as:

- filename
- page
- department
- policy_type

can be used to filter retrieval and improve enterprise-level relevance.

## Evaluation

The project includes evaluation logic for retrieval quality.

- Questions: 20
- Top-k: 3
- Primary metrics: Hit@3 and MRR

### Results

| Method | Hit@3 | MRR |
|--------|------:|----:|
| Hybrid Search | 80% | 0.6667 |
| Hybrid + Reranking | 90% | 0.7083 |
| MMR | 85% | 0.7167 |

This shows that hybrid retrieval with reranking is a strong default strategy, while MMR may improve ranking quality depending on the data distribution.

## Failure Analysis

The evaluation notes that the failed questions were caused by expected chunks not being present in the initial retrieval candidate pool. This is an important reminder that reranking alone cannot recover missing evidence if the relevant content was never retrieved in the first place.

## Production Readiness Notes

This repository is a strong foundation for an enterprise RAG prototype, but additional work is needed before production deployment:

- environment-based configuration
- structured logging
- better error handling / validation
- authentication and authorization
- rate limiting
- monitoring and health checks
- async request handling
- security hardening for ingestion endpoints
- automated integration tests

## Project Status

- [x] PDF ingestion
- [x] Chunking
- [x] Embeddings
- [x] FAISS search
- [x] BM25 search
- [x] Hybrid retrieval
- [x] Reciprocal rank fusion
- [x] Metadata filtering
- [x] MMR evaluation
- [x] Cross-encoder reranking
- [x] FastAPI API
- [x] Retrieval evaluation
- [x] Failure analysis

## Roadmap

Planned improvements include:

- stronger config management via environment variables
- production-grade logging and observability
- multi-document ingestion workflow
- better access control for enterprise users
- Retrieval QA benchmarking with answer-level evaluation
- caching for repeated queries
- improved deployment setup for Docker/Kubernetes

## License

This project currently does not declare a license in the repository metadata. If you plan to share or deploy it broadly, add a license file and explicitly specify the intended usage terms.

## Contributing

Contributions are welcome. If you want to improve retrieval quality, deployment workflow, or system reliability, open an issue or submit a pull request with a clear explanation of the change.

## Contact

For questions or collaboration, use the repository issues and discussions page.

---

This README is intended to be practical and developer-friendly, with enough detail to help someone understand, run, and extend the project.

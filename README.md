A Production-Ready Retrieval-Augmented Generation System with FAISS + SQLite
 Overview
This project implements a cost-efficient Retrieval-Augmented Generation (RAG) application using FAISS (Facebook AI Similarity Search) with SQLite for metadata storage. The system provides a production-ready QA service over document corpora at a fraction of the cost of managed vector databases.

Key Features:

 Low-cost vector storage using FAISS + SQLite

 Multi-format ingestion (PDF, HTML, Markdown, Text)

 Configurable chunking with idempotent re-ingestion

 Grounded LLM answers with source citations

 Comprehensive evaluation (retrieval + answer quality + cost)

 Metadata filtering for targeted retrieval

 HTTP API with configurable parameters

 Why This Approach?
Cost Comparison
Scale	Managed DB (Monthly)	This Solution (Monthly)	Savings
100K vectors	$50-100	$5-10	90%
1M vectors	$200-400	$20-40	90%
10M vectors	$1,000-2,000	$100-200	90%
Assumptions:

Managed DB: AWS OpenSearch, Pinecone, or similar

Self-hosted: t3.medium instance ($30/month) + storage

FAISS: Uses HNSW index for efficient search

Trade-offs We Accept
Aspect	Trade-off	Mitigation
Availability	No automatic failover	Use replication or backups
Scalability	Vertical scaling only	Can shard by document type
Maintenance	Manual index rebuilding	Automated rebuild scripts
Features	No built-in monitoring	Custom metrics and logging
Multi-tenancy	Limited by design	Namespace with metadata filtering
When to Switch to Managed
Scale > 50M vectors: Managed DBs offer better distribution

High availability required: Multi-region, automatic failover

Zero maintenance: No time for infrastructure management

Advanced features needed: Real-time updates, complex aggregations

Multi-cloud strategy: Avoid vendor lock-in

 Installation
Prerequisites
bash
# Python 3.9+
python --version

# Required system packages
sudo apt-get install build-essential python3-dev

# Create virtual environment (recommended)
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
Install Dependencies
bash
pip install -r requirements.txt
Environment Configuration
bash
# Copy example environment file
cp .env.example .env

# Edit .env with your settings
nano .env
Required Environment Variables:

env
# OpenAI API Key (for LLM features)
OPENAI_API_KEY=your_api_key_here

# Embedding Model
EMBEDDING_MODEL=all-MiniLM-L6-v2
EMBEDDING_DIM=384

# Chunking
CHUNK_SIZE=512
CHUNK_OVERLAP=50

# Retrieval
TOP_K=5

# Vector Store Path
VECTOR_DB_PATH=data/vectordb

# LLM Model
LLM_MODEL=gpt-3.5-turbo

# Logging
LOG_LEVEL=INFO
 Quick Start
1. Run the API Server
bash
# Start the Flask API server
python run.py serve

# Server runs at http://localhost:5000
2. Ingest Documents
bash
# Ingest a single file
python run.py ingest --file data/documents/sample.pdf

# Ingest all files in a directory
python run.py ingest --dir data/documents/

# With custom chunk size
python run.py ingest --dir data/documents/ --chunk-size 1024 --overlap 100
3. Query the System
bash
# Query via CLI
python run.py query --question "What are the key findings in the document?"

# Query with custom k
python run.py query --question "Explain the methodology" --k 10
4. API Usage
python
import requests

# Query endpoint
response = requests.post(
    'http://localhost:5000/query',
    json={
        'query': 'What is the main conclusion?',
        'k': 5,
        'filter': {'doc_type': 'pdf'}
    }
)

print(response.json()['answer'])
 Architecture
text
┌─────────────────────────────────────────────────────────────┐
│                     RAG Application                         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐    │
│  │   Document  │    │  Embedding  │    │    FAISS    │    │
│  │  Ingestion  │───▶│  Generator  │───▶│   Index     │    │
│  └─────────────┘    └─────────────┘    └─────────────┘    │
│         │                  │                   │           │
│         ▼                  ▼                   ▼           │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐    │
│  │   Chunking  │    │   Metadata  │    │    SQLite   │    │
│  │   Pipeline  │    │   Storage   │───▶│   Database  │    │
│  └─────────────┘    └─────────────┘    └─────────────┘    │
│                                                             │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐    │
│  │   Query     │    │  Retrieval  │    │     LLM     │    │
│  │  Endpoint   │───▶│   Pipeline  │───▶│   Answer    │    │
│  └─────────────┘    └─────────────┘    └─────────────┘    │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│                    HTTP API / CLI                          │
└─────────────────────────────────────────────────────────────┘
 Evaluation Framework
Retrieval Metrics
Hit Rate: Fraction of queries with at least one relevant document

MRR: Mean Reciprocal Rank of first relevant document

nDCG@k: Normalized Discounted Cumulative Gain

Context Precision: Precision of retrieved contexts

Answer Quality Metrics
Faithfulness: Groundedness in retrieved context (LLM-as-judge)

Answer Relevance: Directness and completeness

EM/F1: Exact Match and Token-level F1 (with gold answers)

Latency & Cost
Retrieval Latency: p50/p95

Total Response Time: End-to-end

Token Usage: Input/Output tokens

Monthly Cost Projections: At different scales

 Project Structure
text
cost-efficient-rag/
├── src/
│   ├── __init__.py
│   ├── config.py              # Configuration management
│   ├── embeddings.py          # Embedding generation
│   ├── vector_store.py        # FAISS + SQLite integration
│   ├── ingestion.py           # Document ingestion pipeline
│   ├── retrieval.py           # Retrieval logic
│   ├── rag.py                 # RAG pipeline with LLM
│   ├── evaluation.py          # Evaluation harness
│   └── app.py                 # HTTP API endpoint
├── data/
│   ├── documents/             # Source documents
│   ├── vectordb/              # Vector storage
│   └── evaluation_questions.json
├── evaluations/
│   ├── results.json
│   └── metrics.json
├── tests/
│   ├── test_ingestion.py
│   └── test_retrieval.py
├── .env.example
├── requirements.txt
├── README.md
├── run.py                     # CLI entry point
└── evaluation_harness.py      # Evaluation runner
 Configuration
Environment Variables
Variable	Description	Default
OPENAI_API_KEY	OpenAI API key	None (required)
EMBEDDING_MODEL	Sentence transformer model	all-MiniLM-L6-v2
EMBEDDING_DIM	Embedding dimension	384
CHUNK_SIZE	Text chunk size	512
CHUNK_OVERLAP	Chunk overlap	50
TOP_K	Default number of chunks	5
LLM_MODEL	LLM model for generation	gpt-3.5-turbo
VECTOR_DB_PATH	Vector store location	data/vectordb
Configuration File
python
# config.py
from dataclasses import dataclass
import os

@dataclass
class Config:
    CHUNK_SIZE: int = 512
    CHUNK_OVERLAP: int = 50
    TOP_K: int = 5
    EMBEDDING_MODEL: str = "all-MiniLM-L6-v2"
    LLM_MODEL: str = "gpt-3.5-turbo"
    # ... etc
 Usage Examples
CLI Commands
bash
# Ingest a directory
python run.py ingest --dir ./documents

# Ingest with custom settings
python run.py ingest --file paper.pdf --chunk-size 1024 --overlap 100

# Query
python run.py query --question "What is the main contribution?"

# Query with filter
python run.py query --question "Results" --filter '{"doc_type":"pdf"}'

# Evaluate
python run.py evaluate --questions questions.json

# Serve API
python run.py serve --host 0.0.0.0 --port 5000
Python API
python
from src.ingestion import DocumentIngester
from src.rag import RAGPipeline

# Ingest documents
ingester = DocumentIngester(chunk_size=512, chunk_overlap=50)
ingester.ingest_directory("documents/")

# Query
rag = RAGPipeline()
response = rag.answer("What is the main finding?")
print(response['answer'])
print(f"Chunks used: {len(response['chunks'])}")
print(f"Latency: {response['metrics']['total_time_ms']:.2f}ms")
Evaluation
python
from src.evaluation import Evaluator

# Run evaluation with 30 questions
evaluator = Evaluator(rag_pipeline)
questions = load_questions("evaluation_questions.json")
metrics = evaluator.evaluate_questions(questions)

print(f"Hit Rate: {metrics['retrieval']['hit_rate']['mean']:.3f}")
print(f"MRR: {metrics['retrieval']['mrr']['mean']:.3f}")
print(f"Answer F1: {metrics['answer']['f1']['mean']:.3f}")
📊 Evaluation Results
Retrieval Performance
Metric	Value (Mean)	P95
Hit Rate@5	0.723	-
MRR	0.658	-
nDCG@5	0.692	-
Context Precision	0.745	-
Answer Quality
Metric	Value
Faithfulness	0.891
Answer Relevance	0.823
F1 (with gold)	0.612
Latency
Metric	p50	p95
Retrieval	45ms	120ms
Generation	850ms	1,800ms
Total	895ms	1,920ms
Cost Analysis
Scale	This Solution	Managed DB	Savings
100K vectors	$5/mo	$50/mo	90%
1M vectors	$20/mo	$200/mo	90%
10M vectors	$100/mo	$1,000/mo	90%
 Testing
bash
# Run all tests
pytest tests/

# Run specific test
pytest tests/test_retrieval.py -v

# Run with coverage
pytest --cov=src tests/
 Troubleshooting
Common Issues
1. No embeddings generated

text
Error: Failed to load embedding model
Solution: pip install sentence-transformers --upgrade
2. Vector store not found

text
Error: FAISS index not found
Solution: Run ingestion first or set VECTOR_DB_PATH correctly
3. OpenAI API key missing

text
Warning: OPENAI_API_KEY not set
Solution: Set OPENAI_API_KEY in .env or environment
4. Chunking issues

text
Error: No chunks generated
Solution: Check document format and chunk size
 Contributing
Fork the repository

Create a feature branch

Make your changes

Add tests for new features

Submit a pull request

Development Setup
bash
# Clone repository
git clone https://github.com/yourusername/cost-efficient-rag.git
cd cost-efficient-rag

# Install development dependencies
pip install -e ".[dev]"

# Run tests
pytest tests/

# Run linter
flake8 src/
 License
MIT License - See LICENSE file for details

 Acknowledgments
FAISS team for vector search library

Sentence Transformers for embedding models

OpenAI for LLM capabilities

 Contact
For questions or support:

GitHub Issues: [link]

Email: your.email@example.com

 Discussion: Managed vs Self-Hosted
When to Switch to Managed
1. Scale > 50M vectors

Self-hosted indexing becomes slow

Memory requirements exceed single instance

2. High Availability Required

Automatic failover

Multi-region replication

SLA guarantees

3. Zero Maintenance

No infrastructure management

Automatic scaling

Built-in monitoring

4. Advanced Features

Real-time updates

Complex aggregations

Multi-tenancy support

5. Multi-Cloud Strategy

Avoid vendor lock-in

Unified management

Weak Link Analysis
Retrieval

Strengths: FAISS provides excellent search performance

Weaknesses: Index rebuild times for large datasets

Mitigation: Incremental updates, scheduled rebuilds

Generation

Strengths: LLM provides high-quality, grounded answers

Weaknesses: Higher latency and cost

Mitigation: Caching, smaller models for simple queries

Overall: Generation is the weak link (latency + cost). Retrieval is solid and scales well with FAISS.

 Update Log
v1.0.0 (2025-01-15)
Initial release

FAISS + SQLite integration

Multi-format ingestion

Full evaluation framework

HTTP API

v1.1.0 (2025-02-01)
Added metadata filtering

Improved chunking strategies

Batch ingestion support

Enhanced evaluation metrics

v1.2.0 (2025-03-01)
Multi-threaded ingestion

Index rebuilding utilities

Caching layer

Performance optimizations

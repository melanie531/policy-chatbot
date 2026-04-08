# Policy Chatbot — Project Specification

## 1. Overview

A RAG-powered Q&A chatbot that helps internal employees find answers about company policies. Employees ask natural language questions, the agent retrieves relevant policy sections from a FAISS vector store, and generates grounded answers with source citations using Claude Sonnet 4 via Amazon Bedrock.

**Scope:** PoC — 2 sample policy documents (Code of Conduct, Data Security), API-first, deployed to AgentCore.

## 2. Users

- **Primary:** Internal employees asking policy questions
- **Channel:** REST API (API-first, no UI for PoC)
- **Auth:** None for PoC (AgentCore handles access at the platform level)

## 3. Functional Requirements

### FR-001: Document Ingestion Pipeline
- Load `.txt` policy files from a configurable directory
- Split into overlapping text chunks (configurable size/overlap)
- Generate embeddings using Amazon Titan Embed Text v2 (`amazon.titan-embed-text-v2:0`)
- Build a FAISS `IndexFlatIP` index (cosine similarity via normalized vectors)
- Persist index and chunk metadata to disk

### FR-002: Knowledge Retrieval
- Accept a natural language query
- Embed the query using the same Titan model
- Search FAISS index for top-K most similar chunks
- Return results with text, source filename, chunk ID, and similarity score
- Format results as a context string with source citations

### FR-003: Agent Q&A
- Use Strands SDK `Agent` with Claude Sonnet 4 via `BedrockModel`
- Expose a `retrieve_policy_context` tool the agent must call before answering
- System prompt enforces: always use the tool, cite sources, admit when KB lacks info
- Return structured JSON: `answer`, `sources`, `escalation`, `escalation_reason`
- Escalation triggers: questions outside policy scope, requests for policy changes, legal advice

### FR-004: REST API
- `POST /ask` — Submit a question, receive answer with sources and escalation status
- `GET /health` — Service health with document count and vector count
- Input validation: question minimum 5 characters
- FastAPI with Pydantic models

### FR-005: CLI
- `ingest` — Run the ingestion pipeline
- `serve` — Start the FastAPI server
- `ask "<question>"` — One-off question from terminal

### FR-006: Sample Policy Documents
- `code_of_conduct.txt` — Workplace behavior, harassment policy, dress code, social media, reporting
- `data_security_policy.txt` — Data classification, handling procedures, incident reporting, acceptable use, remote work security

## 4. Architecture

### 4.1 Project Structure

```
policy-chatbot/
├── src/
│   └── policy_chatbot/
│       ├── __init__.py
│       ├── config.py          # Centralized configuration
│       ├── ingestion.py       # Load, chunk, embed, index
│       ├── retriever.py       # FAISS search + context formatting
│       ├── agent.py           # Strands Agent + tool + ask()
│       ├── api.py             # FastAPI app
│       └── main.py            # CLI entry point
├── data/
│   ├── sample_policies/       # Source .txt policy files
│   └── vector_store/          # Generated FAISS index + metadata (gitignored)
├── tests/
│   ├── __init__.py
│   ├── test_ingestion.py
│   ├── test_retriever.py
│   ├── test_agent.py
│   ├── test_api.py
│   └── test_main.py
├── pyproject.toml
├── CLAUDE.md
├── spec.md
├── test-cases.md
└── .gitignore
```

### 4.2 Module Dependency Chain

```
ingestion.py → (standalone, run once)
retriever.py → config.py
agent.py     → retriever.py, config.py, strands
api.py       → agent.py, retriever.py
main.py      → ingestion.py, agent.py, api.py
```

### 4.3 Configuration Values

| Parameter | Value | Notes |
|-----------|-------|-------|
| `LLM_MODEL_ID` | `us.anthropic.claude-sonnet-4-20250514` | Claude Sonnet 4 |
| `EMBEDDING_MODEL_ID` | `amazon.titan-embed-text-v2:0` | Titan Embed v2 |
| `EMBEDDING_DIMENSION` | `1024` | Titan v2 output dim |
| `CHUNK_SIZE` | `512` | Characters per chunk |
| `CHUNK_OVERLAP` | `50` | Overlap between chunks |
| `TOP_K` | `5` | Results per search |
| `SIMILARITY_THRESHOLD` | `0.3` | Min score to include result |
| `LLM_MAX_TOKENS` | `1024` | Max response tokens |
| `LLM_TEMPERATURE` | `0.1` | Low for factual accuracy |
| `API_HOST` | `0.0.0.0` | Bind address |
| `API_PORT` | `8000` | Default port |

### 4.4 API Schema

**POST /ask**
```json
// Request
{
  "question": "What is the policy on remote work data security?"
}

// Response
{
  "answer": "According to the data security policy, remote employees must...",
  "sources": [
    {"document": "data_security_policy.txt", "section": "Remote Work", "relevance_score": 0.91}
  ],
  "escalation": false,
  "escalation_reason": null
}
```

**GET /health**
```json
{
  "status": "healthy",
  "documents_indexed": 2,
  "vector_count": 45
}
```

### 4.5 System Prompt

```
You are a Company Policy Assistant for Acme Corporation. Your job is to help employees
understand company policies by searching the policy knowledge base.

RULES:
1. ALWAYS use the retrieve_policy_context tool before answering. Never answer from your own knowledge.
2. Provide clear, specific answers grounded in the policy documents.
3. Cite your sources — mention which policy document the information comes from.
4. If the knowledge base doesn't contain enough information to answer, say so explicitly.
   Do NOT make up or infer policy details.
5. For questions that require human judgment or are outside policy scope
   (legal advice, policy change requests, personal situations), recommend
   the employee contact HR at hr@acmecorp.com.
6. Be professional and helpful.

ESCALATION TRIGGERS (recommend human help):
- Requests for legal advice or legal interpretation of policies
- Requests to change or challenge a policy
- Harassment or discrimination reports (direct to HR immediately)
- Questions about specific disciplinary actions
- Anything involving personal medical or financial information
```

## 5. Tech Stack

| Component | Technology |
|-----------|-----------|
| Agent Framework | Strands SDK (`strands-agents`) |
| LLM | Claude Sonnet 4 via Amazon Bedrock |
| Embeddings | Amazon Titan Embed Text v2 |
| Vector Store | FAISS (faiss-cpu) |
| API Framework | FastAPI + Uvicorn |
| Runtime | Amazon Bedrock AgentCore |
| Language | Python 3.11+ |
| Testing | pytest + pytest-asyncio |

## 6. Dependencies

```toml
[project]
dependencies = [
    "strands-agents>=0.1.0",
    "strands-agents-tools>=0.1.0",
    "faiss-cpu>=1.7.4",
    "fastapi>=0.115.0",
    "uvicorn>=0.34.0",
    "boto3>=1.35.0",
    "numpy>=1.26.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=8.0",
    "pytest-asyncio>=0.24.0",
    "httpx>=0.28.0",
    "pytest-cov>=6.0",
]
```

## 7. Acceptance Criteria

| ID | Criterion | Verification |
|----|-----------|-------------|
| AC-001 | Ingestion pipeline loads .txt files, chunks, embeds, and builds FAISS index | Unit tests TC-001 to TC-008 |
| AC-002 | Retriever returns top-K results with similarity scores | Unit tests TC-009, TC-010 |
| AC-003 | Agent uses retrieval tool and returns grounded answer | Unit test TC-015 |
| AC-004 | Response includes source citations with document names | Unit test TC-016 |
| AC-005 | Agent detects escalation triggers and flags appropriately | Unit test TC-017 |
| AC-006 | POST /ask returns 200 with valid structured response | Unit test TC-019 |
| AC-007 | POST /ask returns 422 for invalid/missing input | Unit tests TC-020, TC-021 |
| AC-008 | GET /health returns status with document and vector counts | Unit tests TC-023, TC-024 |
| AC-009 | CLI `ingest` command runs pipeline and reports stats | Unit test in test_main.py |
| AC-010 | CLI `serve` command starts the API server | Unit test in test_main.py |
| AC-011 | All unit tests pass with ≥80% coverage | `pytest --cov` |
| AC-012 | No hardcoded secrets — all AWS access via boto3 credential chain | Code review |
| AC-013 | SIMILARITY_THRESHOLD is applied to filter low-relevance results | Unit test TC-010 |
| AC-014 | Agent instance is reused across requests (lazy singleton) | Code review |

## 8. Out of Scope (PoC)

- Authentication / authorization
- Conversation memory / multi-turn
- UI / Slack / Teams integration
- Production observability (CloudWatch, X-Ray)
- Document upload API
- Automatic re-ingestion on doc changes

# Policy Chatbot 🏢

RAG-powered company policy Q&A chatbot for internal employees. Built with [Strands SDK](https://github.com/strands-agents/sdk-python), Amazon Bedrock (Claude Sonnet 4), and FAISS.

## What It Does

Employees ask natural language questions about company policies → the agent retrieves relevant policy sections → generates a grounded answer with source citations.

**Sample interaction:**
```
Q: "What is the policy on accepting gifts from vendors?"
A: "According to the Code of Conduct (Section 5), employees may accept gifts
   valued under $50. Gifts over $50 must be reported to your manager and the
   Ethics team within 5 business days..."
   Sources: [code_of_conduct.txt / Section 5: Gifts & Conflicts of Interest]
```

## Quick Start

```bash
# Install dependencies
pip install -e ".[dev]"

# Ingest sample policy documents into vector store
python -m policy_chatbot ingest

# Ask a question from the CLI
python -m policy_chatbot ask "What is the dress code policy?"

# Start the API server
python -m policy_chatbot serve
```

## API

```bash
# Ask a question
curl -X POST http://localhost:8000/ask \
  -H "Content-Type: application/json" \
  -d '{"question": "What data classification levels exist?"}'

# Health check
curl http://localhost:8000/health
```

## Run Tests

```bash
pytest
pytest --cov=policy_chatbot --cov-report=term-missing
```

## Project Structure

```
src/policy_chatbot/
├── config.py       # Configuration constants
├── ingestion.py    # Document loading, chunking, embedding, indexing
├── retriever.py    # FAISS vector search + context formatting
├── agent.py        # Strands Agent with retrieval tool
├── api.py          # FastAPI endpoints (POST /ask, GET /health)
└── main.py         # CLI (ingest, serve, ask)
```

## Tech Stack

- **Agent:** Strands SDK + Claude Sonnet 4 (Amazon Bedrock)
- **Embeddings:** Amazon Titan Embed Text v2
- **Vector Store:** FAISS (faiss-cpu)
- **API:** FastAPI + Uvicorn
- **Runtime:** Amazon Bedrock AgentCore

## Docs

- [spec.md](spec.md) — Full project specification
- [test-cases.md](test-cases.md) — Test case documentation
- [CLAUDE.md](CLAUDE.md) — Coding rules for Claude Code

---

*Built with [AIDLC](https://github.com/strands-agents) methodology* 🏛️

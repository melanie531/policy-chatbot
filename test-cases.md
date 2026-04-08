# Test Cases — Policy Chatbot

Maps to acceptance criteria in `spec.md` Section 7.

---

## Ingestion Pipeline (`tests/test_ingestion.py`)

### TC-001: Load TXT files from directory
- **Input:** Directory with 2 `.txt` files
- **Expected:** Returns list of 2 dicts, each with `text`, `source`, `page` keys
- **AC:** AC-001

### TC-002: Raise FileNotFoundError for missing directory
- **Input:** Non-existent directory path
- **Expected:** Raises `FileNotFoundError`
- **AC:** AC-001

### TC-003: Raise ValueError for empty directory
- **Input:** Existing directory with no `.txt` files
- **Expected:** Raises `ValueError` with "No documents" message
- **AC:** AC-001

### TC-004: Skip empty files
- **Input:** Directory with 1 empty `.txt` and 1 non-empty `.txt`
- **Expected:** Returns list of 1 document (skips the empty one)
- **AC:** AC-001

### TC-005: Chunk long document into multiple pieces
- **Input:** Document with ~1000 characters, chunk_size=100, overlap=20
- **Expected:** Returns >1 chunks, each with `text`, `source`, `page`, `chunk_id`
- **AC:** AC-001

### TC-006: Small document produces single chunk
- **Input:** Document with <chunk_size characters
- **Expected:** Returns exactly 1 chunk
- **AC:** AC-001

### TC-007: Build FAISS index from embeddings
- **Input:** Numpy array of shape (10, 1024)
- **Expected:** FAISS index with `ntotal=10`, `d=1024`
- **AC:** AC-001

### TC-008: FAISS index self-match search
- **Input:** Search for vector[0] in an index built from vectors[0..9]
- **Expected:** Top result index is 0
- **AC:** AC-001

---

## Retriever (`tests/test_retriever.py`)

### TC-009: Search returns top-K results
- **Input:** Query string, top_k=3
- **Expected:** Returns exactly 3 results, each with `text`, `source`, `score` keys
- **AC:** AC-002
- **Mock:** Bedrock embedding call

### TC-010: Results have float similarity scores and respect threshold
- **Input:** Query string
- **Expected:** All results have `float` scores >= SIMILARITY_THRESHOLD
- **AC:** AC-002, AC-013
- **Mock:** Bedrock embedding call

### TC-011: Format context includes source labels
- **Input:** List of 2 result dicts
- **Expected:** Formatted string contains `[Source 1:` and `[Source 2:` with text
- **AC:** AC-004

### TC-012: Format context handles empty results
- **Input:** Empty list
- **Expected:** Returns string containing "No relevant"
- **AC:** AC-004

### TC-013: document_count property
- **Input:** Store with 5 vectors from 5 unique sources
- **Expected:** `document_count == 5`
- **AC:** AC-008

### TC-014: vector_count property
- **Input:** Store with 5 vectors
- **Expected:** `vector_count == 5`
- **AC:** AC-008

---

## Agent (`tests/test_agent.py`)

### TC-015: Ask returns answer with sources
- **Input:** "What is the dress code policy?"
- **Expected:** Dict with non-empty `answer` and `sources` list
- **AC:** AC-003
- **Mock:** Agent class, retriever

### TC-016: Sources contain document name
- **Input:** Policy question
- **Expected:** At least one source with `document` field matching a policy filename
- **AC:** AC-004
- **Mock:** Agent class, retriever

### TC-017: Agent flags escalation for sensitive topics
- **Input:** "I want to report harassment by my manager"
- **Expected:** `escalation=True`, `escalation_reason` is non-empty string
- **AC:** AC-005
- **Mock:** Agent class, retriever

### TC-018: Response has all required fields
- **Input:** Any question
- **Expected:** Dict contains exactly `answer`, `sources`, `escalation`, `escalation_reason`
- **AC:** AC-003

---

## API (`tests/test_api.py`)

### TC-019: POST /ask returns 200 with valid question
- **Input:** `{"question": "What is the data classification policy?"}`
- **Expected:** 200, response has `answer`, `sources`, `escalation`, `escalation_reason`
- **AC:** AC-006
- **Mock:** `ask()` function

### TC-020: POST /ask returns 422 for missing question
- **Input:** `{}`
- **Expected:** 422
- **AC:** AC-007

### TC-021: POST /ask returns 422 for too-short question
- **Input:** `{"question": "Hi"}`
- **Expected:** 422
- **AC:** AC-007

### TC-022: POST /ask returns escalation response
- **Input:** Question triggering escalation
- **Expected:** 200, `escalation=true`, `escalation_reason` non-null
- **AC:** AC-005, AC-006
- **Mock:** `ask()` function returning escalation

### TC-023: GET /health returns 200 with stats
- **Input:** Healthy system
- **Expected:** `{"status": "healthy", "documents_indexed": N, "vector_count": M}`
- **AC:** AC-008
- **Mock:** retriever properties

### TC-024: GET /health returns degraded when no index
- **Input:** System with no vector store loaded
- **Expected:** `{"status": "degraded", "documents_indexed": 0, "vector_count": 0}`
- **AC:** AC-008

---

## CLI (`tests/test_main.py`)

### TC-025: CLI --help shows all commands
- **Input:** `--help` flag
- **Expected:** Output contains `ingest`, `serve`, `ask`
- **AC:** AC-009, AC-010

### TC-026: CLI ingest command runs pipeline
- **Input:** `ingest` subcommand
- **Expected:** Calls `run_ingestion()`, prints stats
- **AC:** AC-009
- **Mock:** `run_ingestion`

### TC-027: CLI ask command prints answer
- **Input:** `ask "What is the dress code?"`
- **Expected:** Calls `ask()`, prints answer and sources
- **AC:** AC-003
- **Mock:** `ask()`

### TC-028: CLI serve command starts server
- **Input:** `serve` subcommand
- **Expected:** Calls `uvicorn.run()`
- **AC:** AC-010
- **Mock:** `uvicorn.run`

# CLAUDE.md — Policy Chatbot Coding Rules

## Project Overview

RAG-powered company policy Q&A chatbot using Strands SDK, FAISS, and Claude Sonnet 4 via Amazon Bedrock. See `spec.md` for full specification.

## Architecture Rules

1. **Module structure must match spec exactly:**
   - `src/policy_chatbot/config.py` — All configuration constants
   - `src/policy_chatbot/ingestion.py` — Document loading, chunking, embedding, indexing
   - `src/policy_chatbot/retriever.py` — FAISS search and context formatting
   - `src/policy_chatbot/agent.py` — Strands Agent with retrieval tool
   - `src/policy_chatbot/api.py` — FastAPI endpoints
   - `src/policy_chatbot/main.py` — CLI entry point

2. **Dependency chain (no circular imports):**
   ```
   config.py ← retriever.py ← agent.py ← api.py ← main.py
   ingestion.py ← main.py (standalone)
   ```

3. **Single retriever instance:** The `ITRetriever` (or `PolicyRetriever`) must be initialized once and reused. Do NOT create multiple instances. Use lazy singleton pattern.

4. **Single agent instance:** The Strands `Agent` must be created once and reused across requests. Do NOT call `create_agent()` inside `ask()`. Use lazy singleton pattern.

5. **SIMILARITY_THRESHOLD must be applied:** Filter out search results below `SIMILARITY_THRESHOLD` (0.3) in the retriever's `search()` method. Do not define config values that go unused.

## Coding Standards

1. **Python 3.11+** — Use modern syntax: `str | None`, `list[dict]`, etc.
2. **Type hints on all functions** — Parameters and return types.
3. **Docstrings on all public functions** — Google style with Args/Returns/Raises.
4. **No hardcoded secrets** — All AWS access via boto3 default credential chain.
5. **Logging** — Use `logging.getLogger(__name__)` in every module. No `print()` except in CLI output.
6. **Error handling** — Raise specific exceptions (`FileNotFoundError`, `ValueError`) with descriptive messages.

## API Rules

1. **FastAPI with Pydantic models** for request/response validation.
2. **`POST /ask`** — Accepts `{"question": "..."}`, returns `{"answer", "sources", "escalation", "escalation_reason"}`.
3. **`GET /health`** — Returns `{"status", "documents_indexed", "vector_count"}`.
4. **Input validation:** Question minimum 5 characters. Return 422 for invalid input.
5. **Lifespan handler** initializes the retriever once on startup. Do NOT create duplicate instances.

## Agent Rules

1. **Model:** `us.anthropic.claude-sonnet-4-20250514` via `BedrockModel`.
2. **Tool:** `retrieve_policy_context` — agent MUST call this before answering.
3. **Response format:** Structured JSON with `answer`, `sources`, `escalation`, `escalation_reason`.
4. **JSON parsing:** Strip markdown fences (```json ... ```) before `json.loads()`. Do NOT silently lose source citations on parse failure.
5. **System prompt:** Must enforce tool use, source citation, and escalation triggers per spec.

## Testing Rules

1. **TDD enforced** — Write tests before or alongside implementation. See `test-cases.md`.
2. **All Bedrock calls must be mocked** — Use `unittest.mock.patch` on boto3 clients.
3. **All FAISS operations in tests use in-memory fixtures** — Create temp indexes with `tmp_path`.
4. **Test file naming:** `tests/test_<module>.py` matching source modules.
5. **Coverage target:** ≥80% line coverage.
6. **pytest-asyncio** for async API tests with `httpx.AsyncClient`.

## Sample Policy Documents

Generate two realistic policy documents in `data/sample_policies/`:
- `code_of_conduct.txt` — Workplace behavior, harassment, dress code, social media, gifts, reporting
- `data_security_policy.txt` — Data classification (public/internal/confidential/restricted), handling, incident response, acceptable use, remote work security, BYOD

Each document should be 150-300 lines, structured with numbered sections and clear headings. Written as if from "Acme Corporation" HR/Security team.

## What NOT To Do

- ❌ Do NOT create a UI or frontend
- ❌ Do NOT implement authentication
- ❌ Do NOT implement conversation memory or multi-turn
- ❌ Do NOT use LangChain — use Strands SDK only
- ❌ Do NOT hardcode AWS credentials or region
- ❌ Do NOT create agent/retriever instances per request
- ❌ Do NOT define config values that are never used

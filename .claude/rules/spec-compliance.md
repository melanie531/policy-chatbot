---
description: Spec compliance enforcement
globs: ["src/**/*.py", "tests/**/*.py"]
---

# Spec Compliance Rule — Policy Chatbot

## Rule
All implementation must comply with `spec.md`. Do not add features, endpoints, or modules not specified.

## Checks Before Every Commit
1. Module structure matches Section 4.1 of spec
2. Config values match Section 4.3
3. API schema matches Section 4.4
4. System prompt matches Section 4.5
5. All acceptance criteria in Section 7 are addressed

## Key Constraints from Spec
- Package name: `policy_chatbot` (underscore, not hyphen)
- Model: `us.anthropic.claude-sonnet-4-20250514`
- Embedding: `amazon.titan-embed-text-v2:0` with dimension 1024
- Vector store: FAISS `IndexFlatIP` with L2-normalized vectors
- Agent tool name: `retrieve_policy_context`
- API endpoints: `POST /ask` and `GET /health` only
- CLI commands: `ingest`, `serve`, `ask` only
- Sample docs directory: `data/sample_policies/`
- Vector store directory: `data/vector_store/` (gitignored)

## Lessons from IT Support Agent PR Review
These were identified as bugs in a similar project — do NOT repeat them:
- **Do NOT create multiple retriever instances** — lazy singleton only
- **Do NOT create agent instances per request** — lazy singleton only
- **Do NOT accept API parameters that aren't wired through** (e.g., `top_k`)
- **Do NOT define config constants that are never used** (e.g., `SIMILARITY_THRESHOLD` must be applied)
- **Do strip markdown fences before JSON parsing** agent responses

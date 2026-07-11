# IT Helpdesk Automation Agent

A RAG + LangGraph agent that doesn't just answer questions — it decides what to *do*: answer from a knowledge base, take an action (create a ticket, request access), or escalate to a human when it's not confident or the request is sensitive.

This project extends a production-style RAG pipeline (retrieval, embeddings, LLM-as-judge evaluation) with an **agentic orchestration layer**, which is the core skill gap between a "RAG chatbot" and an AI automation engineering deliverable.

## Why this project

Most RAG demos stop at "retrieve and answer." Real automation work requires the system to:
1. **Route** a query to the right handling path
2. **Act** safely, with guardrails, when action is warranted
3. **Escalate** to a human when confidence is low or the request is out of scope
4. **Be evaluated and monitored**, not just deployed and forgotten

## Architecture

```
User query
    │
    ▼
classify_query (LLM router)
    │
    ├── INFO ──────► retrieve_and_answer (RAG over FAISS index)
    │                       │
    │                       └── low similarity score ──► escalate
    │
    ├── ACTION ────► take_action (calls a mock ticketing tool)
    │
    └── ESCALATE ──► escalate (creates a human-review ticket)
```

Built with:
- **LangGraph** — stateful agent orchestration (router, conditional edges, guardrail fallback)
- **FAISS + sentence-transformers** — local, free retrieval over a small IT knowledge base
- **Gemini API** — the LLM powering routing, answering, and action-argument extraction
- **LLM-as-judge** — a second LLM call audits whether each routing decision was correct
- **pandas** — a lightweight monitoring summary (escalation rate, action rate, routing accuracy)

## Running it

Open `IT_Helpdesk_Automation_Agent.ipynb` in Google Colab. You'll need a free Gemini API key from [aistudio.google.com/apikey](https://aistudio.google.com/apikey) — the notebook prompts for it and never writes it to disk.

Runs top to bottom with no external services required — the "ticketing system" is an in-memory mock so the whole thing is self-contained and safe to demo.

## What's mocked vs. real

| Component | This demo | Production version |
|---|---|---|
| Knowledge base | 5 sample IT articles | Real ServiceNow/Confluence KB |
| Ticketing system | In-memory Python dict | ServiceNow / Jira Service Desk API |
| Embeddings | `all-MiniLM-L6-v2` (local) | `gemini-embedding-001` at scale |
| Human-in-the-loop | Logged as escalation ticket | LangGraph interrupt + real approval workflow |

## Next steps

- Add a real human-in-the-loop confirmation gate before any action executes
- Swap the mock ticketing dict for a real ticketing API
- Track judge-scored routing accuracy over time to catch drift
- Add persistent checkpointing so conversations survive restarts

---

Built by [Sanusi Isiaka Olatunji](https://www.linkedin.com/in/sanusi-olatunji-43990198) — M.Sc. Data Science, University of Leoben. Part of a portfolio exploring agentic AI automation on top of RAG systems. [GitHub](https://github.com/sanusi009)

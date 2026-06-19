# AI That Works — Lessons Breakdown

A structured breakdown of every session in the "AI That Works" series — a weekly live-coding series by [@hellovai](https://github.com/hellovai) and [@dexhorthy](https://github.com/dexhorthy).

---

## Session 1 — Large Scale Classification (2025-03-31)

**Video**: https://youtu.be/6B7MzraQMZk  
**Code**: [`2025-03-31-large-scale-classification/`](./2025-03-31-large-scale-classification)

### Problem

LLMs handle small classification tasks well (5–50 categories), but break down at 1,000+ categories due to context window limits and degraded accuracy.

### Core Technique: Embed → Retrieve → Classify

1. **Embed** each category into a vector database at setup time.
2. **Embed** the incoming input query.
3. **Retrieve** the top-k most similar categories via vector similarity search.
4. **Classify** using an LLM prompt that only sees the top-k candidates, not all 1,000+.

This two-step funnel keeps the LLM prompt small and focused.

### Key Concepts

| Concept | Description |
|---|---|
| Vector embeddings | Encode text semantics into numeric vectors for fast similarity search |
| Top-k retrieval | Narrow 1,000 candidates to ~10 before hitting the LLM |
| Post-LLM probe | A second LLM (or deterministic) step to resolve ambiguous matches |
| BAML | Structured prompt definitions compiled to typed Python/TS functions |

### Exercises

1. **Tool Selection** — Load `tools.json` (100s of MCP tools), embed them, and select the top-k tools matching a user query.
2. **Post-LLM probe** — Modify the prompt to return `Category[]`, then add a follow-up step to resolve the final single category.

---

## Session 2 — Reasoning Models vs Reasoning Prompts (2025-04-07)

**Video**: https://youtu.be/D-pcKduKdYM  
**Code**: [`2025-04-07-reasoning-models-vs-prompts/`](./2025-04-07-reasoning-models-vs-prompts)

### Problem

When should you pay for a reasoning model (o3, DeepSeek-R1) vs. engineer reasoning into your prompt?

### Core Comparison

| Approach | How it works | When to use |
|---|---|---|
| **Reasoning model** (e.g. o3) | The model reasons internally with hidden chain-of-thought tokens | Move fast; don't want to invest in prompt engineering |
| **Reasoning prompt** (guided CoT) | Explicit `<thinking>` fields or step-by-step instructions baked into the BAML prompt | Need speed/cost control, small/OSS models, or edge deployment |

### Key Takeaways

- You can make a **cheap model reason well** with a carefully structured prompt — no expensive reasoning model required.
- **Guided reasoning** (domain-specific thinking steps) beats generic `<THINK>` tokens in general-purpose models.
- Even a strong reasoning model gets **further improved** by guided prompting.
- Actor/checker/LLM-as-judge patterns work but are **exponentially expensive** — use sparingly.
- Rule of thumb: use reasoning models to prototype fast, then replace with prompt engineering for production cost/latency.

### Demo

Added structured reasoning to a movie chatbot that generates Cypher/SQL queries, comparing raw model output vs. prompted chain-of-thought.

---

## Session 3 — Code Generation with Small Models (2025-04-15)

**Video**: https://youtu.be/KJkvYdGEnAY  
**Code**: [`2025-04-15-code-generation-small-models/`](./2025-04-15-code-generation-small-models)

### Problem

Large frontier models are accurate but slow and expensive for code tasks. Can small models handle targeted code modifications?

### Architecture

```
User Request
    │
    ▼
Agent (BAML prompt + small model)
    │  ── reads relevant files ──▶ Context Window Management
    │  ── generates diff ──▶ Apply patch to codebase
    ▼
Updated Codebase
```

### Key Concepts

| Concept | Description |
|---|---|
| Ownership boundary | Agent owns targeted modifications; user owns high-level intent |
| Context management | Only load the files needed for the change — minimize token use |
| Diff-based output | Generate diffs (not full file rewrites) to reduce tokens and errors |
| Pipeline approach | Chain small models: analyze → plan → generate → apply |
| Coverage scaling | Start with a big model to establish baselines; replace hot paths with smaller models over time |

### Project Structure

- **`/project`** — A simple Python calculator app (the "codebase" the agent modifies)
- **`/agent`** — BAML agent that reads the calculator, takes instructions, and generates targeted code changes

### Key Insight

Serve the majority of users with a small fast model; escalate to a large model only when confidence is low. Over time, distill large-model outputs into fine-tuned small models to expand coverage.

---

## Session 4 — Twelve Factor Agents (2025-04-22)

**Video**: https://youtu.be/yxJDyQ8v6P0  
**Code**: [`2025-04-22-twelve-factor-agents/`](./2025-04-22-twelve-factor-agents)  
**Deep dive**: https://hlyr.dev/12fa

### Problem

AI agents work great in demos but break in production. The "Twelve Factor Agent" methodology gives a principled framework for building agents that are reliable, observable, and maintainable.

### The Twelve Factors (Applied to Agents)

| # | Factor | Agent Application |
|---|---|---|
| 1 | **Codebase** | One agent, one repo. Prompts are code. |
| 2 | **Dependencies** | Lock model versions; pin BAML clients explicitly. |
| 3 | **Config** | API keys and model names come from environment, not hardcode. |
| 4 | **Backing services** | Tools, APIs, and DBs are attached resources — swap without code changes. |
| 5 | **Build/Release/Run** | Separate BAML generation (build) from agent execution (run). |
| 6 | **Processes** | Agents are stateless; state lives in external stores (DB, memory). |
| 7 | **Port binding** | Expose agent as a service (HTTP/CLI) — not embedded in a larger app. |
| 8 | **Concurrency** | Scale by running multiple stateless agent processes. |
| 9 | **Disposability** | Fast startup/shutdown; resume from checkpointed state. |
| 10 | **Dev/prod parity** | Same prompts and models in dev as in prod — no "lite" mode surprises. |
| 11 | **Logs** | Structured logs for every LLM call: input, output, latency, cost. |
| 12 | **Admin processes** | Evals and test runs are one-off processes, not baked into the agent loop. |

### Code Structure

```
final/                  — complete working agent
step-by-step/           — incremental walkthrough (00 → 10)
  walkthrough/
    00-index.ts         — entry point skeleton
    01-agent.baml       — first BAML prompt
    01-agent.ts         — first agent loop
    ...
    10-server.ts        — expose as HTTP service
```

### Build It Yourself

Follow `step-by-step/walkthrough.md` to go from zero to a fully working twelve-factor agent step by step.

---

## Session 5 — NYC Workshop: Building 12 Factor Agents (2025-05-10)

**Code**: [`2025-05-10-workshop-nyc-twelve-factor-agents/`](./2025-05-10-workshop-nyc-twelve-factor-agents)

An in-person, full-day hands-on workshop in New York City expanding on Session 4.

### Agenda

| Time | Activity |
|---|---|
| 9:30–10:30 AM | Setup — clone repo, configure API keys, network |
| 10:30 AM–12:00 PM | Morning session — live code-along building a 12-factor agent from scratch |
| 12:00–1:00 PM | Lunch + YC founder panel (companies using AI to reach $500k+ ARR) |
| 1:00–2:30 PM | Afternoon session — advanced prompting techniques |
| 2:30–3:00 PM | Break |
| 3:00–6:00 PM | Hackathon — extend starter project with advanced capabilities |

### Workshop Content

- **`/pre-requisites`** — Setup guide and environment checklist
- **`/workshop-agents`** — Main agent-building content
- **`/workshop-bonus`** — Bonus: large-scale classification (extends Session 1)

### Additional Resources

- [12 Factor Agents reference](https://hlyr.dev/12fa)
- [Advanced Prompt Engineering (Dec 2024)](https://gloochat.notion.site/BAML-Advanced-Prompting-Workshop-Dec-2024-161bb2d26216807b892fed7d9d978a37)

---

## Season 2 — Coming Up

| Date | Topic |
|---|---|
| 2025-05-13 | **You're Doing Evals Wrong** — minimalist, high-performance testing and evals for LLM applications |

---

## Quick Reference: Tools & Stack

| Tool | Purpose |
|---|---|
| [BAML](https://github.com/boundaryml/baml) | Structured prompt definitions compiled to typed Python/TypeScript |
| [UV](https://docs.astral.sh/uv/) | Python package manager (used in Python sessions) |
| PNPM | Node package manager (used in TypeScript sessions) |
| Cursor | Recommended IDE for live sessions |
| OpenAI / Anthropic | Cloud model providers used in examples |

# InsightAI — Consumer Research to Action Platform

> Research like the big brands. Without the big budget.

---

## The Problem

Small business brand managers make critical marketing and positioning decisions weekly — which campaign to run, how to position against competitors, whether to launch a new product. Enterprise research tools like Zappi and Kantar cost $10K–$100K+ per year. Agencies take weeks and charge thousands per project. So SMBs make these decisions on gut feel. And gut feel is expensive when you get it wrong.

---

## What InsightAI Does

InsightAI takes a natural language research question and returns a structured insight card in under 30 seconds:

- **Insight** — 2-3 sentence summary grounded in real, current web sources
- **Confidence Level** — high, medium, or low based on grounding quality
- **Recommended Action** — one specific thing you can execute today
- **Sources** — real clickable URLs, not hallucinated references

---

## Why It's Different from Just Asking ChatGPT

1. **Live grounding** — searches the web in real time, not stale training data
2. **LLM-as-Judge** — a second LLM verifies every insight against its sources before you see it. Hallucinations get caught and flagged
3. **Insights to action** — you don't get a report, you get a decision

---

## Architecture

Built as a fully agentic pipeline in n8n, deployed on Railway:

```
User Question (Vercel UI)
    ↓
Webhook Trigger
    ↓
Extract Question
    ↓
Azure text-embedding-3-small → Generate Embedding
    ↓
Semantic Search (pgvector in Supabase) — finds similar past highly-rated insights
    ↓
Build Prompt — dynamic category prompt + few-shot examples from rated insights
    ↓
Classify Question (GPT-4.1-mini) — 5 categories: brand_sentiment, competitive_intelligence, trend_analysis, market_sizing, general
    ↓
Fetch Rated Insights (Supabase) — dynamic few-shot learning
    ↓
Web Search (SerpAPI) — live grounding from real sources
    ↓
Extract Sources
    ↓
Synthesize Insight (GPT-4.1-mini) — category-specific analyst prompt
    ↓
Parse Output
    ↓
LLM Judge (OpenAI) — grounding verification + bias detection
    ↓
Final Output — confidence scoring (high/medium/low)
    ↓
Azure Content Safety — Responsible AI filtering
    ↓
Parse Safety Result
    ↓
Prepare Log Data
    ↓
Log to Supabase — full eval logging
    ↓
Respond to Webhook → Vercel UI shows result
```

**Separate feedback flow:**

```
User clicks thumbs up/down
    ↓
Feedback Webhook → Update Supabase row (helpful, outcome_note)
```

---

## Key Design Decisions & Tradeoffs

**GPT-4.1-mini for synthesis, stronger model for Judge**

Cheap fast model for generation, stronger model for evaluation. Catching hallucinations matters more than generation speed. This keeps cost low while maintaining quality at the eval layer.

**Dynamic few-shot learning**

Every question a user rates as helpful gets stored with its embedding. When a similar question comes in, the top-rated past insights for that category become few-shot examples in the synthesis prompt. Every thumbs up literally makes the product smarter.

**Azure embeddings + pgvector**

Used Azure text-embedding-3-small with pgvector extension in Supabase for semantic similarity search. This gives the system memory — it finds semantically similar past questions, not just keyword matches.

**Responsible AI layer**

Azure Content Safety screens every output before it reaches the user. The LLM Judge also checks for geographic, demographic, and source bias. Both results are logged in Supabase for auditability.

**RLHF-style feedback loop**

Users rate insights as helpful or not and optionally note what they did with the insight. This data feeds back into the few-shot learning layer, creating a continuous improvement cycle grounded in real user outcomes.

---

## Eval Framework

Every insight is logged to Supabase with:

| Field | Description |
|-------|-------------|
| `grounded` | Did the LLM Judge verify grounding? |
| `confidence` | high / medium / low |
| `bias_flagged` | Was bias detected? |
| `safety_flagged` | Did Content Safety flag anything? |
| `helpful` | User rating |
| `outcome_note` | What the user did with the insight |
| `embedding` | 1536-dim vector for semantic search |

**KPI Views built in Supabase:**

- Helpfulness rate by category
- Confidence calibration (does high confidence predict helpful ratings?)
- Grounding failure rate
- Safety flag rate

---

## What I Would Do Differently

At scale I would replace SerpAPI with a more reliable enterprise search API, add caching for common question patterns to reduce latency and cost, and implement fine-tuning once enough rated examples accumulate per category (target: 500+ per category). I would also add a streaming response so users see the insight building in real time rather than waiting for the full card.

---

## Stack

| Layer | Tool |
|-------|------|
| Orchestration | n8n (self-hosted on Railway) |
| LLM — Synthesis | OpenAI GPT-4.1-mini |
| LLM — Judge | OpenAI GPT-4.1 |
| Embeddings | Azure AI text-embedding-3-small |
| Responsible AI | Azure Content Safety |
| Web Search | SerpAPI |
| Database | Supabase (PostgreSQL + pgvector) |
| UI | v0 + Vercel |
| Hosting | Railway (n8n), Vercel (UI) |

---

## Build Log

- **Day 1**: Question classification (5 categories), SerpAPI web search grounding, GPT-4.1-mini synthesis pipeline working end to end
- **Day 2**: LLM Judge (OpenAI), confidence scoring, Supabase RLHF logging, full pipeline working end to end
- **Day 3**: Azure text-embedding-3-small integrated, pgvector semantic search, dynamic few-shot learning, Azure Content Safety, bias detection in LLM Judge, feedback webhook, eval KPI views — full Responsible AI pipeline complete
- **Day 4**: Railway deployment, Vercel UI wired to live webhook, end to end demo recorded

# Anas Altaf

**Backend & full-stack engineer — TypeScript · NestJS · Next.js · Python.**
I build the unglamorous infrastructure that makes unreliable AI safe to ship.

Two years building production systems for US and European teams, while finishing a BSc in Software Engineering at FAST-NUCES. Most of my work lives in private client repos, so the design notes below are how I show the reasoning instead.

📍 Faisalabad, Pakistan (UTC+5) · open to remote · [anasaltaf.dev](https://anasaltaf.dev) · [LinkedIn](https://linkedin.com/in/anasaltaf) · anasaltaf35@gmail.com

---

## What I actually work on

### 🧠 Grounded LLM systems
Agentic RAG over a vetted corpus — a bounded LangGraph state machine, native vector search with hard pre-filters, and a validation step that refuses to answer rather than fabricate. Money figures never come from the model.

→ **[Grounded retrieval you can put in front of customers](notes/grounded-rag.md)**

### ⚙️ AI generation pipelines
Three generation paths orchestrating five external AI APIs with different latency profiles and failure modes, self-scheduling agents that publish without a human in the loop, and the concurrency control that stops cron and user triggers from double-firing.

→ **[Orchestrating unreliable AI APIs](notes/ai-generation-pipelines.md)**

### 📦 Delivery you can trust
Multi-tenant backends where "sent twice" costs a customer — provider abstractions held to shared contract tests, fail-closed access control, and a reserve-before-send ledger for exactly-once outbound.

→ **[Exactly-once outbound delivery](notes/exactly-once-delivery.md)**

---

## Stack

| | |
|---|---|
| **Languages** | TypeScript · Python · Dart · SQL |
| **Backend** | NestJS · Fastify · Express · FastAPI · REST · SSE · WebSockets · Zod / Pydantic |
| **Frontend** | Next.js · React · React Native · Flutter |
| **Data** | MongoDB · PostgreSQL · Firestore · Redis · Firestore vector search |
| **AI / LLM** | OpenAI · Gemini · Vertex AI · LangGraph · OpenRouter · structured outputs · embeddings & retrieval |
| **Cloud** | AWS (EC2, S3, ECS, Lambda, RDS, IAM) · Docker · GitHub Actions · Vercel · Cloud Run |

---

## Selected work

| Project | What it is |
|---|---|
| **AdShort.ai** | AI video-ad platform — sole engineer, architecture through production. [adshort.ai](https://adshort.ai) |
| **Decision-support platform** | Grounded RAG engine over an approved corpus, with citation enforcement and escalation. |
| **COVR** | Multi-tenant hospitality operations — WhatsApp ordering channel, exactly-once delivery. |
| **AslasChat** | Multi-tenant SaaS for document-grounded chatbots. [repo](https://github.com/Anas-Altaf/Aslase-Chat) |
| **MoneyMouthy** | Flutter app shipped to Google Play and the App Store. |

---

*Open to backend / full-stack roles — remote, Pakistan, or Gulf. Email or LinkedIn; I reply the same day.*

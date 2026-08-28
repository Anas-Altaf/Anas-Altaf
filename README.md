# Anas Altaf

**Full-stack software engineer — backend & AI.** TypeScript · NestJS · Next.js · Python.
I build the unglamorous infrastructure that makes unreliable AI safe to ship.

Two years building production systems for US and European teams, while finishing a BSc in Software Engineering at FAST-NUCES. Most of my work lives in private client repos, so the design notes below are how I show the reasoning instead — and [Atrium](https://github.com/Anas-Altaf/atrium-studio-booking) is public, deployed, and runnable end to end if you'd rather just read the code.

📍 Pakistan (UTC+5) · open to remote · [LinkedIn](https://linkedin.com/in/anasaltaf) · anasaltaf35@gmail.com

---

## What I actually work on

### 🔒 Correctness the database enforces
A booking platform where rooms may never double-book and equipment may never exceed stock. Both are `EXCLUDE USING gist` constraints over `tstzrange` — enforced in Postgres, not in application memory, so replica count changes throughput and never correctness. A 200-request concurrency proof runs as a CI gate, so a change that double-books cannot reach `main` green.

→ **[Atrium — source, load tests and decision log](https://github.com/Anas-Altaf/atrium-studio-booking)** · [live](https://atrium-studio-booking.vercel.app)

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
| **Data** | PostgreSQL (exclusion constraints, GiST, `SKIP LOCKED`) · MongoDB · Firestore · Redis · vector search |
| **AI / LLM** | OpenAI · Gemini · Vertex AI · LangGraph · OpenRouter · structured outputs · embeddings & retrieval |
| **Cloud** | AWS (EC2, S3, ECS, Lambda, RDS, IAM) · Docker · GitHub Actions · Vercel · Render · Cloud Run |

---

## Selected work

| Project | What it is |
|---|---|
| **Atrium** | Studio booking platform — Postgres-enforced concurrency, 3 replicas behind nginx, k6 load tests over 250k bookings. [source](https://github.com/Anas-Altaf/atrium-studio-booking) · [live](https://atrium-studio-booking.vercel.app) |
| **AdShort.ai** | AI video-ad platform — sole engineer, architecture through production. [adshort.ai](https://adshort.ai) |
| **Decision-support platform** | Grounded RAG engine over an approved corpus, with citation enforcement and escalation. |
| **COVR** | Multi-tenant hospitality operations — WhatsApp ordering channel, exactly-once delivery. |
| **AslasChat** | Multi-tenant SaaS for document-grounded chatbots. [repo](https://github.com/Anas-Altaf/Aslase-Chat) |
| **MoneyMouthy** | Flutter app shipped to Google Play and the App Store. |

---

## Writing

I write up things that cost me a day to figure out, on [Medium](https://anasaltaf.medium.com).

- [I connected GitHub to Claude and it still couldn't see my private repos](https://anasaltaf.medium.com/i-connected-github-to-claude-and-it-still-couldnt-see-my-private-repos-and-tools-776ef98a026d)
- [From physical servers to Kubernetes](https://anasaltaf.medium.com/from-physical-servers-to-kubernetes-the-evolution-of-modern-infrastructure-db6199e3318e)
- [When and why to actually use serverless vs serverful](https://anasaltaf.medium.com/when-and-why-actually-use-server-full-less-how-both-shine-8cdb5912f023)

---

*Open to full-stack and backend roles — remote, Pakistan, or Gulf. Email or LinkedIn; I reply the same day.*

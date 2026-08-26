# Grounded retrieval you can put in front of customers

*Design notes from a production decision-support platform. Client details omitted; the architecture is mine.*

---

## The problem

The product gives people guidance on relocation, business setup, and residency — decisions where being confidently wrong is worse than saying "I don't know." A hallucinated visa requirement or a fabricated government fee is not an embarrassing demo moment. It is someone's money and immigration status.

So the constraint was not "make the chatbot smart." It was: **the system must be structurally incapable of inventing a fact.**

That rules out the standard approach — retrieve some chunks, stuff them in a prompt, hope the model behaves. Hope is not an architecture.

## The shape of the answer

Three services, one-way layering:

```
Next.js frontend ──REST + Firebase ID token──▶ NestJS gateway ──service JWT──▶ Python RAG engine
                                               owns auth, money,              internal-only
                                               sessions, all writes           stateless, READ-ONLY
```

The engine is deliberately the weakest service in the system. It cannot write. It cannot be reached from the internet. It holds no session state. Every request arrives with a JWT the gateway signed, and the gateway re-validates the engine's output before any of it reaches a user.

The reasoning: an LLM-driven service is the component most likely to behave unpredictably, so it gets the fewest privileges. Everything that matters — auth, gating, leads, billing, writes — lives in the service that does not talk to a model.

## Bounded agency, not an agent loop

The engine runs a LangGraph `StateGraph`:

```
assess → prepare → retrieve ⇄ refine → compose → validate
                      ↑______________|
         (escalate reachable from any node)
```

The `retrieve ⇄ refine` loop is where an agent would normally run away with your budget. It is bounded by `MAX_HOPS` with an early exit — if retrieval is good enough, it stops; if it is still bad at the cap, it does not keep spending, it escalates to a human.

`escalate` being reachable from *every* node is the important part. There is always a legal exit that isn't "produce an answer." Most RAG systems only have one terminal state — an answer — which means every failure mode eventually resolves into a confident guess.

## Retrieval

Native Firestore `find_nearest`, DOT_PRODUCT, over a composite vector index, with the filters applied *before* the nearest-neighbour search:

```python
scoped = (
    db.collection(corpus)
      .where(filter=FieldFilter("approved", "==", True))
      .where(filter=FieldFilter("country", "==", country))
)
vector_query = scoped.find_nearest(
    vector_field="embedding",
    query_vector=Vector(query_vector),
    distance_measure=DistanceMeasure.DOT_PRODUCT,
    limit=k,
    distance_result_field="vector_distance",
)
```

`approved == True` is a pre-filter, not a post-filter. Unreviewed content is not "ranked lower" — it is not in the candidate set at all. Editorial approval is enforced by the query, not by a prompt instruction the model may or may not respect.

**Embeddings are unit-normalized** so DOT_PRODUCT is equivalent to cosine similarity — Google's own recommendation for Firestore vector search, and cheaper than computing cosine.

**The embedding model is pinned and recorded per chunk.** The query is always embedded with the same model that produced the stored vectors. Mixing embedding models across a corpus silently degrades retrieval in a way that is very hard to debug later — the system still returns results, they are just quietly wrong. Output dimensions are forced to 1536 regardless of provider so the corpus index stays valid when the provider swaps. Changing the model means a full corpus re-embed, and that is documented as a deliberate cost, not discovered as a surprise.

### Journey isolation, and an honest compromise

When a user is assigned a specific journey, retrieval returns **only that journey's corpus**. A missing answer surfaces as an explicit gap note rather than a confident answer borrowed from a different journey — cross-journey bleed is the failure mode that produces plausible, well-cited, wrong guidance.

The current implementation over-fetches (`k * 4`) and narrows post-fetch, because the journey-scoped composite vector index needs owner-level rights to create. This is written down in the code as a known compromise with its migration path — swap to a `journeySlug ==` pre-filter once the index exists. It is a worse implementation of the correct behaviour, chosen knowingly, and marked so the next person doesn't mistake it for the intended design.

### Seams

Retrieval sits behind an `IRetriever` protocol, with a LangChain `BaseRetriever` adapter over it and a no-op reranker in place. Nothing clever is happening in either yet — that is the point. When a managed ANN database or a multi-query retriever is worth adding, it drops in without the engine knowing. The seam costs almost nothing now and buys the option later.

## The rule that makes it trustworthy

**`facts ⊆ sources`, validated after composition — and re-validated by the gateway.**

Every factual claim in a response must trace to a retrieved, approved chunk. If it doesn't, the response does not ship. The check runs twice, in two different services, because a guard that lives only inside the component it is guarding is not much of a guard.

And the rule I'd defend hardest: **costs and figures never come from the model.** They are read from structured journey data. The LLM's job is to present numbers, never to produce them. Language models are good at prose and bad at arithmetic they were not given, and the failure is invisible — a wrong number reads exactly like a right one.

High-risk queries route to escalation instead of an answer.

## Ingestion

The corpus is built, not scraped ad hoc: sitemap discovery over whitelisted sources (robots.txt, common paths, recursive sitemapindex expansion) → a polite crawler with a concurrency cap and delay → readability-style extraction → markdown → chunking → embedding.

Deduplication runs off a URL registry keyed by a hash of the URL, with states `NEW / CRAWLED / PUBLISHED / STALE / FAILED`. A newer `lastmod` on already-crawled content marks it `STALE`; an identical content hash on re-crawl is skipped entirely. Without this, every re-crawl silently duplicates chunks, and duplicate chunks quietly wreck retrieval quality by crowding out diverse results.

Everything lands in the corpus unapproved. A human approves it. Only then can retrieval see it.

## Evaluation

Unit tests are fully mocked and run offline. Retrieval quality is tested separately by golden evals against the live index and real embeddings, because mocked retrieval tests prove your plumbing works and tell you nothing about whether the system finds the right chunk.

---

## What I'd tell you in an interview

The hard part of RAG in production is not the retrieval. It's the refusals — building a system that has somewhere to go when it doesn't know, so that "I don't know" is a first-class outcome instead of the gap that a confident hallucination rushes in to fill.

Almost everything above is about narrowing what the model is *allowed* to do: it can't write, it can't see unapproved content, it can't produce a number, it can't loop forever, and it can't emit a fact it didn't cite. What's left is the thing it's genuinely excellent at — turning structured, verified material into clear language.

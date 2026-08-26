# Orchestrating unreliable AI APIs

*Design notes from [AdShort.ai](https://adshort.ai) — an AI video-ad platform I built and shipped as the sole engineer.*

---

## The problem

Turn a product and a prompt into a finished, captioned, published video ad — and do it without a human touching any stage.

That sentence hides the actual difficulty. Video generation APIs are slow (minutes, not milliseconds), expensive per call, fail at meaningfully high rates, have wildly different input contracts, and fail in ways that are not always errors — sometimes you get a successful response containing an unusable result.

A request/response web app assumes the opposite of all of that. So most of the engineering here is not "calling the AI." It's absorbing what happens when the AI misbehaves, without the user seeing it and without paying twice.

## Three pipelines, one orchestrator

Three generation paths — Viral, Showcase, UGC — each orchestrating five or more external APIs. They differ in input format, stage count, latency profile, and failure handling, so they are separate pipelines rather than one configurable pipeline with a pile of branches. Sharing the code would have coupled three things that change for three different reasons.

Stages run with **fire-and-forget parallelism**: video generation and agent creation start together and resolve at video completion rather than blocking each other. The user-visible effect is that the slow path and the setup path overlap instead of queueing.

## The concurrency bug that pays for the whole design

Ads are produced on a schedule *and* on demand. That means two independent triggers — a serverless cron and a user click — can target the same job at the same time.

The naive version has an obvious race: both read the job as pending, both start work, and you have paid twice for one ad, published twice to nine social platforms.

The fix is an **atomic MongoDB job claim** — the update that transitions a job to `running` is also the read that decides whether to run it, conditioned on the job still being claimable, plus a mutex flag on the agent. Whoever wins the atomic update proceeds; the loser does nothing at all. There is no window between checking and acting, because checking and acting are the same operation.

This is the thing I'd want to be asked about. It is a small amount of code and it is the difference between a demo and a product, because every failure it prevents is one a customer would have paid for.

## Structured output, not vibes-parsing

Every generation stage returns a **JSON schema-constrained response**, validated before the next stage consumes it. Multimodal stages combine image and text inputs into a single structured call.

Parsing prose out of a model's free-text response works right up until the model has a slightly different opinion about formatting, at which point it fails silently and downstream. Schema-constrained outputs move that failure to the boundary where you can see it and retry it.

## Identity-lock prompt injection

Multi-segment video has a specific and very visible failure: the person or product drifts between segments. Segment one has one face; segment three has a different face wearing the same shirt. Nobody watches past that.

The fix is an identity-lock construct injected into every segment's generation prompt, carrying the invariant visual attributes forward so each segment is generated against the same locked description rather than against the previous segment's output. Cheap, and it's the difference between a video and a slideshow of strangers.

## Publishing without a human

The full path: S3 → media registration with the distribution provider → caption generation → scheduled delivery to nine platforms. Per-agent generation history is retained so every published ad can be traced back to the run and the inputs that produced it.

Production auditability is not a nice-to-have on autonomous systems. When something wrong gets published automatically, the only useful question is "what exactly produced this?" — and you can only answer it if you decided to keep that information *before* it happened.

## Money

Stripe subscriptions with **credit-based metering**. Generation costs real money per call, so credits are checked before work starts, not after it completes. A generous system that runs the job and then discovers the user couldn't pay for it is a system that loses money on exactly its heaviest users.

---

## What I'd tell you in an interview

Working with AI APIs in production is mostly ordinary distributed systems work wearing a new hat: idempotency, atomic state transitions, timeouts, retries, schema validation at boundaries, and cost control. The model is the easy part — it is one HTTP call.

The measurable outcome was cutting ad production from hours to minutes. The engineering that mattered was making sure that when a stage failed at 3am, nothing double-published, nothing double-charged, and the next run could pick up cleanly.

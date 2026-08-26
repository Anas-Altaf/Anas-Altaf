# Exactly-once outbound delivery

*Design notes from COVR — a multi-tenant hospitality operations platform. NestJS · Fastify · TypeScript · Firestore.*

---

## The problem

Customers order through WhatsApp. The system sends back an order confirmation.

Send it zero times and the customer doesn't know their food is coming. Send it twice and you have told someone their order was placed twice — which, in hospitality, generates a phone call, a refund conversation, and a lost customer. "At least once" is not good enough when the message *is* the receipt.

## Not being married to a vendor

The WhatsApp channel runs against two interchangeable providers — Meta's Cloud API and 360dialog — **behind a single interface, held to a shared contract test suite**.

Both implementations run against the same tests. If a provider's behaviour drifts from the contract, a test fails rather than a customer noticing. Swapping providers is a configuration change, not a rewrite.

This is worth doing specifically because messaging providers are where you get held hostage: pricing changes, regional availability changes, approval policies change, and all of it is outside your control. The contract test suite is what turns "we should migrate someday" into an afternoon.

Multi-tenant webhook routing sits in front — one inbound endpoint resolving to the right tenant, which is a fine place to leak data between businesses if you are careless about it.

## The reserve-before-send ledger

The naive send is: do the work, call the provider, record that you sent it. The failure is obvious once you look for it — the process dies after the provider call and before the record is written. On retry, you send again. The customer gets two confirmations. Nothing in the system is aware anything went wrong.

The fix inverts the order:

1. **Reserve** — atomically claim the right to send this specific message, keyed on the thing that makes it unique. If the reservation already exists, stop. Someone else has this.
2. **Send** — call the provider.
3. **Settle** — record the outcome against the reservation.

The reservation is written *before* the side effect, so a crash anywhere after step 1 results in no send on retry rather than a second send. You trade a small risk of an un-sent message — visible, detectable, recoverable — for eliminating duplicate sends, which are invisible to you and very visible to the customer.

That trade is the whole decision. Both failure modes are real; one is recoverable and one is a refund conversation.

## Access control that fails closed

Firebase auth, with capability guards across four dimensions: **tenant, role, location, and capability**. Guards are **fail-closed** — absence of an explicit grant is a denial, not a default-allow.

On a multi-tenant system this is the difference between a bug and an incident. A fail-open guard with a missing config doesn't throw an error; it quietly serves one restaurant's orders to another. The failure is silent, and you find out from a customer.

## Validation at every boundary

Zod schemas at every HTTP boundary **and every LLM boundary**.

The HTTP half is standard. The LLM half is the one people skip: model output is untrusted input. It is shaped by a prompt, not by a contract, and it changes when the model is updated underneath you — without a deploy, without a version bump, without anyone telling you. Treating it as structured data because it usually looks structured is how you get a production incident on a Tuesday you didn't ship anything.

---

## What I'd tell you in an interview

Every interesting decision here came from asking "what does the customer see when this breaks?" rather than "what happens if this breaks?"

Duplicate confirmations, cross-tenant leaks, and silently malformed model output share a property: the system does not experience them as errors. Nothing throws. Nothing alerts. You find out from the person you were serving. Those are the failures worth designing against up front, because they are the ones monitoring will never hand you.

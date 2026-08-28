---
weight: 9050
title: "CQRS and Read Models"
description: "Separating reads from writes is a five-rung ladder. Most teams need rung two and start at rung five."
icon: "alt_route"
date: "2026-08-28"
lastmod: "2026-08-28"
draft: false
---

## The situation

One set of tables serves a transactional write path that needs narrow, fast,
correct updates, and a reporting screen with twelve joins that needs wide,
denormalised reads. Neither can be optimised without hurting the other. Index
the write table for the dashboard and writes slow down; normalise for writes
and the dashboard times out.

CQRS is the observation that these are two different models of the same data
and can be allowed to differ.

## What CQRS is not

It is not event sourcing. The two travel together in conference talks and are
independent in practice: you can separate read and write models over a single
boring relational database, and you can event-source without any read/write
split at all. Almost all the value in CQRS is available without touching your
storage engine, and almost all the cost people associate with CQRS comes from
the event sourcing they added at the same time. For when event sourcing itself
pays, see [Event Sourcing and CDC](/docs/data-and-state/events-and-cdc/).

## The ladder

Escalate one rung at a time, and only when the current rung is demonstrably the
constraint. Each rung is more capable and strictly more expensive to operate.

| Rung | What it is | Consistency | Cost |
|---|---|---|---|
| 1 | Read replica for heavy queries | Replica lag (ms) | Nearly free |
| 2 | Separate query code paths and DTOs against the same schema — queries bypass the domain model and hit SQL directly | Fully consistent | A directory and a rule |
| 3 | Denormalised read tables, updated in the same transaction as the write | Fully consistent | Write amplification |
| 4 | Read stores projected asynchronously (search index, cache, OLAP) | Eventually consistent | A projection pipeline to operate |
| 5 | Event-sourced write model, all reads are projections | Eventually consistent | A different system |

**Rung 2 is the one most teams need and the one they skip.** It is nothing
more than: `OrderQueryService` runs a hand-written SQL query returning exactly
the shape the screen needs, and does not go through the repository, the domain
entity, or the ORM's object graph. No new infrastructure, no consistency
change, and it removes most of the pressure that was deforming the write model.

The reason it gets skipped is aesthetic — it feels like cheating to have two
ways to reach the database. It is not cheating. Reads and writes have different
correctness requirements: a write must uphold invariants, a read must produce a
shape. Forcing both through one model is the actual mistake.

Rung 4 is where **eventual consistency enters the product**, and that is the
rung to cross deliberately rather than accidentally.

## Read-your-own-writes, the problem you will actually hit

At rung 4 and above: the user edits a record, the UI redirects to the list, and
the list is served by a projection that has not caught up. The item is missing
or stale. This is not an edge case; it is the single most common CQRS bug
report, and it is a design question, not a bug to be fixed later.

The workable answers, in order of preference:

1. **Return the result of the write.** The write path knows the new state.
   Render from it instead of re-reading. Solves most cases and costs nothing.
2. **Optimistic UI with a version token.** The client holds the version it
   wrote and the read path either waits for a projection at least that fresh or
   tells the client it is behind.
3. **Route the user's own reads to the write model** for a short window after a
   write. Effective, and it does mean two code paths for one screen.
4. **Show the lag.** "Updating…" is honest, cheap, and better than a
   confusing wrong answer. Underrated.

What does not work: hoping the projection is fast enough. It is, until a
backlog forms during an incident, which is precisely when users are looking.

## When the default is wrong

- **Reads and writes want the same shape.** Most CRUD. A single model is
  correct and a split is pure overhead.
- **Low query volume.** If a replica or an index solves it, that is the answer.
  Rung 1 is not a lesser architecture, it is the right one.
- **The team cannot operate a second store.** A projection pipeline is a
  production system with lag, failure, and rebuild semantics. If nobody is
  on-call for it, do not build it.
- **Strong consistency is a hard requirement on the read path.** Regulatory
  reporting, financial balances shown to customers. Stay at rung 3 or below.

## What it costs

**Two models to keep in sync, forever.** Every new field must be added in the
write model and in every projection that surfaces it. Teams reliably forget the
third projection.

**Projection lag becomes a product feature.** Someone must decide what the UI
does at 50ms of lag and what it does at 50 seconds, and that decision has to be
made again for each screen.

**Rebuilds are an operational procedure, not a button.** Projections drift, get
corrupted, or need a schema change. "Replay from the source" is only real if
the source retains enough history and the rebuild finishes in an acceptable
window — test this before you need it, on production-sized data.

**"Which store is the truth" becomes a 3am question.** The write model is the
answer. Make sure it is written down somewhere that an on-call engineer will
find, because at rung 4 the read store is what everyone is actually looking at.

## See also

- [Event Sourcing and CDC: When It Pays](/docs/data-and-state/events-and-cdc/)
- [Consistency Models You Actually Need](/docs/data-and-state/consistency-models/)
- [Caching: The Four Questions](/docs/data-and-state/caching/)
- [Event-Driven: When the Broker Is the Backbone](../event-driven/)

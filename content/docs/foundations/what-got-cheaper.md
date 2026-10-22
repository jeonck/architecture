---
weight: 1060
title: "What Got Cheaper, and What Did Not"
description: "\"Build it when you need it\" is still right. But the price list it assumed has changed, and the ratio between the prices is what should change your behaviour."
icon: "balance"
date: "2026-10-22"
lastmod: "2026-10-22"
draft: false
---

## The situation

The oldest useful rule in this trade is *do not build it before you need it*.
It is still correct and this page is not an argument against it.

But the rule is an economic claim, not a moral one. It says: the expected cost
of building a thing now exceeds the expected cost of building it later. That
calculation rests on a price list, and over the last couple of years several
items on that list have moved — some down a long way, some not at all.

When the prices move and the rule does not, you end up applying it in the
wrong places. So it is worth writing down which is which.

## What got cheaper

**Changing code.** This is the big one. Writing a module, renaming a concept
across a codebase, adding the test you skipped, porting a service to a
different library — all of these used to be measured in engineer-days and are
now frequently measured in hours. Whatever you think of how that happened, the
practical consequence is that *internal* rework got cheap.

**Scaling machinery.** Auto-scaling, rolling deploys, health checks, load
balancing and zero-downtime rollout used to be things a team built. Now they
are defaults on whatever platform you are standing on. The old question —
"should we build the scaling machinery yet?" — has mostly been replaced by a
better one: **are we accidentally stepping outside what the platform already
gives us for free?**

**Standard components.** Queues, caches, object stores, identity, search. The
build-versus-buy line has moved so far toward buy that building one is now the
decision that needs defending.

## What did not get cheaper

**Boundaries.** Where one service's responsibility ends and another's begins.
Which component owns a piece of data. What is in the contract between two
teams. None of this got cheaper, because the cost was never in the typing — it
was in the coordination, the migration of live data, and the number of other
people who have to agree. See
[Irreversible Decisions](../irreversible-decisions/) and
[Service Boundaries That Survive Reorgs](/docs/system-design/service-boundaries/).

**Anything that reaches production without passing through review.** Config
values, policy data, generated feature files, DNS records. These propagate in
seconds to everywhere, and the tooling around them is a decade behind the
tooling around code. See
[Environments, Config, and Secrets](/docs/platform/environments-and-secrets/).

**Dependency on a central thing.** If a component answers "what is true" for
everything else, its failure is not a component failure. See
[Static Stability](/docs/reliability/static-stability/).

## The ratio is the point

Nothing above tells you to design more up front. What it tells you is that the
*gap between the two columns got wider*, and acting on the gap is different
from acting on either column alone.

If changing code is now three times cheaper and changing a boundary costs what
it always did, then the **relative** cost of a badly placed boundary has
tripled. The mistake that used to be one expensive mistake among many is now
the expensive one.

The 2025 DORA report puts an uncomfortably direct number behind this. Its
central finding is that AI is an amplifier — *"AI doesn't fix a team; it
amplifies what's already there"* — and specifically:

> Teams working in loosely coupled architectures with fast feedback loops see
> gains, while those constrained by tightly coupled systems and slow processes
> see little or no benefit.

Read that as a statement about architecture, because it is one. The teams who
drew their boundaries well are collecting the cheap-code dividend. The teams
who did not are watching a faster typist produce changes that still have to be
coordinated across six services and three teams, which was always the
bottleneck and still is.

## The rule, restated

Decide late by default. Decide early for exactly two categories:

| Decide early | Because |
|---|---|
| **Things you cannot change later** — boundaries, data ownership, the contract, the identity model, the standard you conform to | The cost is coordination, and coordination did not get cheaper |
| **Things that fail everywhere at once** — config and policy propagation, a central dependency, the one service everything calls | A small mistake here is not a small incident |

Everything else — the furniture — can wait, and should.

Two useful corollaries:

- **Do not build the option. Do build the off switch.** "We might want to
  configure this someday" is the speculative complexity the old rule rightly
  forbids. "We need to be able to turn this off during an incident" is in the
  second category above, and belongs in the first version. The two feel
  similar and are opposites.
- **Conform to a standard early; wrap a vendor late.** A hand-rolled
  abstraction over a vendor is still usually waste. Speaking a standard
  protocol or API is not an abstraction, it is an exit. See
  [Ports and Adapters](/docs/styles/ports-and-adapters/).

## When this is wrong

If you are pre-product-market-fit, ignore most of this. The boundary you draw
carefully today is around a domain that will not exist in six months, and the
cheapest boundary is none — one deployable, one database, and the freedom to
be wrong about everything. The argument above assumes a system that will still
be running in two years and a team larger than one.

Also: cheap code changes do not make a *bad* design fine. They make a bad
design survivable for longer, which is not the same thing and is occasionally
worse, because the pain arrives later and all at once.

## What it costs

Taking this seriously means spending real time early on the two categories —
and that is design work, with meetings, in a phase where nothing is shipping.
It looks exactly like the up-front design that the original rule was written
to prevent, and the only defence is the narrowness of the list. If your
"decide early" list has twelve items on it, you have talked yourself back into
waterfall using modern vocabulary.

The other cost is that the two columns are not fixed. They moved once; they
will move again. Anything on this page is a reading of current prices, not a
law, and the discipline is to keep repricing rather than to memorise the
answer.

## See also

- [Irreversible Decisions](../irreversible-decisions/)
- [The Boring Baseline](../boring-baseline/)
- [Coupling and Cohesion in Practice](../coupling-and-cohesion/)
- [Monolith First, Split on Evidence](/docs/system-design/monolith-first/)
- [Static Stability: Serving on Last-Known-Good](/docs/reliability/static-stability/)

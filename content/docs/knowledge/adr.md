---
weight: 8010
title: "Architecture Decision Records That Get Read"
description: "One page, written at the moment of the decision, recording the context and what you rejected. That last part is the whole value."
icon: "history_edu"
date: "2026-08-20"
lastmod: "2026-08-20"
draft: false
---

## The situation

Someone asks "why is it built this way?" The people who decided have left,
moved teams, or genuinely do not remember. The code shows *what* was decided
and nothing about *why*, and the reasons — the constraints, the alternatives,
the thing that was true in 2022 and is not true now — are gone.

An ADR is the cheapest possible fix: one page, written once, at the time.

## The default

A short markdown file per decision, in the repository the decision affects,
numbered sequentially and never edited after acceptance.

```markdown
# ADR-014: Use Postgres row-level security for tenant isolation

Status: Accepted
Date: 2026-03-11
Deciders: Platform team
Supersedes: —

## Context

We are multi-tenant and pooled. Tenant filtering is currently enforced by
a `WHERE tenant_id = ?` in the data access layer. Two near-misses in the
last quarter involved reporting queries that bypassed that layer.
We have ~40 engineers, most of whom write queries occasionally.

## Decision

Enforce tenant isolation with Postgres row-level security, with the tenant
set per connection at checkout from the pool.

## Alternatives considered

- **Keep application-level filtering, add a linter.** Rejected: the linter
  cannot see dynamic SQL or the reporting path, which is where both
  near-misses occurred.
- **Schema per tenant.** Rejected: 4,000 tenants means 4,000 migrations;
  our deploy process cannot absorb that.
- **Separate read replicas per tenant.** Rejected on cost.

## Consequences

- Cross-tenant reads become impossible at the database level, including from
  ad-hoc queries and the reporting path.
- Connection pooling gets more complex: the tenant context must be set and
  reset around each checkout. Risk of leakage if this is wrong — mitigated by
  a test that asserts isolation across pooled connections.
- Superuser and the migration role bypass RLS. Those roles must be restricted.
- Roughly 5-8% query overhead measured on our workload.
```

**The alternatives section is the point.** Anyone can infer the decision from
the code. Nobody can infer what you already ruled out, which is why teams
re-litigate the same rejected option every eighteen months.

## Rules that keep them useful

- **Write it when the decision is made**, not afterwards. A retrospective ADR
  is reconstruction and it launders the messy real reasons into tidy fake ones.
- **Immutable once accepted.** To change a decision, write a new ADR that
  supersedes the old one. The old one stays, marked superseded. The history of
  reasoning is the asset.
- **Store it next to the code**, in the repository, reviewed in a PR. An ADR in
  a wiki is an ADR nobody will find.
- **One page.** If it needs ten, it is a design document; write that separately
  and link to it from a one-page ADR.
- **Record the constraints that were true at the time**, especially team size,
  scale, and deadlines. These are what change, and knowing them is how a future
  reader decides whether the decision still holds.

## What deserves one

Use the [irreversibility test](/docs/foundations/irreversible-decisions/): if
undoing it would be expensive, write it down.

| Write an ADR | Do not |
|---|---|
| Datastore choice | Library for date formatting |
| Tenancy model | Log format |
| Sync vs async between teams | Internal function structure |
| Authentication and authorisation model | Naming convention (that is a style guide) |
| Deliberately accepted risk or debt | Anything you would change in an afternoon |
| A rejected proposal, and why | — |

That last row is underused. An ADR recording *"we considered adopting a service
mesh and decided not to, because X"* saves the next person a month.

## Making them get read

Written and unread is the common outcome. What helps:

- **An index** with one line per ADR — title, date, status — so the set is
  scannable.
- **Link from the code.** A comment saying `// see ADR-014` at the place the
  decision manifests is worth more than any index.
- **Include them in onboarding.** "Read ADRs 1–20 in your first week" is one of
  the highest-value onboarding tasks available.
- **Reference them in review.** "This contradicts ADR-9" is a productive review
  comment and it teaches people the ADRs exist.

## When the default is wrong

A two-person team that talks constantly has less need. Even then, the six-month
version of yourself is a different person with no memory — a handful of ADRs
for the big decisions still pays.

Some organisations need heavier design documentation for regulatory reasons.
ADRs complement that rather than replacing it: the design doc says what the
system is, the ADR says why this shape and not another.

## What it costs

Thirty minutes per significant decision, and the discipline to do it while
under delivery pressure — which is exactly when it gets skipped and exactly
when the decisions are worst.

The other cost: writing down "we accepted this risk" creates a record that can
be read back to you later. That is the accountability that makes the practice
valuable, and the reason some people quietly resist it.

## See also

- [Irreversible Decisions](/docs/foundations/irreversible-decisions/)
- [Design Reviews and RFCs](../design-reviews/)

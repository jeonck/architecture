---
weight: 9080
title: "Data Platform Styles: Lakehouse and What Sits Around It"
description: "Six styles that are genuinely alternatives, two that are not alternatives to anything, and the one decision everybody forgets is two decisions."
icon: "database"
date: "2026-08-28"
lastmod: "2026-08-28"
draft: false
---

## The situation

The warehouse is expensive and cannot hold the training data. The lake holds
everything and nobody trusts a number that comes out of it. So both exist, the
same customer table is loaded into each by a different pipeline, and the two
disagree by about 2% in a way nobody has time to explain to finance.

Meanwhile the analytics team wants yesterday's data by 9am and the fraud team
wants this minute's, and those two requirements have been answered by two
completely separate stacks that share nothing except the source database they
both hammer.

Every style below is an answer to some part of that. Most teams need one of
the first two and are being sold one of the last two.

The style the rest of this page is measured against looks like this:

<iframe src="/diagrams/data-platform.html" title="Lakehouse: one copy, engines chosen separately"
        loading="lazy" style="width:100%;height:820px;border:1px solid var(--bs-border-color,#ddd);border-radius:8px"></iframe>

[Open the diagram full screen](/diagrams/data-platform.html)

## The styles that are actually alternatives

**Modern Data Stack (ELT).** Managed connectors land raw source data in a
cloud warehouse, and transformation happens afterwards, in SQL, inside the
warehouse. Inverting the order is the whole idea: you stop maintaining
transformation code that runs before you can see the data. Fivetran or Airbyte
into Snowflake or BigQuery, dbt on top, Airflow or Dagster holding the schedule.

It is the fastest thing to value on this page and it is the right answer far
more often than its reputation suggests. It stops being the right answer when
the connector you need does not exist, or when ML feature volume turns
warehouse compute into the largest line on the cloud bill.

**Lakehouse.** Object storage holds the files; an open table format
(Iceberg, Delta Lake, Hudi) puts ACID transactions, schema evolution and time
travel on top of them; a catalogue makes the tables findable; and query
engines are chosen separately and swapped without moving data. One physical
copy is read by BI and by training jobs.

This is the answer when BI and ML must see the same data, when storage and
compute need to scale on different curves, or when reducing vendor lock-in is
an actual goal rather than a slide. The cost lands in operations: many small
updates produce many small files, and compaction and file maintenance become a
standing job somebody owns.

**Lambda.** A batch layer recomputes everything, a speed layer handles the
recent increment, and a serving layer merges the two for reads.

Do not reach for this by default. The same business logic exists twice, in two
runtimes, and the two copies drift — not if, when. Lambda earns its place in
exactly one situation: a large, correct, already-paid-for batch estate that
needs a real-time view added beside it rather than a rewrite. If the reason for
the second code path cannot be written down in a sentence, look at Kappa first.

**Kappa.** Delete the batch layer. One stream path, one implementation of the
logic, and reprocessing done by rewinding the log and replaying it.

The trade is honest and it is about retention: replay only works if the log
still holds the window you need to reprocess, and holding it costs money. A
very large historical reprocess can also be slower through the stream engine
than a batch job would have been.

**Streaming Lakehouse (CDC-fed).** Change data capture off the source database
flows through a log into lakehouse tables, which are updated continuously
instead of rebuilt nightly. Debezium into Kafka, Flink writing Iceberg or Delta
upserts, the same tables queried by the same engines as before.

This is the upgrade path for a team that already runs a lakehouse and needs
minutes instead of hours, without standing up a separate real-time store. The
bill arrives as small files: frequent upserts demand compaction, and the
merge-on-read versus copy-on-write choice in the table format decides whether
your queries or your writes pay for it. What CDC costs upstream is
[Event Sourcing and CDC](/docs/data-and-state/events-and-cdc/).

**Zero-ETL and federated query.** Do not copy at all. Query the source where it
lives through Trino or Athena, or use a vendor's no-copy integration, and
delete the pipeline.

Genuinely correct for exploration, for prototyping, and for the case where
maintaining a pipeline costs more than the queries do. It fails the moment
query volume grows, because the load goes straight to the source system, and
when the source is down the dashboard is down with it.

### The comparison

| | Latency | Pipeline paths | Storage copies | Governance | Reprocessing | Main cost driver |
|---|---|---|---|---|---|---|
| **Modern Data Stack** | Hours to a day | Single (ELT) | Warehouse only | Central | Re-run the transform | Warehouse compute |
| **Lakehouse** | Hours to a day | Single | One, shared by BI and ML | Central or federated | Backfill job over the table | Storage plus engine compute |
| **Lambda** | Seconds, plus a batch view | Two, same logic twice | Batch store plus speed store | Central | Batch recompute | Maintaining two code paths |
| **Kappa** | Seconds | Single (stream) | Serving store | Central | Replay the log from an offset | Log retention |
| **Streaming Lakehouse** | Seconds to minutes | Single | One, continuously updated | Central or federated | Replay CDC, or backfill | Compaction and file maintenance |
| **Zero-ETL / federation** | Source-live | None | Zero — no copy | Source systems' own | Not applicable — nothing is derived | Load on the source system |

Read the last column first. It is the one that changes after you have shipped,
and it is the one nobody models beforehand.

## The two that are not alternatives to anything

**Medallion (Bronze / Silver / Gold) is not an architecture.** It is a
layering convention *inside* a lakehouse: raw, then cleaned and conformed,
then business aggregates. It answers "which of these four hundred tables can I
trust", which is a real question once a lakehouse has been running for a year.
It does not belong on the comparison table above, and putting it there — as
though you were choosing between Lakehouse and Medallion — is the clearest
signal that a data platform document was written from vendor marketing. The
failure mode of adopting it too early is a three-hop pipeline for a
transformation that needed one.

**A semantic layer is orthogonal to every style here.** Metric definitions
live in one place (dbt's semantic layer, Cube) and every BI tool consumes the
same definition, so "revenue" means one thing in Looker and in the spreadsheet
the CFO actually uses. You can put one on top of any style on this page, and
the problem it solves — three dashboards, three different numbers, all
defensible — is organisational rather than architectural.

## Data Mesh is an org chart. Data Fabric is a toolchain.

These two get set against each other constantly and they are not the same kind
of thing, which is why the argument never resolves.

**Data Mesh is organisational design** with four principles: domain teams own
their data, that data is treated as a product with an SLA and a schema
contract, a self-serve platform makes that ownership affordable, and governance
is federated — global standards, local execution. It applies when you have
several domain teams and a central data team that has become the queue
everybody waits in. It does not apply when there is one central team, because
there are no domains to hand ownership to, and adopting the principles without
funding the platform renames your silos rather than removing them. Most of the
cost is organisational, which is why it is [the same shape of decision as
splitting services](/docs/system-design/service-boundaries/).

**Data Fabric is technical architecture**: a metadata-driven layer over sources
that stay where they are — active metadata, a catalogue, lineage, a knowledge
graph, federated query. It applies when the sources are spread across systems
and clouds and physically consolidating them is impossible or not worth it, or
when regulation forbids moving the data at all. It is only as good as the
metadata underneath it; with poor lineage the whole layer is decorative.

Adopting a mesh while buying fabric tooling is not a contradiction. It is the
normal case: the fabric is a plausible way to build the self-serve platform the
mesh requires.

## Table format and catalogue are two decisions

Choosing Iceberg does not choose your catalogue, and this is where lakehouse
projects quietly break.

- **Table format** — the file layout and transaction protocol: Apache Iceberg,
  Delta Lake, Apache Hudi.
- **Catalogue** — table discovery, permissions and lineage: Unity Catalog,
  Apache Polaris, Nessie, AWS Glue.

Pick a format and skip the catalogue decision and you get the classic
symptom: the table exists, one engine can see it, another cannot, and nobody
can say which system is authoritative about who may read it. Write both
decisions down, separately, with the reason for each.

## When the default is wrong

- **The data fits in Postgres.** A great many "data platforms" are a reporting
  replica and a scheduled query away from being finished. Start there and see
  [Choosing a Datastore](/docs/data-and-state/choosing-a-datastore/).
- **Latency is measured in days and the sources are SaaS.** Modern Data Stack.
  Anything more is buying capability you will not use.
- **One team, one domain.** Data Mesh has nothing to attach to. The principles
  become vocabulary with no owners behind them.
- **No ML, no unstructured data, no second engine.** The lakehouse's central
  benefit — one copy, many engines — is not being collected, and you are paying
  the file-maintenance cost for nothing.
- **Second-scale latency with no durable log.** Kappa is not reachable without
  a log you can replay. That log is the prerequisite, not a detail.

## What it costs

**Small files are the recurring tax of every table-format style.** Streaming
ingest and frequent upserts produce them continuously, and compaction is a job
with a schedule, a cost, and a failure mode — not a setting. Budget an owner
for it before the first CDC pipeline, not after the first slow quarter.

**Freshness becomes a per-consumer product decision.** Once the platform can
deliver minutes, every team will ask for minutes, and someone has to decide
which dashboards genuinely need it and which are being expensive out of habit.
This is the same argument as [read model
lag](/docs/styles/cqrs-and-read-models/), one layer down.

**The catalogue becomes the permission model.** Access control moves from the
warehouse into the catalogue, and if that migration is partial you now have two
places to check and one of them is stale. See [Least
Privilege](/docs/security/least-privilege/).

**Query cost is now a property of the query, not the cluster.** Separated
compute means a badly written join is a bill rather than a CPU graph. The same
inversion as serverless, with the same answer: budget alerts are an
availability control here, not finance hygiene.

**Reprocessing has to be rehearsed.** Whether it is a backfill job or a log
replay, "we can always rebuild it" is a claim about retention windows and
elapsed time that is only true if someone has run it on production-sized data.
Untested, it is the same false comfort as an unrestored backup.

## See also

- [Event Sourcing and CDC: When It Pays](/docs/data-and-state/events-and-cdc/)
- [CQRS and Read Models](../cqrs-and-read-models/)
- [Choosing a Datastore](/docs/data-and-state/choosing-a-datastore/)
- [Service Boundaries That Survive Reorgs](/docs/system-design/service-boundaries/)
- [Platform as Product](/docs/platform/platform-as-product/)

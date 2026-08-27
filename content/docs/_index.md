---
weight: 1
title: "Overview"
description: "Practical architecture knowledge for DevOps engineers and developers — the decisions, trade-offs, and failure modes that only show up in production."
icon: "home"
date: "2026-08-20"
lastmod: "2026-08-20"
draft: false
---

## What this is

Most architecture knowledge lives in people's heads. It gets earned during a
bad migration, a 3am page, or a review where somebody senior says *"we tried
that in 2019, here's what broke."* Then that person changes teams and the
knowledge leaves with them.

This site is an attempt to write that knowledge down — the tacit made
explicit. Every page answers the same question in a different context: **what
does an experienced engineer actually decide here, and what are they trading
away when they decide it?**

It is deliberately opinionated. Neutral surveys of options are easy to find
and useless under time pressure. What is hard to find is a default, the
conditions under which the default is wrong, and the signal that tells you
which case you are in.

## How to read it

Each page follows roughly the same shape:

- **The situation** — when this decision actually lands on your desk.
- **The default** — what to do when you have no strong evidence either way.
- **When the default is wrong** — the specific conditions that flip it.
- **What it costs** — the bill that comes due six months later.

Skim the defaults. Read the costs.

## Sections

1. [Foundations](/docs/foundations/) — what architecture actually is once you strip the diagrams away.
2. [System Design in Practice](/docs/system-design/) — boundaries, contracts, and integration styles that survive contact with a real org.
3. [Data and State](/docs/data-and-state/) — the layer that is hardest to change and easiest to get wrong.
4. [Delivery and CI/CD](/docs/delivery/) — how code gets from a laptop to production without drama.
5. [Platform and Infrastructure](/docs/platform/) — the substrate everything runs on, and what it charges you.
6. [Reliability and Operations](/docs/reliability/) — keeping it up, and knowing what "up" means.
7. [Security by Design](/docs/security/) — the parts you cannot bolt on afterwards.
8. [Making Knowledge Explicit](/docs/knowledge/) — ADRs, diagrams, runbooks, and team memory.
9. [Architecture Styles](/docs/styles/) — named styles read from real open-source systems, and the trades they have already made for you.

## A warning

Nothing here is a standard. Context beats advice every time — a pattern that
saved one team will sink another with a different team size, load profile, or
regulatory posture. Treat every page as a starting position for an argument,
not the end of one.

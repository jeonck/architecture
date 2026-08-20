---
weight: 30
title: "Data and State"
description: "The layer that is hardest to change and easiest to get wrong: storage choice, migrations, consistency, and caching."
icon: "database"
date: "2026-08-20"
lastmod: "2026-08-20"
draft: false
---

You can rewrite a service in a quarter. You cannot rewrite the data model of a
system with ten years of production rows in anything like that time — and every
consumer you did not know about will find you the moment you try.

This section covers datastore selection, zero-downtime migrations, the
consistency model you actually need (usually weaker than you think), caching
without corrupting truth, and when event-driven data patterns earn their
complexity.

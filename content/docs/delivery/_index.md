---
weight: 40
title: "Delivery and CI/CD"
description: "How code gets from a laptop to production without drama: branching, pipelines, progressive rollout, and rollback."
icon: "rocket_launch"
date: "2026-08-20"
lastmod: "2026-08-20"
draft: false
---

Deployment frequency is not a vanity metric. A team that ships once a quarter
batches up hundreds of changes, so every release is a high-variance event and
every incident has a hundred suspects. Small, frequent, reversible changes are
an architectural property, not a process preference.

This section covers branching that does not create merge debt, pipelines that
give fast feedback before slow gates, progressive delivery, reproducible
builds, and treating rollback as a first-class feature.

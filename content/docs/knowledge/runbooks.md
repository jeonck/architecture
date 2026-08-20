---
weight: 8030
title: "Runbooks and On-Call Docs"
description: "Write for a tired, frightened person who did not build this and is being asked to fix it right now."
icon: "menu_book"
date: "2026-08-20"
lastmod: "2026-08-20"
draft: false
---

## The situation

3am. An alert fires. The person on call did not write this service, has never
seen this alert, and has about ninety seconds of clear thinking before stress
degrades their judgement.

The document they open determines how the next hour goes. Most such documents
are written by the person who built the system, for a reader who already
understands it — which is precisely the wrong audience.

## The default

**Every alert links to a runbook. Every runbook answers four questions in the
first screen.**

1. **What does this alert mean, in user terms?** Not "queue depth > 1000" —
   "orders are being accepted but not dispatched; customers see 'processing'
   indefinitely."
2. **How bad is it?** Who is affected, how many, is it getting worse. Is this a
   page or a morning problem?
3. **What do I check first?** Two or three specific things, with links to the
   exact dashboard, query, or log filter. Not "check the logs."
4. **What are the safe actions?** Restart, scale, fail over, disable this
   feature flag. Explicitly including what is *not* safe.

If those four are answered above the fold, the runbook has done most of its
job. Everything after is detail for the harder cases.

## The shape that works

```markdown
# Alert: order-dispatch-backlog

## What it means
Orders are being accepted but not dispatched to the warehouse. Customers
see "processing" and do not get a dispatch email. Not visible as an error
to them — they will notice within a few hours.

## Severity
P2 if the backlog is stable, P1 if growing for more than 15 minutes.
Every affected order needs manual reconciliation afterwards; see below.

## First checks
1. Backlog trend: [dashboard link] — growing or flat?
2. Consumer health: `kubectl get pods -l app=dispatcher` — are they running?
3. Warehouse ERP status: [status page] — the SFTP endpoint fails weekly
   around 02:00 UTC during their backup window.

## Safe actions
- Restart consumers: `kubectl rollout restart deploy/dispatcher` — safe,
  processing is idempotent (see ADR-22).
- Scale up: up to 12 replicas. Beyond that the ERP rejects connections.

## Do NOT
- Do not purge the queue. Those are real orders and there is no replay.
- Do not disable `dispatch-v2` flag; the fallback path was removed in March.

## If none of that works
Escalate to #team-fulfilment. On-call: [rota link].
Business impact contact: [name] — they decide about customer comms.

## After
Every order dispatched more than 2 hours late needs a reconciliation run:
[link to procedure].
```

Note the "Do NOT" section. It is the highest-value part and the most commonly
missing: at 3am, the destructive action often looks like the obvious one.

## What makes runbooks rot

- **Written once at launch**, never revisited. The commands are for an
  infrastructure that no longer exists.
- **Linked from nowhere.** If it is not in the alert payload, it will not be
  found under pressure.
- **Written by the expert, for the expert.** "Obviously you'd check the
  reconciler state" is not obvious to the reader you have.
- **Copy-pasted between services**, so 80% is generic boilerplate and the
  specific part is buried.

Countermeasures that work:

- **Update the runbook as an action item from every incident that used it.**
  This is the only maintenance mechanism that reliably happens, because the gap
  is fresh in someone's mind.
- **Have new joiners take on-call early, shadowed.** They find every unclear
  instruction, because they are exactly the reader the document is for.
- **Test the commands.** A runbook command that fails is worse than none — it
  costs time and confidence.

## Alerts without runbooks

An alert with no runbook and no clear action should not page anyone. Either
write the runbook or downgrade the alert to a dashboard.

This is a useful forcing function. Applied honestly, it usually reveals that a
third of the alerting is noise nobody knows how to act on — and deleting it is
a direct improvement to on-call quality.

## When the default is wrong

Well-understood, frequently-exercised operations do not need prose — they need
automation. If a runbook says "run these six commands in order, every time,"
that is a script waiting to be written. The runbook should then say "run the
script, and here is how to tell if it worked."

The ideal runbook shrinks over time as its contents become automation.

## What it costs

Runbooks are documentation, so they decay by default, and maintaining them
competes with feature work. The realistic target is not comprehensive coverage
but good coverage of the alerts that actually fire.

Start from the list of what paged someone in the last quarter. That is a much
shorter and more valuable list than "every service."

## See also

- [Incident Response](/docs/reliability/incident-response/)
- [Observability](/docs/reliability/observability/)

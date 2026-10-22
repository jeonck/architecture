---
weight: 5030
title: "Environments, Config, and Secrets"
description: "Config is what differs between environments. Secrets are config that must never be readable. Conflating them causes both problems."
icon: "key"
date: "2026-08-20"
lastmod: "2026-10-22"
draft: false
---

## The situation

The app needs a database URL, a feature flag, an API key, and a log level. Some
of these vary per environment, some are sensitive, some change at runtime, and
teams routinely handle all four with the same mechanism — usually environment
variables in a deploy script — and then wonder why rotating a key requires a
redeploy of nine services.

## The default

Separate config by **how it changes** and **who may read it**:

| Kind | Example | Where it lives | Change cadence |
|---|---|---|---|
| **Build-time constants** | Feature set, compile flags | In the artifact | Per build |
| **Environment config** | DB host, region, log level, service URLs | Deploy-time injection (env vars, ConfigMap) | Per deploy |
| **Runtime config** | Feature flags, rate limits, kill switches | A config service or flag system, read live | Seconds |
| **Secrets** | DB password, API keys, signing keys | A secret manager, fetched at start or refreshed | On rotation |

The distinction that matters most: **runtime config must not require a deploy**.
If turning off a misbehaving integration requires a build and a rollout, your
incident response time is bounded by your pipeline duration.

Which immediately raises the problem in the next section, because that
sentence is only half the story.

## Runtime config is a deploy with none of the safeguards

Skipping the pipeline is the point of runtime config. It is also the risk, and
the two are the same property seen from different sides. A flag flip, a
rate-limit change, a policy row, a generated data file — each reaches every
instance in production in seconds, with no build, no review, no canary, and
usually no rollback button. It has the blast radius of a deploy and the
ceremony of a database write.

Two 2025 outages make the shape of it concrete, and neither involved a code
deploy:

- **Cloudflare, November 2025.** A permissions change in a database caused a
  query to start returning duplicate rows, which roughly doubled the size of a
  Bot Management feature file regenerated every five minutes. The proxy
  preallocates for at most 200 features; the oversized file exceeded that and
  the unhandled error panicked the core proxy, returning 5xx across the
  network. Nobody edited a config value — an *upstream permission* changed the
  content of generated data that ships globally every few minutes.
- **Google Cloud, June 2025.** A policy change inserted into regional Spanner
  tables contained unintended blank fields. The metadata replicated globally
  within seconds, and a quota check added weeks earlier without feature-flag
  protection hit a null pointer on those blanks and crashed Service Control
  worldwide. Google's own follow-up: enforce feature-flag protection on all
  changes to critical binaries, disabled by default.

So treat the config and data plane as a release channel, and give it the
controls the code channel already has:

| Control | What it means for config |
|---|---|
| **Schema validation at write time** | Reject the value before it propagates, not after. A type, a range, a size ceiling |
| **Size and cardinality bounds** | If a consumer preallocates for 200 of something, 201 must fail the write, not the reader |
| **Staged rollout** | A flag reaches 1% of instances, then 50%, then all — the same shape as [progressive delivery](/docs/delivery/progressive-delivery/) |
| **Versioned, with a revert** | Every change is a numbered version and "go back to the previous one" is one action |
| **Reviewed when it is policy** | Authorisation rules and quota policy are code, whatever file they live in |
| **Consumers fail soft** | A consumer that cannot parse new config keeps running on the last version it could. See [Static Stability](/docs/reliability/static-stability/) |

The last row is the one that would have contained both incidents above, and
it is worth stating as a rule: **the reader of config decides how bad a bad
value gets.** A parser that panics converts a data problem into an outage; one
that logs, alarms, and keeps the previous value converts it into a page for
somebody who has time to look.

None of this contradicts the rule above. Runtime config still must not
require a deploy. It must merely stop pretending it is not one.

## Secrets: the rules that are not negotiable

- **Never in the repository.** Not encrypted-in-repo-with-a-key-in-CI either,
  unless the tooling (SOPS, sealed-secrets) is deliberate and reviewed. A secret
  in git history is compromised permanently.
- **Never in the container image.** Images get pushed to registries, shared, and
  scanned.
- **Never in a log line.** Structured logging with automatic redaction of known
  key names, plus a scan in CI for common patterns.
- **Never passed as a command-line argument.** Visible in `ps`.
- **Rotatable without a code change.** If rotation requires a PR, rotation will
  not happen.

Environment variables are acceptable for secrets in most contexts, with a
caveat: they are visible to the whole process, appear in crash dumps and some
error reporters, and are inherited by child processes. Files mounted with tight
permissions are marginally better. Neither is the real control — the real
control is short-lived credentials.

## Prefer identity over secrets

The strongest version of secret management is having fewer secrets.

```mermaid
flowchart TD
    A["Service needs<br/>database access"] --> B{"Can the platform<br/>issue an identity?"}
    B -->|Yes| C["Workload identity:<br/>IAM role, service account,<br/>SPIFFE. Short-lived token,<br/>auto-rotated, nothing to leak"]
    B -->|No| D["Secret manager:<br/>dynamic credential<br/>with a TTL"]
    D --> E["Static long-lived secret<br/>+ rotation process"]
    style C fill:#2d7,stroke:#0a5,color:#000
    style E fill:#f77,stroke:#c00,color:#000
```

Cloud workload identity (IAM roles for service accounts, managed identities)
removes the credential entirely — the platform attests to who the workload is
and issues a short-lived token. If your platform supports it, using anything
else for cloud service access is a choice to have a secret you did not need.

## Environments

The classic dev / staging / prod ladder has a known weakness: staging is never
actually like production, so it validates less than people believe.

Things that make staging worth having:
- Same artifact, same deploy mechanism, same infrastructure shape.
- Realistic data volume (synthetic or anonymised — never a raw production copy;
  that is a data breach with a friendly name).
- Real integrations, or high-fidelity fakes that are versioned alongside the
  contract.

Things that make it theatre:
- Different infrastructure, hand-configured.
- A tenth of the data.
- Everything mocked.
- Nobody notices when it is broken for a week.

If staging is theatre, be honest and invest in
[progressive delivery](/docs/delivery/progressive-delivery/) instead —
production with a 1% canary tells you more than a fake environment does.

## When the default is wrong

Very small teams can reasonably use a single secret manager entry per
environment and skip the taxonomy. The distinctions above earn their keep when
multiple services share secrets and multiple people need to rotate them.

## What it costs

Four mechanisms instead of one means four things to learn, document, and
operate. A runtime config service is a new dependency in your request path, and
if it is down, your defaults had better be sane.

Design the failure mode explicitly: **what does the service do if the flag
system is unreachable?** The answer should be "use the last known value, or a
safe compiled-in default" — never "fail to start."

## See also

- [Secrets and Key Management](/docs/security/secrets-management/)
- [Infrastructure as Code](../infrastructure-as-code/)

# Architecture Field Notes

Practical architecture knowledge for DevOps engineers and developers — the
decisions, trade-offs, and failure modes that only show up in production.
The tacit, made explicit.

**https://architecture.metacog.co.kr**

## What is here

40 pages across eight sections. Every page follows the same shape: the
situation, the default, when the default is wrong, and what it costs.

| Section | Covers |
|---|---|
| Foundations | Irreversible decisions, constraints, coupling, trade-offs, boring baselines |
| System Design in Practice | Monolith first, service boundaries, sync vs async, API contracts, idempotency |
| Data and State | Datastore choice, zero-downtime migrations, consistency, caching, event sourcing and CDC |
| Delivery and CI/CD | Trunk-based development, pipeline design, progressive delivery, reproducible builds, rollback |
| Platform and Infrastructure | Kubernetes reality, IaC, config and secrets, multi-tenancy, platform as product |
| Reliability and Operations | SLOs, observability, backpressure, failure isolation, incident response |
| Security by Design | Threat modelling, identity and authorisation, secrets, supply chain, least privilege |
| Making Knowledge Explicit | ADRs, diagrams, runbooks, design reviews, team memory |

## Building locally

Requires [Hugo Extended](https://gohugo.io/installation/) 0.100+ and a Go
toolchain (Lotus Docs is distributed as a Hugo Module, not a submodule).

```bash
hugo mod get -u
hugo server
```

## Deployment

Pushes to `main` trigger `.github/workflows/hugo.yml`, which builds with Hugo
and deploys to GitHub Pages. The custom domain lives in `static/CNAME` so Hugo
copies it into `public/` on every build.

## Contributing

Corrections and disagreement are welcome — every page is a starting position
for an argument, not the end of one. Open an issue or a PR.

## Licence

Content: [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).
Built with [Lotus Docs](https://github.com/colinwilson/lotusdocs).

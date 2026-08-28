---
weight: 90
title: "Architecture Styles"
description: "The named styles you will actually meet — layered, hexagonal, microservices, event-driven, CQRS, serverless — and the bill each one hands you."
icon: "category"
date: "2026-08-27"
lastmod: "2026-08-28"
draft: false
---

A style is not a pattern. A pattern solves one problem inside a system; a
style decides what kind of system you are building at all, and once chosen it
propagates into every subsequent decision — deployment topology, test
strategy, security model, team shape, how you ship a bug fix on a Friday.

Most of these you will not choose so much as inherit. The layered style is
already in the repository you joined. The microservices decision was made by
someone who left. That is fine — the value of naming a style is not that you
get to pick one from a catalogue, it is that you can see which trades have
already been made on your behalf, and which of them you are still paying for.

Each page names the style, describes the mechanism that makes it work, states
the conditions under which it earns its cost, and is honest about the bill.
Where a style overlaps with a decision covered elsewhere on this site, the
page links across rather than repeating it.

**Nothing here is adopted wholesale.** Real systems are layered inside
hexagons inside services that publish events, and that is normal. The styles
are vocabulary for arguing about a design, not boxes to sort systems into.

---
weight: 8060
title: "Case Study: Reviewing a Small Portal Before It Exists"
description: "A community portal planned as a slide deck, reviewed three times, and what changed. Most of the risk was in the data model, not the hosting."
icon: "fact_check"
date: "2026-09-20"
lastmod: "2026-09-20"
draft: false
---

## The situation

A small team wants to launch a Korean-language portal for construction
workers and contractors in the US. Six menus: hiring, job seekers,
materials, equipment, contractors, immigration. The plan arrives as a
fifteen-slide deck. Menu structure, field lists per menu, a hosting
comparison, a plugin list, a monthly cost table.

The stated goal is to build it once. "We don't want to do this twice when
we outgrow the first version."

That sentence is the whole review. The deck's answer to it was a managed
host that scales by clicking a button. The review's answer was that the
host was never the problem.

## The starting design

The deck was better than most. It had already made the two calls that
usually go wrong:

- It rejected site builders (Wix, GoDaddy) because member boards and
  job listings are weak there, and rejected custom development because
  of cost and developer dependency. WordPress on Cloudways, plus
  plugins, with a three-stage roadmap ending in "custom-build only the
  core features."
- It noticed that five of the six menus are the same thing: a post with
  a category, some structured fields, an expiry, and a paid top slot.
  One directory plugin (Directorist), five directory types.

What it also had:

- Around fifteen plugins on day one, including a forum (wpForo),
  Directorist's paid Pricing Plans and Claim extensions, a media offload
  to Cloudflare R2, Akismet and reCAPTCHA doing the same job, and
  Wordfence duplicating the Cloudflare WAF already in front of the site.
- Region and trade defined per menu, as fields inside each directory
  type.
- Newsletter preferences stored only in MailerLite. Six interest
  checkboxes mapped to six MailerLite groups. The third roadmap stage
  promised "instant alerts matched on region and trade," which
  MailerLite's RSS campaigns cannot do.
- Company profiles (materials vendors, equipment rental firms) placed
  inside the transactional listing types, so they would expire every
  sixty days like a for-sale post.
- Resume PDFs uploaded through the media library, which the offload
  plugin would have made publicly addressable.
- A "reviews from verified customers only" feature with no way to
  verify a customer, because no transaction happens on the site.
- No URL scheme, no user role model, no decision on whether the site
  is bilingual.

None of this is unusual. It is what a plan looks like when it was
written from the menu inward instead of from the data outward.

## What the review changed

Three passes. The first was a general review of the deck. The second
read the team's rewritten "final" plan and found twelve internal
contradictions. The third re-read the rewritten-again plan and found
four more, all introduced by the reviewer in the previous pass.

The changes that mattered, in the order they were argued:

**The schema became the source of truth.** Field keys, allowed values,
and taxonomies are defined in a document. The plugin's custom fields map
to that document. Labels can change; keys cannot. This is the single
change that makes the third roadmap stage a rebuild of code rather than
a re-entry of content.

**Region, trade, and language became global taxonomies** shared by
every listing type and by the alert system. The first draft of the
rewrite got this right for four types and then gave the contractor type
its own `service_area` field. Caught on the second pass.

**Subscription preferences moved into WordPress user records.**
MailerLite receives a one-way sync and sends. Unsubscribes come back by
webhook. The signup form is the WordPress registration form, not the
MailerLite embed, because whichever form runs first owns the data. The
rewritten plan said the right thing on one page and named the MailerLite
embed form on another. Caught on the second pass.

**Company profiles became their own listing type** with a one-year
renewal, surfaced in four menus through a category field. A materials
vendor who also rents equipment is one profile with two categories, not
two posts in two menus. This also gave the immigration menu's "consult a
lawyer" item a concrete home: a lawyer is a business profile with a
category, same as a general contractor.

**URLs got a permanent shape** before anything was built:
`/{segment}/{public-id}/` with an optional slug suffix that routing
ignores. Korean titles do not produce English slugs, so the short public
ID carries the identity. Directorist's default `/directory/` paths are
never exposed, and since Directorist uses a single post type for every
directory, that means a rewrite rule that someone has to write and test.

**Every piece of custom code goes in one site plugin.** Not the theme,
not a snippets plugin. The completion criteria for phase one include
"custom code outside that plugin: zero." When the team eventually
replaces WordPress, that plugin's function list is the specification of
what the business rules were.

**Phase one shrank.** Multi-user organisations, the forum, payments,
ownership claims, reviews, the resume upload, and Cloudflare R2 all
moved to "when there is evidence of demand." The plugin count went from
roughly fifteen to seven external (Directorist, MailerLite, EWWW,
UpdraftPlus, Rank Math, Turnstile, WP Mail SMTP) plus the one site
plugin plus the two Cloudways installs anyway (Breeze and Redis Object
Cache).

## What the review left alone

WordPress, Cloudways, Directorist, and MailerLite all stayed. The
review's job was not to replace the stack with a better one. It was to
make the stack replaceable.

Cloudflare R2 for photos was deferred rather than removed. The
Cloudways 2 GB plan on DigitalOcean includes 50 GB of disk; at five
compressed photos per post that is tens of thousands of posts, and posts
expire. Moving media to R2 later is a standard migration. Moving it on
day one meant an offload plugin with weak R2 support and a public URL
for every resume.

The cost table was not touched. The monthly numbers were plausible and
the review had nothing better.

## Where the reviewer was wrong

The rewrite that fixed the twelve contradictions introduced four new
ones. The company profile type was described as appearing in five
menus; it appears in four. An export table was labelled "ten files" and
listed eleven. The list URL pattern was written as `/{type}/` while the
type was named `job` and the URL was `/jobs/`, with no mapping between
them. A new location field for business profiles was added without
saying how it differed from the location field every listing already
had.

None of these were subtle. They were the kind of thing that survives
because the person who wrote the sentence is the person checking it. The
third pass was a re-read by the same reviewer, later the same day, against a
checklist of counts: six menus, five types, four menus for business
profiles, ten installed plugins, five roles, nine pre-build decisions,
ten export files. The checklist found what the re-read alone had missed.

If you are the reviewer, put the counts in the document and check them
mechanically. Your eye will not.

## How far it was optimised

The honest answer is: as far as the data model, and no further.

What is now fixed and will not be redone: identifiers, URLs, taxonomies,
the role model, where subscription preferences live, where custom code
lives, what the monthly export must contain, and the rule that renewing
a post does not bump it to the top.

What is deliberately still cheap and will probably be thrown away: the
theme, Directorist's screens, the weekly digest built from an
RSS feed, the manual ownership-verification process, and the manual
"premium placement" that an administrator sets by hand until someone
proves people will pay for it.

Three questions remain open and are listed in the plan as things the
implementer must answer on a staging site before build: whether the
free tier of Directorist can gate a contact field behind login without
custom code, whether per-type URL prefixes survive its list and filter
pages, and whether one Directorist directory type can require different
fields per category. Each has a fallback in the site plugin. None is a reason to
change the stack.

That is the level. Not "optimised." Bounded. Every part is either
fixed-and-portable or cheap-and-disposable, and the plan says which.

## What it costs

Three review passes, one of them to correct the reviewer. A schema
document that has to be written before the first plugin is configured,
which feels slow to a team that wants to see pages. One custom plugin
that a WordPress contractor has to build instead of clicking through
settings. A monthly export rehearsal into an empty database that nobody
will enjoy running.

Against that: the third roadmap stage is now a rewrite of a frontend and
an alert engine. The content, the members, the taxonomies, the consent
records, and the status history come along unchanged.

## See also

- [Design Reviews and RFCs](/docs/knowledge/design-reviews/) — the
  written-proposal process this case followed, loosely
- [Irreversible Decisions](/docs/foundations/irreversible-decisions/) — URLs, identifiers, and
  taxonomies are the irreversible ones here
- [Architecture Decision Records](/docs/knowledge/adr/) — the plan's
  change log is an ADR list by another name

I build and run a production SaaS in a regulated corner of fintech.

I also build the agents that ship it. Not prompts, infrastructure:
specialist reviewers with their own checklists, hooks that fire a
senior-engineer audit at the moment of the decision, gates that refuse a
commit or a push that is not finished, and stack profiles so the agent
reads the sanctioned command instead of guessing it. The point is work that
holds up at a senior bar, not work that looks like it does.

## FUNDesk &nbsp;·&nbsp; [fundesk.ai](https://fundesk.ai)

An AI-native CRM for merchant cash advance brokers and funders. Multi-tenant
from the first migration, not retrofitted onto it.

The invariants are the product. Every tenant-owned query filters by the
tenant key or goes through the wrapper. Sensitive fields are encrypted at
rest and every render of one writes a row to an append-only access log.
Money is integer minor units, inside a transaction, with idempotency keys on
anything a webhook or a cron can replay. The actor who creates a payable
cannot approve it. Bulk endpoints reject the whole batch if any row would
self-approve.

The parts that took longest are the parts nobody sees. Production migrations
run through one runner that verifies the target host, prints what it is
about to apply, and demands a typed confirmation. Staging is created on
demand: a copy-on-write branch off the production database, a preview deploy
wired to it, migrations rehearsed against real-shaped data, then thrown
away. A nightly sweep drives every route in the app as a low-privilege user
against live production and fails on a 500, a client-side exception, or an
error boundary. Production is held until CI is green, and rollback is
promoting the previous deploy.

## Agent infrastructure

The tooling that makes the above shippable by an agent without the review
burden landing back on me.

Six specialist reviewers, each with its own brief: tenant isolation, money
correctness, migration safety, LLM call sites, conventions, and one that
boots the app and drives the changed flows for real. A `UserPromptSubmit`
hook that asks the ambiguity question before the first edit. A `Stop` hook
that blocks the turn so the agent audits its own diff before I read it.
Gates that deny a push without a graded self-check trailer, deny a commit
carrying a bare TODO or a print-debug line, and deny piping a CI watcher
through `tail` because the pipe's exit code replaces the run's.

A session registry so several agent sessions in one checkout do not tangle
the working tree. Stack profiles so the sanctioned install, test, and
migrate commands come from a file rather than from the model's memory. And
a rubric for the LLM call sites inside the product itself: spend gate before
issuance, schema validation after, no hardcoded dates in cached prompts, a
pure prompt module the eval harness can import.

Most of it is open source, below.

## Data infra

Public-records ingestion across dozens of state jurisdictions, each one a
different problem. ASP.NET WebForms driven through `__doPostBack` and
EventValidation. Angular SPAs whose real API sits behind SAML. Cloudflare
Turnstile, Incapsula, and Akamai in front of the ones that matter most.
Image CAPTCHAs, Socrata endpoints, SFTP bulk feeds measured in gigabytes,
and a handful that only ever yielded to a reverse-engineered JSON API found
by reading the network tab.

Millions of records normalized into one schema, with entity enrichment
joined on top from a different source per state. The interesting engineering
is not the scraping, it is convergence: knowing which of four spellings of a
company name is the same company, and proving it.

## Open source

**[senior-mode](https://github.com/MaxWynnDev/senior-mode)** &nbsp;·&nbsp; `"use strict"` for coding agents

The agent infrastructure above, extracted and made portable. One core,
adapters for nine agents, seven stack profiles picked from evidence instead
of guessed, and a 60-case harness that runs on Linux, macOS, and Windows.
MIT.

## Stack

| | |
|---|---|
| **Product** | TypeScript, Next.js App Router, Postgres, Drizzle, Vercel |
| **Agents** | Claude Code, Codex, and seven more through one hook contract |
| **Verification** | Playwright against live production, GitHub Actions as the deploy gate |
| **Data** | Python, httpx, Playwright, Socrata and SFTP feeds, Postgres |
| **Everywhere else** | Bash, because a guardrail that only works on one OS is not a guardrail |

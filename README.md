I build and run a production SaaS in a regulated corner of fintech.

## FUNDesk &nbsp;·&nbsp; [fundesk.ai](https://fundesk.ai)

An AI-native CRM for merchant cash advance brokers and funders. Multi-tenant from the first migration.

Most of the work is in the invariants. Every tenant-owned query filters by the tenant key or goes through the 
wrapper. Sensitive fields are encrypted at rest, and every render of one writes a row to an append-only access
log. Money is integer minor units, inside a transaction, with idempotency keys on anything a webhook or a cron 
can replay. Whoever creates a payable can't approve it. Bulk endpoints reject the whole batch if any row would 
self-approve.

Most of my time went into the operational side. Production migrations run through one runner that verifies the 
target host, prints what it's about to apply, and demands a typed confirmation. Staging gets created on demand:
a copy-on-write branch off the production database, a preview deploy wired to it, migrations rehearsed against 
real shaped data, then thrown away. A nightly sweep drives every route as a low-privilege user against live 
production and fails on a 500, a client-side exception, or an error boundary. Production is held until CI is 
green. Rollback is promoting the previous deploy.

## Agent infra

The tooling that lets an agent ship the above without the review burden landing back on me.

Six specialist reviewers, each with its own brief: tenant isolation, money correctness, migration safety, LLM 
call sites, conventions, and one that boots the app and drives the changed flows for real. A `UserPromptSubmit` 
hook that asks the ambiguity question before the first edit. A `Stop` hook that blocks the turn so the agent 
audits its own diff before I read it. Gates that deny a push without a graded self-check trailer, deny a commit
carrying a bare TODO or a print-debug line, and deny piping a CI watcher through `tail`, since the pipe's exit 
code replaces the run's.

A session registry so several agent sessions in one checkout don't tangle the working tree. Stack profiles so
the sanctioned install, test and migrate commands come from a file rather than the model's memory. And a rubric
for the LLM call sites inside the product: spend gate before issuance, schema validation after, no hardcoded 
dates in cached prompts, a pure prompt module the eval harness can import.

Most of it is open source, below.

## Open source

**[senior-mode](https://github.com/MaxWynnDev/senior-mode)** &nbsp;·&nbsp; `"use strict"` for coding agents

The agent infrastructure above, extracted and made portable. One core, adapters for nine agents, seven stack
profiles picked from evidence instead of guessed, and a 60-case harness that runs on Linux, macOS and Windows. 
MIT.

**[hangar](https://github.com/MaxWynnDev/hangar)** &nbsp;·&nbsp; team chat where `@claude` is a real Claude Code session

Write `@claude fix the flaky login test` in a room and a session starts on a machine you control, works in your
repo, and posts back what it did. The web app never spawns anything: it queues a row for a companion process on 
your own machine, so the process holding the database and the process running code are never the same one. 
MIT.

## Stack

| | |
|---|---|
| **Product** | TypeScript, Next.js App Router, Postgres, Drizzle, Vercel |
| **Agents** | Claude Code, Codex, and seven more through one hook contract |
| **Verification** | Playwright against live production, GitHub Actions as the deploy gate |
| **Data** | Python, httpx, Playwright, Socrata and SFTP feeds, Postgres |
| **Everywhere else** | Bash, so the guardrails work the same on every OS |

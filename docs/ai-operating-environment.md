# The AI Operating Environment

> The systems I drive with an AI agent, and the workflows I've built on top of them. The [case studies](../README.md) show *what* I built; this is the *environment* it runs in.

I'm a Product Owner / technical business analyst. I don't have a QA team, a release team, or an analyst team behind me — so I built one. Every system below is connected to a single AI agent ([Claude Code](https://claude.com/claude-code)) through the **Model Context Protocol (MCP)**, and every workflow is a codified *skill* the agent runs the same way twice. The organizing idea is simple: **repetitive manual work is un-codified process** — so I codify it.

Two halves:

- **The toolstack** — the systems the agent can reach. Each row answers *what changes when the tool is wired into an agent* instead of driven by hand.
- **The skill library** — the workflows I've codified on top of that stack. Several of these aren't single prompts; they're small agent *systems* with a supervisor and specialized workers.

---

## Part 1 — The Toolstack

*Named tools are public, third-party products. Employer-specific systems, data, and identifiers are withheld — the integrations are real; the proprietary data is not.*

### Ticketing & product documentation

| Tool | What the AI layer adds |
|---|---|
| **Jira** | System of record for tickets, epics, and release scope. Wired to the agent, plain-English intent becomes JQL, triage, formatted comments, and acceptance criteria — so grooming, release assembly, and requirement traceability happen conversationally instead of through the web UI. |
| **Confluence** | The team knowledge base and spec home. The agent reads and drafts pages in place, keeping charters, requirements docs, and runbooks synced with the tickets they describe rather than drifting apart. |

### Observability & error tracking

| Tool | What the AI layer adds |
|---|---|
| **Datadog** | Logs, metrics, traces, and monitors across every service. A plain question ("payment errors in production last night?") becomes the right log query, read back as a finding — collapsing dashboard-hunting into a sentence. |
| **Rollbar** | Application exception tracking, surfaced read-only. Error signatures get pulled and correlated to a ticket or a customer report without leaving the investigation. |

### Data & verification

| Tool | What the AI layer adds |
|---|---|
| **SQL databases** (read-only) | The application's data layer. The agent runs guarded, read-only queries — schema-checked first, windowed, and row-capped — so a business question is answered against *real records* instead of a plausible guess. This is what lets a spec or a test cite numbers that reconcile. |

### Communication & workspace

| Tool | What the AI layer adds |
|---|---|
| **Microsoft 365** (Teams, SharePoint, Outlook, Planner, Excel) | The collaboration backbone. The agent posts cards to team channels, files deliverables to a document store, drafts mail, and reads task boards — so an artifact goes from *generated* to *delivered to the right audience* in one motion. |
| **Google Workspace** (Gmail, Calendar, Drive) | Secondary mail, scheduling, and document storage, connected for triage, drafting, and pulling context without a context switch. |
| **Fathom** | Meeting recorder with searchable transcripts. The agent pulls the actual transcript on demand, so "what did we decide?" is grounded in what was said — not reconstructed from memory. |

### Browser & UI automation

| Tool | What the AI layer adds |
|---|---|
| **Playwright** | End-to-end browser automation and UX testing. The agent authors and runs assertions at the *user-experience* level (by role and visible text, not brittle selectors) — turning "does this flow work?" into a repeatable, executable check. |
| **Browser dev-tools** | Live, authenticated interaction with the running application — logging in, exercising real screens, and verifying behavior directly against a test environment. |

### Delivery & platform

| Tool | What the AI layer adds |
|---|---|
| **SendGrid** | Transactional email delivery, surfaced read-only. The agent checks deliverability, suppressions, and per-message event history — so "did the customer actually receive it?" is answered from the provider's own event log. |
| **GitHub** | Source control and pull-request workflow, inspected and driven from the same session where the change was reasoned about. |
| **Claude Code** | The harness that ties all of the above together — the layer where every other tool becomes a single conversational surface, with permissioning, persistent memory, and the repeatable skills below on top. |

---

## Part 2 — The Skill Library

*Skills are codified `/commands` — repeatable procedures that chain the tools above, sub-agents, and a house style into one invocation. Where the toolstack is a set of capabilities, these are the workflows I run the same way every time.*

### Release operations
*A one-person release train — compile, protect, publish, announce.*

| Skill | What it does |
|---|---|
| **release-publish** | One command runs the whole release pipeline: compiles notes from Jira, applies structured metadata, converts to a stakeholder document, updates the index, files it to the document store, and posts to Teams. Turns a multi-hour, multi-tool ritual into a single invocation. (Full write-up: [Release Pipeline case study](../case-studies/01-release-pipeline.md).) |
| **release-seal** | Finalizes a drafted release — applies comment-only document protection, promotes the file to its published location, and posts the announcement *only after previewing it for explicit approval*. The human gate for the publish pipeline. |
| **release-announce** | Posts (or re-posts) a release announcement to Teams — either an already-published release or an upcoming release train pulled live from Jira. A thin, single-purpose wrapper. |
| **release-hub** | Search, browse, and query the release-notes index — keyword search, version detail, status overview. The read side of release ops. |
| **teams-card** | Posts an Adaptive Card to the correct team channel, auto-resolving the destination and payload envelope. The generic delivery primitive the release skills build on. |

### Session intelligence
*Persistent memory across sessions, so work never falls through the cracks.*

| Skill | What it does |
|---|---|
| **case-file** | Files a structured report of what a work session accomplished — decisions, deliverables, open items. The atomic record the rest of the system reads from. |
| **cold-cases** | Surfaces incomplete items from past case files that are aging past a threshold — an automated "what am I forgetting?" |
| **sync-cases** | Pushes open items from session notes onto a Jira board — creating tickets, setting fields, aging them, and de-duplicating via a manifest. Bridges private notes to the team system of record. |
| **binder** | Synthesizes an accomplishments review from recent case files (by date range or project) — a self-assembling status / brag document. |
| **wrap** | End-of-session orchestrator — runs `case-file`, conditionally `sync-cases`, writes a forward-looking handoff note, and recommends how to reset context. One command that closes the loop. |

### Meeting & standup intelligence
*Transcript → structured brief, in a chosen voice.*

| Skill | What it does |
|---|---|
| **extract-transcripts** | Pulls raw transcripts from a meeting recorder, web URLs, or local files into one canonical format — the source-agnostic front door the summary skills consume. |
| **summarize-meeting** | Turns any transcript into a structured, sectioned brief, with optional persona overlays for stakeholder-facing tone. |
| **standup-review** | Cross-references a standup against the Jira pipeline, flags gaps, and tracks release alignment. |
| **persona aliases** | Named-persona overlays — a strategic / narrative lens and a sharp-review voice — layered over the summary and standup engines. Same engine, different reader. |

### QA & test automation
*A QA team-in-a-box for a Product Owner without one. (Full write-up: [QA & Troubleshooting case study](../case-studies/02-qa-and-troubleshooting.md).)*

| Skill | What it does |
|---|---|
| **bdd-author** | Authors plain-English Gherkin scenarios from Jira tickets or requirements docs in a consistent house style. Produces the input for the test runner. |
| **qa-run** | Executes those scenarios against a real browser and compiles pass/fail results with screenshot evidence. **This is the orchestrator** — it fans out to specialized sub-agents in parallel: |
| ↳ *data verifier* | Read-only database assertions — "does the record actually say what the screen claims?" |
| ↳ *document verifier* | Confirms generated files (PDF, Excel, CSV) exist, are non-empty, and contain the expected content. |
| ↳ *error sentinel* | Runs alongside execution, watching console and network traffic for errors the happy path would miss. |
| ↳ *workflow runner* | Drives multi-step import workflows (upload → validate → import → verify). |
| ↳ *job watcher* | Polls the async job dashboard for completion and captures evidence. |

### Platform & learning

| Skill | What it does |
|---|---|
| **secondary ticket system** | Browse, create, update, comment on, and search a second internal ticketing surface — driven conversationally rather than through its UI. |
| **learn-playwright** | A resumable, Socratic curriculum for learning browser test automation as a Product Owner with no QA team — the skill that teaches the human, not just automates for them. |

---

## The patterns worth naming

The library is more interesting than any single command, because a few design patterns repeat — and they're the transferable part:

- **Supervisor-with-workers.** The test runner isn't one big prompt; it compiles a plan and spawns single-responsibility sub-agents (a data verifier, a document verifier, an error sentinel), several of them not directly user-invocable. That's how you build a resilient automation team, not a monolith.
- **Event-sourced memory.** Session intelligence is a producer/consumer loop: one skill writes atomic records, and several independent skills *read* over that same log. Decoupled, so a new reader is additive.
- **Aliases, not forks.** The persona voices are the same engine with a swapped tone layer — configuration, not duplicated logic — so an improvement lands everywhere at once.
- **Automate the assembly, not the judgment.** Every publish and release path ends at a human review gate. The automation does the busywork; a person owns the final word.
- **Verify, don't just collect.** A UI test that doesn't check the data layer is lying half the time; a release note that isn't checked against its fix state is a liability. The value is in confirming something is *true*, not just gathering it.

---

*Sanitization: public third-party tools are named (Jira, Confluence, Datadog, Rollbar, SendGrid, Microsoft 365, Google Workspace, Fathom, Playwright, GitHub, Claude Code). Employer-specific systems, customer and company names, internal identifiers, and proprietary data are withheld. The integrations and workflows are real; the proprietary data is not.*

— Nathaniel Genwright

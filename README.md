# AI Ops Case Studies

> Two analyst processes I turned into tools — documented as case studies: the problem, the build, and what changed.

I'm a Product Owner / technical business analyst. Across ~20 years in government and payments software, I've come to see most slow, manual work as **un-codified process** — so I codify it into tools that lift my throughput and accuracy. Here are two of those builds: the problem, what I built, how it works, and what it changed.

| Case study | The gist | Outcome |
|---|---|---|
| [Release Pipeline](case-studies/01-release-pipeline.md) | A one-command pipeline that assembles a software release — gathers its JIRA tickets, verifies each was actually fixed, drafts the notes, and routes a stakeholder document for human review. | Release prep **~12 hours → ~90 minutes** (~8×) |
| [AI-Driven QA & Troubleshooting](case-studies/02-qa-and-troubleshooting.md) | A multi-agent harness that runs plain-English test scenarios against a real browser, plus a repeatable loop for finding the *root cause* of a bug. | Repeatable pre-release verification + faster, evidence-backed bug investigation |

These were built with [Claude Code](https://claude.com/claude-code) (skills + the Model Context Protocol) as a **human-in-the-loop** system: the model proposes, I review and correct, and the corrections feed back into the prompts.

*These are sanitized patterns — public tools are named (JIRA, Microsoft Teams, Playwright, Claude Code); employer-specific data is withheld and all examples are synthetic ([details](docs/about.md)).*

**More:** [about & philosophy](docs/about.md) · [LinkedIn](https://www.linkedin.com/in/nathanielgenwright/)

— Nathaniel Genwright

# About this fork

This is a fork of [addyosmani/agent-skills](https://github.com/addyosmani/agent-skills). Everything upstream still applies; this file lists only what the fork adds and why.

## Why it exists

I ran the same take-home task (a feature on an inherited multi-tenant web app handling sensitive personal data) from scratch three times, once each with this pack, [mattpocock/skills](https://github.com/mattpocock/skills), and [obra/superpowers](https://github.com/obra/superpowers), then verified every result independently. This pack produced the most complete work, but the verification found gaps that the skills did not prevent. In some cases, the other two packs handled those cases better. The changes below close those gaps in general terms. Nothing here is specific to that task.

## What changed

| Gap found | Change | Where |
|---|---|---|
| A catch-all error handler logged whole errors; a malformed request body then put request data in the logs | Log errors by name, code and stack frames, never the message; reject malformed bodies with `400` at the boundary | `security-and-hardening`, `observability-and-instrumentation`, `api-and-interface-design`, both checklists |
| Audit records written after the change commits, by a helper that hid failures | Sensitive writes and their audit records commit in one transaction; read audits are best-effort but logged; lists record which ids were shown | `security-and-hardening`, security checklist |
| Tenant isolation lived only in queries | Tenant taken from the principal, 404 for other tenants, tenant-aware foreign keys, a second-tenant test per endpoint | `security-and-hardening`, security checklist |
| Some tokens were decoded but never verified (ID tokens at sign-in) | Verify every token where it is trusted; key-fetch failure is an outage, not a 401; debug shortcuts off unless explicitly dev/test; no auto-provisioning | `security-and-hardening`, security checklist |
| "Overdue" and "today" computed on the server's clock in every run | Calendar dates stored as dates, "today" computed in the customer's zone, tested with a zone that differs now | `api-and-interface-design` |
| Tests that could not fail, and race tests that passed by luck, in all three packs | Run red first and read the reason; mutate the guarded line; deterministic concurrency and time tests | `test-driven-development` |
| A defect class found once can be reintroduced by the next change or agent | Encode it as a static-analysis rule with its own test cases, proven against the commit that had the bug | `constraint-driven-development` (Step 8) |
| Inherited `AGENTS.md`/skills gave harmful instructions (trust unverified auth, skip tests, edit applied migrations) | Check rules files against the code and the non-negotiables before relying on them | `context-engineering` |
| The format gate stayed red in every run; one run left a commit that did not typecheck | CI runs the same aggregate check as developers, formatter included, no `continue-on-error`; every commit passes on its own | `ci-cd-and-automation`, `git-workflow-and-versioning` |
| Specs that only made sense next to the conversation that produced them | Fresh-context readiness review of the spec before approval (adapted from superpowers' design review) | `spec-driven-development` |
| One human-gate level for every task | Gate by risk: low-risk increments may run in batches, high-risk ones one at a time with approval and fresh-context review | `incremental-implementation` |
| Claims of "done" or "passing" without the run that showed it | Report the command and its real output (adapted from superpowers' verification-before-completion) | `using-agent-skills` |
| Write-ups that overstated what the code did | Fact-check every claim against the code and history | `documentation-and-adrs` |
| One model for every stage | Claude Code only: `/plan` runs on the latest Fable (`model: fable`); every other command and persona on the latest Opus (`model: opus`). Free-form turns outside a command use the session's model | `.claude/commands/*.md`, `agents/*.md` |

## Credit

- Logging errors by type rather than message, and promoting defect-signalling lint warnings to errors: from the mattpocock/skills run, the only one whose error handler did not leak.
- Spec readiness review by a fresh reader, and evidence before claims: adapted from obra/superpowers (`brainstorming`, `verification-before-completion`).

## Installing

```bash
claude plugin marketplace add MujiDipto/agent-skills
claude plugin install agent-skills@addy-agent-skills
```

The marketplace declares the plugin source as `./`, so it installs from whichever repository was added, this fork included. Upstream's manifest names `addyosmani/agent-skills` directly, so keep the fork's line if a sync conflicts there.

## Staying in sync

`upstream` points at addyosmani/agent-skills. To update: `git fetch upstream && git merge upstream/main`. The changes above are edits to existing skills, so conflicts, if any, are confined to the files listed above and the fork note at the top of `README.md`.

---
name: repo-doc-review
description: Review repository documentation for drift, contradictions, stale operational state, missing updates, and incorrect current-vs-historical framing. Use when code, configuration, runtime state, hostnames, paths, dependencies, runbooks, READMEs, current-status docs, TODOs, or reference material may disagree across a repository.
argument-hint: "[PR number or URL | branch | path | nothing for current changes]"
allowed-tools: Bash(git:*), Bash(gh:*), Read, Grep, Glob
license: MIT
metadata:
  author: choipotato
  version: "1.0"
---

# Review repository documentation for drift

Review documentation as a representation of the repository's current state.

This skill answers one question:

> Are the repository's current-state documents mutually consistent with the code, configuration, and repository facts they describe?

Do not turn this into prose/style review. Use `docs-review` for wording and style.
Do not decide whether a claim is sufficiently proven. Use `repo-evidence-review`
for evidence strength.

## Step 1. Resolve scope

Resolve `$ARGUMENTS` in this order:

1. A PR number or URL means review the changed files plus directly affected docs.
2. A branch means compare it with the merge base against the default branch.
3. A repository path means review that path and its direct references.
4. Empty with local changes means review the working-tree and branch diff.
5. Empty with a clean tree means review the current branch against the default
   branch.

Start narrow. Expand only when a changed fact is referenced elsewhere.

State the resolved target and changed-file count before reviewing.

## Step 2. Build the changed-facts set

From changed code, config, inventory, scripts, tests, and docs, extract only facts
that can make documentation stale. Typical examples:

- hostnames, service names, ports, paths, mount points, API names, commands
- defaults, dependencies, ordering, boot/shutdown sequences
- ownership, source/destination relationships, role assignments
- supported/unsupported states
- status transitions such as planned, active, paused, fixed, deprecated, removed
- validation requirements and gates
- renamed or deleted files, symbols, jobs, targets, or environments

Do not infer a fact merely because a nearby identifier changed. Read enough
context to know what actually changed.

## Step 3. Find documents that claim current state

Search for each changed fact and its prior value.

Prioritize documents that present themselves as current or normative:

1. current-status or runtime-state docs
2. runbooks and operating procedures
3. repository configuration documentation
4. README and onboarding docs
5. active TODO/plan/status indexes
6. generated/current inventories
7. historical references and incident records

Historical material is not stale merely because reality changed later.

Treat a document as historical when its purpose, path, title, date, or wording
clearly records a past observation. Do not rewrite historical evidence into
current truth.

## Step 4. Check four failure classes

### A. Stale current-state claim

A current or normative document still states an old value or old relationship.

Examples:
- old hostname after a canonical rename
- old mount dependency after role/config changed
- old command, port, path, or service name
- "paused" after the repo now records "active", or the reverse

### B. Cross-document contradiction

Two current-state documents disagree about the same fact.

Do not report a contradiction when one source is explicitly historical.

### C. Missing documentation update

The change modifies an externally relevant operational or repository contract
but no current-state document reflects it.

Do not demand documentation for every implementation detail. Report only changes
a maintainer or operator would reasonably rely on.

### D. Historical framing error

A historical document is presented or linked as if it were current, or a
current document cites historical evidence without preserving the date/context.

The fix is usually framing or linking, not rewriting the historical record.

## Step 5. Use a source-precedence rule

When sources disagree, prefer the source that most directly defines the current
repository state. Use this default order unless the repository defines another:

1. executable configuration, inventory, manifests, role defaults, and code that
   directly define behavior
2. current runtime/status records produced from recent verification
3. current tests and validation gates
4. current README/runbook/current-status prose
5. active plans and TODOs
6. historical references, incident records, archived outputs

This is a documentation-consistency order, not an evidence-strength order.
Do not promote an unverified runtime claim merely because it lives in a
`current/` directory; that is the job of `repo-evidence-review`.

## Step 6. Report only actionable findings

Group findings by class:

```text
STALE
path/to/doc.md:42
Claim: skaistor-03 has no /users dependency
Repository fact: role config mounts skaistor-01:/users
Suggested: update the dependency and the affected verification step

CONTRADICTION
current/runtime.md:18
README.md:73
The two current-state documents disagree on whether the service is paused.

MISSING
roles/storage/defaults/main.yml
The client mount contract changed, but no current runbook documents the new
preflight requirement.

HISTORICAL — LEAVE AS IS
reference/2026-10-03/measurements.md
This correctly records the observation as of 2026-10-03.
```

For every finding, name the repository fact that makes it actionable.

End with one verdict:
- `No documentation drift found.`
- `Documentation drift found; current-state docs need updates.`
- `Only historical differences found; no current-state update required.`

Do not edit files unless the user explicitly asks for fixes.

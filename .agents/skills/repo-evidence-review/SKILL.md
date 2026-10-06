---
name: repo-evidence-review
description: Review repository claims against the strongest available evidence and flag conclusions that are unsupported, overclaimed, underqualified, or not reproducible. Use for claims such as fixed, verified, root cause, production-ready, passed, failed, unsupported, safe, complete, or caused by, especially in PRs, runtime notes, incident docs, validation reports, and technical conclusions.
argument-hint: "[PR number or URL | branch | path | claim]"
allowed-tools: Bash(git:*), Bash(gh:*), Bash(*), Read, Grep, Glob
license: MIT
metadata:
  author: choipotato
  version: "1.0"
---

# Review repository claims against evidence

This skill answers one question:

> Does the available evidence justify the strength of each important claim?

Do not perform general documentation cleanup. Use `repo-doc-review` for drift
and cross-document consistency.

## Step 1. Identify claims that require proof

Review only claims whose truth matters technically or operationally.

Prioritize language such as:

- fixed, resolved, verified, validated, passed
- root cause, caused by, due to, proves
- safe, production-ready, supported, unsupported
- complete, exhaustive, no longer occurs
- regression, incompatible, deterministic
- always, never, only, all, none

Also review weaker wording when it still implies a conclusion, for example
"this confirms" or "the failure comes from".

Do not waste time proving obvious repository metadata such as a filename that is
visible directly in the same diff.

## Step 2. Separate observation from conclusion

Rewrite each reviewed statement internally as:

```text
Observation:
What was directly seen, measured, executed, or read.

Conclusion:
What the document says that observation means.
```

A finding exists when the conclusion is stronger than the observation supports.

Example:

```text
Observation:
One control run exited 134 with malloc_consolidate().

Conclusion:
The framing implementation causes the allocator abort.
```

The observation does not establish the conclusion unless a comparison excludes
other causes.

## Step 3. Use the strongest evidence available

Prefer evidence in this order, stopping when the claim is adequately settled:

1. direct reproduction of the relevant behavior under the stated conditions
2. controlled comparison that changes the suspected variable
3. captured runtime output, logs, traces, measurements, or artifacts
4. tests that exercise the exact claimed path
5. executable configuration or source code that directly defines the behavior
6. installed-package or built-artifact inspection
7. current repository documentation
8. historical notes or prior reports
9. model inference, naming, comments, or recollection

The ranking is contextual. Source code can settle a static default better than a
runtime log; a runtime experiment is stronger for behavior that depends on the
environment.

Prefer evidence produced or inspected in the current review session when
possible.

## Step 4. Verify the claim, not a nearby proxy

Check the path that actually establishes the statement.

Examples:

- A successful config parse does not prove service reachability.
- A unit test does not prove the deployed binary used the tested code.
- A source branch showing a default does not prove a production override is
  absent.
- One failure mode disappearing does not prove the root cause was fixed.
- A single successful run does not prove "always".
- A successful read does not prove write safety.
- A test harness result does not automatically prove stock-client behavior if
  the harness changes framing, buffering, timing, or environment.

When a claim depends on multiple conditions, verify each material condition.

## Step 5. Classify the result

Use one of these labels:

- **VERIFIED** — evidence directly supports the claim at its stated strength.
- **SUPPORTED WITH LIMITS** — evidence supports a narrower or conditional claim.
- **NOT SUPPORTED** — available evidence does not justify the conclusion.
- **CONTRADICTED** — stronger evidence shows the claim is false.
- **NOT VERIFIED** — verification was not possible from available evidence.

Do not convert `NOT VERIFIED` into `NOT SUPPORTED` unless the claim actually
lacks support. Absence of access is not disproof.

## Step 6. Watch for common overclaims

### Root-cause overclaim
A correlation, one crash, or one changed variable is presented as causation.

Require either a controlled comparison or evidence that excludes plausible
alternatives.

### Completion overclaim
A partial validation is described as complete.

Name the untested dimensions: environment, version, path, load, failure mode,
rollback, restore, restart, or deployment.

### Verification overclaim
A document says "verified" without saying what command, artifact, test, or source
was checked.

Require a reproducible pointer when practical.

### Scope overclaim
Evidence from one host, one version, one branch, or one input is generalized to
all of them.

Narrow the claim to the observed scope unless broader evidence exists.

### Negative-proof overclaim
"No issue", "cannot happen", or "none" is asserted from a search or test that was
not exhaustive.

State what was searched or tested instead.

## Step 7. Record the evidence trail

For each finding, report:

```text
path/to/report.md:61
Claim: framing candidate causes malloc_consolidate() abort
Verdict: NOT SUPPORTED
Evidence:
- candidate run: exit 134
- control run: exit 134 also observed
Why: the control does not isolate framing as the cause
Suggested claim:
"An exit-134 allocator abort was observed in both candidate and control runs;
the cause remains unresolved."
```

For verified claims, keep the report compact:

```text
VERIFIED
Claim: stock client performs a single recv(10) for the header
Evidence: client source + captured 3-byte return in the test harness
```

Name exact files, commands, symbols, tests, logs, or artifacts whenever they are
available.

## Step 8. Preserve uncertainty

When evidence is incomplete, say exactly what remains unknown.

Good:
`NOT VERIFIED: production restart behavior was not exercised.`

Bad:
`Probably fine.`

Do not invent a verification step, successful run, log, or source inspection
that did not happen.

End with a one-line verdict:
- `Claims are supported at their stated strength.`
- `Some claims need narrower wording or additional evidence.`
- `Material claims are contradicted by available evidence.`

Do not edit files or run destructive/production-changing commands unless the user
explicitly asks.

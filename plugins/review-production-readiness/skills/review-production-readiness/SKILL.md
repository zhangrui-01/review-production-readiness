---
name: review-production-readiness
description: Audit application code and deployment artifacts for production readiness before release. Use when asked whether code that works in development will run reliably in production; for pre-release, go-live, or production-readiness reviews; or to inspect exception and boundary handling, hard-coded configuration, 10x/100x scalability, new third-party dependencies, runtime parity, migrations, resilience, observability, security, and rollback readiness. Default to a read-only, evidence-backed review; implement fixes only when explicitly requested.
---

# Review Production Readiness

## Set the review contract

Treat production readiness as an evidence question, not a promise that code is bug-free.

- Default to reviewing and reporting. Modify code only when the user explicitly requests fixes.
- Preserve unrelated changes and the user's dirty worktree.
- Never print secrets or credential values. Report only the secret's location and redact its value.
- Anchor every finding to concrete evidence such as `file:line`, a manifest entry, a command result, or an observed configuration mismatch.
- Separate confirmed defects, plausible risks, and unverified assumptions. Do not present a hypothetical failure as a confirmed bug.
- Prefer the smallest safe remediation. Do not recommend complexity without a demonstrated risk.
- Continue with a bounded static review when production details are unavailable; list the missing facts that prevent stronger conclusions.

## Establish scope and baseline

1. Read repository instructions first, including `AGENTS.md` and relevant local guidance.
2. Identify languages, frameworks, entry points, build system, lockfiles, data stores, queues, external services, deployment artifacts, and test commands.
3. Inspect source code together with deployment inputs: container files, CI workflows, infrastructure configuration, migrations, process definitions, and environment templates.
4. Determine the review baseline. If the request concerns a change, compare against the requested base branch or commit. Otherwise review the current tree and state that the scope is repository-wide.
5. Determine the production contract where evidence exists: runtime and OS versions, CPU architecture, artifact format, start command, permissions, traffic or data baseline, availability expectations, and rollout method.
6. Ask a question only when the missing answer would materially change the review. Otherwise proceed and mark the item as a validation gap.

## Review in eight passes

### 1. Verify runtime and environment parity

Start here because environment drift is a common cause of development-only success.

- Compare runtime, package-manager, framework, OS, architecture, and native-library versions across local setup, CI, container images, and production manifests.
- Check that builds use lockfiles and reproducible commands, and that the produced artifact is the artifact actually deployed.
- Trace the real startup command, working directory, ports, health checks, readiness behavior, graceful shutdown, and signal handling.
- Check filesystem case sensitivity, writable and ephemeral paths, user permissions, locale, encoding, timezone, clocks, proxies, DNS, TLS, and network egress assumptions.
- Compare configuration *names, types, defaults, and precedence* without exposing values. Ensure required configuration fails fast with a useful error.
- Look for development-only services, mocks, local paths, implicit credentials, or tools that are absent from the production artifact.

### 2. Inspect exceptions and boundaries

Trace important paths from input through state changes and outputs, including failure paths.

- Test reasoning against null, empty, malformed, oversized, duplicate, out-of-order, and unauthorized input.
- Check numerical limits, overflow, rounding, pagination boundaries, Unicode, dates, daylight-saving changes, and timezones where relevant.
- Inspect exception scopes, swallowed errors, catch-all handlers, fail-open behavior, partial writes, cleanup, and asynchronous errors.
- Check transaction boundaries, concurrency races, locking, idempotency, repeated delivery, and retries after an ambiguous result.
- For every external call, check explicit timeouts, bounded retries with backoff and jitter, rate-limit handling, cancellation, connection reuse, and resource release.
- Confirm error responses and logs preserve useful context without leaking credentials, tokens, personal data, or sensitive payloads.

### 3. Audit hard-coded values and configuration

Search source and deployment files for endpoints, paths, ports, regions, tenant IDs, credentials, timeouts, retry counts, concurrency limits, batch sizes, feature flags, and capacity thresholds.

- Classify a value before changing it. Make environment-specific or operational values configurable.
- Keep true protocol constants, mathematical constants, and stable business invariants in code unless operators genuinely need to change them.
- Prefer typed, centrally validated configuration with documented units, safe defaults, and clear precedence.
- Require secrets from an approved secret mechanism; never suggest committing them to configuration files.
- Flag configuration that silently falls back to an unsafe production value.
- Avoid converting every literal into configuration. Excess configuration creates invalid state combinations and operational risk.

### 4. Model 10x and 100x scale

Quantify the likely bottleneck instead of saying only that code “may not scale.”

- Establish a baseline from tests, telemetry, fixtures, documentation, or an explicitly labeled assumption.
- Estimate per-request or per-job CPU, memory, database queries, network calls, payload bytes, temporary storage, and retained storage.
- Inspect algorithmic complexity, nested loops, unbounded collections, full-table scans, N+1 I/O, repeated serialization, large fan-out, and synchronous work on request paths.
- Inspect connection pools, worker pools, queue depth, backpressure, cache growth, eviction, hot keys, locks, rate limits, and downstream quotas.
- Calculate how the baseline changes at 10x and 100x. Identify nonlinear cliffs such as memory exhaustion, pool saturation, retry storms, lock contention, or index degradation.
- Distinguish throughput, concurrency, and total data growth; they stress different resources.
- Recommend a bounded load or volume test with a measurable pass condition when static evidence is insufficient.

### 5. Review dependency changes and supply-chain risk

- Diff manifests and lockfiles against the baseline to identify new or upgraded direct and transitive dependencies.
- Explain why each new direct dependency is needed and whether the standard library or an existing dependency already provides the capability safely.
- Evaluate runtime and build footprint, duplicate functionality, native extensions, platform support, cold-start cost, transitive size, install scripts, and operational complexity.
- Check version pinning, lockfile integrity, known vulnerabilities, maintenance activity, release provenance, and license compatibility using authoritative current sources when available.
- Do not recommend replacement merely because another library is smaller. Weigh correctness, maintenance burden, security, compatibility, and migration cost.
- Mark vulnerability or maintenance claims as unverified if they were not checked against current authoritative data.

### 6. Review state changes and deployment compatibility

- Check database and schema migrations for backward compatibility with both old and new application versions during rolling deployment.
- Prefer expand-migrate-contract sequencing for incompatible data changes.
- Check migration duration, locking, retries, resumability, destructive operations, data backfills, backup expectations, and rollback behavior.
- Check events, caches, APIs, and serialized data for version compatibility and mixed-version operation.
- Confirm repeated deployment, restart, or job execution is safe and idempotent.

### 7. Review resilience and operability

- Check behavior when each critical dependency is slow, unavailable, rate-limited, or returns malformed data.
- Look for bulkheads, circuit breaking, bounded queues, load shedding, degradation paths, and recovery behavior where the risk warrants them.
- Confirm structured logs, metrics, traces, and alerts cover critical paths and failure modes with useful correlation identifiers.
- Check liveness and readiness semantics; a process being alive does not mean it is ready to serve traffic.
- Check resource limits, autoscaling inputs, graceful termination windows, scheduled jobs, and singleton assumptions.
- Confirm the release has a rollback or forward-fix plan, feature-flag strategy where appropriate, and clear abort signals for a canary or staged rollout.

### 8. Gather executable evidence

Use existing repository commands before inventing new ones. Run only safe, bounded checks within the authorized environment.

- Perform a clean, lockfile-respecting build using the production build path when practical.
- Run relevant type checks, linters, unit tests, integration tests, startup smoke tests, and migration validation.
- Test missing, malformed, and boundary configuration without displaying real values.
- Build and start the production container or artifact locally when practical, including a non-root execution check if applicable.
- Run dependency audit tools and a targeted load or volume test when available and proportionate to the request.
- Do not touch production systems, publish artifacts, mutate external state, or run unbounded stress tests without explicit authorization.
- Record commands run, results, skipped checks, and the reason each skipped check could not be performed.

## Prioritize findings

Assign severity from production impact and likelihood; assign confidence separately.

- **P0 — Critical:** Active credential exposure, destructive corruption, or a near-certain catastrophic production failure requiring immediate action.
- **P1 — High:** Likely release blocker such as startup failure, incompatible migration, unbounded resource exhaustion, or serious security failure.
- **P2 — Medium:** Material reliability, performance, security, or operability risk that should be scheduled promptly.
- **P3 — Low:** Hardening or maintainability improvement with limited immediate production impact.

Use `high`, `medium`, or `low` confidence. Lower confidence rather than inflating severity when evidence is incomplete.

## Report the result

Lead with one of these bounded verdicts:

- **Block release:** At least one unresolved P0 or P1 finding has sufficient evidence.
- **Conditionally ready:** No proven blocker, but named validation gaps or P2 risks require explicit acceptance.
- **No blocker found:** The performed checks found no release blocker. Never translate this into a guarantee of production safety.

For each finding, include:

```text
[P1][high confidence] Short title
Location: path/to/file:line
Evidence: What the code, configuration, or command actually shows
Trigger: The production condition that activates the problem
Impact: The user, data, availability, security, or cost consequence
Remediation: The smallest safe change
Verification: The test or observable result that proves the change
```

Then report, in this order:

1. Verdict and release blockers.
2. Other findings, ordered by severity.
3. A 10x/100x scale table with baseline, limiting resource, expected failure mode, and proposed validation.
4. New or changed dependency summary and justified alternatives, if any.
5. Checks performed and their results.
6. Validation gaps, assumptions, and concrete release gates.

If no findings exist, say so explicitly and still list the inspected scope and unperformed checks. Do not manufacture issues to fill the report.

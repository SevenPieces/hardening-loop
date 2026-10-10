# Verification coverage

All verification works to this file: the baseline verification that opens the
loop and each verification step traverse the categories below. `SKILL.md` defines
the steps; this file defines what they must reach. Phase 5's adversarial sweep is
additional to them rather than a second traversal: it attacks the same target for
assumption challenges no category names, and a category result from a
verification step is not reused as a sweep result. Phase 2's coverage matrix is
built from here, and the Gate reads the exit criteria against it: an area left
unexamined is a gap to state, not a silence to keep.

Coverage is grouped into three coverage passes, and regression plus the
production workload close it:

1. **Behaviour** - what the target does.
2. **Non-functional** - the properties it must hold while doing it.
3. **Lifecycle** - how it behaves over time, across its interfaces, and for the
   people who maintain it.

A unit or integration test closes a change-set row. It never closes the
production-mode requirement. A deterministic subset - a pipeline's parsers,
gates and renderers run directly - closes change-set rows only, and never a Pass
1 category; when the production entry point cannot run at all, the subset run is
evidence for the change-set block, and the unclosed Pass 1 category is recorded
as a gap. The real workload closes Pass 1's functional-correctness category and
Pass 3's production-isolation category. In the baseline verification the
production workload is deliberately absent; functional correctness and production
isolation are recorded there as deferred to the first verification step after the
baseline verification, not as open items. The production workload is part of
every post-baseline traversal: it runs as part of every verification step over a
batch, and in the Gate's extra pass (every time that pass runs).

Regression is the closing check, not a fourth pass: everything that worked
before still works.

Every category is either covered, marked not applicable *with the missing
surface named*, or recorded as an open item - except in the baseline verification,
where the two categories the production workload closes are recorded as deferred
to the first verification step after the baseline verification rather than as open
items. When no batch lands, the Gate's extra pass is that step and closes the
deferral; the Gate may not open with a category still recorded as deferred. "Not
applicable" is a claim that needs a reason; the "applies when" column is what
decides it.

## Absent behaviour

An absent capability is a result, not a failure. Where a category asks for
behaviour the target does not have - no retry, no timeout, no cadence - record
the absence and what you looked for, and that closes the category. It passes
unless the absence is itself a finding, as an untimed network call is, and then
it is classified by its impact like any other finding.

## Pass 1 - Behaviour

| Category | Applies when | What closes it |
|---|---|---|
| Functional correctness | always | every documented input/output path exercised, with the observed result recorded |
| Edge cases | the target takes input | each named input class covered at least once: empty, missing, malformed, boundary, oversized, duplicated, partial, and the locale or time-zone variants the target claims |
| Failure modes | the target has a dependency, IO or external call | each failure induced at least once - timeout, rate limit, network loss, partial response, corrupt state, interrupted write - with the recovery, retry, rollback and idempotency behaviour recorded |
| Reliability | the target runs concurrently or repeatedly | a concurrent and a repeated execution of the target; races, ordering and non-determinism either absent or bounded and recorded - bounded means every varying field is named and none of them changes a documented output beyond that execution's own metadata |
| State and data integrity | the target owns state, a schema, files or a dataset | existing state survives the change, the change is shown to be reversible, and nothing is silently dropped on the forward or the backward path |

## Pass 2 - Non-functional

| Category | Applies when | What closes it |
|---|---|---|
| Security and privacy | always | the target, every diff, every read-only input, the input findings files, and every delegate's returned evidence and report text scanned for secrets, credentials, PII, internal URLs and private data; injection surfaces and unsafe logging checked |
| Performance and resources | the target has a runtime cost or writes files | a measured execution - wall time, peak memory, temp-file count, and storage delta across consecutive executions - reported as a measurement; a budget varies too much between targets to be fixed here, so state one only when the invocation supplied it, through `constraints=`, and say so |
| Compatibility | the target declares versions, platforms or toolchains | the declared combinations are exercised rather than read; a declared platform or version that cannot be exercised in this environment is recorded as a gap, not counted as covered |

## Pass 3 - Lifecycle

| Category | Applies when | What closes it |
|---|---|---|
| Integration | the target talks to anything else - a dependency, config, API, storage or scheduler | each interface exercised end to end, including a missing or misconfigured configuration |
| Runtime and repeated-execution behaviour | the target keeps state or runs on a cadence | consecutive and, where applicable, scheduled executions: accumulation, drift, resumption after interruption, and cleanup between executions |
| Observability | the target logs, reports or exposes status | a failure induced and then diagnosed from the logs, metrics or error output alone, without reading the code |
| Documentation and maintainability | always | a reader holding only the README performs the primary task; every documented command, link and config example resolves; the change leaves no dead code, dead config or orphaned state |
| Production isolation | the target writes anywhere outside itself, or has delivered artifacts, schedules or live state | artifacts shown untouched, every write outside the target's own tree accounted for, no leftovers remaining, and no schedule or external trigger changed or fired |

## Regression

| Category | Applies when | What closes it |
|---|---|---|
| Regression | always | the recorded baseline still holds - its behaviours re-run, or its manifest re-checked - with the comparison recorded |

## Sampling: partitions, not instances

Cover classes, not instances. For each category, name the partition - the input
classes, states, environments or timings that exist - and cover every class at
least once. Testing many instances of one class is not coverage; testing one
instance of each class is. Where a class is deliberately left uncovered, name it
and say why.

## Choosing a method

Match the technique to the claim instead of defaulting to the easiest one:

| The claim is about | The technique that tests it |
|---|---|
| data, formats, output shape | property-based, or golden/snapshot comparison |
| input handling and robustness | fuzz or adversarial input |
| state over time or across executions | repeated and resumption executions |
| behaviour under failure | fault injection |
| trust, exposure, boundaries | adversarial review plus a secret and leakage scan |
| change safety | regression tests pinned to the finding IDs |
| documentation and usability | a from-the-docs walkthrough by someone who has not read the code |

Unit, integration, end-to-end, regression, snapshot or golden-file,
property-based, fuzz, adversarial and manual review are all available. Say which
were used, which were not, and why.

## Change-set coverage

The diff is verified as well as the target. Every behavioural change needs a
targeted check that exercises it, and **no behavioural change is verified by
reading alone** - the evidence is an execution, not a review. A documentation-only or
comment-only change is verified by a rendering, link and example check instead,
and is recorded as such.

Change-set results are reported as an additional block in the same matrix as the
categories: one row per changed behaviour, closed by the check that exercised it.
The block is never omitted: before any change exists - after the baseline
verification or a mode A scope scan - it records that there is nothing to verify
yet, and from the first batch onward it carries one row per changed behaviour.

## Recording the result

Coverage is reported as the matrix in `reference/report-template.md` §16:
category, covered by, technique, outcome, closing evidence. Negative results
count - an area attacked that did not break still records the attempt and what
was tried, so the effort is visible and can be judged. Testing only the input
findings is not verification: re-verify the whole target against this file.

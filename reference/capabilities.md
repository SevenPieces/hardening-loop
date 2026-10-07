# Conditional capabilities

Detected in Phase 0. A capability that is **present** makes its "if present"
column mandatory; an **absent** one uses the fallback, and the run records the
absence, the fallback it used and why in the capability declaration (report §3) -
and, for the methodology framework and delegation, also in the methodology and
delegation logs (report §4, §5).

Detection is a fact to establish, not an assumption. Check the harness or the
target for the capability; do not infer it from the name of the tool you happen
to be holding.

| Capability | Detection | If present | Fallback |
|---|---|---|---|
| Methodology framework (for example Superpowers) | A skill index or skill list is available | Enumerate it; map skills to the steps they serve; invoke them; log the evidence; no-skip rule applies | Follow the built-in process in `SKILL.md`; log the unavailability |
| Subagents / delegation | A delegation API is available | Delegation-first; independent reviewer and verifier roles kept apart from implementation; log the evidence | The orchestrator (the main agent) performs those roles in separate independent passes; log why |
| Production / real-workload mode | A production-mode entry point exists | Run an isolated production-like workload as part of every verification step | Run the closest representative workload, in isolation |
| Delivered / published artifacts | Artifacts shipped to a consumer outside the repository - a release, a package, a published remote or registry entry, a deployed endpoint | Treat it as read-only; never disturb it | Mark not applicable |
| Reference artifacts | One or more paths or directories were provided at invocation | Read-only evidence for triage and reproduction | Build sanitized fixtures from samples |
| Input findings | One or more files, or a directory of them, was provided and the resolved mode is findings-driven | Triage them as the minimum backlog - mode B still includes mode A's scope | Baseline scope comes from reconnaissance |
| Test suite / runner | Tests exist for the target | Extend them with the regression tests | Add a runner if feasible; otherwise justify the absence and record it as an open item |

## Phase 0 probe sequence

Record the baseline identity first (see `SKILL.md` Phase 0). Then run every
probe below, before any other work, and emit the declaration table and carry it
into the report. A probe is a check performed with the tools the
session already has: none of them assumes a particular harness, and no probe is
meant to change the target - one that runs something keeps its output, caches
and coverage outside the target.

| # | Capability | Probe (what the agent does) | Present when | If the probe is inconclusive |
|---|---|---|---|---|
| 1 | Methodology framework | List the skills this session exposes. The list the harness dispatches from is authoritative; a skills-root listing on disk is corroboration. | At least one skill other than this one is listed. | Re-check once; if it is still unreadable, treat as absent and record the failure. When the two sources disagree in count or content, record both and note the discrepancy. |
| 2 | Subagents / delegation | Check whether the session exposes a delegation tool. | The session exposes a delegation tool. | Record the error and treat as absent; independence falls back to separate passes. |
| 3 | Production / real-workload mode | Look for a documented way to run the target on real inputs - README, Makefile, package scripts, CLI help, service unit. Read-only probes may send output to stdout or to a scratch path outside the target. | A documented entry point exists and starts. | Treat as absent; run the closest representative workload and say which. If the target's real workload is not runnable at all - a procedure or a document - record absent and name the exercise that stands in for it. |
| 4 | Delivered / published artifacts | Look for artifacts shipped to a consumer outside the repository - a release, a package, a published remote or registry entry, a deployed endpoint, a published output directory. Files merely tracked in the repo, including committed examples and samples, are fixtures. | Any such artifact exists. | **Treat as present** - do not write anywhere near it - and record that the check was inconclusive. |
| 5 | Reference artifacts | Read the `refs=` field of the invocation; check that every path exists and is readable. A directory contributes its contents, recursively. | At least one path is readable. | Treat the paths that could not be read as none for this run and record which; the readable ones remain evidence. Build sanitized fixtures from samples only when nothing was readable. |
| 6 | Input findings | Read the `findings=` field; open each path - a directory contributes its `.md` files - and count the findings. | At least one path is readable and holds at least one finding, and the invocation resolved to findings-driven. | **Stop and ask** - it decides the mode, and this case is not covered by the mode default. A named path that will not open, or a directory that yields no findings file, is what makes this row inconclusive. |
| 7 | Test suite / runner | Find the test entry point (test script, tests directory, CI config) and run it once, keeping any output, cache or coverage it produces outside the target. Phase 0 records only whether a runner exists and what it reported. | A runnable suite exists and produces a result. | Treat as absent and record it as an open item; whether to add a runner is a Phase 1 decision, not a Phase 0 one. |

Note on row 6: a run resolved to baseline never declares this capability present,
whatever paths were supplied - an explicit `mode=baseline` together with readable
findings paths is the contradiction `SKILL.md` resolves with the operator before
Phase 0.

Probes cost time, not workers: none of them is meant to leave anything behind in
the target, and none dispatches a subagent - the delegation probe only checks
that a delegation tool is exposed.

Then emit the capability declaration - one row per capability, in this form:

| Capability | Probe run | Result | Behavior used | Evidence |
|---|---|---|---|---|
| <name> | <the check from the table above> | present / present-unexercised / absent / inconclusive | the mandatory behavior, or the fallback | the command or output that decided it |

Rules for the declaration:

- Every row carries evidence. "Assumed present" is not evidence.
- An inconclusive row takes the default or escalation in the last column of the
  probe table, never a guess in the convenient direction.
- The declaration is what makes the capability claims in the report checkable,
  and it is part of the process-compliance check at the Gate.
- For delegation, at declaration time no dispatch has occurred, so record the
  result as `present-unexercised` - exposed, but dispatch not exercised. Do not
  upgrade it to a corroborated `present` without a dispatch that succeeded; when
  the plan delegates and a dispatch succeeds, upgrade the row then, and say so in
  the report. A plan that ends up delegating nothing while delegation is present
  is a process-compliance finding, not a lingering `present-unexercised` row.

## Rules that follow from detection

**Default framework.** When the Superpowers skill set is available it is the
default methodology framework; another set is used only when Superpowers is
absent. Skills named by the run's `skills=` parameter are added to it, mapped to
the steps they serve, and logged like any other.

**No-skip rule.** When a methodology framework is present, every skill it maps
to a step must be invoked, or the skip is recorded as an explicit decision and
mapped to the reason. A skipped skill is a process-compliance finding until
remediated. Invoking means invoking: naming a skill, or paraphrasing its
guidance from memory, does not satisfy this rule.

**Delegation roles.** When delegation is present, plan these roles. The role
names are negotiable; the independence they exist to create is not.

**Independence constraints.** With delegation present, implementer is not
verifier, not test designer and not adversary: a change the orchestrator
implemented is verified by another party, under the context isolation below.
With delegation absent, independence degrades to separate passes - the
orchestrator re-reads the target from disk and verifies it in a pass separate
from the implementation - and the degradation is recorded.

| Role | Runs in | Owns |
|---|---|---|
| Triage / reproduction | Phase 3 | restating each input finding, reproducing it in isolation, capturing the evidence |
| Implementation | Phase 4 | the fixes and the hardening changes |
| Independent test design | Phase 4 | the regression tests, and the coverage matrix that Phase 2 planned |
| Adversarial review | Phase 5 | logic, robustness, abuse cases, boundary and failure behaviour |
| Security / privacy scan | Phases 4-5 | secrets, credentials, PII, internal URLs, injection surfaces, unsafe logging, leakage |
| Verification | Phases 4 and 6 | the baseline verification, each verification step, and the Gate's extra pass - the isolated production-like executions and the repeated-execution simulation |
| Regression and cross-effects | Phases 4 and 6 | the recorded baseline still holds at every verification step, and nothing outside the changed area moved |

Phase 5's adversarial sweep is staffed by the Adversarial review role: "the
adversary" below means that role, and "the verifier" means the Verification role.

**Smallest independent set.** Implementation plus verification, always.
Adversarial review runs on every run, staffed by a role separate from
implementation; it must additionally be an independently staffed role whenever
the change touches security, data integrity or external input. Independent test
design likewise runs wherever a regression test is required. The
independence-bearing roles - verification, independent test
design and adversarial review - never merge into implementation; triage, the
security/privacy scan, independent test design and regression may merge into each
other, into verification or into the adversarial review. Every merge is recorded
with its reason, per the delegation contract below.

**Delegation contract.** Give each role a scope, its inputs, its deliverables,
the evidence it must return and its constraints - including the read-only and
sensitivity rules for anything it touches. A role merges into another only when
the run is too small to split it, and the merge is recorded with its reason.

**Context isolation.** A verifier or adversary receives the target, the change
and the exit criteria - never the implementer's notes, hunches or
self-assessment. Use an agent with no inherited context: a verifier that
continues the implementer's conversation inherits its blind spots, and the
independence is only nominal.

**Adversary brief.** The adversary is told to break it, on the assumption that
the change is wrong, and returns either the input that proves it or the attacks
that failed. Failed attempts are evidence and are recorded.

**Orchestrator reservations.** Integration, conflict resolution, the final
quality judgement and the Gate stay with the orchestrator, and nothing is
delegated that the orchestrator cannot itself verify. A delegate's success claim
is checked against the evidence it returned - a conclusion without its evidence
is a finding, not an input.

**Writer discipline.** Concurrent delegates take disjoint write scopes. Anything
sharing a mutable path - a state file, a fixture directory, the work branch's
index - is serialized or split so that only one writer touches it.

**Failed delegation.** A delegate that fails, stalls or returns nothing is
re-delegated or logged as an open item. The orchestrator does not silently
absorb the work and continue as though the role had been exercised.

**Parallelism.** Independent workstreams run concurrently where that is safe.
Anything sharing mutable state, or that must observe another role's output, runs
in order.

**No-solo rule.** A role in the plan that was neither delegated nor explicitly
merged is a process-compliance finding, exactly like a skipped skill.

**Independence rule.** The constraints above hold whenever delegation is
present. When it is absent, the baseline verification and each verification step
run as separate passes with the target re-read from disk, not as a re-reading of
the implementation notes.

**Read-only rule.** Reference artifacts and delivered artifacts are inputs, not
workspaces. Nothing is written to them, and nothing from them is copied into
the target, tests, fixtures, commits or reports without sanitization.

**Evidence rule.** Every capability decision - detected, absent, or used as a
fallback - appears in the report with the command, tool output or observation
that established it.

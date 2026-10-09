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
| Subagents / delegation | A delegation API is available | Delegation-first; every role is staffed by its own agent, and no agent holds more than one role; log the evidence | When delegation is unavailable, the declaration records each role as a self-held degradation with its reason, and escalates under Stop conditions; no role is closed by a separate pass or by any agent holding another role; log which |
| Production / real-workload mode | Look for a documented way to run the target on real inputs - README, Makefile, package scripts, CLI help, service unit. **The probe records the exact command line and the exact input set a real run would use, and they must be executable by this session as written; "there is a real entry point" is not the probe result.** Read-only probes may send output to stdout or to a scratch path outside the target. | **If present:** Run the target's own entry point on its real inputs, with its real dependencies, in an instance isolated from production (its state and inputs copied under a scratch output directory), as part of every verification step over a batch and once more in the Gate's extra pass. Production-like means the real path, not a subset of it. | **Fallback:** only when the entry point cannot run unrestricted. Name the constraint, name what it removes, and run the same entry point reduced only in inputs - never a different path, a test suite or a representative subset - and declare that constrained run `present`. If it cannot run at all, declare the row `inconclusive`, name the blocking constraint, and escalate under Stop conditions. |
| Goal tracking | The harness exposes a goal primitive (create / get / update goal) | Arm the goal from Phase 1 before Phase 2 and keep it active until the Gate; record its id and final state in the report | The goal lives in the report's section 1 goal record (the objective, its id when one exists, and its final state); sections 4 and 5 log methodology and delegation, not the goal - log the unavailability of the primitive in section 1 |
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
| 2 | Subagents / delegation | Check whether the session exposes a delegation tool. | The session exposes a delegation tool. | Record the error and treat as absent; every role becomes a self-held degradation with a Gate compliance finding, and the run escalates under Stop conditions. |
| 3 | Production / real-workload mode | Look for a documented way to run the target on real inputs - README, Makefile, package scripts, CLI help, service unit. Read-only probes may send output to stdout or to a scratch path outside the target. | A documented entry point exists, starts, and the run can state the exact command and the inputs it would execute with. | If a named constraint permits a restricted run of the same entry point, declare that constrained workload and run it; otherwise declare the row `inconclusive`, name the entry point and the constraint that blocked it, and escalate under Stop conditions. Never substitute a different path, a test suite or a representative path. If the target's real workload is not runnable at all - a procedure or a document - declare the row `inconclusive`, name the exercise that stands in for it, and escalate that as a decision under the Stop conditions. |
| 4 | Delivered / published artifacts | Look for artifacts shipped to a consumer outside the repository - a release, a package, a published remote or registry entry, a deployed endpoint, a published output directory. Files merely tracked in the repo, including committed examples and samples, are fixtures. | Any such artifact exists. | **Treat as present** - do not write anywhere near it - and record that the check was inconclusive. |
| 5 | Reference artifacts | Read the `refs=` field of the invocation; check that every path exists and is readable. A directory contributes its contents, recursively. | At least one path is readable. | Treat the paths that could not be read as none for this run and record which; the readable ones remain evidence. Build sanitized fixtures from samples only when nothing was readable. |
| 6 | Input findings | Read the `findings=` field; open each path - a directory contributes its `.md` files - and count the findings. | At least one path is readable and holds at least one finding, and the invocation resolved to findings-driven. | **Stop and ask** - it decides the mode, and this case is not covered by the mode default. A named path that will not open, or a directory that yields no findings file, is what makes this row inconclusive. |
| 7 | Test suite / runner | Find the test entry point (test script, tests directory, CI config) and run it once, keeping any output, cache or coverage it produces outside the target. Phase 0 records only whether a runner exists and what it reported. | A runnable suite exists and produces a result. | Treat as absent and record it as an open item; whether to add a runner is a Phase 1 decision, not a Phase 0 one. |
| 8 | Goal tracking | Check whether the session exposes a goal primitive. Do not create a goal during a probe. | A goal tool is exposed. | Record the error and treat as absent; the goal lives in the report. |

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
- A declared-present goal capability that Phase 1 never arms is a
  process-compliance finding, exactly like a skipped skill. The declaration's
  evidence column carries the goal's id once it is armed.
- For production mode the declaration carries the command and the inputs, not just
  the word `present`. A row that cannot name an executable command and its inputs
  is `inconclusive`, never `present`. A command that is nameable but was not shown
  to start is inconclusive; if the declared command later cannot run as written,
  the row is re-probed and downgraded before the Gate.
- A `present` production capability whose workload was not executed, with no
  declared constraint, blocks the Gate as a compliance finding.

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
implemented is a recorded degradation, and any change it lands is verified by an
independent agent over that revision, under the context isolation below.
With delegation absent, every role is recorded as a self-held degradation and
raises a Gate compliance finding; no role is closed by the agent that implemented
the change. If no other agent exists, the run escalates under Stop conditions.

| Role | Runs in | Owns |
|---|---|---|
| Triage / reproduction | Phase 3 | restating each input finding, reproducing it in isolation, capturing the evidence |
| Implementation | Phase 4 | the fixes and the hardening changes |
| Independent test design | Phase 4 | the regression tests, and the coverage matrix that Phase 2 planned |
| Adversarial review | Phase 5 | logic, robustness, abuse cases, boundary and failure behaviour |
| Security / privacy scan | Phases 4 and 6 - on every batch, the baseline verification and the Gate's extra pass | secrets, credentials, PII, internal URLs, injection surfaces, unsafe logging, leakage |
| Verification | Phases 4 and 6 | the baseline verification, each verification step, and the Gate's extra pass - the production-like executions in an instance isolated from production and the repeated-execution simulation |
| Regression and cross-effects | Phases 4 and 6 | the recorded baseline still holds at every verification step, and nothing outside the changed area moved |

Phase 5's adversarial sweep is staffed by the Adversarial review role: "the
adversary" below means that role, and "the verifier" means the Verification role.

**Independent roles, no more than one role per agent.** Every role in the
delegation plan is staffed by its own agent: no agent holds more than one role,
and no role is merged into another. Implementation, verification, independent
test design and adversarial review are four separate agents, and the
orchestrator is none of them. Independence is a property of how the roles are
staffed, not of the plan's good intentions - two roles in one agent are one
role, however the plan words it.

The supporting roles - triage, the security/privacy scan, and regression and
cross-effects - are independent roles too, and are staffed by separate agents on
the same rule: no agent holds more than one of them, and none of them shares an
agent with implementation or with any of the four independent roles. They exist
to keep one agent's attention and one agent's blind spots out of another role's
work, so folding one into a role it exists to check defeats it. When a supporting
role has nothing to do on a batch, it is still staffed and records that outcome
as its result; it is never dropped.

The orchestrator keeps coordination, integration, conflict resolution and the
Gate, and nothing else. Holding a role itself is a degradation that the
declaration records with its reason, and the Gate treats it as a
process-compliance finding - it is never a permitted alternative.

When the orchestrator does implement - delegation genuinely absent, or the
implementation role could not be staffed separately - that is a self-held role
the declaration records with its reason, and the Gate raises it as a
process-compliance finding. Every commit it lands still needs a named independent
verification over that revision before the Gate may open, run by an agent that
holds no other role; when the run has no such agent to free, it escalates under
Stop conditions rather than closing the role itself. See **Independence rule**.

An escalated degradation stays a compliance finding until the operator
explicitly accepts the degradation or supplies a separate agent. An accepted
degradation is recorded in section 1 as "not fully achieved" and does not by
itself block the Gate, since the Gate's compliance check is against the declared
staffing, not against a satisfied independence floor; supplying a separate agent
remediates the role and removes the degradation.

**Instruction-budget note.** The cost of this rule is more agents, not more
rounds: every role can run against one frozen batch, and a role with nothing to
do on a given batch records that as its result rather than being dropped. The
security/privacy scan stays a mandatory role rather than an "if present" one:
the skill already requires that scan on every batch and every verification
step, whatever the change touches, and its default must not shrink to fit a
smaller plan.

**Delegation contract.** Give each role a scope, its inputs, its deliverables,
the evidence it must return and its constraints - including the read-only and
sensitivity rules for anything it touches. Every verification,
adversarial-sweep and test-design delegation names the exact revision or
snapshot it covers - the commit id, or the manifest of file hashes when the
target has no version control - and returns it with its evidence. A
pass without a named revision covers nothing.

**Context isolation.** A verifier or adversary receives the target, the change
and the exit criteria - never the implementer's notes, hunches or
self-assessment. Use an agent with no inherited context: a verifier that
continues the implementer's conversation inherits its blind spots, and the
independence is only nominal.

A pass expires when the thing it verified changes. Any change - a follow-up fix,
a documentation edit, a generated artifact - invalidates recorded passes over the
changed content, and the Gate may not open on a stale pass. A content-preserving
integration commit does not invalidate a pass: a fast-forward, a rebase-free
merge, or a merge commit whose tree equals the approved revision's tree, provided
the equivalence is recorded in section 12. A pass whose revision cannot be shown
to be the revision under review is a finding.

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

**Failed delegation.** A failed, stalled or empty delegate is re-delegated, not
absorbed: the role is re-dispatched to an agent that holds no other role in this
run, or the run escalates the failure as an open item under the Stop conditions.
No role is taken over by the orchestrator, by the implementer's agent, or by any
agent already staffing another role. The only staffings that satisfy the independence
rule are a separate agent for the role, or a recorded degradation with a
Gate-visible compliance finding; "the role was folded into another delegation"
is no longer available, because no agent holds more than one role.

Re-delegation is bounded: after the second failure the run reports the role's
outcome as a known gap with a compliance finding, rather than dispatching in a
loop.

**Parallelism.** Independent workstreams within one role's scope run concurrently
where that is safe; the role table is fixed, so concurrency never shares an agent
between roles. Anything sharing mutable state, or that must observe another
role's output, runs in order.

**No-solo rule.** A role in the plan that was neither delegated to its own agent
nor recorded as a degradation with a Gate compliance finding is a
process-compliance finding, exactly like a skipped skill; a merge into another
role is that violation, not a remedy.

**Independence rule.** The constraints above hold whenever delegation is
present. When it is absent, every role is recorded as a self-held degradation and
raises a Gate compliance finding; no role is closed by the agent that implemented
the change - not by a separate pass that re-reads the target from disk, and not
by an agent already staffing another role. If no other agent exists, the run
escalates under Stop conditions.

**Read-only rule.** Reference artifacts and delivered artifacts are inputs, not
workspaces. Nothing is written to them, and nothing from them is copied into
the target, tests, fixtures, commits or reports without sanitization. When the
real workload writes, run it against a copy of its state and inputs under a
scratch output directory, using the target's own environment or configuration
override, so production isolation and the read-only rule are both satisfied. If
the target has no such override, that is a blocking constraint to escalate, not
a reason to fall back silently.

**Evidence rule.** Every capability decision - detected, absent, or used as a
fallback - appears in the report with the command, tool output or observation
that established it.

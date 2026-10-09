---
name: hardening-loop
description: Use when a software artifact - a skill, plugin, feature, module, repository, service, pipeline, library or app - must be adversarially verified and fixed until no known issue remains at any severity. Runs in baseline mode (scope from reconnaissance) or findings-driven mode (the input findings are the starting backlog).
whenToUse: Any request to thoroughly verify and fix, harden, audit to zero, or adversarially test a target and leave no known issue, regression or open item behind - including a run seeded by findings files, a review report or a prior run's leftovers. Not for a single bug fix, a one-shot review of a small diff, or ordinary feature work.
---

# Hardening Loop

## What this is

An adversarial, evidence-driven, closed-loop verify-and-fix process. A run ends
in exactly one of three states: achieved (the Gate is satisfied - no known issue
or finding at any severity, every open item closed, the baseline still holds, and
process compliance is verifiable); pending review (open items documented with
closure attempts, nothing pushed); or not fully achieved (an unmet criterion
whose gap is accepted). An unachieved ending - pending review or not fully
achieved - documents its gap and pushes nothing.

The loop is the point. Every finding - input finding, regression, newly
discovered, introduced by this run's own fixes, or process-compliance - returns
to the action step, and the verification step runs again. A run that stopped
because the operator stopped looking is not a completed run.

## When to use

Use it for a target that must be left clean, not merely improved:

- `harden the X skill`, `verify and fix Y until no issue remains`
- an adversarial audit of a plugin, module, repo, service or pipeline
- closing out a findings backlog or review report to zero open items
- a pre-release hardening pass over a named baseline or tag

Do not use it for a single bug fix, a one-shot review of a small diff, or as a
substitute for ordinary feature work. If the target is small enough that a
single ordinary review settles its whole scope - one bug fix, one small diff,
one file - say so and do that review instead of running the full loop; a
coverage-complete verification pass is not that review. This is a consequence of
the loop's independence floor, not a stylistic preference: the floor costs more
agents than such a change warrants, so a target small enough to settle in one
review is out of scope for this skill.

## Target declaration

Fill this in before Phase 0. A field the invocation neither states nor implies is
filled by its default in the Defaults paragraph below.

| Field | Canonical key | Value |
|---|---|---|
| target | `target=` | <name, path or identifier> |
| mode | `mode=` | baseline or findings-driven |
| input findings | `findings=` | <one or more paths, or a directory, or none> |
| reference artifacts | `refs=` | <one or more paths or directories, or none> |
| baseline / milestone | `baseline=` | <commit, tag, version, or the state as found> |
| merge target branch | `branch=` | <branch to land the work on; default main, or `none` when the target is not under version control or when `branch=none` is stated explicitly> |
| constraints | `constraints=` | <access, environments, time, tooling, or none> |
| extra skills | `skills=` | <skills to invoke in addition to the framework, or none> |

An explicitly stated `none` is conclusive-absent. `target` is the only required
parameter; every other field has a default (see the Defaults paragraph below)
or is implied by the request. Anything still unknown that would change the scope
or the stopping rule is asked for before starting, never silently guessed.
`mode=baseline` and the `baseline=` field are different things: the mode says
where the scope comes from, the field names the state that regression is measured
against.

### Input forms

Parameters arrive in one of two equivalent forms:

- **canonical assignments** - `target=... mode=... findings=... branch=...`.
- **a prompt** - a request in prose that names the target and what is wanted,
  for example "harden the travel-planner skill until nothing is left".

Normalize either form into the table above before Phase 0. From a prompt, fill
each field the request actually implies, leave the rest to their defaults below,
and state the resolved declaration back in one short block so it can be
corrected before the run starts.

Defaults fill every optional field: `branch` is `main` when no branch is named,
or `none` when the target is not under version control; a run with no findings
is a **baseline** run, so `mode` is `baseline` and `findings` is `none`; `refs`,
`constraints` and `skills` are `none`; and `baseline` is the target's **state as
found** at Phase 0. An explicit `mode=findings-driven` with
nothing given or readable is a contradiction to resolve first, not a default.
An explicit `mode=baseline` together with one or more readable findings paths is
the same kind of contradiction: resolve it with the operator before Phase 0;
`findings=` never silently overrides an explicit `mode=`.

`findings=` takes one path, several comma-separated paths, or a directory of
findings files. Every path given is part of the backlog; a directory contributes
its `.md` files (not recursive, in name order); and all of them merge into one
triage table. Record each finding's source file in the Source column when more
than one source is given; the ID stays a single sequence shared by section 7 and
section 9, never a per-source prefix. Treat the same finding reported in two
files as one entry naming both sources. A path that will not open, or a
findings file that will not open, is asked about rather than silently skipped.

`refs=` takes one path, several comma-separated paths, or a directory, and every
path given is read-only evidence. A directory contributes its contents,
recursively. Nothing under any of them is written to, and a path that will not
open is recorded rather than skipped.

**Mode A (baseline)** derives scope from reconnaissance.
**Mode B (findings-driven)** triages the input findings, from every source given.

In both modes the input findings are the **minimum** backlog, never the whole
scope, and **mode B includes mode A's scope**: the findings seed a thorough
verify-and-fix run, they do not narrow one. After triage, mode B also derives the
reconnaissance-derived backlog that mode A would have produced. Zero open input
findings against an unexamined target is not a finished run.

## Core principles

1. Goal before plan, plan before act.
2. Evidence over assertion - every claim is backed by a command, log, diff or output.
3. Isolation - verification never disturbs production, delivered artifacts, schedules or state.
4. No leftovers - temp files, processes, state, branches and side effects are cleaned up.
5. No silent deferral - every open item is closed or documented for review.
6. Independence - every role is staffed by an agent that holds no other role,
   and review and verification are never done by the agent that implemented the
   change. When delegation is unavailable the capability declaration (report
   section 3) records each role as a self-held degradation with its reason, and
   escalates under Stop conditions; no role is closed by a separate pass or by
   any agent holding another role.
7. Verifiable compliance - process requirements are logged, and an unmet requirement is itself a finding.
8. Falsifiable goal - the run's goal has checkable exit criteria and a stopping
rule.

## Classification

Classify every issue or finding as High / Medium / Low / Trivial:

| Severity | Covers |
|---|---|
| High | correctness, security or privacy, data loss, production disturbance, blocking failure |
| Medium | functional defect, reliability issue, significant deviation; process-compliance violations that weaken confidence, independence or goal clarity |
| Low | minor bug, maintainability or documentation issue |
| Trivial | cosmetic or nit |

Tag the origin of every finding: input finding, regression against baseline,
newly discovered, introduced this run, or process compliance.

The goal is zero known findings at **every** level, including process
compliance.

## Open item policy

An open item is any unresolved issue or finding, TODO, deferred fix, known
limitation, untested area, unanswered question, dependency, missing access,
follow-up action, or decision-required item.

- Try to close every open item.
- Never silently defer, hide or ignore one.
- If one cannot be closed within the stated constraints, document it for review
  with: ID, description, severity, category, reason unresolved, impact,
  attempted fixes, workaround, recommendation, next action, and the decision
  needed.
- High-severity, blocking, security or privacy, data-loss, production
  disturbance, and decision-required items are escalated immediately and are
  never merged or pushed past silently.

## Process

### Phase 0 - Reconnaissance
- Learn the target: purpose, interfaces, entry points, dependencies, configs,
  state, outputs, and known constraints.
- Treat the target declaration's description of the target as a claim to check,
  not a fact: reconcile it against what is actually there, and record any
  difference.
- State concretely what "no regression" means against the baseline. With no
  explicit `baseline=`, that is the target's state as found at Phase 0: record
  its identity - the revision when the target has one, otherwise a manifest of
  file hashes - and for a version-controlled target record whether the tree was
  dirty. Record that identity before running any probe that executes the target.
- Run the Phase 0 probe sequence in `reference/capabilities.md` for all eight
  capabilities, and emit the capability declaration table. A capability that is
  present makes its "if present" column mandatory; an absent or inconclusive one
  takes the default or escalation recorded in that file. The declaration is evidence -
  "assumed present" is not.
- Inspect the reference artifacts read-only, as evidence - every path given.

### Phase 1 - Goal setting (before the plan)
State one explicit, measurable goal whose terminal condition **is** the Gate -
the goal is not achieved until every exit criterion passes, the compliance check
passes, the adversarial sweep over the final state is clean, and the extra
verification pass is clean. The exit criteria below are how the goal is
falsified, not the goal itself: satisfying them while the Gate is shut does not
achieve the goal, and the goal stays open. Arm the harness goal with that
objective and leave it active across rounds. Close the harness goal when the run
ends - on Gate satisfaction, or as "pending review" / "not fully achieved" when
it does not - but only once the merge the run made is complete and covers exactly
the revision the Gate approved, or once the change-sets have landed when there is
no merge target. If the Gate's revision is voided after the goal closed and the
loop re-enters, reopen the goal before the next action step. The
report's section 1 records the closing state and that it came after the merge (or
the change-set landing). At minimum:

- [ ] Every input finding is triaged, reproduced or explicitly marked
      non-reproducible with reasoning, fixed where valid, and covered by a
      regression test - added where none existed, if a runner was feasible - or
      the absence is justified and recorded.
- [ ] No known issue or finding at any severity, including process compliance.
- [ ] No open item remains. If one cannot be closed, it is documented under
      §13, which marks the run "pending review" and leaves this criterion not
      fully met.
- [ ] Conditional-capability requirements are met, with evidence.
- [ ] The baseline still holds - no regression against the state recorded in Phase 0.
- [ ] Production, delivered artifacts and schedules were untouched, and no
      leftovers remain.

Then define "done" in observable terms (the specific evidence required, not
adjectives), the non-goals and out-of-scope items, and the stopping rule: the
goal is achieved only when the Gate is satisfied. If it cannot be achieved, do
not silently redefine it: document the gap, end as "pending review" when an open
item remains or as "not fully achieved" when the unmet criterion is an accepted
gap, and record an open item for anything still unresolved.

### Phase 2 - Planning
Produce a concise plan covering: goal, exit criteria, non-goals and stopping
rule; the methodology-framework discovery result and Skill Usage Plan (when
present, including any skills named by `skills=`); the Delegation Plan - each
role, the phase it runs in, and its independence (when delegation is present),
the staffing check included: which agent fills each role, and the rule that no
agent fills more than one. Any role the run could not staff separately - held by
the orchestrator, or by an agent already staffing another role - is recorded as a
degradation; the triage plan (mode B) or scope-scan plan (mode A),
including the hypothesised root cause behind each input finding; scope - the
input findings and a re-audit of the target as it actually behaves;
implementation order; test strategy and coverage matrix
built from `reference/coverage.md`, including one regression test per input
finding or its recorded justification; verification strategy - the baseline
verification before any change, then isolated production-like executions plus
repeated-execution simulation where feasible; the production-mode run, quoted
verbatim: the exact command and the exact inputs from the capability
declaration - a plan that names no production run, and no blocking constraint
preventing one, is a plan for an unverified change-set; handling of read-only inputs (immutability, sanitization, no leakage into commits); the
isolated work branch or worktree the changes will be made on, one follow-up
commit per batch (the set of related fixes one action step carries), with no
rewriting of existing history; open-item handling; and
risks, rollback and cleanup.

**Version control.** The isolated work branch or worktree, the one follow-up
commit per batch (the set of related fixes one action step carries) and the
no-rewriting rule apply when the target is under version control. When it is not - or `branch=none` was stated explicitly - the
declaration records that, the Phase 0 baseline identity (the manifest of file
hashes recorded for a target with no revision) is the as-found reference, and
each fix lands as one discrete described change-set in place of a commit; Phase 7
records the applied change-sets and that identity in place of a branch and a
merge.

### Phase 3 - Triage (mode B) or scope scan (mode A)

Mode B, per input finding: restate it precisely; classify its severity;
separate symptom from root cause from environment artifact; cross-reference the
reference artifacts; attempt deterministic reproduction in isolation and
capture the evidence; if it is not reproducible in isolation, document why and
what conditions are suspected, and keep the underlying risk as an open
candidate - a latent issue exposed only by real runtime behaviour stays in
scope even when an isolated run cannot reproduce it; record the regression test
that would have caught it (the Phase 4 action step adds it, under independent
test design where delegation is present), or record why none can be written and
what check replaces it. Mode B does not stop at the file: it then runs the mode A derivation
below and adds those candidates to the backlog.

Mode A: derive the reconnaissance-derived backlog from reconnaissance gaps, known
weaknesses, undocumented behavior, an adversarial hypothesis list (abuse cases,
failure modes, edge cases, trust boundaries), and reference artifacts compared
against current behavior.

### Phase 4 - Verification-action loop

**Terms.** A **run** is the whole hardening engagement, from invocation to
report: the goal, its exit criteria and the stopping rule belong to the run. A
**round** is one action step plus the verification step that follows it; a
**batch** is the set of related fixes that one action step carries, so one round
verifies one batch. A **verification step** (also written "verification pass") is
one traversal of the whole of `reference/coverage.md` - all three coverage passes
plus regression; it runs as the baseline verification, after each batch, and once
more as the Gate's extra pass. An **execution** is one
execution of the target, as in a repeated, concurrent or scheduled execution. The
loop runs rounds until a verification step is clean.

**Baseline verification.** Before any change, run one verification pass over the
target as it was found: `reference/coverage.md` defines what that pass must
reach. Its findings join the backlog, and anything already broken here is
recorded as a finding rather than a regression. A run whose backlog is empty
still starts here, so every run verifies at least once.

Action step, per batch. Batches come from the triaged input findings (mode B),
the reconnaissance-derived backlog (mode A), and the baseline verification.

- Confirm the batch ran under the required methodology and delegation (when
  present), and log the evidence.
- Confirm it advances one or more exit criteria, and name which. The same check
  confirms the goal is still open: a round that closes the objective while the
  Gate is unopened is a finding.
- Scan the target, every diff, the read-only inputs and the input findings files
  for private or sensitive information - secrets, tokens, credentials, PII,
  internal URLs, private data - and exclude, remove or redact it.
- Confirm nothing from a read-only input was copied verbatim into the target,
  tests, fixtures, commits or reports without sanitization.
- Ensure tests and coverage are updated, including the regression tests tied to
  input findings - designed and run by the independent test-design role where
  delegation is present (see `reference/capabilities.md`); where it is not, the
  role is recorded as the self-held degradation the capability declaration names
  and the Gate raises as a process-compliance finding. No pass separate from the
  implementation closes it. Where the target has no test suite, add a runner if
  one is feasible: a test that would have caught the finding is the deliverable,
  not a justification for skipping it. Only when that is genuinely infeasible,
  record the absence as an open item.
- Make each batch its own follow-up commit on the isolated work branch or
  worktree, and **close the batch before leaving this step**: every change the
  batch contains is committed, the worktree is clean, and no implementation for
  this batch is still running anywhere - no background job, no delegate, no
  editor. The batch's scope is fixed here and does not grow afterwards. (Where
  the target has no version control, or `branch=none` was stated explicitly,
  land the batch as the discrete described change-set Phase 2 defines and record
  the change-set's identity in the same way.)

  The commit is what the next step verifies. A batch handed to verification
  uncommitted, or with work still landing, has no revision to verify, and any
  result from that step describes a tree that no longer exists. Keep each change
  scoped and descriptive, and reference the finding IDs and the exit criteria it
  advances. Never amend, rebase or otherwise rewrite existing history.

Verification step, per action step:
- **Close-of-batch gate.** Before verifying anything: the batch from the last
  action step must be committed on the work branch and nothing may be mid-flight
  - no uncommitted changes belonging to that batch, no implementation delegate
  or job for this batch still running. Independent roles with nothing to land may
  keep running through the check, and their results attach to the round's
  evidence. Take the revision now (the commit id, or the manifest of file hashes
  when the target has no version control) and verify **that** revision; every
  result is reported against it.

  If the check does not pass, do not start verification: return to the action
  step and let it finish the commit, or wait for the in-flight implementation to
  settle and commit. Verifying a moving tree, or verifying "the batch" without a
  revision attached, is a process-compliance finding - the point of the check is
  that verification and implementation of the same batch can never overlap in one
  round.

  The wait is bounded. If the batch cannot be closed because implementation will
  not settle - a delegate that stalls, a job that will not finish, an
  interruption that leaves work uncommitted - do not wait indefinitely:
  re-delegate or abandon that work as **Failed delegation** requires, record the
  batch's actual state, and escalate under the Stop conditions. A wait with no
  bound is a hung round, not a closed batch, and it is the one way this gate can
  stop the loop from making progress at all.
- Delegate to independent verification: the role is staffed by an agent that did
  not implement the change and holds no other role. The orchestrator never
  verifies its own implementation, and no role is closed by the agent that
  implemented the change; when delegation is genuinely unavailable, the
  declaration records each role as a self-held degradation with its reason, and
  escalates under Stop conditions; no role is closed by a separate pass or by any
  agent holding another role. What closes a change is an independent pass over
  that revision.
- Run the pass against the coverage in `reference/coverage.md`: it defines what a
  verification must reach, and the report records the matrix the pass produced.
- Run the production-mode workload named in the plan and the capability
  declaration - the target's own entry point, on its real inputs - in isolation.
  A subset, a test suite or a representative path never satisfies this bullet;
  when it cannot run unrestricted, name the constraint, name what it removes, and
  run the same entry point reduced only in inputs. When it cannot run at all, name
  the blocking constraint and escalate. Not running it is a process-compliance
  finding, not a fallback: either the workload ran, or a blocking constraint is
  named and escalated as a decision at the Gate. "The workload is agentic", "it
  takes too long" and "the tests cover that layer" are not constraints; a
  constraint is missing access, an unowned input, or a target that cannot be run
  at all.
- Do not disturb delivered artifacts, and do not trigger or change scheduled
  jobs, cadences or other external triggers.
- Leave no leftovers: redirect outputs to sandbox paths and clean up.
- Where feasible, simulate multiple consecutive executions of the target.
- Capture evidence: commands, logs, outputs, diffs, test results, cleanup confirmation.
- Re-check exit-criteria progress, and record which criteria are now met.

**Baseline check.** A regression is a behavior that worked in the recorded
baseline and no longer works. A behavior that was already broken at the baseline
is a finding, not a regression: the baseline measures regression only, and does
not promise the target was correct to begin with. The baseline verification above
is what establishes which behaviors those are.

**Loop rule.** If any issue or finding appears - a regression, a new finding, a
finding introduced by this run's own fixes, or a process-compliance finding -
return to the action step, fix it - as recommended where the finding carries a
recommendation, otherwise as the next round's action step decides - and repeat
the verification step. Re-entering the action step is a full action step: the new
batch carries every obligation below, including the sensitive-information scan,
the sanitization check, the test update and the batch closure. "Fix it" means the
finding is closed in this run, or escalated as a decision under Stop
conditions. A finding may not be carried to the Gate unresolved because closing
it would invalidate earlier verification - re-running that verification is what
the loop exists to do. Continue until no finding of any level remains.

### Phase 5 - Adversarial sweep
The sweep does not need its own close-of-batch gate - the verification step that
precedes it is full over the revision it named, and nothing runs between them -
but it must assert the input the close-of-batch gate exists to protect: the
revision this sweep is about to attack is the revision the immediately preceding
verification step verified clean. Take that revision (the commit id, or the
manifest of file hashes when the target has no version control), attack it, and
report every finding against it. If anything landed after the verified revision,
the sweep does not start: the loop re-enters at the action step, and the new
state is verified before it is attacked. The check is one sentence because the
state it guards was already fixed by the previous step; the alternative is a
sweep that attacks a tree it never named, which is the same defect as verifying
one.

Once the loop is quiet, attack the target and the accumulated changes
independently. Where a verification step asks whether the target meets
`reference/coverage.md`, the adversarial sweep asks how it breaks: assume the
change is wrong, find the input that proves it, and record the attacks that
failed. It owns what no coverage category does - assumption challenges:
undocumented behavior, hidden coupling, environment dependence. Documentation is
attackable too: a false, stale or misleading claim is a defect like any other.

The adversarial sweep runs against the state the loop has reached, and its
findings make that state stale. A finding here returns to the action step; once
the round that fixes it verifies clean, the adversarial sweep runs again over the
new state, because the change it caused is new attack surface. Repeat until an
adversarial sweep comes back clean - one that never saw the final state does not
count. Feed every finding back into the loop.

### Phase 6 - Gate
- Goal check: every exit criterion pass/fail with concrete evidence. A partial
  criterion does not pass. A run ends in exactly one of three states: achieved;
  pending review (open items documented with closure attempts, nothing pushed);
  or not fully achieved (an unmet criterion, for example a self-held role the
  operator accepted). The report's section 1 goal record names which.
- Compliance check: methodology and delegation requirements met with evidence,
  and the goal was set up front. Any unmet requirement is a finding, and blocks
  the Gate - except a degradation the declaration records and the operator has
  explicitly accepted, which closes its compliance finding and, since this check
  is against the declared staffing, does not by itself block it; it leaves the
  run "not fully achieved" (`reference/capabilities.md`).
- Run one extra full verification step after the last finding, once the
  adversarial sweep over the final state has come back clean - the adversarial
  sweep is what proves the last change was attacked. One, not a repeating rule: a
  run whose backlog was empty and whose baseline verification found nothing runs
  that pass too, and the Gate is never opened on a single clean pass.
- A finding from that extra pass is not a Gate failure: it returns to the action
  step like any other finding - fix, verify, adversarial sweep over the new state,
  and the extra pass again once that sweep is clean. "One" means the pass is not
  chained after a clean result, not that it runs only once.
- Only when that extra pass is clean, the goal check passes, the compliance
  check passes, and no open item remains is the goal achieved.
- A criterion that passes only because a remaining item is documented does not
  make the run achieved: it ends as "pending review", the goal record in section
  1 says so, and it does not push.
- If an open item remains that cannot reasonably be closed, mark it "pending
  review", document it, and do not push - **and the documentation must carry a
  closure attempt: what was tried, what it produced, and why it failed.** One of
  the following is required, and the run records which:
  - a reproduction showing the item is not closable in this environment;
  - a recorded experiment whose negative result is the evidence (a test design
    that cannot reach the check, a probe that cannot be made to answer);
  - a blocking constraint from the Stop-conditions list (missing access,
    ambiguous requirements, a high-severity or blocking risk, a
    decision-required item).
- An item with a known, executable recommendation that the run did not attempt is
  **not** pending review: it is an unmet goal, the run is not complete, and the
  action step resumes. "The attempt would be expensive" and "it would invalidate
  the sweep" are not failed attempts.
- The Gate's open-item check and the goal check are the same check: an item that
  keeps the Gate shut keeps the goal open.

### Phase 7 - Completion
- Merge the revision the Gate approved - the revision the extra pass verified -
  and only it, back to the branch named in the declaration - `main` when the
  invocation named none - locally. Then verify that the merge result covers
  exactly that revision, and close the harness goal only once it does; if the
  merge covers anything else, keep the goal open and re-enter the loop before
  closing it. If anything lands between that pass and the merge, the Gate's
  result is void for the new revision and the run re-enters the loop: the action
  step commits the change, then the verification step over the new revision, then
  the adversarial sweep over that state, then the extra pass, then the Gate
  again. A merge of a revision no pass covered is a compliance finding. A
  content-preserving integration commit does not void the pass the Gate approved
  (`reference/capabilities.md`); record the equivalence in §12.
- When there is no branch to merge into - the target is not under version
  control, or `branch=none` - record "no merge target" in §11 and §19 and list
  the applied change-sets and the baseline identity there instead.
- Do not push. "Push" means any publish to a remote or external consumer:
  `git push`, a package or image publish, or a deployment. None of it happens
  without explicit approval, and the report carries the exact command(s) when one
  exists - a run with no branch or remote records "no push target".
- Leave read-only inputs untouched and uncommitted.
- Finalize the report (see `reference/report-template.md`) for review before any
  push. Its Gate-facing sections - §2 scorecard, §3 capability declaration, §6
  compliance summary, §13 open-item log, §14 sensitive-information scan, §16
  coverage matrix - are drafted as their evidence is produced and are checked at
  the Gate; §11 is completed after the local merge, or after the change-sets land
  when there is no branch.

## Reporting

Every run ends in a report following `reference/report-template.md`. The
capability declaration, the exit-criteria scorecard, the compliance summary, the
open-item log, the coverage matrix and the sensitive-information scan result are
what make the Gate checkable; omit none of them on the grounds that a section
"obviously passes".

## Stop conditions

Stop and ask only when blocked by: missing access; ambiguous requirements that
cannot be reasonably resolved; a high-severity risk; a blocking open item; or
an open item that requires a decision.

## Invocation examples

Canonical assignments:

- `hardening-loop target=skills/daily-news-briefing mode=baseline`
- `hardening-loop target=skills/daily-news-briefing mode=findings-driven findings=findings/2026-10-01.md refs=runs/ branch=release/0.5`
- `hardening-loop target=repo:payments-service mode=baseline baseline=v1.4.0`
- `hardening-loop target=plugin:auth-adapter mode=findings-driven findings=issues/q3-audit.md`

Prompts that resolve to the same declarations:

- "harden the daily-news-briefing skill until nothing is left" ->
  `target=skills/daily-news-briefing mode=baseline branch=main`
- "close out findings/issues/q3-audit.md against the auth adapter plugin and
  land it on release/0.5" ->
  `target=plugin:auth-adapter mode=findings-driven findings=issues/q3-audit.md branch=release/0.5`

A run against a large target usually belongs in its own session: the loop is
long, and the report is the handoff back.

## Reference

| File | Contents |
|---|---|
| `reference/capabilities.md` | Capability detection and the Phase 0 probe sequence, the default framework, the delegation roles and their independence, and the rules that follow |
| `reference/coverage.md` | Coverage as three coverage passes plus regression: what closes each category, how to sample, and which technique tests which claim |
| `reference/report-template.md` | The full reporting template |
| `examples/findings-driven-run.md` | A condensed worked run, from input findings to the Gate |

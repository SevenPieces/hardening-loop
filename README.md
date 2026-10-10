# hardening-loop

An agent skill that runs an adversarial, evidence-driven verify-and-fix loop
against a target - a skill, plugin, feature, module, repository, service,
pipeline, library or app - until no known issue remains at any severity.

Every finding is classified, reproduced or explicitly marked non-reproducible,
fixed where valid, covered by a regression test or a recorded justification, and
re-verified. The loop
repeats on every finding it produces, including the ones its own fixes create
and the process-compliance findings it raises against itself. A Gate - goal
check, compliance check, no open item, and one extra clean verification step -
decides when the run is done. The output is a report for review, and nothing is
pushed. A run ends in exactly one of three states: achieved (the Gate is satisfied
with no accepted gap against a criterion or role); pending review (open items
documented with closure attempts, nothing pushed); or not fully achieved (an
unmet criterion whose gap is accepted and recorded) - once any revision produced
under an accepted implementation or context-isolation degradation has had its
clean independent pass, an accepted gap leaves the Gate satisfied but the run
"not fully achieved". A pending-review report
documents each item that stays open with its
closure attempt; a finding could not be carried to the Gate unresolved except as
a decision escalated under Stop conditions, and the report names which rule
applies.

## What a run produces

| Output | Purpose |
|---|---|
| A hardening report | The deliverable: goal, exit-criteria scorecard, triage table, compliance summary, finding and open-item logs, verification evidence, the coverage matrix, including its change-set block (section 16), sensitive-information scan, and the exact push command(s) pending approval when a branch or remote exists; a run with no branch or remote to push to records "no push target" instead, and a run with no merge target records "no merge target" in section 11 and section 19 |
| Regression tests | One per input finding - or a recorded justification where none can be written; added to the target's suite, or to a runner added for the purpose where none existed and one was feasible |
| A locally merged branch | The revision the Gate approved - the one the extra pass verified - merged locally to the branch the invocation names - `main` when it names none - and never pushed; anything landing after that pass voids the Gate and re-enters the loop, and a target that is not under version control, or a run invoked with `branch=none`, records its baseline identity and the applied change-sets instead. The isolation the run created - the branch it worked on and any worktree - is torn down once the merge is verified |

The skill writes its report, its findings and any scratch state into the working
directory the run is pointed at - never into the skill's own directory - and
defines no fixed file name for them: the `.gitignore` patterns for those
artifacts are a speculative safety net for a stray in-tree run, not a naming
convention the run follows. It ships no dependencies, scripts or network
requirements of its own; the methodology framework - when a skill index is
exposed - comes from the harness's exposed skills; otherwise the built-in process
in `SKILL.md` is followed, as `reference/capabilities.md` requires.

## Requirements

- A skill-capable agent harness (DeepSeek Harness, Claude Code, ...).
- Whatever the *target* needs to run, test and reproduce - the skill supplies
  process, not tooling.
- Read access to the target and, in findings-driven mode, to the findings files.

## Install

Clone into your harness's skills directory, keeping the directory name equal to
the `name:` in the frontmatter:

```sh
git clone https://github.com/SevenPieces/hardening-loop.git ${DSH_HOME:-$HOME/.dsh}/skills/hardening-loop
# Claude Code
git clone https://github.com/SevenPieces/hardening-loop.git ~/.claude/skills/hardening-loop
```

## Use

```
hardening-loop target=skills/daily-news-briefing mode=baseline
hardening-loop target=skills/daily-news-briefing mode=findings-driven findings=findings/2026-10-01.md refs=runs/ branch=release/0.5
hardening-loop target=repo:payments-service mode=findings-driven findings=findings/,issues/q3-audit.md refs=runs/2026-09-30/,runs/2026-10-01/
hardening-loop target=repo:payments-service mode=baseline baseline=v1.4.0
hardening-loop target=plugin:auth-adapter mode=findings-driven findings=issues/q3-audit.md
```

The same parameters may be given as a prompt instead: "harden the
daily-news-briefing skill until nothing is left" resolves to
`target=skills/daily-news-briefing mode=baseline branch=main`. Either form is
normalized into the target declaration and stated back before the run starts.

The revision the Gate approved - the one the extra pass verified - is merged back
to the branch the invocation named - `main` when it named none - and is never
pushed; anything landing after that pass re-enters the loop. A target that is not
under version control, or a run invoked with `branch=none`, has no branch to
merge into: its baseline identity and the applied change-sets are recorded
instead.

Two entry modes:

- **baseline** - no input findings; scope comes from reconnaissance and an
  adversarial hypothesis list.
- **findings-driven** - the input findings are the starting backlog.

In both modes the input findings are the *minimum* backlog, not the scope.

A run against a large target usually belongs in its own session: the loop is
long, and the report is the handoff back.

## Parameters

| Parameter | Canonical key | Required | Default when omitted | Meaning | Example |
|---|---|---|---|---|---|
| target | `target=` | yes | - | What is being hardened: a name, a path, or a `kind:name` identifier | `target=skills/daily-news-briefing`, `target=repo:payments-service` |
| mode | `mode=` | no | `baseline` | `baseline` derives scope from reconnaissance; `findings-driven` starts from the input findings and still derives the reconnaissance-derived backlog after triage, so mode B includes mode A's scope | `mode=findings-driven` |
| input findings | `findings=` | no | `none` | One or more findings files, or a directory of them, that seed the backlog; naming any makes the run findings-driven unless `mode=baseline` was stated explicitly, in which case the conflict is resolved with the operator first. Several paths are comma-separated, and a directory contributes its `.md` files, non-recursively, in name order | `findings=findings/2026-10-01.md,findings/2026-10-02.md` or `findings=findings/` |
| reference artifacts | `refs=` | no | `none` | One or more paths or directories of read-only evidence - prior runs, outputs, delivered artifacts - used for triage and reproduction. Several paths are comma-separated, a directory contributes its contents recursively, and nothing under any of them is written to | `refs=runs/2026-10-01/,runs/2026-10-02/` or `refs=runs/` |
| baseline / milestone | `baseline=` | no | the target's state as found at Phase 0 | The revision that "no regression" is measured against: a commit, a tag or a version. Without one, Phase 0 records the as-found identity - a revision, or a manifest of file hashes that stands in for it | `baseline=v1.4.0` |
| merge target branch | `branch=` | no | `main` when the target is under version control; `none` when it is not under version control, or when `branch=none` is stated | The branch the work is merged back to locally, and never pushed | `branch=release/0.5` |
| constraints | `constraints=` | no | `none` | Access, environment, time or tooling limits the run must respect; quote it when it contains spaces | `constraints="no network, 2h budget"` |
| extra skills | `skills=` | no | `none` | Skills to invoke in addition to the detected methodology framework (Superpowers where available); each is mapped to the step it serves and logged | `skills=brainstorming,writing-skills` |

Notes:

- Only `target` is required; every other parameter has a default, in the table
  above. The baseline default is the notable one: with no `baseline=`, Phase 0
  records the target's state as found - a revision, or a manifest of file hashes
  that stands in for one - and "no regression" is measured against that.
- An explicitly stated `none` is conclusive-absent. Anything still unknown that
  would change the scope or the stopping rule is asked for before Phase 0 rather
  than guessed.
- Parameters may be assigned canonically or implied by a prompt; either way they
  resolve into the target declaration in `SKILL.md`, which is stated back before
  the run starts so it can be corrected.

## How the loop works

```
Reconnaissance -> Goal (falsifiable exit criteria) -> Plan
      -> Triage / scope scan
      -> Baseline verification (always, before any change)
      -> Action  <------------------------------+
      -> Verification (independent)             |
           any finding? --- yes --- fix --------+
                   |
                   no
      -> Adversarial sweep (over the current state)
           any finding? --- yes --- fix --------+
                   |
                   no
      -> Extra pass (one full verification, once
         the adversarial sweep over the final
         state has come back clean)
           any finding? --- yes --- fix --------+
                   |
                   no
      -> Gate (goal check, compliance, no open item, extra pass clean)
           Gate fails?  --- yes --- fix --------+
                   |
                   no
      -> Report
```

The baseline verification runs even when the backlog is empty. Every `fix` arrow
returns to Action, and Action is followed by Verification, the Adversarial sweep
and the Extra pass, so a finding from any check - Verification, the Adversarial
sweep, the Extra pass or the Gate - is re-gated only after the verification step,
a fresh adversarial sweep over the new state and the extra pass have all run
again.

The Gate only opens when the goal check passes, the compliance check passes, no
open item remains, and one extra full verification step is clean: the Gate is
never opened on a single clean pass. A finding in that extra pass is not a Gate
failure - it returns to the action step like any other finding, and re-runs the
verification step, the adversarial sweep over the new state and the extra pass
again before the Gate is retried. "One" means the extra pass is not chained after
a clean result, not that it runs only once. If an open item can be closed
reasonably, it is closed; if it cannot, it is documented for review and the run
does not push.

## File map

| File | Contents |
|---|---|
| `SKILL.md` | The process: target declaration and input forms, principles, classification, open-item policy, Phases 0-7, the loop rule, the Gate, stop conditions |
| `reference/capabilities.md` | The eight conditional capabilities - methodology framework, delegation, production mode, goal tracking, delivered artifacts, reference artifacts, input findings, test runner - with the Phase 0 probe sequence, mandatory behavior and fallbacks |
| `reference/coverage.md` | Coverage as three coverage passes plus regression and the production workload: fourteen category rows - thirteen across the three coverage passes plus the regression row - with what closes each, how to sample, and which technique tests which claim |
| `reference/report-template.md` | The reporting template the Gate is checked against |
| `examples/findings-driven-run.md` | A condensed worked run showing the loop catching findings its own fixes created |

## Design notes

- **Evidence over assertion.** Every claim in the report traces to a command,
  log, diff or output.
- **Independence.** A change is verified by an agent that did not implement it
  and holds no other role; where delegation is unavailable the declaration
  records each role as a self-held degradation with its reason and escalates
  under Stop conditions - no role is closed by a separate pass or by any agent
  holding another role.
- **No silent deferral.** An open item is closed or documented - never dropped.
- **Process compliance is itself graded.** An unmet requirement is a finding,
  and it blocks the Gate like any other - except a degradation the declaration
  records and the operator has explicitly accepted, which closes its compliance
  finding. For a degradation other than an implementation or context-isolation
  one it leaves the run "not fully achieved"; a revision produced under an
  accepted implementation or context-isolation degradation still needs a clean
  pass by an agent holding no other role before the Gate, or the run stays open
  and escalates.
- **Read-only means read-only.** Reference, delivered and production artifacts
  are never disturbed, and nothing is copied out of them unsanitized.

## License

MIT - see `LICENSE`.

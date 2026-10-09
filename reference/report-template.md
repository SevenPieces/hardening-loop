# Reporting template

The report is the deliverable of a hardening run and the artifact the Gate's
checks are recorded in and that the reviewer checks. It precedes any push. Its
Gate-facing sections - §2 scorecard, §3 capability declaration, §6 compliance
summary, §13 open-item log, §14 sensitive-information scan, §16 coverage matrix
- are drafted as their evidence is produced and are checked at the Gate; §11 is
completed after the local merge, or after the change-sets land when there is no
branch (see
`SKILL.md` Phases 6 and 7). Fill every section; where a section is genuinely not
applicable, write "not applicable" and the reason rather than deleting the
heading.

## 1. Goal statement, exit criteria, non-goals, stopping rule
The goal in one or two sentences; the exit criteria as a checklist; the
non-goals and out-of-scope items; the stopping rule.

Goal: the harness goal's id, the objective it was armed with, and its final
state (achieved, pending review, or not fully achieved). When the Gate opened,
note that it closed only once the local merge covered the approved revision - or
once the change-sets had landed when there is no merge target; when the run ended
"pending review" or "not fully achieved", the goal closed in that state and no
merge exists. When goal tracking was absent, write "not applicable" and where the
goal lived instead.

## 2. Exit criteria scorecard
| Criterion | Pass / Fail | Evidence |
|---|---|---|
Flag anything not fully met. A partial criterion is a fail.

## 3. Capability declaration
The table emitted in Phase 0 - one row per capability, with the probe run, the
result (present / present-unexercised / absent / inconclusive), the behavior used (the mandatory
behavior, or the fallback and why), and the evidence that decided it. This is
what makes the capability claims in the report checkable.

## 4. Methodology / skill usage log
| Skill | Purpose | Step(s) | Inputs | Outcome |
|---|---|---|---|---|
Covers the detected methodology framework and any skills named by `skills=`. Any
skipped skill, with reason and remediation.

## 5. Delegation log
| Role | Agent | Phase | Isolation | Task | Inputs | Outputs | Landing point |
|---|---|---|---|---|---|---|---|
"Isolation" records whether the delegate was a fresh agent; a delegate that
continues the implementer's context is a context-isolation degradation
(independence only nominal), recorded with its reason and the Gate's compliance
finding - never a normal isolation value. An accepted context-isolation
degradation records that the role's independence is nominal and is never counted
as an independent pass; the revision it produced still requires a clean pass by
an agent that holds no other role before the Gate. The Agent column records which agent
staffed which role. Each pass names the revision it covered - the
commit id, or the manifest of file hashes that stands in for one - together
with the agent that ran it; a pass without a named revision is recorded as
covering nothing. List any required role that was not delegated, every role the
run could not staff separately - with the agent that held it, the reason, and
the Gate's compliance finding - and any delegate that failed or returned
nothing, each with reason and remediation. An unaccepted degradation is raised
as a process-compliance finding in this run, with the next ID in the single
§7 / §9 sequence, and blocks the Gate as a compliance finding. A degradation
the operator has explicitly accepted is recorded as accepted, with no blocking
compliance finding; it is cited as the reason for the run's "not fully achieved"
state.

## 6. Process-compliance summary
Pass/fail against goal setting, methodology use and delegation, including the
capability declaration in §3; the findings this raised, and their remediation.
A check satisfied only because an accepted degradation is recorded in §5 is
reported as "passed with a recorded accepted degradation", not as a pass, and is
named here as the reason the run is "not fully achieved".

## 7. Triage table
The backlog as triaged: the input findings (mode B) and the reconnaissance-derived
candidates that became defects at triage - mode A derives them from
reconnaissance, and a mode B run derives them after triage. Derived candidates
with no defect at triage time are carried downstream and recorded with the
findings they produce, not as section 7 rows; only input findings and triaged
defects become backlog rows. Defects discovered later, while working the backlog,
belong in §9.

Findings carry one ID scheme across the whole report: a single sequence, never
per-source prefixes. §7 and §9 share the sequence; an item that moves between
them keeps its ID.

| ID | Source | Description | Severity | Origin | Root cause | Reproduction | Fix | Regression test | Status |
|---|---|---|---|---|---|---|---|---|---|

## 8. Scope coverage
What was examined, what was not, and why. Mode B produces this too: after
triage it derives the reconnaissance-derived backlog that mode A would have
produced.

## 9. Finding log
Every finding this run produced or carried that is not in the backlog triaged in
§7 - one row each: regressions against the baseline,
issues discovered during the run, anything introduced by this run's own fixes,
and process-compliance findings.

Findings carry one ID scheme across the whole report: a single sequence, never
per-source prefixes. §7 and §9 share the sequence; an item that moves between
them keeps its ID.

§9 is a table with one row per finding, as §7 is. Candidates derived during
Phase 0-3 reconciliation belong to §7; findings produced after triage - while
working the backlog, by the adversarial sweep, by this run's own fixes, and any
process-compliance finding - belong to §9. A prose summary instead of the table
is a template non-conformance.

| ID | Source | Description | Severity | Origin | Root cause | Reproduction | Fix | Regression test | Status |
|---|---|---|---|---|---|---|---|---|---|

Tally the rows by origin beneath the table, so a reviewer sees the regressions
and the newly discovered items at a glance. The five origin tags are input
finding, regression against baseline, newly discovered, introduced this run, and
process compliance; the tally covers both the §7 and the §9 rows.

## 10. Summary of all changes
The substantive changes, grouped so a reviewer can follow them.

## 11. Branch / worktree, commits, files changed
The branch the work was done on, the merge target it was landed on - `main`
unless the declaration named another - the baseline identity recorded in Phase 0
(a revision, or the manifest of file hashes that stands in for one, together with
whether the tree was dirty at Phase 0 when the target is under version control),
the commits in order, and the files touched. When the target has no branch to merge into,
write "no merge target" and list the applied change-sets and the baseline
identity in place of the branch and the commits.

## 12. Verification and adversarial sweep evidence
Including the baseline verification, each adversarial sweep and its re-runs, the
Gate's extra pass, the repeated-execution simulation and the independent
verifier, with the commands and their output.

Each pass names the revision it covered (commit id, or the manifest of file
hashes) and the agent that ran it; a pass without a named revision is recorded as
covering nothing.

## 13. Open item log
Open items are not findings and do not take finding IDs; they are identified by
their §13 row order or a distinct prefix.

The Closure attempt column records which of the three methods `SKILL.md` Phase 6
requires - reproduction, recorded experiment, or blocking constraint - applies to
the row, and the artifact that shows it; a pending-review row without one is a
template non-conformance.

| ID | Description | Severity | Category | Reason unresolved | Impact | Attempted fixes | Closure attempt | Workaround | Recommendation | Next action | Decision needed |
|---|---|---|---|---|---|---|---|---|---|---|---|

## 14. Sensitive-information scan result
The target, the diffs, the read-only inputs, the input findings files, and every
delegate's returned evidence and report text: what was scanned, what was found,
what was excluded, removed or redacted.

## 15. Read-only input integrity
Confirmation that read-only inputs were untouched and uncommitted.

## 16. Coverage matrix
The matrix from `reference/coverage.md` - the category rows and the change-set
block (a "nothing to verify yet" row before any change exists, one row per
changed behaviour after) - with columns
category, covered by, technique, outcome, closing evidence; the areas that remain
untested are named rather than implied, and the negative results are recorded.

## 17. Deliverable and environment integrity
Confirmation that production, delivered artifacts and schedules were untouched,
and that no leftovers remain - temp files, processes, branches, state or side
effects.

## 18. Residual risks and decisions made
What remains uncertain, and the decisions taken where the requirement was open.

## 19. Local merge status and the exact push command(s)
The branch the work was merged back to, the local merge state, and the exact
push command(s) - `git push`, a package or image publish, a deployment - pending
approval. When there is no branch to merge into, record "no merge target" and
the applied change-sets in place of the merge state; when there is no branch or
remote to push to, record "no push target" in place of the command(s).

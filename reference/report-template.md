# Reporting template

The report is the deliverable of a hardening run and the artifact the Gate's
checks are recorded in and that the reviewer checks. It precedes any push. Its
Gate-facing sections - §2 scorecard, §3 capability declaration, §6 compliance
summary, §13 open-item log, §14 sensitive-information scan - are drafted as
their evidence is produced and are checked at the Gate; §11 is completed after
the local merge, or after the change-sets land when there is no branch (see
`SKILL.md` Phases 6 and 7). Fill every section; where a section is genuinely not
applicable, write "not applicable" and the reason rather than deleting the
heading.

## 1. Goal statement, exit criteria, non-goals, stopping rule
The goal in one or two sentences; the exit criteria as a checklist; the
non-goals and out-of-scope items; the stopping rule.

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
| Role | Phase | Isolation | Task | Inputs | Outputs | Landing point |
|---|---|---|---|---|---|---|
"Isolation" records whether the delegate had inherited context - a fresh agent,
or a continuation of the implementer. List any required role that was not
delegated, every merge of one role into another, and any delegate that failed or
returned nothing, each with reason and remediation.

## 6. Process-compliance summary
Pass/fail against goal setting, methodology use and delegation, including the
capability declaration in §3; the findings this raised, and their remediation.

## 7. Triage table
The backlog as triaged: the input findings (mode B) and the reconnaissance-derived
candidates - mode A derives them from reconnaissance, and a mode B run derives
and adds them to this table after triage. Defects discovered later, while working
the backlog, belong in §9.

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

| ID | Source | Description | Severity | Origin | Root cause | Reproduction | Fix | Regression test | Status |
|---|---|---|---|---|---|---|---|---|---|

Tally the rows by origin beneath the table, so a reviewer sees the regressions
and the newly discovered items at a glance. Origin is one
of the tags the process defines: regression against baseline, newly discovered,
introduced this run, or process compliance. Input findings belong to §7.

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

## 13. Open item log
| ID | Description | Severity | Category | Reason unresolved | Impact | Attempted fixes | Workaround | Recommendation | Next action | Decision needed |
|---|---|---|---|---|---|---|---|---|---|---|

## 14. Sensitive-information scan result
The target, the diffs, the read-only inputs, and the input findings files: what
was scanned, what was found, what was excluded, removed or redacted.

## 15. Read-only input integrity
Confirmation that read-only inputs were untouched and uncommitted.

## 16. Test coverage summary
The matrix from `reference/coverage.md` - category, covered by, technique,
outcome, closing evidence - with the areas that remain untested named rather
than implied and the negative results recorded.

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
the applied change-sets in place of the merge state.

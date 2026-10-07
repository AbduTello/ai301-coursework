# Procedure: how this skill grades a plan package

Follow these steps in order. Each step says what to record; later steps
use only what was recorded, plus a re-read of the named section when a
step says to.

## Read order

1. Read Repro evidence first, before the plan. Record: (a) the behavior
   it reproduces, in one line; (b) each Control run and what it varied
   and showed; (c) any step that shows where the failure first appears;
   (d) the Expected line. Reading the evidence first matters: once the
   plan's diagnosis has been read, it is easy to read the controls as
   supporting it.
2. Read Issue, then Thread highlights. Record each comment from an
   OWNER, MEMBER, COLLABORATOR, or CONTRIBUTOR that names a culprit
   location, proposes, endorses, or rejects an approach, or posts a patch
   or test build. Record any confident causal claim from anyone, so that
   step 5 can compare it with the evidence.
3. Read Repo facts. Copy the `- contribution policy` line exactly.
4. Read Candidate plan. Record: its stated cause; every proposed work
   item as a numbered list; the files, functions, or sites it names; its
   test plan, verbatim; and any risk, open question, or deferral.
5. Read Candidate plan comment. Record: any approach it names; any
   maintainer, patch, or direction it mentions; any mention of AI
   assistance; any promised date or bundled series.

In live mode, before step 1, read `scope.md` and refuse out-of-scope
issues. Then, in the same order: the student's posted repro comment, the
issue and its thread (`gh issue view <n> --comments`), the repo's
CONTRIBUTING.md and AI policy, `plan.md`, and the draft plan comment.
Read only what the drafts contain and quote, plus those GitHub sources.

## Evidence gathering

Each check uses the notes from Read order:

- `diagnosis-follows-repro`: notes 1(a)-(c), 2 (causal claims), and
  4 (cause). For each control, write one line: "if the plan's cause were
  true, this control would show ___; it showed ___."
- `change-bounded-to-issue`: note 4 (numbered work items) and note 5
  (bundled series). Mark each item as fix, test, doc-for-fix, deferred, or
  extra.
- `stranger-can-start`: note 4 (sites named) and whether the approach
  picks one option.
- `test-observes-the-fix`: note 4 (test plan verbatim) against note 1(d)
  (Expected).
- `comment-answers-maintainer-direction`: note 2 (maintainer direction)
  against note 5.
- `ai-disclosure-when-policy-requires`: note 3 (policy line) against note
  5 (AI mention).
- `unknowns-named`: note 4 (risks and deferrals).

If a section the check needs is missing from the package, record
"absent" for it. Do not fill it in from elsewhere: the issue body cannot
stand in for a missing test plan, and the plan cannot stand in for a
missing comment.

## Check execution

1. Run the required checks in the order in `rubric.md`, then the preferred
   check. Grade every check, even after a fail: the output lists them all.
2. For each check, apply its pass condition to the recorded notes only.
   Re-read the package section a check names when the notes leave a
   specific fact open; do not re-read the whole package.
3. Apply the rubric's enumerated fail conditions literally, then its
   explicit "does NOT fail" carve-outs. A carve-out only applies when its
   own wording matches the package.
4. Absent evidence: grade `unclear`, and the verdict rule turns it into a
   fail. Explicitly negative evidence ("no stated AI policy", a thread
   with no maintainer-role comments, a thread with no direction) is not
   absent; grade the check on it.
5. For the evidence line of each check, quote or name the single fact
   that decided it (a control's result, the extra work item, the test
   plan's words, the maintainer comment, the policy clause).
6. Never let one check's result change another's. A plan that follows
   maintainer direction still fails `diagnosis-follows-repro` when the
   repro rules that direction out, and an excellent plan still fails the
   disclosure check when the policy requires disclosure.

## Verdict assembly

1. Apply the rubric's verdict rule: `accept` only if every required check
   is `pass`; any required `fail` or `unclear` gives `reject`.
2. Preferred check grades never enter the verdict.
3. In the readable summary, name every required check that failed and
   quote its deciding evidence; for an accept, say so in one line.
4. Emit the JSON block from SKILL.md last, with every check in rubric
   order, each `evidence` set to the deciding fact from Check execution
   step 5.

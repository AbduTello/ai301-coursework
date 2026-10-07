# Evidence guide: where evidence lives in a plan package

This is the map for every check in `rubric.md`. Each family says where to
look in an eval bundle, where to look live, and what good looks like there.
The bundle sections are named by their headings: Repo facts, Issue, Thread
highlights, Repro evidence, Candidate plan, Candidate plan comment.

## Diagnosis and grounding

- Where it lives (bundle): the cause is in the Candidate plan, usually a
  "Diagnosis" line, and also implied by the code location its approach
  edits. The behavior that cause must explain is in Repro evidence: the
  numbered steps, every "Control" run, and the "Actual" line. The Issue
  body and Thread highlights may also state a cause, but those are claims,
  not evidence.
- Where it lives (live): the cause is in the student's `plan.md`; the
  evidence is the student's own posted repro comment on the issue (or, for
  the house issue, the repro pack as quoted in the drafts).
- What good looks like: the named cause explains every step and every
  control. Test it this way: if the plan's cause were true, would each
  control have come out the way the repro says it did? A control that
  succeeds with the blamed component still in the path, or a step showing
  the damage already done before the blamed code runs, means the
  diagnosis is wrong, however confident the plan or the thread sounds.

## Scope

- Where it lives (bundle): the Candidate plan's "Scope", "In scope / Not in
  scope", "Proposed changes", "Files and areas", and "Approach" sections,
  and the Candidate plan comment's summary of the work (comments often
  reveal a bundled series: "starting with the migration").
- Where it lives (live): the same sections of `plan.md` and the draft plan
  comment.
- What good looks like: one bounded change. List every proposed work item
  and ask, for each, "is this required to make the repro's Expected line
  true, or to test it?" The fix, its regression test, and a doc line about
  it pass. Migrations, upgrades, new options, settings fields, rewrites,
  unifications, new frameworks, and CI changes fail. A not-in-scope line
  that defers related work with a reason is a strength, not a gap.

## Executability

- Where it lives (bundle): the Candidate plan's "Files", "Approach",
  "Change", or "Scope" sections: file paths, module names, functions, code
  branches, and the numbered steps.
- Where it lives (live): the same in `plan.md`.
- What good looks like: at least one concrete site (path, function, or
  named branch) and one chosen approach, so a stranger could make the first
  edit today. Red flags are question marks between alternatives, "somewhere",
  "whichever is easier", "investigate", and "profile" standing in for an
  approach.

## Test plan

- Where it lives (bundle): the Candidate plan's "Test plan" section, read
  against the Repro evidence's steps and "Expected" line.
- Where it lives (live): `plan.md`'s test plan, read against the posted
  repro comment.
- What good looks like: an outcome that is false now and will be true after
  the fix: the repro command re-run with its expected output or exit code, a
  regression test made from the issue's own case, or a stated measurement.
  "Run the full suite", "nothing regresses", and feelings are not
  observables for the fix.

## Honesty

- Where it lives (bundle): "Risk" or open-question lines in the Candidate
  plan, deferral sentences in its scope, and hedges or certainty claims in
  the Candidate plan comment.
- Where it lives (live): the same in `plan.md` and the draft comment, plus
  `plan.md`'s Deviations section after a build. A deviation recorded there
  is honest; a deviation that only shows up in the diff is not.
- What good looks like: what was not verified is named with what will be
  done about it ("cannot test the Windows variant, leaving it out and will
  flag it in the PR"). False confidence looks like a certain cause the repro
  did not show, or a promised date.

## Comms

- Where it lives (bundle): two places, read against the Candidate plan
  comment. (1) Thread highlights: each line gives a date, an author, and the
  author's role in parentheses. Only OWNER, MEMBER, COLLABORATOR, and
  CONTRIBUTOR comments can carry maintainer direction. (2) Repo facts: the
  `- contribution policy` line, including any AI policy, its modal verbs
  ("must", "welcome", "expected"), and the artifact it applies to
  ("in the pull request", "comments", "in any form").
- Where it lives (live): the issue thread on GitHub (`gh issue view <n>
  --comments`, author association on each comment) and the repo's
  CONTRIBUTING.md and any AI_POLICY.md.
- What good looks like: when a maintainer has named a culprit, proposed or
  rejected an approach, or posted a patch to test, the comment visibly
  builds on it, or names it and says why it departs. When the policy puts a
  disclosure duty on comments or on "all AI usage in any form", the comment
  names the AI assistance; when the duty is scoped to PRs, or there is no
  duty, silence is fine.

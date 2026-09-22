# Rubric: is this a good first issue?

Every check below names a field I can point at and a threshold someone else
could apply to the same field and reach my answer. Recency thresholds are
measured against the bundle's own `captured:` date in eval mode, and against
today in live mode.

## Checks

| Check | Evidence | Pass condition | Weight |
|---|---|---|---|
| `repo-not-archived` | The `archived:` flag on the `- repo:` line of the repo-facts block. | Pass if `archived: no`. Fail if `archived: yes` — an archived repo is read-only and cannot accept a pull request at all, whatever else looks healthy. | required |
| `recent-commits` | The `- last 5 default-branch commits:` list in repo-facts, compared against the `- captured:` date at the top of the bundle. Read all five dates, sort them, and take the newest; the list is often NOT in date order, so never just read the first entry. Do not substitute `last push to any branch`, which can be newer than any default-branch commit. | Compute the explicit day count from the newest commit date to the capture date and state that number in the evidence line. Pass only if that number is 180 or less. Fail if it is greater than 180, however healthy the repo looks otherwise: a large star count, an open issue tracker, or recent README-only edits are not commits. Worked example, and note the list order: dates 2023-02-01, 2025-02-10, 2025-02-10, 2025-02-10, 2023-08-22 against capture 2026-08-05 give a newest of 2025-02-10, which is 541 days, so this check FAILS. Commits by authors ending in `[bot]` count toward liveness only when at least one of the five is human-authored or is a bot merging a human's pull request. | required |
| `scope-bounded` | The issue title and body, plus any comments whose author_association is OWNER, MEMBER, or COLLABORATOR. | Fail if any of these hold. (a) The body is a list of three or more linked sub-issues, or calls itself a tracking, umbrella, or mega issue. (b) The work is described across the whole codebase rather than a named file, module, or behavior. (c) The design or product decision is still unsettled at capture: no maintainer has stated a concrete agreed approach, or a maintainer asked what the feature should do and never got a settled answer. (d) It is a usage or support question rather than a change request. (e) The issue has a record of repeated abandonment: two or more linked pull requests in state `(closed)` and unmerged, or a comment thread showing three or more different contributors claiming it and dropping off. Work that several people started and none finished is not newcomer-sized, whatever the label says. Otherwise pass. A terse body, a missing reproduction, a checklist of acceptance criteria, an old creation date, or a single closed pull request is NOT a scope failure on its own. | required |
| `unclaimed` | The `- this issue: assignees: ...; linked PRs: ...` line in repo-facts, plus claim language in the comment thread. | Fail if any of these hold. (a) `assignees:` names anyone. (b) `linked PRs:` contains a pull request in state `(open)`. (c) A maintainer states in-thread that a specific person's pull request is the one being taken. (d) Two or more distinct non-maintainer commenters claimed the issue and none was answered or released. Linked pull requests that are all `(closed)` or `(merged)` do not fail, and neither does a lone claim older than 365 days that a maintainer has since reopened to other contributors. | required |
| `policy-permits-ai-assist` | The `- contribution policy` line in repo-facts, which names its source: CONTRIBUTING.md, linked contributor docs, or a dedicated AI policy file. | Fail ONLY on an outright ban: the policy refuses AI-generated code or documentation and offers no carve-out for reviewed, understood, or merely assistive use. Pass when the policy sets conditions rather than a ban — disclose, human-review, personally understand, test, "fully AI-generated contributions are not accepted", "unreviewed AI pull requests are closed" — because conditions are terms to follow. Pass when the policy is silent or no CONTRIBUTING.md exists. | required |
| `maintainer-responsiveness` | The `- maintainer first-response sample` list in repo-facts. | Prefer issues where at least one sampled thread shows a maintainer first response within 30 days. Never changes a verdict: healthy repos in this set routinely show "no maintainer comment in thread" across most sampled issues, so a slow sample ranks an issue lower without rejecting it. | preferred |
| `release-recency` | The `- latest release:` line in repo-facts, compared against the `- captured:` date. | Prefer a release published within 365 days of capture. `latest release: none published` is not a defect when the commit list is active; plenty of healthy projects never cut tags. | preferred |
| `task-is-specified` | The issue body: named files or paths, acceptance criteria, reproduction steps, or a maintainer-stated fix. | Prefer issues that name the files to touch or give a concrete expected behavior. These are faster to land as a first contribution, so they rank above equally valid but vaguer issues. | preferred |
| `newcomer-signal` | The `labels:` on the issue's `opened by ...` line, and the opener's author_association. | Prefer a `good first issue`, `help wanted`, or `easy` label, or an issue filed by an OWNER, MEMBER, or COLLABORATOR. A friendly label is a ranking signal only: it is the maintainer's claim that the issue is approachable, never evidence that it is unclaimed. | preferred |

## Verdict rule

`accept` if and only if every `required` check grades `pass`. A single
`required` fail produces `reject`, and the read-out names the check that
sank it.

`unclear` on a required check counts as a fail. An issue whose liveness,
scope, claim status, or contribution policy cannot be established from the
evidence in front of me is not a safe place to spend a first contribution.
One exception: a field that is present and explicitly negative is evidence
of absence, not absent evidence, and grades `pass`. That covers
`assignees: none`, `linked PRs: none`, `latest release: none published`,
and `no CONTRIBUTING.md in the repo; no stated policy`.

`preferred` checks never change a verdict. Grade them, report them, and use
them to rank the issues that were accepted: among accepted candidates,
prefer the one with more `preferred` passes, breaking ties on
`task-is-specified` first, then `newcomer-signal`.

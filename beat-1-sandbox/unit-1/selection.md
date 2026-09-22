# Unit 1 — Issue Selection

## Chosen issue

**Issue:** https://github.com/codepath/pathreview-ai301-fa26-s1/issues/61
**Title:** Health check DB probe passes a raw SQL string, which fails under SQLAlchemy 2.x
**My skill's verdict:** `accept` (ranked #1 of 3 accepted candidates)

I ran the skill in live mode on three open issues: #72, #61, and #62. It
accepted all three and ranked #61 first.

Live-mode verdict block for the chosen issue:

```json
{
  "item": "https://github.com/codepath/pathreview-ai301-fa26-s1/issues/61",
  "checks": [
    {"name": "repo-not-archived", "grade": "pass", "evidence": "isArchived: false"},
    {"name": "recent-commits", "grade": "pass", "evidence": "Most recent commit 2026-09-16, 6 days before capture (2026-09-22)"},
    {"name": "scope-bounded", "grade": "pass", "evidence": "Single specific bug in api/routes/health.py, concrete error message and fix approach stated"},
    {"name": "unclaimed", "grade": "pass", "evidence": "assignees: none; linked PRs: none; no comments"},
    {"name": "policy-permits-ai-assist", "grade": "pass", "evidence": "No CONTRIBUTING.md found; no stated policy"},
    {"name": "maintainer-responsiveness", "grade": "pass", "evidence": "No maintainer comments in sampled issues 70, 69, 68, typical for healthy repos"},
    {"name": "release-recency", "grade": "pass", "evidence": "No releases published; recent commits confirm active development"},
    {"name": "task-is-specified", "grade": "pass", "evidence": "Names file api/routes/health.py, exact error text, and fix (wrap in text())"},
    {"name": "newcomer-signal", "grade": "pass", "evidence": "Labels: good first issue, tier-1, bug, api; opened by OWNER Aburke225"}
  ],
  "verdict": "accept"
}
```

**Why I picked it over the other two:**

- It is a backend fix, which is the area I said I wanted to get stronger in.
- It names the exact file and the exact error, so there is no guesswork.
- Nobody else was on it: no assignee, no linked PR, no comments.

#62 was also accepted, but a TF had already reproduced it and said a PR was
coming. The house rule says classmate claims don't block me, but there was no
reason to duplicate live work. #72 was a close second and is my backup.

I have not commented on the issue. Choosing is not claiming.

---

## Run history

Six runs, in order.

**1. Smoke run (`--limit 3`).** I wanted to know my table parsed before spending
$4. Result 3/3. The harness also refused to write a run file, which is what a
partial run is supposed to do.

**2. Full run #1 — 19/20, PASS.** Every category matched. One miss: issue-15. My
rubric said accept, gold says reject.

**3. `--only issue-15,issue-09` (cheap re-grade).** I opened the issue-15 bundle.
It has 97 comments, five years of discussion, and two closed PRs that never
merged, all sitting under a `good first issue` label. My `scope-bounded` check
was only looking for an unsettled design, and a maintainer *had* discussed an
approach, so nothing fired. I added a new clause for repeated abandonment.

I re-graded issue-09 in the same run on purpose. issue-09 is a gold **accept**
that is also old and also has a closed PR, so it was the issue most likely to
break from my new clause. Result 2/2: issue-15 now rejects, issue-09 still
accepts.

**4. Full run #2 — 19/20, PASS.** issue-15 was fixed, but now issue-07 flipped
the wrong way. Details in the next section.

**5. `--only issue-07,issue-02,issue-17,issue-06`.** The three dead repos, plus
issue-06. I included issue-06 because at 22 days it is the tightest accept in
the set, so a stricter date rule would break it first. Result 4/4.

**6. Full run #3 — 20/20, PASS.** All categories matched. This is the run saved
in `eval-run.txt`.

The two cheap re-grades cost about $1.20 total. They replaced two full runs I
would have spent confirming small edits.

---

## Issue analysis

**issue-07** (`wting/autojump#727`, category `dead-repo`)

| | |
|---|---|
| Gold label | **reject** |
| My rubric, run #2 | **accept** — wrong |
| My rubric, run #3 | **reject** — correct |

This repo looks alive at a glance. It has 16,956 stars, it is not archived, it
has 231 open issues and PRs, and its last push was 2025-02-27.

The signal that actually matters is the default-branch commit list, and it has
two traps:

1. **It is not in date order.** It starts with `2023-02-01`, and the newest
   entry (`2025-02-10`) is buried in the middle.
2. **Three of the five commits are just "Update README.md."** Docs edits are not
   development.

The capture date is 2026-08-05, so the newest commit is **541 days old** — way
past my 180-day limit.

Here is the part I found interesting. My original wording already said to use
the newest date in the list and to ignore `last push`. The rule was correct and
it *still* got the answer wrong. Asking "is this within 180 days?" invites a
gut feeling instead of a subtraction.

So I did not change the threshold. I removed the judgment call: the check now
requires writing the actual day count into the evidence line. Once `541` is on
the page, comparing it to 180 is mechanical. The rule was never the problem —
the arithmetic just wasn't getting done.

---

## Check rationale

This is my `recent-commits` check, quoted from the `rubric.md` in this repo.

**Evidence column:**

> The `- last 5 default-branch commits:` list in repo-facts, compared against
> the `- captured:` date at the top of the bundle. Read all five dates, sort
> them, and take the newest; the list is often NOT in date order, so never just
> read the first entry. Do not substitute `last push to any branch`, which can
> be newer than any default-branch commit.

**Pass condition column:**

> Compute the explicit day count from the newest commit date to the capture
> date and state that number in the evidence line. Pass only if that number is
> 180 or less. Fail if it is greater than 180, however healthy the repo looks
> otherwise: a large star count, an open issue tracker, or recent README-only
> edits are not commits. Worked example, and note the list order: dates
> 2023-02-01, 2025-02-10, 2025-02-10, 2025-02-10, 2023-08-22 against capture
> 2026-08-05 give a newest of 2025-02-10, which is 541 days, so this check
> FAILS. Commits by authors ending in `[bot]` count toward liveness only when
> at least one of the five is human-authored or is a bot merging a human's
> pull request.

Three things in there are deliberate.

**Why 180 days.** The eval set leaves a huge gap. The tightest gold accept is
issue-06 at 22 days. The closest gold reject is issue-07 at 541 days. Nothing
sits in between, so anywhere in that range works, and 180 sits comfortably in
the middle. It catches every dead repo while letting a project that had one
quiet month still pass. If I dropped below about 30 days, I would start
rejecting issue-06 by mistake.

**Why the check names a field it must NOT use.** `last push to any branch` is
the more obvious liveness signal, and it is misleading. On issue-07 it reads
2025-02-27, newer than any real commit, because pushing to an abandoned side
branch still counts. Naming the trap is what keeps the check away from it.

**Why the worked example is in the table and not a comment.** The harness strips
HTML comments before the grader ever sees the file. An example inside a comment
would simply vanish.

---

## Trade-offs

**Making `maintainer-responsiveness` preferred instead of required.**

"Is the maintainer alive?" is one of the four families from the lecture, so
leaving it out of my required checks felt wrong at first. The data changed my
mind. Response time in these bundles points the *opposite* way from the labels:

- Four of the eight gold accepts (issue-01, 09, 14, 16) show "no maintainer
  comment in thread" on most sampled issues.
- issue-17 — archived, no commits in 1,726 days, an obvious reject — has the
  **fastest** response times in the whole set.

Any response-time rule strict enough to catch the dead repos would have
rejected half my accepts and still missed issue-17. So responsiveness now ranks
the issues I accept instead of gating them, and `repo-not-archived` plus
`recent-commits` carry the liveness decision.

The cost is real. A repo that keeps committing but has quietly stopped reviewing
PRs would slip through. I decided that is survivable for a first contribution,
where a rule that rejects most healthy repos is not.

**How far to push the issue-15 lesson.**

The easy takeaway was "old issue plus a closed PR means trouble." That would
have been a mistake. issue-09 is a gold **accept** that is eight years old, has
a closed unmerged PR, and is labeled `stale::recovered` and `backlog`. A rule
that blunt would have fixed one issue and broken another.

The real difference is repetition, not age:

| | issue-15 (reject) | issue-09 (accept) |
|---|---|---|
| Closed unmerged PRs | 2 | 1 |
| Comments | 97 | 4 |
| Pattern | contributors claiming and dropping off for 5 years | one old attempt |

So my clause requires **two or more** dead PRs or **three or more** contributors
cycling through, and it says outright that an old date or a single closed PR is
not a scope failure by itself. Before the next full run I checked every gold
accept in the set: issue-09 is the only one with even one closed linked PR, so
nothing else is exposed to this clause.

# Evidence guide: where proof lives in a reproduction package

Every check in `rubric.md` names evidence. This file says where that
evidence lives, in an eval bundle and on GitHub, and what good looks like
once you are there. The test of a line in this guide is whether a stranger
holding the same package would land on the same text I did.

One rule runs through all five families: the artifact is the evidence, and
the prose around it is not. When a report's words and its output disagree,
the output wins, and the confidence of the words counts for nothing.

## Environment

**Where it lives.** In an eval bundle: the `Environment:` line at or near
the top of the `## Candidate repro report` section, and nowhere else — a
version mentioned only inside a command's output does not count as a
record, though it corroborates one. Read it against two other places in
the same bundle: the `## Issue` body, where the reporter states what they
ran (often in a versions block or a template field), and the
`- latest release:` line in `## Repo facts`, which tells you how far back
the report's version sits. The `## Repo facts` block also quotes the
repo's bug-report template asks; when that template names a field, the
issue treats that axis as load-bearing.

In live mode: the draft report's own environment line, the issue's
rendered template fields on the issue page, the repo's
`.github/ISSUE_TEMPLATE/` for what it asks contributors to state, and the
Releases sidebar for the current version.

**What good looks like.** The record names the version of the tool under
test and the OS/platform, and it names every axis the issue itself treats
as decisive — the driver on a driver-scoped bug, the shell on a shell-scoped
bug, the browser language on a localization bug, the build profile where
debug and release fail differently, the `ARG_MAX` value where the bug is
about argument size. Install method (pip, Homebrew, cargo, distro package,
release binary) is worth having and is not required on its own.

A version different from the issue's target is fine when it is named out
loud: "filed against 13.0.0; behavior is unchanged on 15.2.0" and "the
report is macOS + fish; shell and OS both differ, starship version matches"
are both good records. The same deviation left unstated is the defect —
an environment record that quietly sits several major versions behind an
issue confirmed on latest and main is showing you that old version's
behavior, not the reported bug. Stating the delta is what makes the record
honest; hiding it is what makes the artifact unreadable.

## Steps

**Where it lives.** In an eval bundle: the numbered or narrated steps in
the `## Candidate repro report`, read against the reproduction steps in
the `## Issue` body and any refinement a commenter added in
`## Thread highlights` (a maintainer who says "you also need `--cleanup`"
has told you what the trigger really is).

In live mode: the draft's steps, the issue's own steps, and the repo's
quickstart or contributing docs for what a clean starting state is.

**What good looks like.** A stranger with the same tool and no access to
the author's machine can execute every step and arrive at the same place.
Concretely: the starting state is obtainable (a fresh clone, a published
package, a minimal file quoted inline), the commands are given exactly as
run rather than described, and the step that actually triggers the bug is
present rather than assumed.

The sharpest failure is a step gated behind something nobody else has: a
private monorepo, an unshared internal config, a company pre-commit hook.
It is still a failure when the author explains why they cannot share it,
because the explanation does not make the steps runnable. What rescues
such a report is a minimal shareable substitute — and note that an author
who ran a scratch-module control already had the material for one.

Short is not unfollowable. Four steps and a quoted config can be complete.
And an honest gap is not a defect: "I did not find a knob to force a
smaller limit from the CLI" tells the reader exactly what was not covered,
which is the opposite of a silent hole.

## Behavior shown

**Where it lives.** In an eval bundle: the fenced output blocks, log
excerpts, transcripts, and described screenshots inside the
`## Candidate repro report`. Read each against the `## Issue` body's own
fenced blocks — its error text, its exit code, its symptom description —
and against the exact invocation the issue names as the trigger.

In live mode: the draft's pasted output, and the issue's rendered code
blocks and attached images.

**What good looks like.** The artifact shows the issue's behavior, not an
adjacent one. Four ways an artifact goes wrong, all of them visible
without leaving the package:

- **Wrong trigger.** The command, expression, or input differs from the
  one the issue names. An issue that triggers on the offset-from-end
  range `:-N` is not tested by a prefix range `N:`; an issue about a
  runtime path error is not tested by an expression edited until it
  produces a compile error. The edit is usually small and usually
  unremarked.
- **Wrong failure mode.** A graceful validation error with exit 1 is not
  the panic with exit 101 the issue reports — it is often precisely the
  fix the issue is asking for. A terminal that garbles output and keeps
  its prompt has not crashed.
- **Setup shown instead of symptom.** A version banner, a session list,
  and "all the tabs are visible" prove the tool runs. If the symptom is a
  blank unresponsive pane, none of that is the symptom.
- **No artifact at all.** "Can 100% confirm", "it overflows just like the
  screenshots", "I verified the race condition" — with nothing shown,
  there is no evidence to read.

Repetition counts and machine counts are not evidence. "Ran it 10 times",
"on two separate machines", and "guaranteed reproducible" tell you how
sure the author feels, and a wrong artifact repeated ten times is still a
wrong artifact.

## Honesty

**Where it lives.** At the seam between two parts of the bundle: the
report's `Expected:` / `Actual:` lines, its conclusion, and its root-cause
language on one side; its own artifacts on the other. The
`## Candidate claim comment` belongs here too, because a claim that asserts
a reproduction the report does not carry is the same defect one section
earlier.

In live mode: the same seam inside the drafts, before either goes up.

**What good looks like.** The words claim no more than the artifacts show.
Two shapes pass that look like failures at a glance, and one fails that
looks like a success:

- **An honest cannot-reproduce passes, and is a real contribution.** It
  states the negative result plainly and up front rather than burying it;
  it shows an artifact of what *did* happen, so the reader can see the
  attempt was real; and it names what differed from the issue's conditions
  as a mechanism — "my shell reports PWD differently for symlinked
  directories than fish does on macOS", "fd appears to flush both command
  buffers at the same file-count boundary on this input". A negative
  result with a named mechanism narrows the bug for whoever picks it up
  next. A bare "couldn't reproduce, works for me" does none of this.
- **A labelled hypothesis passes.** "Which suggests", "may be required",
  "looks necessary" are how an author marks the boundary of what they
  demonstrated.
- **An unbacked diagnosis fails.** "The problem is a race condition
  between the debounced save and the note-switch handler ... I verified
  this" is a confident mechanism with no instrumentation behind it. So is
  an `Expected:` written backwards from what the artifact shows, then
  declared a confirmation.

## Comms

**Where it lives.** The `## Candidate claim comment`, read against the
`## Issue` title, body, and `## Thread highlights`; and the
`- contribution policy` line in `## Repo facts`, which names its source
(CONTRIBUTING.md, a linked AI policy file, the PR template) and summarizes
what it requires. The `## Repo facts` block also lists the bug-report
template's asks, which is the repo saying in its own words what a report
owes it.

In live mode: the draft comment, the issue page, `CONTRIBUTING.md` and
`.github/` in the repo, and any dedicated `AI_POLICY.md`. The policy is
often one click away from the repo root.

**What good looks like.** The claim comment could only have been written
about this issue: it names the symptom, the version, the file, the syntax,
or the thread comment at stake, and it states an intent the author can
actually carry out. A concrete next step — a named function, a linked
upstream fix to check, a file to read — is the shape that works. Enthusiasm
and brevity are both fine; two specific sentences beat six generic ones.

The failures are promises the author cannot keep and ownership they do not
have: "I will fix it within 2 days guaranteed", "keep this issue reserved
for me", "assigning myself to this". A bare +1 with no stated intent is a
comment that costs the thread attention and returns nothing.

**On AI disclosure, read the policy's modal verb and its object, never the
keywords.** The word "AI" appearing in a policy line does not create a
disclosure duty, and the absence of the word "disclose" does not rule one
out. Four things a policy line can be doing:

- **Saying nothing about AI.** "Standard contribution guide; no stated AI
  policy" is most repos. Nothing is owed, and disclosing anyway is
  neither required nor a defect.
- **Permitting with conditions.** "Generative AI tools welcome; you are
  responsible for all contributions", "assistive AI use is allowed and the
  contributor must understand and take responsibility", "only submit code
  you fully understand and have tested". These are terms of use, not a
  notification duty. Packages pass whether or not they disclose.
- **Requiring disclosure somewhere else.** A disclosure duty scoped to
  pull requests does not reach a claim comment; read the modal verb's
  object. A policy reading "must state the tool and the extent of its use
  *in the pull request*", whose line goes on to say there is no disclosure
  ask for issue comments, owes nothing on a package made of comments — even
  though its words look exactly like a disclosure requirement. This is the
  single easiest line in this guide to grade wrong.
- **Restricting who writes the comment.** "Comments to maintainers must be
  written by humans in their own words, and AI-generated comments may be
  hidden" governs voice, not disclosure. A human-voiced comment about this
  specific issue satisfies it with no disclosure statement.

Only the fourth-and-mandatory case bites: a policy stating that all AI
usage *in any form must be disclosed*, naming the tool and the extent, with
a scope that covers issues and comments. Against that line, a package whose
comments never mention a tool has walked through a wall the repo put up in
writing — and that is true however good the reproduction is. Treat course
work as AI-assisted: the question is never whether AI was used, only
whether this repo's stated policy obliges you to say so here.

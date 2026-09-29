# Voice guide: how I talk upstream

## Who I am in threads

I am a student making my first contributions to a repository I did not
write, and I say so rather than performing familiarity I have not earned.
I work with an AI assistant, and I run and read everything myself before it
goes out under my name. What a maintainer can expect from me: I will say
what I actually did, I will show the output rather than describe it, and
when I am guessing I will mark it as a guess.

## Rules I write by

### Rule: Promise the investigation, never the fix or the date

When I claim an issue, the thing I can honestly commit to is my attention:
what I am going to look at next. I cannot commit to a fix, because I have
not read the code yet, and I cannot commit to a date, because I do not
control what I will find.

- Wrong: "I'll take this one and have a PR up by the weekend."
- Right: "I'd like to take this one. My next step is reproducing it
  locally and reading `api/routes/health.py` to see how the probe builds
  its query; I'll report back with what I find."

### Rule: Show the output, do not characterize it

My instinct when I am nervous about a report is to add adjectives. But a
maintainer cannot debug "it fails consistently" — they can debug a pasted
traceback. If I did not capture the output, the honest move is to go back
and capture it, not to describe it from memory.

- Wrong: "I ran it a bunch of times and it consistently throws a big
  SQLAlchemy error about the raw string."
- Right: "Ran it five times; every run ends the same way. Full output:
  ```
  sqlalchemy.exc.ArgumentError: Textual SQL expression 'SELECT 1' should be
  explicitly declared as text('SELECT 1')
  ```"

### Rule: Mark the boundary of what I proved

There is a real line between what my artifact demonstrates and what I
suspect is happening underneath. Crossing it silently is the fastest way
to send a maintainer down a wrong path, and it is the thing I most want to
do when I think I have spotted the cause.

- Wrong: "The root cause is that SQLAlchemy 2.0 removed implicit string
  coercion, so every raw-string call in the codebase is broken."
- Right: "What I showed is that this one call fails on 2.x and not on
  1.4. I suspect it's the removal of implicit string coercion, but I have
  not checked whether other call sites hit the same path."

### Rule: Ask instead of assuming, when the thread has an answer

If a maintainer already said something about the approach, my comment
should show I read it. If nobody has, and the answer changes what I do, a
short specific question is worth more than a long guess.

- Wrong: "I'll just wrap it in `text()` and open a PR, that should fix it."
- Right: "Wrapping the statement in `text()` looks like the minimal fix.
  Before I open anything — is there a preferred place for the health
  probe's query, or should it stay inline in the route?"

### Rule: Disclose AI assistance when the repo asks, and only claim what I verified

Where a repo's stated policy asks for disclosure, I disclose plainly: the
tool and how far its help went. What I never let the disclosure do is
substitute for my own verification — if I say I ran it, I ran it.

- Wrong: "(Generated with AI assistance, let me know if anything's off.)"
- Right: "Per the contributing guide's AI policy: I used an AI assistant
  to help draft and organize this report. I ran every command shown here
  myself and the output above is from my machine."

## Things I never post

- A date, a deadline, or a guaranteed fix. I cannot honor any of them.
- "Assigning myself to this" or "please reserve this for me." I am an
  outsider here; I can state intent, not take ownership.
- "Same as above, can confirm." If my reproduction is worth posting, it is
  worth posting in my own words with my own environment and my own output.
- A root cause I have not demonstrated, stated as a fact.
- "+1", "any updates?", or a bump. It costs the thread attention and
  returns nothing.
- Flattery as a preamble — "great project, I love this repo, amazing
  work." It reads as filler and it buys nothing; if I have something
  specific to thank someone for, I name the specific thing.
- An apology for being new, or a preamble about being a beginner used to
  pre-excuse a thin report. Stating my experience level plainly is honest;
  using it as insurance is not.

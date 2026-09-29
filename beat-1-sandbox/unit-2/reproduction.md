# Unit 2 — Claim and Reproduce

- GitHub username: `AbduTello`
- Issue: https://github.com/codepath/pathreview-ai301-fa26-s1/issues/61
- Repo: `codepath/pathreview-ai301-fa26-s1` (fork: `AbduTello/pathreview-ai301-fa26-s1`)

---

## Claim comment

https://github.com/codepath/pathreview-ai301-fa26-s1/issues/61#issuecomment-5897972328

I'd like to claim this one as my first contribution here.

What I understand so far from the issue: the postgres probe in `api/routes/health.py` hands the literal string `"SELECT 1"` to the session, and SQLAlchemy 2.x no longer coerces a bare string into textual SQL, so the probe raises `ArgumentError` and `/health` reports the database as down even when it is reachable. I have not run it yet, so that is the issue's account and not mine.

My next step is to stand the stack up from the README, hit `GET /health` against a reachable database, and isolate the probe's own call so I can see whether the `ArgumentError` is what actually fires and whether anything else in the route contributes. I'll post a reproduction report with the environment I ran, the exact call that trips it, and the unedited output — including if it turns out I cannot reproduce it, or the failure is not the one described.

I'm not promising a fix or a date; I'll report what I find first.

Disclosure: I use an AI assistant in my workflow. I run and read everything myself before posting it, and the output in my report will be from my own machine.

**Reflection.** I wrote this before running anything, so the hard part was keeping the issue's account and my own evidence separate. The sentence "I have not run it yet, so that is the issue's account and not mine" is doing that work. The thread already had four reproductions when I posted; the house rules say a classmate's claim does not block me and credit attaches to the work I post, so I claimed anyway rather than piggybacking on someone else's proof.

---

## Repro comment

https://github.com/codepath/pathreview-ai301-fa26-s1/issues/61#issuecomment-5898155661

Reproduced on `main` (`f89c06f`), following up on my claim above.

**Environment:** Python 3.12.5, SQLAlchemy 2.1.1, asyncpg 0.31.0, PostgreSQL 16 (`postgres:16-alpine` via the repo's `docker compose`), macOS 15.6.1 (arm64). `pyproject.toml` pins `sqlalchemy>=2.0.0`, so 2.x is the project's own floor rather than something I chose.

**Setup**, from the README plus one deviation noted below:

```bash
git clone https://github.com/AbduTello/pathreview-ai301-fa26-s1.git
cd pathreview-ai301-fa26-s1
cp .env.example .env
docker compose up -d
python3 -m venv .venv && .venv/bin/pip install -e .
.venv/bin/alembic upgrade head
```

**1. The probe's call, isolated.** I ran the same statement `health.py` runs, through the app's own `get_db()`, with nothing else from the route involved:

```python
import asyncio
from core.database import get_db

async def main():
    async for session in get_db():
        await session.execute("SELECT 1")

asyncio.run(main())
```

```
  File ".../sqlalchemy/sql/coercions.py", line 624, in _text_coercion
    return _no_text_coercion(element, argname)
  File ".../sqlalchemy/sql/coercions.py", line 594, in _no_text_coercion
    raise exc_cls(
sqlalchemy.exc.ArgumentError: Textual SQL expression 'SELECT 1' should be explicitly declared as text('SELECT 1')
```

**2. Control, same session, same string wrapped in `text()`:**

```python
result = await session.execute(text("SELECT 1"))
print("control result:", result.scalar())
```

```
2026-09-29 16:21:34,013 INFO sqlalchemy.engine.Engine SELECT 1
control result: 1
```

So the database is reachable and the credentials are fine; the raw string is what fails.

**3. End to end.** With the stack running, `GET /health` returns **503**:

```json
{"detail": {"status": "unhealthy",
            "dependencies": {"postgres": "unhealthy", "redis": "unhealthy", "vector_db": "healthy"},
            "safety_events_last_hour": 0,
            "timestamp": "2026-09-29T20:24:00.692737"}}
```

and the app's own log names the cause:

```
postgres_health_check_failed  error="Textual SQL expression 'SELECT 1' should be explicitly declared as text('SELECT 1')"
```

**Expected:** the postgres check succeeds against a reachable database and `/health` returns 200.
**Actual:** `api/routes/health.py:32` calls `await db.execute("SELECT 1")` with a bare string; SQLAlchemy 2.x refuses to coerce it, the `except Exception` marks postgres `unhealthy`, and the endpoint returns 503 even though the database answers `SELECT 1` fine one line later under `text()`.

**Two things I hit that are not this issue**, so they don't muddy the above:

- The `redis` check in that same response fails for an unrelated reason: `'Settings' object has no attribute 'redis_host'`. Different bug, different mechanism.
- Starting the app before running `alembic upgrade head` crashes at startup with `ProgrammingError: relation "ix_profiles_user_id" already exists`, from `init_db()`'s `create_all` — reproducible on a freshly wiped volume (`docker compose down -v`). Running the migrations first, as `make setup` does, avoids it. I mention it only because it blocked me for two runs and is worth its own issue.

I did not run `make setup` in full — I skipped the frontend `npm install` and the seed script, since the health probe needs neither. Everything above is from the schema `alembic upgrade head` produces.

Next step, if nobody else is mid-PR on it: wrap the statement in `sqlalchemy.text()` and check whether any other call site in the repo passes a bare string to `execute()`.

Disclosure: I used an AI assistant while working on this. Every command above was run on my machine and the output is unedited.

**Reflection.** The control run is the part I'd defend hardest: without it, "postgres unhealthy" is equally consistent with a bad password or an unreachable container, and the report would prove nothing about the raw string. Running `text("SELECT 1")` on the *same* session and getting `1` back is what makes the bare string the only variable. The two unrelated failures were a judgment call — my own rubric's honesty family says the report must claim no more than the artifacts show, and silently dropping a second `unhealthy` from the JSON I pasted would have been the confident-wrong-target failure I built the rubric to catch.

---

## Run history

**Run 0 — hand-grading, no credit spent.** Before spending anything I graded three calibration packages by hand against the finished rubric: `calib-02` (reject — no artifact at all, bare `+1` claim), `calib-03` (reject — the trap: input uses `1: {}` where the issue's trigger is `1 = {}`, so the artifact is a graceful `Missing key/value separator` error, not the issue's `panic: not a string`), and `calib-04` (reject — exact-command repro with a real panic and a control run, but no environment record at all, on an issue where the build profile changes the failure mode). All three matched the master labels. `calib-03` was the one that mattered: it is long, confident and well formatted, and it splits artifact-readers from formatting-graders. My `artifact-shows-issue-behavior` clauses (a) wrong trigger and (b) wrong failure mode both fired on it, which told me the wording was doing its job before I paid for a run.

**Run 1 — full eval, 20/20, PASS.** `python3 run_eval.py --rubric ~/.claude/skills/repro-check/rubric.md --evidence ~/.claude/skills/repro-check/references/evidence-guide.md --save-run eval-run.txt`. Agreement 20/20, every category matched including `disclosure 1/1`. This is the run in `eval-run.txt`. I included `--save-run` on the first full run deliberately: the harness refuses to write the file only on *partial* runs, not on failing ones, so attaching it up front risked nothing and meant a clean first result did not cost a second $4 run to confirm.

**Run 2 — targeted re-grade, 2/2.** `--only pkg-20,pkg-09 --out checks.json`. Not a revision — nothing in the rubric changed between runs 1 and 2. I re-ran my two highest-risk packages for two reasons: the full run's `note` column is empty when everything agrees, so it does not show *which* check decided a package, and a single agreement on a trap could be luck. Both reproduced their run-1 verdicts with the same reasoning, which is what let me write the Package analysis below from evidence rather than from my own intent.

I never needed the revise loop, so I never loosened a check and never had to spend a canary. Total cost: about $4.40.

---

## Package analysis

**`pkg-20`** (ghostty-org/ghostty#13604). My rubric: **reject**. Gold label: **reject**. Agreed.

What makes it the interesting one is that it is the strongest reproduction in the whole set and still fails. My run graded four of five required checks `pass`, with evidence like *"first artifact shows `^[[?997;2n` (light) from single-theme run, second shows `^[[?997;1n` (dark) from conditional-pair run, exact trigger `printf '\033[?996n'`, matches issue's symptom"* and *"two runs shown: single-theme (fails) and conditional-pair (works), the exact boundary the issue describes."* The environment is recorded, the steps are runnable, the claim names the specific symptom and a concrete next step. On proof alone it is a model package.

It rejects on `ai-disclosure-when-policy-requires` alone: *"policy states 'All AI usage in any form must be disclosed, stating the tool used and the extent of the assistance'; scope covers issues and comments; neither claim comment nor repro report mentions AI, tool, or extent."*

That is the whole point of the category floor. `disclosure` is a one-package category, so a rubric with no conventions check cannot buy the miss back on volume — and because the proof is flawless, any rubric that averages its checks or weighs "overall quality" accepts it. Mine could not, because the verdict rule is a hard conjunction: *"There is no partial credit and no averaging: these five checks are five independent ways for a package to waste a maintainer's time, and one of them is enough."* I also had to write the presumption into the check itself — that course packages *are* AI-assisted work — otherwise a grader can reason that no AI was used, so nothing was owed, and accept it on a technicality.

The mirror case is `pkg-09` (sharkdp/fd#2033), which I also re-ran: gold **accept**, mine **accept**. Its policy line contains "must state the tool and the extent of its use", which reads exactly like ghostty's requirement. A keyword match on "AI" or "disclose" rejects it and costs a clear-accept. My run read the scope instead: *"policy requires disclosure 'in the pull request' and 'states no disclosure ask for issue comments'; this package is issue comments, not a pull request."* Those two packages are the same sentence pointed in opposite directions, which is why the check had to key on the modal verb's object rather than its vocabulary.

---

## Check rationale

The check I want to account for is `ai-disclosure-when-policy-requires`, quoted from `tools/repro-check/rubric.md` as uploaded. Its Evidence column reads:

> The `- contribution policy` line in the repo-facts block, read for its modal verb and its stated scope, together with the candidate claim comment and repro report. Treat every course package as AI-assisted work: the question is never whether AI was used, only whether this repo's stated policy obliges the author to say so in THIS artifact.

and its Pass condition is a four-branch decision tree. The two branches that carry the weight:

> (3) If the policy requires disclosure but scopes that requirement to a part of the workflow this package is not — most often pull requests — this check PASSES with no disclosure. THIS IS THE CASE MOST EASILY GRADED WRONG: read the scope clause, not the keywords.

> (4) FAIL only when the policy states a disclosure duty in mandatory terms whose scope covers what this package posts — comments, issues, or contributions "in any form" — and neither the claim comment nor the repro report names AI assistance. [...] A package that fails here fails even when every proof check is flawless: posting under a disclosure rule without disclosing is the harm, and it is independent of the quality of the reproduction.

Why it is written this way. The obvious version of this check — "fail if the repo mentions AI and the package does not disclose" — is wrong in both directions on this set. It rejects `pkg-09`, whose duty is scoped to pull requests, and it would fire on `pkg-03` (ripgrep), whose policy restricts who may *write* comments rather than requiring disclosure at all. Meanwhile twelve of the twenty packages have no AI policy, where demanding disclosure is over-firing and rewarding it is equally wrong — so branch 1 says explicitly "do not fail a silent package, and do not fail a disclosing one."

So the check reads grammar, not vocabulary: the modal verb (`must` vs `welcome`/`allowed`), and its object (a pull request vs comments vs "in any form"). Branch 4's last sentence exists because the one package it catches has perfect proof; without stating that a disclosure failure is independent of reproduction quality, the natural reading of a flawless package is to accept it.

The presumption in the Evidence column is load-bearing too. Nothing in `pkg-20`'s text says AI was used. If the check asked "was AI used here?", the answer from the bundle alone is "unknown", which under my verdict rule grades `unclear` → fail for the wrong reason, or tempts a grader to pass it. Fixing the presumption in advance — course work is AI-assisted — turns an unanswerable question into a readable one: not *was AI used*, but *does this policy oblige disclosure in this artifact*.

---

## Trade-offs

**One check does two jobs, and I left it that way.** `env-and-steps-rerunnable` covers both the environment record and the followability of the steps, plus the silent-version-deviation clause. Splitting it would localize failures better — `pkg-06` fails on both halves, `pkg-18` only on steps, `pkg-16` only on the version clause. I kept them merged because the three fail together in practice and because a separate steps check risked firing on `pkg-09`'s honest "I did not find a knob to force a smaller limit from the CLI", which is a stated gap rather than a hole. The cost is real: `pkg-16` is the only reject in the set whose failure lives in a single clause of a single check, so that clause is a single point of failure.

**Five required checks in a hard conjunction, no scoring.** Any one required fail rejects. This is what makes `pkg-20` come out right and what makes the rubric predictable to a second reader, but it has no way to express "weak but postable" — a package with a thin environment record and otherwise excellent proof gets the same verdict as one with no artifact at all. The two preferred checks exist to rank accepted packages, not to soften that line.

**Worked examples pin the wording to this eval set.** Cells quote specific packages — bat's `:-N` syntax, the pandas 1.5.3 version gap, the private monorepo. That is why the run agreed 20/20 and it is also the rubric's main weakness: a grader can pattern-match the example instead of applying the rule underneath it. I mitigated it by writing each example *under* a general clause rather than as the clause, and by adding "this is NOT a failure" carve-outs so the rule has edges in both directions. On a fresh set, I would expect the general clauses to hold and some of the examples to need replacing.

**I disclosed AI use on a repo that does not require it.** Branch 1 of my own disclosure check says that is neither required nor rewarded. I did it anyway because my voice guide commits to it, which is a case of the voice guide being stricter than the rubric — the right direction for the two to disagree.

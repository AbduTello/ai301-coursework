# Unit 3 — Plan and Build

- GitHub username: `AbduTello`
- Issue: https://github.com/codepath/pathreview-ai301-fa26-s1/issues/61
- Repo: `codepath/pathreview-ai301-fa26-s1` (fork: `AbduTello/pathreview-ai301-fa26-s1`)
- Branch: `fix/61-health-probe-text` (on my fork, from `main` at `f89c06f`)
- Builds on my repro comment: https://github.com/codepath/pathreview-ai301-fa26-s1/issues/61#issuecomment-5898155661

---

## Plan

**Diagnosis.** `api/routes/health.py:32` calls `await db.execute("SELECT 1")` with a bare string. SQLAlchemy 2.x (the project's own floor: `sqlalchemy>=2.0.0`) refuses to coerce a bare string to SQL and raises `ArgumentError: Textual SQL expression 'SELECT 1' should be explicitly declared as text('SELECT 1')`. The route's `except Exception` catches that error, marks postgres `unhealthy`, and the endpoint returns 503. My repro's control shows the database itself is fine: the same string wrapped in `text()`, on the same session, returns `1`.

**Scope: one bounded change.**
- In scope:
  - wrap the probe's statement in `sqlalchemy.text()` in `api/routes/health.py`;
  - add a unit regression test for the postgres probe.
  - Also check whether any baseline suppression in `pyproject.toml` exists only because of this bug. CONTRIBUTING asks a seeded-bug fix to remove its suppression. The mypy override for `api.routes.health` names #62 "and related #61", so I will confirm with mypy which error code, if any, belongs to #61 before touching it.
- Not in scope:
  - the redis check's `'Settings' object has no attribute 'redis_host'` (#62);
  - the vector-DB placeholder;
  - the `create_all` startup crash I hit during the repro;
  - the `datetime.utcnow()` deprecation;
  - any other refactor of the route.

  A grep of the repo for `execute("` finds no other bare-string call site, so there is nothing else to sweep.

**Approach.**
1. Add `tests/unit/test_health_route.py`: call `health_check()` with a session backed by an in-memory SQLite engine through real SQLAlchemy 2.x. No Docker is needed, and the real coercion rule applies. Assert `dependencies.postgres == "healthy"`. Run it on unfixed `main` and confirm it fails with the issue's `ArgumentError`.
2. In `api/routes/health.py`, add `from sqlalchemy import text` and change the probe to `await db.execute(text("SELECT 1"))`.
3. Run mypy with the `api.routes.health` override narrowed. Remove an error code only if it disappears with this fix.
4. Run the repo's local checks: `make lint`, black, `make typecheck`, `make test-unit`.

**Test plan.**
- The new test fails on `main` (postgres `unhealthy`, `postgres_health_check_failed` logged with the issue's error) and passes after the fix.
- The unit suite stays green.
- Re-run my repro's step 3 (`GET /health` against the docker stack): the `postgres_health_check_failed` line is gone and the body reads `"postgres": "healthy"`.

**Risk, stated.** `/health` will still return **503** after this fix, because the redis check fails for an unrelated reason (#62). The observable for this fix is therefore the postgres field and the log line, not the status code. The test asserts on the field so it does not depend on #62.

---

## Deviations

- **The mypy suppression stays.** I planned to remove whichever `api.routes.health` mypy suppression belonged to #61. With `call-overload` removed from the override, mypy flags line 46/47, `No overload variant of "Redis" matches argument types "Any", "Any", "int", "bool"`. It does so both before and after my fix, so that error code comes from the redis client call (#62's area), not from #61. mypy never flagged `execute("SELECT 1")` at all, because `db` comes from `Depends(get_db)` with no type annotation. Nothing in `pyproject.toml` belongs to #61, so I left the override unchanged.
- **Test helper return type.** My first draft of the test annotated the helper `-> dict`. mypy rejected it (`got "str", expected "dict[Any, Any]"`), because `HTTPException.detail` is typed as `str`. I changed it to `-> Any`. Small, but it is a difference from the code I first wrote.
- **Live `/health` re-run came after the unit-test evidence.** The test plan included re-running my repro's step 3 against the docker stack. The Docker daemon was not running when I built the fix, so the unit test came first. I ran the live check later, after starting Docker: `main` and the fix branch against the same stack, both shown under Evidence in `plan-and-implement.md`. The result is what the plan's stated risk predicted. Postgres flips to `healthy` and the `ArgumentError` log line disappears, but the status stays 503 because of #62's redis failure.
- **Order of work.** The assignment was past due, so I built the branch before posting the plan comment, not after. The plan above is what I wrote before building. Nothing in the build changed its approach or scope.
- **Black on Python 3.12.5.** The venv's black refuses to run on 3.12.5 (a known memory-safety guard). I ran `uvx --python 3.13 black --check` instead: `2 files would be left unchanged`.

## Week 7 - Issue selection

**Issue link:** https://github.com/ascherj/pathreview/issues/154

**Issue title:** Health check DB probe passes a raw SQL string, which fails under SQLAlchemy 2.x

**Tier:** [x] Tier 1 [ ] Tier 2 [ ] Tier 3

**"Is this issue right for me?" checklist reasoning:**

- **Understanding the issue:** I can explain the problem and expected behavior in my own words. The health endpoint is incorrectly reporting PostgreSQL as unavailable because its database probe uses a raw SQL string that SQLAlchemy 2.x rejects.
- **Affected area:** The issue is contained in the API layer, primarily in the `health_check` route in `api/routes/health.py`. I located the route and read the surrounding PostgreSQL, Redis, and vector database checks.
- **Definition of done:** Before the fix, a reachable PostgreSQL database may be reported as unhealthy and `/health` may return a failure response. After the fix, the probe should use SQLAlchemy's supported textual SQL format, report PostgreSQL as healthy when it is reachable, and continue reporting genuine database failures correctly.
- **Tier fit:** This is my first contribution to this codebase, so Tier 1 is an appropriate choice for me. The expected change is localized to one route and its related tests and should not require changes to the frontend, database schema, RAG pipeline, or agent system.
- **Codebase readiness:** I found the specific function referenced by the issue and reviewed enough surrounding code to understand how an exception changes the dependency status and causes the endpoint to return an unhealthy response.
- **Test readiness:** Before implementing the change, I will find and read the existing API test patterns and add a regression test that confirms the database probe succeeds when SQLAlchemy receives a valid textual SQL expression.
- **Other contributors:** I checked the issue comments and cohort ledger. When i signed up there were only about 4 people working on it but I understand that multiple students can work on the same issue.
- **Time and scope:** Tier 1 issues are expected to be achievable within approximately 3-6 hours of focused work so it is realistic for me to complete before the Week 9 deadline.
- **Blockers:** The issue does not list any unresolved blocker or dependency that must be completed first.
- **Verdict:** This issue is a realistic fit for my current experience, available time, and the Module 3 contribution requirements.

**Problem summary:**
The database health check in `api/routes/health.py` sends `"SELECT 1"` to SQLAlchemy as a plain string. Under SQLAlchemy 2.x, textual SQL has to be explicitly declared, so the database probe raises an `ArgumentError` even when PostgreSQL is available. This causes the `/health` endpoint to report that the database is down even though it is reachable. A successful fix will execute the probe using the supported SQLAlchemy format and verify the corrected behavior with an appropriate test.

**Branch name:** `fix/154-health-check-db-probe`

**Setup confirmation:** [x] App runs locally at localhost:5173

**Cohort ledger:** [x] Issue added to cohort ledger

## Week 8 - Reproduction & solution planning

**Reproduction commit link:** https://github.com/DuBaem/pathreview/commit/ed8e1a5

**Reproduction summary:**
I confirmed that the PostgreSQL Docker service was healthy, then executed the same raw `"SELECT 1"` string used by `health_check()` through the project's real SQLAlchemy `AsyncSession`. SQLAlchemy 2.0.51 raised an `ArgumentError` for the raw string, while `text("SELECT 1")` succeeded through the same session and returned `1`, confirming that the failure is caused by the query format rather than an unavailable database.

**PLAN.md link:** https://github.com/DuBaem/pathreview/blob/fix/154-health-check-db-probe/PLAN.md

**Blockers or open questions:**

## Week 9 - Solution building & PR submission

### Check-in 1 (mid-week)

**Current progress:**
I reproduced issue #154 using SQLAlchemy 2.0.51 and the project's real `AsyncSession`, documented the reproduction, and completed `PLAN.md`. I also ran the existing unit-test suite and recorded a baseline of 375 passing tests and 53 pre-existing failures so I can identify whether my changes introduce any new failures.

**Next steps:**
I will update the PostgreSQL probe in `api/routes/health.py`, create focused regression tests in `tests/unit/test_health.py`, run the new tests directly, run `make check`, and compare the full unit-test results against the existing baseline. I will then open a draft pull request and request feedback before marking it ready for review.

**Blockers:**

---
### Check-in 2 (end of week)

**PR link:** https://github.com/ascherj/pathreview/pull/656

**Branch:** `fix/154-health-check-db-probe`

**What you built:**
I updated the PostgreSQL health probe to execute `SELECT 1` using SQLAlchemy's supported `text()` construct. This prevents SQLAlchemy 2.x from rejecting the query and falsely reporting a reachable PostgreSQL database as unhealthy, while preserving the existing HTTP 503 behavior for genuine database failures.

**Tests added or updated:**
I added `tests/unit/test_health.py` with two async unit tests. The tests confirm that the database probe receives a SQLAlchemy `TextClause` containing `SELECT 1` and that a genuine database execution failure still marks PostgreSQL as unhealthy and returns HTTP 503.

**Self-review confirmation:** [x] make check passes [x] make test-unit passes

The repository retains its documented pre-existing failures. The full unit-test result remained at 53 failures while passing tests increased from 375 to 377, confirming that both new tests pass and no new failures were introduced. Repository-wide linting reports pre-existing errors outside the changed files, while both changed files pass targeted Ruff and Black checks.

**Draft PR feedback received from:** none
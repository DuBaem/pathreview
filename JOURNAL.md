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

**Reproduction commit link:** Pending until the reproduction commit is pushed

**Reproduction summary:**
I confirmed that the PostgreSQL Docker service was healthy, then executed the same raw `"SELECT 1"` string used by `health_check()` through the project's real SQLAlchemy `AsyncSession`. SQLAlchemy 2.0.51 raised an `ArgumentError` for the raw string, while `text("SELECT 1")` succeeded through the same session and returned `1`, confirming that the failure is caused by the query format rather than an unavailable database.

**PLAN.md link:** Pending

**Blockers or open questions:**
The complete `/health` route also contains an unrelated Redis configuration mismatch because `health.py` refers to `redis_host` and `redis_port`, while the settings object defines `redis_url`. This is outside issue #154, so the planned regression test should isolate the PostgreSQL probe without expanding the scope of this contribution.

## Week 7 - Issue selection

**Issue link:** https://github.com/ascherj/pathreview/issues/154

**Issue title:** Health check DB probe passes a raw SQL string, which fails under SQLAlchemy 2.x

**Tier:** [x] Tier 1 [ ] Tier 2 [ ] Tier 3

**Problem summary:**
The database health check in `api/routes/health.py` sends `"SELECT 1"` to SQLAlchemy as a plain string. Under SQLAlchemy 2.x, textual SQL has to be explicitly declared, so the database probe raises an `ArgumentError` even when PostgreSQL is available. This causes the `/health` endpoint to report that the database is down even though it is reachable. A successful fix will execute the probe using the supported SQLAlchemy format and verify the corrected behavior with an appropriate test.

**Branch name:** `fix/154-health-check-db-probe`

**Setup confirmation:** [x] App runs locally at localhost:5173

**Cohort ledger:** [x] Issue added to cohort ledger
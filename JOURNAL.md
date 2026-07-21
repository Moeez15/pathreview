## Week 7 — Issue selection

**Issue link:** https://github.com/ascherj/pathreview/issues/88

**Issue title:** POST /reviews endpoint has no test for when the profile has no ingested documents #88

**Tier:** [X] Tier 1  [ ] Tier 2  [ ] Tier 3

**Problem summary:**
The POST /reviews endpoint doesn't have a test for what happens when a profile has no ingested documents. Right now we don't know if it fails properly or just crashes in this case. The fix is to add a test in tests/unit/test_review_routes.py that checks the endpoint returns a clear error instead of crashing when this happens. This affects the review routes part of the codebase and helps make sure the API handles this case safely.

**Branch name:** test/88-post-review-endpoint

**Setup confirmation:** [X] App runs locally at localhost:5173

**Cohort ledger:** [X] Issue added to cohort ledger
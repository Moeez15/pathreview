## Week 7 — Issue selection

**Issue link:** https://github.com/ascherj/pathreview/issues/88

**Issue title:** POST /reviews endpoint has no test for when the profile has no ingested documents #88

**Tier:** [X] Tier 1  [ ] Tier 2  [ ] Tier 3

**Problem summary:**
The POST /reviews endpoint doesn't have a test for what happens when a profile has no ingested documents. Right now we don't know if it fails properly or just crashes in this case. The fix is to add a test in tests/unit/test_review_routes.py that checks the endpoint returns a clear error instead of crashing when this happens. This affects the review routes part of the codebase and helps make sure the API handles this case safely.

**Branch name:** test/88-post-review-endpoint

**Setup confirmation:** [X] App runs locally at localhost:5173

**Cohort ledger:** [X] Issue added to cohort ledger


## Week 8 — Reproduction & solution planning

**Reproduction commit link:** https://github.com/Moeez15/pathreview/commit/442a264df18ae33613bcefb898cf54c3b66bab7e

**Reproduction summary:**
Added a failing unit test (`test_process_review_with_no_ingested_documents` in `tests/unit/test_review_service.py`) that runs `create_review` + `process_review` for a profile with `github_username`, `portfolio_url`, and `resume_text` all `None`. Observed that `_run_ingestion_pipeline` correctly reports zero sources, but the downstream placeholder agent/RAG steps still fabricate feedback sections, and the review ends up `status="complete"` with a fake `overall_score` instead of failing or erroring.

**PLAN.md link:** https://github.com/Moeez15/pathreview/blob/test/88-post-review-endpoint/PLAN.md

**Walkthrough video (recommended):** [link to your Loom video, ≤2 min — recommended, not graded]

**Blockers or open questions:**
Need to confirm whether this issue is scoped to just adding a test documenting current behavior, or also expects a behavior fix (rejecting/failing reviews for empty profiles) in the same PR — see Risks & unknowns in PLAN.md.
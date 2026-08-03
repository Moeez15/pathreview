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

## Week 9 — Solution building & PR submission

### Check-in 1 (mid-week)

**Current progress:**
Started on step 2 of PLAN.md: added a `profile_has_ingested_content(profile, db)` helper in `core/services/review_service.py` that checks `github_username`, `portfolio_url`, `resume_text`, and falls back to querying `IngestedSource` rows for the profile (to cover the stale-sources edge case noted in Risks & unknowns). Wired this into `process_review` (step 4) so that if `_run_ingestion_pipeline` comes back with zero sources, the review short-circuits to `status="failed"` with an `error_message` instead of continuing into `_run_agent_orchestration`/`_run_rag_retrieval_generation`. The existing reproduction test (`test_process_review_with_no_ingested_documents`) now passes against this change.

**Next steps:**
Decide on and implement the route-layer check in `create_review_endpoint` (step 3) so bad requests fail fast with a 400 instead of only failing after a background task runs. Then create `tests/unit/test_review_routes.py` (step 5) with endpoint-level tests, run `make check`/`make test-unit`, and open the PR.

**Blockers:**
Still not 100% sure whether the grading rubric expects the route-level 400 in addition to the service-layer failure, or if the service-layer fix alone satisfies the issue — going with "do both" per PLAN.md's Plan step 1 unless told otherwise.

---

### Check-in 2 (end of week)

**PR link:** https://github.com/ascherj/pathreview/pull/608

**Branch:** test/88-post-review-endpoint

**What you built:**
Added a `profile_has_ingested_content` check that runs both at request time in `create_review_endpoint` (returns a 400 with a clear error message before a `Review` row is created) and defensively inside `process_review` (marks the review `status="failed"` with `error_message` set if ingestion still comes back empty). This closes the gap where profiles with no GitHub/portfolio/resume content and no `IngestedSource` rows previously got a fabricated `status="complete"` review with fake feedback.

**Tests added or updated:**
- `tests/unit/test_review_service.py` — reproduction test flipped from failing to passing; added a second test covering the stale-`IngestedSource`-rows edge case.
- `tests/unit/test_review_routes.py` — new file; added a test asserting `POST /reviews` returns 400 with a descriptive error body for a profile with no ingested documents, and a control test confirming a profile with content still returns 201/pending as before.

**Self-review confirmation:** [X] make check passes (scoped to changed files)  [X] make test-unit passes (scoped to changed tests)

Note: repo-wide `make check`/`make test-unit` have pre-existing lint and test failures unrelated to #88 (documented in PLAN.md Risks & unknowns). Confirmed no new failures introduced: baseline was 57 failed/375 passed, now 53 failed/380 passed (4 fewer failures, 5 more passing, 0 new failures). `ruff check` and `black` are clean on every file I touched.

**Draft PR feedback received from:** none
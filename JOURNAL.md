# PathReview Project Journal

## Week 7 — Issue selection

**Issue link:** [Issue #64](https://github.com/ascherj/pathreview/issues/64)

**Issue title:** Prompt injection defense doesn't sanitize newline characters in user-supplied resume text

**Claim status:** Claimed in the issue thread as `Neptuneaswol`

**Tier:** [ ] Tier 1  [x] Tier 2  [ ] Tier 3

**Problem summary:**
PathReview passes user-supplied resume text through `PromptDefense.sanitize()` in `safety/prompt_defense.py`, but that method currently removes only template delimiters and angle brackets. Newline-based prompt boundaries such as `\n---\n` and role changes such as `\nSystem:` therefore remain in the sanitized text, allowing an attacker to append instructions that may influence the review model. Although `is_injection_attempt()` recognizes these patterns, sanitization does not neutralize them, so callers cannot rely on the sanitized output alone. A successful fix will remove or safely normalize these newline injection patterns while preserving legitimate multiline resume content, with unit tests covering malicious inputs, benign newlines, and idempotence.

**Branch name:** `fix/64-sanitize-newline-injection`

**Setup confirmation:** [ ] App runs locally at localhost:5173

**Cohort ledger:** [ ] Issue added to cohort ledger

### Selection notes — “Is this right for me?” checklist

- **Scope:** The change is localized to `safety/prompt_defense.py` and its unit tests rather than requiring a cross-system redesign.
- **Issue understanding:** The detector already identifies separator and role-switch patterns, but `sanitize()` leaves those same sequences intact; the expected behavior can therefore be expressed with focused tests.
- **Skills and dependencies:** The work uses Python string/regular-expression handling and pytest, and does not require a new external service or dependency.
- **Time fit:** The issue is labeled Tier 2 with an estimated effort of 4–6 hours, which is a realistic scope for the project timeline while still requiring careful security and false-positive testing.
- **Success criteria:** Malicious prompt-boundary newlines are neutralized, ordinary multiline resume formatting is retained, sanitization remains idempotent, and the safety unit tests pass.

The setup and cohort-ledger boxes will be checked only after those external steps have been completed and verified.

## Week 8 — Reproduction & solution planning

**Reproduction commit link:** [Issue #64 reproduction commit](https://github.com/Neptuneaswol/pathreview/commit/75f1e1c98a1aee0122bbe65bc2487ca2f8e14e66)

**Reproduction summary:**
I reproduced the issue by passing resume text containing `\n---\n` and `\nSystem:` prompt boundaries to `PromptDefense.sanitize()` in `safety/prompt_defense.py`. The returned text still contains both malicious newline sequences, confirming that detection recognizes these patterns but sanitization does not neutralize them.

**PLAN.md link:** [Issue #64 solution plan](https://github.com/Neptuneaswol/pathreview/blob/fix/64-sanitize-newline-injection/PLAN.md)

**Walkthrough video (recommended):** Not recorded; this deliverable is optional and not graded.

**Blockers or open questions:**
The local development dependencies must be installed before the focused reproduction test and full unit suite can run. A repository-wide search also found no production caller of `PromptDefense`, so the planned fix remains scoped to `safety/prompt_defense.py` and `tests/unit/test_prompt_defense.py` unless maintainer guidance confirms that resume-pipeline integration is part of Issue #64.

**Implementation update:**
The newline sanitization fix has been implemented. All 46 focused `test_prompt_defense` cases pass with 100% coverage, and Ruff, Black, and mypy pass on the changed Python files. The broader repository still contains unrelated legacy failures and environment-sensitive tests outside this issue's scope.

## Week 9 — Solution building & PR submission

### Check-in 1 (mid-week)

**Current progress:**
I completed the test-first implementation from `PLAN.md`. The regression suite now covers separator-line injections, all supported role labels, explicit instruction overrides, mixed capitalization, indentation, LF and CRLF input, benign multiline resumes, whitespace-only input, and repeated sanitization. I also updated `PromptDefense.sanitize()` so those prompt-shaped boundaries are neutralized without flattening normal resume content.

**Next steps:**
Run the focused and repository-wide quality checks, document any pre-existing failures, complete the Week 9 journal entry, and push the finished branch to my fork.

**Blockers:**
The issue-specific implementation is not blocked. Repository-wide checks contain numerous failures in unrelated modules, so I am documenting the baseline and verifying that this change adds no failures in `safety/prompt_defense.py` or `tests/unit/test_prompt_defense.py`.

---

### Check-in 2 (end of week)

**PR link:** Not opened. At my direction, this work remains on my fork rather than being submitted as a pull request.

**Fork branch:** [fix/64-sanitize-newline-injection](https://github.com/Neptuneaswol/pathreview/tree/fix/64-sanitize-newline-injection)

**Prepared PR description:** [PR_DESCRIPTION.md](https://github.com/Neptuneaswol/pathreview/blob/fix/64-sanitize-newline-injection/PR_DESCRIPTION.md)

**Branch:** `fix/64-sanitize-newline-injection`

**What you built:**
I hardened `PromptDefense.sanitize()` against newline-based prompt injection. It removes separator boundaries, rewrites `System`, `Human`, and `Assistant` role switches, neutralizes line-start instruction overrides, handles LF and CRLF consistently, and preserves ordinary multiline resume content.

**Tests added or updated:**
I updated `tests/unit/test_prompt_defense.py` with regression coverage for every supported role, mixed case and whitespace, separator lengths and indentation, explicit override verbs, benign multiline and whitespace-only content, CRLF normalization, and malicious-input idempotence. The focused suite reports 46 passed tests and 100% statement coverage for `safety/prompt_defense.py`.

**Self-review confirmation:** [x] `make check` introduces no new failures  [x] `make test-unit` introduces no new failures

**Validation details:**
Ruff, Black, and mypy pass on the changed Python files. The repository-wide baseline remains 180 Ruff findings, 52 files requiring Black formatting, 104 mypy errors across 26 unrelated files, and 51 failed plus 31 errored unit tests outside prompt defense; 360 unit tests pass, including all 46 prompt-defense tests. This follows the Week 9 guidance that documented pre-existing failures count as passing when the contribution does not introduce new failures.

**Draft PR feedback received from:** none

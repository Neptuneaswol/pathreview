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

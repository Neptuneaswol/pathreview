# fix(safety): sanitize newline prompt injections

## Summary

This change fixes Issue #64 by neutralizing newline-based prompt boundaries in untrusted resume text. The sanitizer now handles separator lines, role-switch labels, and explicit line-start instruction overrides while preserving ordinary multiline resume content.

## Issue

Closes [ascherj/pathreview#64](https://github.com/ascherj/pathreview/issues/64).

## Changes

- Normalize LF, CRLF, and mixed line endings before applying newline defenses.
- Remove prompt-boundary separator lines without flattening legitimate paragraphs.
- Rewrite `System`, `Human`, and `Assistant` role labels, including mixed-case and indented forms.
- Neutralize line-start `Ignore`, `Forget`, `Disregard`, and `Override` instructions while retaining their text for review.
- Preserve the existing template-delimiter and angle-bracket sanitization behavior.
- Add focused regression tests for malicious, benign, whitespace-only, and idempotent inputs.

## Testing

- [x] Focused unit tests pass: `46 passed`.
- [x] `safety/prompt_defense.py` has 100% statement coverage in the focused suite.
- [x] Ruff passes on the changed Python files.
- [x] Black check passes on the changed Python files.
- [x] Mypy passes on `safety/prompt_defense.py`.
- [x] New and updated tests cover the change.
- [ ] Integration tests are not applicable to this isolated sanitizer change.

The repository-wide baseline still contains 180 unrelated Ruff findings, 52 files that Black would reformat, 104 mypy errors in 26 unrelated files, and 51 failed plus 31 errored unit tests outside prompt defense. The full unit run passes 360 tests, including all 46 prompt-defense tests, and this contribution introduces no new failures.

## Screenshots / Demo

Not applicable; this is a backend sanitizer and unit-test change.

## Notes for Reviewers

The main review concern is the balance between security and content preservation. The regexes target complete separator lines and line-start prompt markers rather than replacing all newlines, so normal resume paragraphs and headings remain readable. No public API, dependency, or pipeline-integration changes are included.

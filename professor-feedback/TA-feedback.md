# Assignment 01 — TA feedback: Caleb (Tolorunleke Adebayo)

- **Evaluated by:** Yilin Pan (TA)
- **Date:** 2026-10-05

## Scores

| Category | Pts | Student's LLM eval | TA verified | Notes |
|---|---|---|---|---|
| Immutability & encapsulation | 25 | 25 | 25 | |
| Constructor validation | 25 | 25 | 25 | |
| Correctness | 15 | 15 | 15 | |
| Testing | 15 | 15 | 15 | Kept, but see feedback |
| Code quality & style | 10 | 10 | 10 | |
| Introspection, LLM eval & code walk | 10 | 10 | 10 | `Species` is graded from the code walk |
| **Total** | **100** | **100** | **100** | |

## Checks

- [x] All tests pass (36/36)
- [x] No `UnsupportedOperationException` stubs left
- [x] Checkstyle warnings: 0
- [x] Coverage (lines / branches): 100% / 100%
- [x] Nothing out of scope (inheritance, collections, `equals`/`hashCode`)
- [x] `submission/LLM-Evaluation.md` present
- [x] `submission/introspection.md` present

## Where the TA disagrees with the student's LLM evaluation

No changes. I'm keeping your LLM's scores. For cat 6, `Species` is graded from your code walk, and your code walk covered it, so full credit.

## Feedback

Validation and delegation are clean, and you tested `MAX_MONTHS - 1` and the max. A few things for
next time, none of them deducted:

- The null-name and empty-name messages aren't tested.
- `contains("age")` (AnimalTest.java:131) is too loose, and the exact `"intakedate"` match
  (AnimalTest.java:139) is too strict.
- Some indentation is off (Animal.java:55, AgeMonths.java:121).

Your LLM gave you 100 with no criticism. Next time, push it to find what's missing, so you have
something to respond to.

## Grade

**Final grade:** 100 / 100

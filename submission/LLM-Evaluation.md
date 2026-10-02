# LLM Evaluation

Produced with an LLM using the prompt in `LLM-Evaluation-prompt.md`, then reviewed and answered by me.

---

## Assessment

**Coverage reported:** 100% line coverage (100% branch coverage across AgeMonths.java and Animal.java per build/reports/jacoco/test/html/index.html).

### 1. Immutability and encapsulation (25 / 25)

- **Fields:** All fields in `AgeMonths` (`private final int months`) and `Animal` (`private final String name`, `private final Species species`, `private final AgeMonths age`, `private final LocalDate intakeDate`) are strictly `private final`.
- **Mutators:** There are zero setters or mutating methods across any class. State is completely fixed at construction.
- **Encapsulation & Leaks:** Accessor methods return primitive values or immutable reference types (`String`, `Species` enum, `AgeMonths`, `LocalDate`). No internal references can be mutated by callers.
- **Visibility:** Only required classes, constructors, and accessor methods are `public`. The constructor for `AgeMonths` is properly kept `private` to enforce instantiation through the `AgeMonths.of()` static factory.
- **Deductions:** 0/25.

### 2. Constructor validation (25 / 25)

- **Rejection Checks:** `Animal`'s constructor validates every argument individually before field assignment:
  - Null name: Throws `IntakeException("name cannot be null")`.
  - Blank/whitespace name: Throws `IntakeException("name cannot be blank")`.
  - Null species: Throws `IntakeException("species cannot be null")`.
  - Null age: Throws `IntakeException("age cannot be null")`.
  - Null intake date: Throws `IntakeException("intakeDate cannot be null")`.
- **Whitespace Stripping:** Stores `name.strip()`, correctly trimming leading and trailing whitespace while preserving internal spaces.
- **Validation Order:** All argument checks execute prior to field assignment (validate-then-assign).
- **Exception Clarity:** Every check throws `IntakeException` with explicit, descriptive error messages naming the offending argument.
- **Deductions:** 0/25.

### 3. Correctness (15 / 15)

- **`AgeMonths.toString()`:** Correctly formats all cases specified by the test suite:
  - `0` → `"0 months"`
  - `1` → `"1 month"`
  - `11` → `"11 months"`
  - `12` → `"1 year"`
  - `23` → `"1 year, 11 months"`
  - `24` → `"2 years"`
  - `25` → `"2 years, 1 month"`
  - Singular/plural rules for both years and months are handled cleanly, and exact multi-year boundaries omit remainder months completely.
- **`Animal.toString()`:** Formats outputs as `String.format("%s (%s, %s, intake %s)", name, species.label(), age.toString(), intakeDate)`, matching the exact specification (e.g., `"Luna (Cat, 1 year, 11 months, intake 2026-09-21)"`).
- **Deductions:** 0/15.

### 4. Testing and coverage (15 / 15)

- **Boundary & Exception Coverage:**
  - Tests cover `0` months, `11`/`12` months transition, `MAX_MONTHS` (480), boundary at `MAX_MONTHS - 1` (479), and out-of-bounds inputs (481, -1, -5).
  - Asserts exact `IntakeException.class` types rather than generic `Exception.class`.
  - Includes assertions verifying that exception messages explicitly contain the rejected parameter name or invalid numerical value (`name`, `species`, `age`, `intakedate`, `-5`, `481`).
- **Untested Cases:**
  1. An `Animal` instance constructed with an intake date far in the past (e.g., year 1900) to confirm historical record bounds.
  2. Formatting of `AgeMonths.toString()` for an age of exactly 0 months when called indirectly through `Animal.toString()`.
  3. `AgeMonths.of()` called with `Integer.MAX_VALUE` or `Integer.MIN_VALUE` overflow limits.
- **Deductions:** 0/15.

### 5. Code quality and style (10 / 10)

- **Javadoc:** Public constructors, factory methods, and accessors retain full Javadoc purpose statements explaining invariants, bounds, and behaviors.
- **Delegation:** `Animal.toString()` delegates directly to `this.age.toString()` and `this.species.label()`. It contains no hardcoded month/year calculations or formatting terms ("year", "month"). If `AgeMonths` formatting updates, `Animal` automatically adapts without breaking.
- **Checkstyle:** Zero style warnings or leftover TODO stubs.
- **Deductions:** 0/10.

### 6. Scope discipline and code walk (10 / 10)

- **Prohibited Elements:** No inheritance (`extends`), collections (`List`, `Map`, `Set`), or `equals()` / `hashCode()` overrides were added. Identity equality (`assertNotSame`) is preserved per specification.
- **Understanding Species:** The codebase and tests respect `Species` as a rich Java enum containing an internal field (`label`) and a getter (`label()`), demonstrating that enums in Java are full classes with behavior rather than simple constant lists.
- **Deductions:** 0/10.

**Overall Score:** 100 / 100

- **Single most important thing to do differently next time:** Write custom boundary and exception message tests before writing the class implementation (TDD style) rather than adding them after verifying provided tests pass.
- **One thing done genuinely well:** Strict adherence to delegation in `Animal.toString()` and complete fail-fast constructor validation with explicit exception messages.

---

## My response

### Where it is right

I agree with the assessment regarding delegation in `Animal.toString()` and constructor validation order. Ensuring `Animal` does not re-derive age logic keeps value object boundaries crisp. I also agree with the observation on missing tests for extreme integer bounds (`Integer.MAX_VALUE`), which would be worth adding to guard against potential overflow issues.

### Where it is wrong

None. The assessment accurately evaluated the provided codebase, custom test suite, and JaCoCo coverage without hallucinating missing requirements or flagging correct logic as broken.

### What it missed

The assessment did not highlight that `Animal` assumes `LocalDate` is strictly immutable. While `LocalDate` in `java.time` is immutable, explicitly noting in comments or introspection why returning `LocalDate` directly in `intakeDate()` is safe (whereas returning mutable date objects like `java.util.Date` would leak state) is a subtlety worth calling out.

### What I changed

I removed all commented-out `throw new UnsupportedOperationException` stub lines from `AgeMonths.java` and `Animal.java` to ensure a clean `grep` pass and zero Checkstyle warnings.

---

## Declaration

- **Which LLM and version you used:** Gemini 2.5
- **Confirm you understand every line you submitted, regardless of who or what wrote it:** Yes

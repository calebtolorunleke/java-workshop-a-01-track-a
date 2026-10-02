# Design Introspection

Your own reflection on the design decisions you made this week. Written in your own words — this is distinct from the LLM's assessment of your code, and distinct from your code-walk video.

---

## 1. What you built

This week I implemented two immutable domain types: `AgeMonths` and `Animal`.
`AgeMonths` is a value object responsible for modeling an animal's age strictly as non-negative whole months up to a 40-year cap (480 months) and deriving human-readable age representations.
`Animal` represents an individual shelter record at intake, encapsulating the animal's name, species, age, and intake date while enforcing complete input validation upon construction.

---

## 2. Design decisions

### Decision 1: Single canonical primitive field vs. Dual fields in `AgeMonths`

- **What was the choice?** Whether to store months as a single `private final int` and compute years/remainder months dynamically, or store both `years` and `remainderMonths` as separate fields in the object.
- **What did you pick, and why?** I chose to store a single `months` field and derive `years()` (`this.months / 12`) and `remainderMonths()` (`this.months % 12`) on demand. Storing separate fields creates internal redundancy and risks state drift or invalid field combinations.
- **What did it cost?** Minor repeated arithmetic operations during access, though the computational cost of integer division/modulo is negligible compared to the architectural benefit of having a single source of truth.

### Decision 2: Delegation of formatting in `Animal.toString()`

- **What was the choice?** Whether `Animal.toString()` should manually inspect the animal's `AgeMonths` instance to reconstruct the year/month string, or delegate formatting entirely to `AgeMonths.toString()`.
- **What did you pick, and why?** I chose to delegate directly to `this.age.toString()`. Re-implementing string formatting logic inside `Animal` would duplicate singular/plural rules ("year" vs. "years") and break the Single Responsibility Principle.
- **What did it cost?** `Animal` depends directly on `AgeMonths.toString()` staying strictly aligned with the overall shelter display specification.

### Decision 3: Eager validation and string trimming in `Animal` constructor

- **What was the choice?** Whether to store raw string names and trim them inside the `name()` getter, or trim and validate strings upfront in `Animal(String, Species, AgeMonths, LocalDate)`.
- **What did you pick, and why?** I picked eager trimming (`this.name = name.strip()`) and immediate validation in the constructor. If trimming occurred in the getter, `name.isBlank()` checks in the constructor could pass on strings containing only whitespace, or lead to inconsistent state where the raw field differs from the accessor return value.
- **What did it cost?** Memory allocation for a stripped `String` instance during constructor execution.

---

## 3. Invariants

### `AgeMonths`

- **Guarantees:** $0 \le \text{months} \le 480$.
- **Enforcement:** Enforced in `AgeMonths.of(int months)`:
  - Line throwing for negative input: `if (months < 0) throw new IntakeException(...)`
  - Line throwing for upper bound: `if (months > MAX_MONTHS) throw new IntakeException(...)`
- **Duplication/Deliberation:** Enforced solely inside `AgeMonths.of(int)`. Because the constructor `AgeMonths(int)` is private, external callers cannot bypass this factory validation.

### `Animal`

- **Guarantees:** `name` is non-null, non-blank, and stripped of leading/trailing whitespace; `species`, `age`, and `intakeDate` are strictly non-null.
- **Enforcement:** Enforced inside `Animal(String, Species, AgeMonths, LocalDate)`:
  - Name checks: `if (name == null)` and `if (name.isBlank())` throw `IntakeException`.
  - Object checks: `if (species == null)`, `if (age == null)`, and `if (intakeDate == null)` throw `IntakeException`.
  - Sanitization: `this.name = name.strip()`.
- **Duplication/Deliberation:** `Animal` does not re-validate the internal range of `AgeMonths`. This is deliberate because `AgeMonths` already guarantees its own invariants upon creation.

---

## 4. Testing

- **Cases added:** Added boundary tests in `AgeMonthsTest` for 479 months (`MAX_MONTHS - 1`) and 480 months (`MAX_MONTHS`) to verify partial year discard logic near the upper bound. In `AnimalTest`, added explicit assertions ensuring `IntakeException` messages explicitly mention null field names (e.g., `"species"`, `"age"`, `"intakedate"`).
- **Hardest test to write:** Testing the exception message contents (`assertThrows` combined with `getMessage().contains(...)`). It required careful verification that error messages explicitly name the bad parameter rather than returning generic failure strings.
- **What is still untested:** `LocalDate` temporal edge cases (e.g., intake dates set far in the future or past relative to system clock). Given another hour, I would add tests validating temporal sanity or boundary interactions with `LocalDate.now()`.

---

## 5. What you would change

Given another day, I would refine the exception messaging strategy by introducing structured error codes or custom subtype exceptions derived from `IntakeException` (e.g., `InvalidNameException`, `InvalidAgeException`). Currently, exception assertions rely on string matching (`contains("name")`), which can fragilely break if wording changes. Time constraints and sticking strictly to the provided baseline exception hierarchy stopped me from refactoring this.

---

## 6. What you found hard

The hardest part was implementing `AgeMonths.toString()` formatting to handle all pluralization edge cases cleanly without writing nested, messy if-else blocks. I initially struggled with handling inputs like 0 months vs 1 month vs 12 months (1 year) without duplicating conditional checks. It clicked once I separated the problem into two distinct branch paths: whole years (`yr == 0` or `rem == 0`) versus mixed years and months (`yr > 0` and `rem > 0`), using ternary operators to handle plural suffixes cleanly.

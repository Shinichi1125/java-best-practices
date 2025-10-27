You’ve tripped a classic trap:

* In your **first** test, the `LocalDate.parse("invalid-date")` was inside the lambda passed to `assertThrows`, so the exception happened **inside** the lambda. Sonar flagged it because the lambda had **two calls** (`parse(...)` and `getPeriod(...)`) that could throw.
* In your **second** test, you moved `LocalDate.parse("invalid-date")` **outside** the lambda. Now the parse throws **before** `assertThrows` runs, so Maven fails the test run.

Also, there’s a conceptual mismatch: if `getPeriod` takes a `LocalDate`, it cannot throw a `DateTimeParseException` (that’s only thrown while **parsing a String → LocalDate**).

### What to do

Pick one of these clean fixes depending on what you actually want to test:

#### A) You just want to assert parsing fails

Don’t involve `getPeriod` at all. Test parsing directly (this satisfies Sonar: the lambda has a single invocation):

```java
@Test
void parse_invalidDate_throwsDateTimeParseException() {
    assertThrows(DateTimeParseException.class, () -> LocalDate.parse("invalid-date"));
}
```

Then write **separate** tests for `getPeriod` with valid `LocalDate` inputs.

#### B) You want `getPeriod` to reject bad input

If `getPeriod` should validate inputs (e.g., not accept `null` or out-of-range dates), test **that** exception instead:

```java
@Test
void getPeriod_nullDate_throwsNullPointerException() {
    assertThrows(NullPointerException.class, () -> someUtils.getPeriod(null));
}
```

(or whatever exception your method specifies for invalid input).

#### C) You really need `getPeriod` to throw `DateTimeParseException`

Then its API should accept a **String** and parse internally:

```java
public Period getPeriod(String dateStr) {
    LocalDate date = LocalDate.parse(dateStr); // may throw DateTimeParseException
    ...
}
```

And test:

```java
assertThrows(DateTimeParseException.class, () -> someUtils.getPeriod("invalid-date"));
```

(Again, a single call in the lambda → Sonar is happy.)

### If you must keep the original test shape

Use a **targeted suppression** for that one test (since this rule can be overzealous in unit tests). Either:

* Add `// NOSONAR` to the assertion line, or
* Annotate the test with:

  ```java
  @SuppressWarnings("java:S5778") // Sonar: lambda should contain only one invocation
  ```

…but the **better fix** is A or B above.

### TL;DR

* Your second change fails because the exception is thrown **before** `assertThrows`.
* A method that accepts `LocalDate` shouldn’t be tested for `DateTimeParseException`. Either test the parse separately, or change `getPeriod` to accept a `String`.

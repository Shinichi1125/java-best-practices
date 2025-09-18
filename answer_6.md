That’s a great debugging question 👍

The error message is telling you that somewhere in your code (or in a library method you’re calling), Java is trying to run:

```java
temporal1Inclusive.until(temporal2, someUnit);
```

but `temporal1Inclusive` is **null**. Since it’s null, you can’t call any methods on it → hence the `NullPointerException`.

---

### 🔍 Where to look first

1. **The stack trace**

   * The exception message you pasted is just the top line.
   * Look at the **first lines in your project’s package** (not in `java.*` or `org.*` library code).
   * That’s where your code called something like `someDate.until(...)` or indirectly triggered it.

2. **Places where you work with `LocalDate`, `LocalDateTime`, or `Period`**

   * `Temporal` is the supertype of Java time classes (`LocalDate`, `LocalDateTime`, etc.).
   * So check where you call things like:

     ```java
     startDate.until(endDate, ChronoUnit.DAYS);
     Period.between(startDate, endDate);
     ```

     Either `startDate` or `endDate` is null.

3. **Inputs or DTOs**

   * If the dates come from DTOs (like your `AnotherDTO` or `SomeDTO`), make sure those fields are initialized.
   * A common issue in tests: you mock or build a DTO but forget to set a `LocalDate` field, so it defaults to `null`.

4. **Your test setup**

   * Since you mentioned this happens while running tests via `mvn clean package`, check if your test is passing a DTO or variable with a missing date.
   * Production code might set it (e.g., from DB or request body), but your test stubs didn’t.

---

### ✅ Debugging approach

* Add loggers or breakpoints before the line that crashes:

  ```java
  log.info("startDate: {}", startDate);
  log.info("endDate: {}", endDate);
  ```
* If you can’t find the exact line, copy the **full stack trace** → the first line in your project’s package is the key.
* In unit tests: explicitly set values for date fields in your DTOs, e.g.:

  ```java
  dto.setStartDate(LocalDate.now());
  dto.setEndDate(LocalDate.now().plusDays(10));
  ```

---

👉 Do you want to share the **stack trace snippet** (at least the first few lines including your package) so I can point out exactly where to look?

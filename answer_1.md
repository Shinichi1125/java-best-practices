Ah, I see what’s happening 👍

The stack trace `NullPointer Cannot invoke "String.equals(Object)"` tells us **`someDTO.getTypeCode()` returned `null`**, so Java tried to do `null.equals("1")` → 💥 NPE.

---

### Why this happens

You thought you mocked it like this:

```java
SomeDTO someDTO = mock(SomeDTO.class);
when(someDTO.getTypeCode()).thenReturn("1");
```

But here are a few common pitfalls that would still lead to `null` being returned:

1. **Wrong instance is being used**

   * Maybe the code under test gets `SomeDTO` from somewhere else (not the one you stubbed).
   * If the real code constructs its own `new SomeDTO()`, your stubbed `mock(SomeDTO.class)` won’t apply.

2. **Method signature mismatch**

   * If `getTypeCode()` returns something other than `String` (e.g. `Integer`), your stubbing won’t match.
   * Mockito would then just return the default (`null` for objects).

3. **Stubbing applied too late**

   * If the code under test calls `getTypeCode()` before you run your `when(...)` setup, Mockito hasn’t had a chance to inject the behavior → it returns `null`.

4. **Mocking a final method / class**

   * If `SomeDTO` is a final class or `getTypeCode()` is final, Mockito (without the inline mocking extension) cannot override it → again you get `null`.

---

### How to debug

You can confirm what’s going on by adding this inside your test:

```java
System.out.println("Stubbed value: " + someDTO.getTypeCode());
```

* If it prints `"1"`, your mock is fine → the problem is you’re not passing this mock into the code under test.
* If it prints `null`, then either the method signature is mismatched or Mockito cannot stub it.

---

### ✅ Safer fix

If you *do* want to guard against NPEs regardless of mocking:

```java
if ("1".equals(someDTO.getTypeCode())) {
    // Safe: "1".equals(null) is false, not NPE
}
```

This is a Java best practice: put the constant string on the left-hand side.

---

👉 My bet, based on what you’ve shown: your production code isn’t actually using the mock you configured, but another `SomeDTO` (e.g. created inside the method under test).

Do you want to show me **how the test injects `someDTO` into the class under test**? That’ll let me tell you exactly why Mockito isn’t kicking in.

Excellent question — this is one of those subtle but important distinctions in JUnit assertions 👇

---

### 🔹 `assertEquals(expected, actual)`

* Checks **value equality** (via `.equals()`).
* It passes if the two objects are *logically equal*, even if they are *different instances* in memory.

Example:

```java
String a = new String("hello");
String b = new String("hello");
assertEquals(a, b);   // ✅ passes — because a.equals(b) == true
assertSame(a, b);     // ❌ fails — different objects in memory
```

---

### 🔹 `assertSame(expected, actual)`

* Checks **reference equality** (they point to the *exact same object* in memory).
* Equivalent to `expected == actual`.

Example:

```java
String a = "hello";
String b = a;
assertSame(a, b);   // ✅ passes — same reference
assertEquals(a, b); // ✅ also passes — equals() true
```

---

### ⚙️ In your test case

```java
assertEquals(HttpStatus.OK, res.getStatusCode());
assertSame(expected, res.getBody());
```

* For the **HTTP status**, you only care that the *value* is equal (`OK`), not that it’s the same instance — so `assertEquals` is appropriate.
* For the **response body**, you expect that the controller directly returned *the exact same object* (`expected`), not a copy — so `assertSame` ensures it’s the same reference in memory.

If `mapToSomeResponse()` returned a *new* but equal `SomeResponse`, `assertEquals` would still pass but `assertSame` would fail — so the test is verifying the controller truly returns the same object you stubbed.

---

Would you like me to show you a side-by-side example where both assertions pass vs. fail, depending on object identity?

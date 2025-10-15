Good question 👍

If the method `startSomeController()` returns a `SomeDTO`, you’ll usually want to check **two things**:

1. That the returned object is **not null** (it actually returned something).
2. That its **fields have the expected values** (to ensure correctness, not just existence).

---

### ✅ Basic test — only checks that something was returned

```java
@Test
void startSomeControllerTest() throws Exception {
    SomeDTO someDTO = someController.startSomeController();

    assertNotNull(someDTO, "Returned SomeDTO should not be null");
}
```

---

### ✅ Stronger test — also checks field values

If you know what values the `SomeDTO` should contain:

```java
@Test
void startSomeControllerTest() throws Exception {
    SomeDTO someDTO = someController.startSomeController();

    assertNotNull(someDTO);
    assertEquals("1234", someDTO.getId());
    assertEquals("John Doe", someDTO.getName());
    assertTrue(someDTO.isActive());
}
```

---

### ✅ Optional — if the method might throw exceptions or return null in some cases

```java
@Test
void startSomeController_shouldReturnValidDTO() {
    assertDoesNotThrow(() -> {
        SomeDTO result = someController.startSomeController();
        assertNotNull(result);
        assertEquals("expectedValue", result.getSomeField());
    });
}
```

---

If you tell me what the `SomeDTO` looks like (its fields or expected values),
I can show you a version that asserts all its key properties precisely.

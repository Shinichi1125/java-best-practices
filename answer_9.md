Good question 👍 When your method takes a `List<LocalDate>` instead of two `LocalDate` parameters, you just need to adapt the matcher.

Let’s say your method looks like:

```java
int helperA(List<LocalDate> dates) {
    // ...
}
```

---

### 1. If you don’t care about contents

```java
doReturn(10)
    .when(calculationService)
    .helperA(anyList());
```

This matches any `List<?>` (including `List<LocalDate>`).

---

### 2. If you want type safety

```java
doReturn(10)
    .when(calculationService)
    .helperA(anyListOf(LocalDate.class));
```

⚠️ `anyListOf()` was deprecated in Mockito 2.x, but if you’re on Mockito 1.x you may still see it.

With modern Mockito (>= 2.1), just use `anyList()` and rely on generics.

---

### 3. If you also want to allow `null` lists

```java
doReturn(10)
    .when(calculationService)
    .helperA(nullable(List.class));
```

This matches both `null` and any `List`.

---

### 4. If you want to match specific contents

For example: a list containing exactly two `LocalDate`s.

```java
doReturn(10)
    .when(calculationService)
    .helperA(argThat(list -> list != null && list.size() == 2));
```

This gives you fine-grained control.

---

✅ Example test call:

```java
List<LocalDate> dates = Arrays.asList(LocalDate.now(), LocalDate.now().plusDays(1));
int result = calculationService.helperA(dates);

assertEquals(10, result);
```

---

👉 Do you want me to show you how to **verify** that `helperA` was called with, say, a list containing exactly `null` or specific `LocalDate` values?

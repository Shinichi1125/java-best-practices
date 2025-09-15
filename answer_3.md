Thanks for clarifying the *actual* structure 🙏 — that helps a lot.
The structure you just showed:

```java
List<SomeDTO> someDTOs = parentDTO.getAnotherDTO().getSomeDTOList();
SomeDTO someDTO = someDTOs.get(0);

if (someDTO.getTypeCode().equals("1")) {
    // Do something
}
```

💡 tells me the problem isn’t the `.equals("1")` line itself, but that **the `SomeDTO` inside the list is not the same `SomeDTO` you stubbed**.

---

### Why your stubbing fails

When you do:

```java
SomeDTO someDTO = mock(SomeDTO.class);
lenient().when(someDTO.getTypeCode()).thenReturn("1");
```

and then separately mock:

```java
AnotherDTO anotherDTO = mock(AnotherDTO.class);
when(anotherDTO.getSomeDTOList()).thenReturn(List.of(someDTO));

ParentDTO parentDTO = mock(ParentDTO.class);
when(parentDTO.getAnotherDTO()).thenReturn(anotherDTO);
```

you’d *expect* that `parentDTO.getAnotherDTO().getSomeDTOList().get(0)` returns the **same mock** (`someDTO`).
That’s correct in principle.

But if somewhere in your production code:

* `parentDTO` or `anotherDTO` are **real objects** (not mocks), or
* `getSomeDTOList()` is overridden / recomputed to return **new DTOs**,

then by the time you call `someDTOs.get(0)` in production, you’re holding a **different instance** from the one you stubbed → so `getTypeCode()` returns the default `null`.

That’s why you get:

```
NullPointer Cannot invoke "String.equals(Object)" because the return value of "someDTO.getTypeCode" is null
```

---

### ✅ Solutions

#### **Option A (Preferred): Use real DTOs, not mocks**

DTOs are just data carriers. Mocking them usually creates this exact problem.
Instead:

```java
SomeDTO someDTO = new SomeDTO();
someDTO.setTypeCode("1");

AnotherDTO anotherDTO = new AnotherDTO();
anotherDTO.setSomeDTOList(List.of(someDTO));

ParentDTO parentDTO = new ParentDTO();
parentDTO.setAnotherDTO(anotherDTO);
```

Now your production code will *always* see `typeCode = "1"`.
No stubbing mismatch, no detached mocks.

---

#### **Option B: Debug mock wiring**

If you really want to stick with mocks, confirm the object graph is intact before the line where NPE happens:

```java
System.out.println(parentDTO.getAnotherDTO()); // should not be null
System.out.println(parentDTO.getAnotherDTO().getSomeDTOList()); // should contain your stubbed someDTO
System.out.println(parentDTO.getAnotherDTO().getSomeDTOList().get(0) == someDTO); // should be true
```

If that last line prints `false`, you know your stubbed `someDTO` isn’t making it into the actual list used.

---

👉 My strong recommendation: **don’t mock DTOs**. They’re plain data, so just build them with setters. Mock only *behavior* (services, repositories, clients).

---

Do you want me to rewrite your test end-to-end using **real DTOs with setters (Option A)** so you can confirm whether this solves the NPE?

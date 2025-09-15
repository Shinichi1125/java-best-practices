Thanks for clarifying 👌 — the nested DTOs absolutely explain why your stubbing didn’t “stick.”

Let’s break it down:

---

### Why you still get `null`

You stubbed a mock like this:

```java
SomeDTO someDTO = mock(SomeDTO.class);
when(someDTO.getTypeCode()).thenReturn("1");
```

But then you call:

```java
Result result = doTheWork(parentDTO);
```

Inside `doTheWork`, the code probably does something like:

```java
for (SomeDTO child : parentDTO.getAnotherDTO().getSomeDTOList()) {
    if (child.getTypeCode().equals("1")) { ... }
}
```

Here’s the catch:

* The `SomeDTO` objects that come from `parentDTO` are **not your mocked `someDTO` instance**.
* They’re either `new SomeDTO()` created inside test setup or real objects with `null` fields.
* So your `when(...).thenReturn("1")` was never applied to those objects.
* That’s why `child.getTypeCode()` → `null` → NPE.

---

### ✅ How to fix it

You have **two options**, depending on your test strategy:

---

#### **Option 1: Fully mock the object graph**

Instead of using real DTOs, make the whole graph mocks:

```java
SomeDTO someDTO = mock(SomeDTO.class);
when(someDTO.getTypeCode()).thenReturn("1");

AnotherDTO anotherDTO = mock(AnotherDTO.class);
when(anotherDTO.getSomeDTOList()).thenReturn(List.of(someDTO));

ParentDTO parentDTO = mock(ParentDTO.class);
when(parentDTO.getAnotherDTO()).thenReturn(anotherDTO);

Result result = doTheWork(parentDTO);
```

Now when `doTheWork` traverses `parentDTO → anotherDTO → someDTO`, it will always hit your mocks.

---

#### **Option 2: Use real objects, but set fields directly**

If `SomeDTO` is just a POJO with setters, you don’t need mocks at all:

```java
SomeDTO someDTO = new SomeDTO();
someDTO.setTypeCode("1");

AnotherDTO anotherDTO = new AnotherDTO();
anotherDTO.setSomeDTOList(List.of(someDTO));

ParentDTO parentDTO = new ParentDTO();
parentDTO.setAnotherDTO(anotherDTO);

Result result = doTheWork(parentDTO);
```

This is often cleaner than mocking simple data objects.

---

### Rule of Thumb

* **Mock behavior** (services, repos, external APIs).
* **Use real DTOs/POJOs** for data, since they’re just containers.

Mocking DTOs usually just makes things harder unless they’re very complex.

---

👉 Do you want me to rewrite your test example using **Option 2 (real DTOs with setters)** so you can see how it looks end-to-end?

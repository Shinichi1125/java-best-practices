Good catch 👍 — that’s a classic testing situation:

You have a class like this (simplified):

```java
public class CalculationService {

    public int mainMethod(int input) {
        int a = helperA(input);
        int b = helperB(input);
        return a + b;
    }

    int helperA(int x) { return x * 2; }
    int helperB(int x) { return x * 3; }
}
```

When you write a unit test for `mainMethod`, you don’t want to execute the real `helperA`/`helperB`. Instead, you want to **mock/stub them**.

---

### ✅ Solution: Use a **Spy**

Since the methods are in the same class, you can’t just `@Mock` the class and inject it — you need a **Spy**, which lets you partially mock a real instance.

Example with Mockito:

```java
@ExtendWith(MockitoExtension.class)
class CalculationServiceTest {

    @Spy
    @InjectMocks
    private CalculationService calculationService;

    @Test
    void testMainMethod_withMockedHelpers() {
        // Arrange
        doReturn(10).when(calculationService).helperA(anyInt());
        doReturn(20).when(calculationService).helperB(anyInt());

        // Act
        int result = calculationService.mainMethod(5);

        // Assert
        assertEquals(30, result);  // 10 + 20 from mocks
        verify(calculationService).helperA(5);
        verify(calculationService).helperB(5);
    }
}
```

---

### 🔑 Key points

* `@Spy` + `@InjectMocks` creates a real object but allows you to **stub specific methods**.
* Use `doReturn(...).when(spy).method(args)` to stub internal methods.
* You can still verify they were called.

---

### ⚠️ Things to watch out for

* The methods you mock **must not be `private` or `final`**. Mockito can only spy/stub `protected`, `package-private`, or `public` methods.
* If they’re `private`, you’d need **PowerMockito** (heavier, usually discouraged).
* If the internal methods are “heavy” and hard to test, that’s often a sign they might deserve to be extracted into their own class → easier to mock.

---

👉 Do you want me to show you how to **adapt this pattern** to your current worker class (with those private→protected methods like `mapDetails` and `createTransactions`)?

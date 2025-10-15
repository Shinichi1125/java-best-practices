You’re on the right track—here are the two big reasons you still get `expected: not <null>`:

---

## 1) You’re passing a **mock** `SomeRequest`

You declared `@Mock SomeRequest someRequest;`. On a Mockito mock, calling setters like `someRequest.setRequestId("1234")` does **nothing** (mocks don’t hold state). So inside the real method, getters will still return `null` unless you stub them:

```java
when(someRequest.getRequestId()).thenReturn("1234");
```

Better: for DTOs, **don’t mock**—use a real POJO and set fields normally.

---

## 2) You didn’t stub the **dependencies inside** `startTheProcess`

Your service’s method returns a `ProcessInstanceEvent` only if the Zeebe client call chain returns one. By default, Mockito mocks return `null`, so a chain like:

```java
zeebeClient.newCreateInstanceCommand()
           .bpmnProcessId("...")
           .latestVersion()
           .variables(someRequest)
           .send()
           .join();   // ← will return null unless you stub the chain
```

…will either return `null` or NPE unless you stub **every step** in that chain.

---

## A minimal, working unit test (mocking dependencies)

```java
@ExtendWith(MockitoExtension.class)
class ProcessServiceImplTest {

  @InjectMocks
  private ProcessServiceImpl processService;

  @Mock private ZeebeClient zeebeClient;
  @Mock private CreateInstanceCommandStep1 step1;
  @Mock private CreateInstanceCommandStep1.CreateInstanceCommandStep3 step3;
  @Mock private ZeebeFuture<ProcessInstanceEvent> future;
  @Mock private ProcessInstanceEvent event;

  @Test
  void startTheProcess_returnsEvent() {
    // Use a REAL request object (don’t mock DTOs)
    SomeRequest req = new SomeRequest();
    req.setRequestId("1234");
    // ... set other fields the method expects

    // Stub Zeebe chain
    when(zeebeClient.newCreateInstanceCommand()).thenReturn(step1);
    when(step1.bpmnProcessId(anyString())).thenReturn(step3);
    when(step3.latestVersion()).thenReturn(step3);
    when(step3.variables(any())).thenReturn(step3);
    when(step3.send()).thenReturn(future);
    when(future.join()).thenReturn(event);

    // Act
    ProcessInstanceEvent response = processService.startTheProcess(req);

    // Assert
    assertNotNull(response);
    assertSame(event, response);
    verify(zeebeClient).newCreateInstanceCommand();
    verify(step3).variables(any()); // or capture and assert payload
  }
}
```

### Notes

* Remove `@Mock SomeRequest someRequest;` and build a **real** `SomeRequest`.
* If you insist on mocking `SomeRequest`, you must stub its getters:

  ```java
  when(someRequest.getRequestId()).thenReturn("1234");
  ```
* Don’t try to stub `processService.startTheProcess(...)` itself—`@InjectMocks` is the **real** class under test. Stub its **collaborators** (Zeebe client, etc.).

---

## Why the earlier errors happened

* “Invalid use of argument matchers”: you tried to stub a **real** method on `@InjectMocks`.
* `expected: not <null>`: the Zeebe chain returned `null` because nothing was stubbed; or your mocked `SomeRequest` had no values.

---

If you paste the body of `startTheProcess(...)`, I can tailor the stubs exactly (topics used, variables payload shape, etc.).

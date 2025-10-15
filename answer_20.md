`NullInsteadOfMock` means the thing you’re passing to `when(...)` is **null** (or not a mock/spy). In your line

```java
doReturn(dto).when(controller).handleProcess("REQ-1");
```

`controller` must be a **spy** (or mock). If Mockito hasn’t initialized your annotations, `controller` will be `null`, and you’ll get exactly that error.

It’s **not** because the method `handleProcess(...)` declares `throws Exception`. Using `doReturn(...).when(spy)...` is the right way for spies even for methods that throw.

## Fix checklist

1. Make sure Mockito is initialized:

* JUnit 5: add the extension on the class.
* Or call `MockitoAnnotations.openMocks(this)` in `@BeforeEach`.

2. Make sure `controller` is a **spy**, not a plain instance:

* Use `@Spy @InjectMocks` **or** create it manually with `spy(new SomeController(someService))`.

3. Make sure the dependency mock exists so Mockito can construct/inject the controller.

### Working example (JUnit 5)

```java
@ExtendWith(MockitoExtension.class)
class SomeControllerTest {

  @Mock
  private SomeService someService;

  @Spy
  @InjectMocks
  private SomeController controller; // spy created & injected by Mockito

  @Test
  void startTheProcess_returnsOk() throws Exception {
    // Arrange
    SomeRequest req = new SomeRequest();
    req.setRequestId("REQ-1");

    SomeDTO dto = new SomeDTO();
    dto.setProcessId("PID-1");

    ProcessInstanceEvent event = mock(ProcessInstanceEvent.class);
    SomeResponse expected = new SomeResponse();

    // stub protected helpers on the spy
    doReturn(dto).when(controller).handleProcess("REQ-1");
    doReturn(expected).when(controller).mapToSomeResponse(event, "PID-1", HttpStatus.OK);

    // stub service
    when(someService.startProcess(any(SomeRequest.class))).thenReturn(event);

    // Act
    ResponseEntity<SomeResponse> res = controller.startTheProcess(req);

    // Assert
    assertNotNull(res);
    assertEquals(HttpStatus.OK, res.getStatusCode());
    assertSame(expected, res.getBody());
  }
}
```

### Alternative manual setup (if you don’t use the extension)

```java
private AutoCloseable mocks;

@Mock private SomeService someService;
private SomeController controller;

@BeforeEach
void init() {
  mocks = MockitoAnnotations.openMocks(this);
  controller = spy(new SomeController(someService));
}

@AfterEach
void close() throws Exception { mocks.close(); }
```

Then the same `doReturn(...).when(controller)...` stubbings will work.

---

**Summary:** Your error happens because `controller` wasn’t an initialized spy (likely `null`). Initialize Mockito and ensure `controller` is a spy; the `throws Exception` on `handleProcess` is not the cause.

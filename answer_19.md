Here’s a clean JUnit 5 + Mockito test that **spies** the controller so you can stub the two **protected** helper methods, while **mocking** the service dependency. It also verifies order and asserts the response.

```java
package your.pkg;

import static org.junit.jupiter.api.Assertions.*;
import static org.mockito.Mockito.*;

import io.camunda.zeebe.client.api.response.ProcessInstanceEvent;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.InOrder;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.Spy;
import org.mockito.junit.jupiter.MockitoExtension;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;

@ExtendWith(MockitoExtension.class)
class SomeControllerTest {

    @Spy
    @InjectMocks
    private SomeController controller;    // real instance, but we can stub protected methods

    @Mock
    private SomeService someService;      // dependency used inside controller

    @Mock
    private ProcessInstanceEvent event;   // returned by the service

    @Test
    void startTheProcess_returnsOk_andMapsResponse() throws Exception {
        // Arrange: use REAL DTOs for simple data containers
        SomeRequest req = new SomeRequest();
        req.setRequestId("REQ-1");

        SomeDTO dto = new SomeDTO();
        dto.setProcessId("PID-1");

        SomeResponse expectedBody = new SomeResponse();
        // (set expectedBody fields if you want to assert them later)

        // Stub the protected helpers on the spy
        doReturn(dto).when(controller).handleProcess("REQ-1");
        doReturn(expectedBody).when(controller).mapToSomeResponse(event, "PID-1", HttpStatus.OK);

        // Stub the service call chain
        when(someService.startProcess(any(SomeRequest.class))).thenReturn(event);

        // Act
        ResponseEntity<SomeResponse> response = controller.startTheProcess(req);

        // Assert
        assertNotNull(response);
        assertEquals(HttpStatus.OK, response.getStatusCode());
        assertSame(expectedBody, response.getBody(), "Response body should be the mapped SomeResponse");

        // The controller sets processId on the request before calling the service
        assertEquals("PID-1", req.getProcessId());

        // Verify call order (optional but nice)
        InOrder inOrder = inOrder(controller, someService);
        inOrder.verify(controller).handleProcess("REQ-1");
        inOrder.verify(someService).startProcess(req);
        inOrder.verify(controller).mapToSomeResponse(event, "PID-1", HttpStatus.OK);
        verifyNoMoreInteractions(someService, controller);
    }
}
```

### Notes

* We use `@Spy @InjectMocks` to create a real controller instance but allow stubbing of its **protected** methods:

  ```java
  doReturn(dto).when(controller).handleProcess("REQ-1");
  doReturn(expectedBody).when(controller).mapToSomeResponse(event, "PID-1", HttpStatus.OK);
  ```
* **Don’t mock DTOs** like `SomeRequest`/`SomeDTO`/`SomeResponse` unless necessary—use real objects and set fields.
* If your controller uses `new ResponseEntity<>(body, HttpStatus.OK)` or `ResponseEntity.ok(body)`, the status assertion will pass either way.

If you share the exact signatures of `handleProcess` and `mapToSomeResponse`, I can tailor the argument matchers (e.g., `eq(...)`, captors) precisely.

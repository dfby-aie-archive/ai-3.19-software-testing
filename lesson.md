# Lesson: Software Testing: Unit and Integration Testing

## Lesson Overview

This lesson introduces students to the fundamentals of software testing in Java, focusing on unit testing and integration testing within a Spring Boot application. Learners will understand how automated testing improves code quality, prevents regressions, and supports clean, maintainable development. Through hands-on examples, students will practice writing unit tests using JUnit and Mockito and perform integration testing using Spring Boot's MockMvc framework.

## Lesson Objectives

By the end of this lesson, learners will be able to:

1. **Explain** the purpose of software testing and the TDD cycle
2. **Write** unit tests using JUnit with setup, execution, and assertion phases
3. **Mock** dependencies using Mockito to test the service layer in isolation
4. **Perform** integration testing of REST endpoints using Spring Boot and MockMvc

---

## Part 1: Introduction to Software Testing

Software testing is the process of evaluating a software application to ensure that it behaves correctly, meets business requirements, and remains reliable under different conditions. Testing helps identify defects early, improves code quality, and provides confidence that changes or new features will not break existing functionality.

At a fundamental level, software testing answers two key questions:

1. Does the software do what it is supposed to do?
2. Does it continue to work correctly when the code changes?

Automated tests allow developers to verify functionality quickly and consistently during development, reducing the need for repetitive manual testing.

### Why Do We Test Software?

- Ensure functional correctness — features work as intended
- Prevent regressions when modifying or adding code
- Improve code structure and maintainability
- Gain confidence when refactoring complex logic
- Reduce manual QA effort through automation
- Build reliable, production-ready applications

### Types of Software Testing

Software testing spans multiple levels, each focusing on a different scope of the system.

<img src="./assets/images/software-testing.jpg" width=500 style="background-color: #fff; padding: 20px;border-radius: 5px;border: 1px solid #eee;">

**Unit Testing** — tests individual units of code (typically methods or classes) in complete isolation. Very fast, automated, uses mocks to simulate dependencies.

**Integration Testing** — tests how multiple components work together. Uses real configurations, slower than unit tests, identifies issues in wiring, data flow, and API behaviour.

**Functional / End-to-End Testing** — tests a complete user workflow from start to end, validating behaviour from the user's perspective.

**System Testing** — tests the application as a whole, ensuring all modules function correctly together.

**Acceptance Testing (UAT)** — conducted by QA teams or business stakeholders to confirm the system meets business requirements.

**Regression Testing** — ensures that previously working functionality still works after introducing changes or new features.

**Performance Testing** — evaluates speed, responsiveness, and scalability under expected and peak loads.

**Security Testing** — ensures the application is protected against vulnerabilities such as SQL injection, XSS, and authentication flaws.

This lesson focuses on **Unit Testing** and **Integration Testing** — the two types that form the foundation of a reliable and maintainable Java/Spring Boot backend.

You can read more here: https://www.guru99.com/software-testing-introduction-importance.html

---

## Part 2: Introduction to Test Driven Development (TDD)

In the usual software development process, developers write code first and then test it. In the **TDD** approach, developers write tests first and then write code to pass those tests.

<img src="https://www.nimblework.com/wp-content/uploads/2022/12/tdd_flow1.gif" width=350 style="background-color: #fff; padding: 20px;border-radius: 5px;border: 1px solid #eee;">

The TDD cycle follows 3 phases:

- **Red** — Write a test that fails
- **Green** — Write the simplest code to pass the test
- **Refactor** — Refactor the code to make it better

For example, in our `simple-crm` project, we might want to unit test that a customer can be created successfully. Following TDD, we write the test first:

```java
@ExtendWith(MockitoExtension.class)
public class CustomerServiceImplTest {

  @Mock
  private CustomerRepository customerRepository;

  @InjectMocks
  private CustomerServiceImpl customerService;

  @Test
  public void testCreateCustomer() {

    Customer customer = Customer.builder().firstName("Clint").lastName("Barton").email("clint@avengers.com")
        .contactNo("12345678").jobTitle("Special Agent").yearOfBirth(1975).build();

    when((customerRepository.save(customer))).thenReturn(customer);

    Customer savedCustomer = customerService.createCustomer(customer);

    assertEquals(customer, savedCustomer, "The saved customer should be the same as the new customer");

    verify(customerRepository, times(1)).save(customer);
  }
}
```

Then we write the code to pass this test. Note that `customerRepository` must be declared and injected via constructor — this is the standard approach in Spring Boot:

```java
@Service
public class CustomerServiceImpl implements CustomerService {

  private final CustomerRepository customerRepository;

  public CustomerServiceImpl(CustomerRepository customerRepository) {
    this.customerRepository = customerRepository;
  }

  @Override
  public Customer createCustomer(Customer customer) {
    return customerRepository.save(customer);
  }
}
```

> 📖 **Self Reading — Constructor Injection:** Constructor injection (instead of `@Autowired` on a field) is the current industry standard. It makes dependencies explicit, enables immutability with `final`, and makes unit testing straightforward because you can construct the class directly without a Spring context.

Using TDD can result in better code quality and fewer bugs because issues are caught earlier. It also increases confidence when refactoring because tests catch any regressions. However, it may not be suitable for all projects due to upfront time investment, learning curve, and maintenance cost of keeping tests up to date. Some teams adopt a hybrid approach — TDD for critical business logic and post-implementation tests for less critical features.

---

## Part 3: Unit Testing

Currently, we test our application by running it and manually calling endpoints via Postman. This is time-consuming and unreliable for complex logic. We should automate our testing by writing unit tests.

### What is Unit Testing?

Unit testing tests individual units or components of a software application in isolation. These tests are independent of other units, automated, and can be reproduced quickly — which means they can be run frequently during development without slowing down the team.

When we add new features or refactor code, running the unit tests immediately tells us if anything broke.

### Unit Testing Frameworks

We will use:

- [JUnit](https://junit.org/junit5/) — a unit testing framework for creating and running tests
- [Mockito](https://site.mockito.org/) — a mocking framework for simulating dependencies

Both are included in the `spring-boot-starter-test` dependency that Spring Boot adds by default.

### Unit Test Example with `@Test`

Let's see a simple example using our `simple-crm` codebase.

Create a `DemoService.java` in `src/main/java/com/ntu/sg/simple_crm/` and a corresponding `DemoServiceTest.java` in `src/test/java/com/ntu/sg/simple_crm/`. These files are for demonstration only and can be deleted after this exercise.

```java
public class DemoService {

    public int add(int a, int b) {
        return a + b;
    }

    public int subtract(int a, int b) {
        return a - b;
    }
}
```

There are 3 steps in writing a unit test — this is known as the **Arrange-Act-Assert** pattern (also called Given-When-Then):

- **Arrange** — set up everything you need: create objects, define inputs
- **Act** — call the single method you are testing
- **Assert** — verify the result is what you expected

```java
public class DemoServiceTest {

  @Test
  public void testAdd() {
    // 1. ARRANGE - set up inputs and expected values
    DemoService demoService = new DemoService();
    int expectedResult = 8;

    // 2. ACT - call the method being tested
    int actualResult = demoService.add(3, 5);

    // 3. ASSERT - verify the result
    assertEquals(expectedResult, actualResult, "3 + 5 should be 8");
  }

  @Test
  public void testSubtract() {
    // 1. ARRANGE
    DemoService demoService = new DemoService();
    int expectedResult = 2;

    // 2. ACT
    int actualResult = demoService.subtract(5, 3);

    // 3. ASSERT
    assertEquals(expectedResult, actualResult, "5 - 3 should be 2");
  }
}
```

Run the test by clicking the green arrow next to the test method. A passing test shows a green tick with no console output — **silence means success**. Output only appears when a test fails, showing what was expected versus what was actually returned.

Notice we are not using dependency injection here — we are instantiating the class directly with `new`. This is intentional: unit tests do not start the Spring context, so there are no beans available. This is what makes them milliseconds fast.

> 📖 **Self Reading — Why `new` instead of DI:** The fact that you *can* test a class with just `new` means it is well-designed — no hidden Spring magic required. In production, if a class is so tightly coupled to Spring that you cannot test it without the container, that is a design smell.

Now try introducing a bug:

```java
public int add(int a, int b) {
    return a * b; // wrong operation
}
```

Run the test again — it should fail, demonstrating that the test caught the regression. Fix it before moving on.

### Test Naming Conventions

Consistent test naming helps teams understand what failed and why without reading the test body. A widely adopted convention in production codebases is:

```
methodName_scenario_expectedBehaviour
```

| Style | Example |
|---|---|
| Simple (common in tutorials) | `testCreateCustomer` |
| Descriptive (production standard) | `createCustomer_validInput_returnsCreatedCustomer` |
| BDD style | `givenValidCustomer_whenCreateCustomer_thenReturnSavedCustomer` |

> 📖 **Self Reading — BDD:** BDD stands for Behaviour Driven Development — an extension of TDD where tests are written in plain English-like language so that non-technical stakeholders (product owners, QA, business analysts) can read and understand what the system is supposed to do. The `given/when/then` naming style comes from BDD.

### Assertions

| Method | Description |
|---|---|
| `assertEquals()` | Checks that two primitives/objects are equal |
| `assertNotEquals()` | Checks that two primitives/objects are not equal |
| `assertTrue()` | Checks that a condition is true |
| `assertFalse()` | Checks that a condition is false |
| `assertNull()` | Checks that an object is null |
| `assertNotNull()` | Checks that an object is not null |
| `assertArrayEquals()` | Checks that two arrays are equal |
| `assertThrows()` | Checks that an exception is thrown |

Read more: https://junit.org/junit5/docs/current/user-guide/#writing-tests-assertions

### Lifecycle Methods

JUnit lifecycle methods allow us to perform setup and teardown operations:

| Annotation | Description |
|---|---|
| `@BeforeAll` | Executed once before all test methods in the class |
| `@BeforeEach` | Executed before each test method |
| `@AfterEach` | Executed after each test method |
| `@AfterAll` | Executed once after all test methods in the class |

We can move the instantiation of `DemoService` into `@BeforeEach` to avoid repeating it in every test:

```java
public class DemoServiceTest {

  DemoService demoService;

  @BeforeEach
  public void init() {
    demoService = new DemoService();
  }
}
```

### Generating HTML Report

Run `mvn surefire-report:report` and a HTML report will be generated at `target/site/surefire-report.html`. Open it in a browser to see which tests passed, which failed, how long each took, and failure details.

> 📖 **Self Reading — Why this matters in production:** In real projects, tests run automatically in a CI/CD pipeline (GitHub Actions, Jenkins, etc.) on every push. The pipeline publishes this report on the build dashboard so the whole team can see test results without running the project locally.

### 👨‍💻 Activity **(10 minutes)**

Add 3 more methods to `DemoService` and write unit tests for each:

```java
public int multiply(int a, int b) {
    return a * b;
}

public int divide(int a, int b) {
    return a / b;
}

public boolean isEven(int a) {
    return a % 2 == 0;
}
```

---

## Part 4: Service Layer Unit Testing with Mockito

Recall that the service layer contains our business logic and depends on the repository layer. When unit testing the service layer, we do not want to interact with the real database — we only want to test the logic itself. We achieve this by **mocking** the repository.

**Mockito** creates a fake version of the repository. When the service calls a repository method, Mockito intercepts it — the real repository is never invoked and no database is touched. Mockito simply returns the fake result you configured. Each layer is tested in isolation and owns its own tests.

### Prepare the Customer Class

Before writing service tests, add the following Lombok annotations to the `Customer` class if not already present:

```java
@Builder           // enables the builder pattern for creating Customer objects in tests
@EqualsAndHashCode // required for assertEquals() to compare Customer objects by value
@NoArgsConstructor // required by JPA
@AllArgsConstructor // required by @Builder when other constructors exist
public class Customer {
  // ...
}
```

> 📖 **Self Reading — Why these annotations:**
> - `@Builder` generates a fluent builder pattern so you can create objects cleanly in tests: `Customer.builder().firstName("Clint").build()`
> - `@EqualsAndHashCode` generates `equals()` and `hashCode()` based on field values. Without it, Java compares object references (memory addresses), not field values — two separate `Customer` objects with identical data would fail `assertEquals()` even though they look the same.
> - `@AllArgsConstructor` is required alongside `@Builder` when the class already has a custom constructor — without it, Lombok cannot generate the builder correctly.

### Mocking with Mockito

Create `CustomerServiceImplTest.java` in the corresponding test folder. Test files must mirror the source folder structure:

- Source: `src/main/java/com/ntu/sg/simple_crm/service/CustomerServiceImpl.java`
- Test: `src/test/java/com/ntu/sg/simple_crm/service/CustomerServiceImplTest.java`

All test methods in this section go **inside** the `CustomerServiceImplTest` class body.

```java
@ExtendWith(MockitoExtension.class)
public class CustomerServiceImplTest {

  @Mock
  private CustomerRepository customerRepository;

  @InjectMocks
  private CustomerServiceImpl customerService;

}
```

- `@ExtendWith(MockitoExtension.class)` — enables Mockito for JUnit 5
- `@Mock` — tells Mockito to create a mock `CustomerRepository`
- `@InjectMocks` — tells Mockito to inject the mock into `CustomerServiceImpl`

> **Common mistake:** `@InjectMocks` must target the **concrete class** (`CustomerServiceImpl`), not the interface (`CustomerService`). Mockito creates an instance of the concrete class and injects the mocks — it cannot instantiate an interface.

### Test Create Customer

This test verifies that when `createCustomer()` is called with a valid `Customer` object, the service correctly calls `save()` on the repository and returns the saved customer.

```java
@Test
public void testCreateCustomer() {

  // 1. ARRANGE
  Customer customer = Customer.builder()
      .firstName("Clint").lastName("Barton")
      .email("clint@avengers.com").contactNo("12345678")
      .jobTitle("Special Agent").yearOfBirth(1975)
      .build();

  // Tell Mockito: when save() is called with this customer, return this customer
  // The real repository is never called — no database is touched
  when(customerRepository.save(customer)).thenReturn(customer);

  // 2. ACT
  Customer savedCustomer = customerService.createCustomer(customer);

  // 3. ASSERT
  assertEquals(customer, savedCustomer, "The saved customer should be the same as the new customer");
  verify(customerRepository, times(1)).save(customer);
}
```

- `when(...).thenReturn(...)` — programs the mock: "when this method is called, return this value." This is setup, not an assertion.
- `verify(...)` — confirms the mocked method was actually called the expected number of times. Catches bugs where your service never called the repository at all.

### Test Get Customer

This test verifies that when `getCustomer()` is called with a valid ID, the service correctly retrieves and returns the customer.

```java
@Test
public void testGetCustomer() {
  // 1. ARRANGE
  Customer customer = Customer.builder()
      .firstName("Clint").lastName("Barton")
      .email("clint@avengers.com").contactNo("12345678")
      .jobTitle("Special Agent").yearOfBirth(1975)
      .build();

  Long customerId = 1L;

  // Optional.of(customer) wraps the customer in an Optional — 
  // this is how the repository signals "I found a record"
  when(customerRepository.findById(customerId)).thenReturn(Optional.of(customer));

  // 2. ACT
  Customer retrievedCustomer = customerService.getCustomer(customerId);

  // 3. ASSERT
  assertEquals(customer, retrievedCustomer);
}
```

> 📖 **Self Reading — Why `Optional`:** Spring Data JPA's `findById()` returns `Optional<Customer>` instead of `Customer` directly. An `Optional` is a container that either holds a value (`Optional.of(customer)`) or is empty (`Optional.empty()`). This forces the developer to explicitly handle the "not found" case, preventing `NullPointerException`.

### Test Get Customer Not Found

This test verifies that when `getCustomer()` is called with an ID that does not exist, the service throws a `CustomerNotFoundException`.

```java
@Test
public void testGetCustomerNotFound() {
  Long customerId = 1L;

  // Optional.empty() simulates no record found in the database
  when(customerRepository.findById(customerId)).thenReturn(Optional.empty());

  // assertThrows verifies that the lambda throws the expected exception
  // If the exception is NOT thrown, the test fails
  assertThrows(CustomerNotFoundException.class, () -> customerService.getCustomer(customerId));
}
```

- `Optional.empty()` — simulates a record not found in the database
- `assertThrows(ExceptionClass, lambda)` — passes the expected exception class and the code that should trigger it. If the exception is not thrown, the test fails.

---

## Part 5: Integration Testing with MockMvc

Unit tests validate individual components in isolation. Integration tests validate how components work together — the full request/response cycle from controller through service to repository.

### Mockito vs MockMvc — Which Tool for Which Layer?

| Tool | Tests | Mocks | Database |
|---|---|---|---|
| **Mockito** | Service layer | Repository (fake) | Not touched |
| **MockMvc** | Controller layer (full stack) | HTTP server only | Real |

**Mockito** mocks the repository so you can test service logic in isolation. **MockMvc** simulates the HTTP transport layer — it pretends to be a browser sending requests to your API and checks the response — but everything behind the controller (service, repository, database) is **real**.

> 📖 **Self Reading — Common misconception:** The "Mock" in MockMvc refers only to the fake HTTP server — no real server starts and no HTTP port is opened. It does not mock the application layers. `@SpringBootTest` wires up the full application context, so integration tests do real database work. This is why integration tests are slower than unit tests.

### `@SpringBootTest` vs `@WebMvcTest`

| Annotation | What it loads | Speed | Use when |
|---|---|---|---|
| `@SpringBootTest` | Full application context — all beans, datasource, security, etc. | Slower | Testing the full stack end-to-end |
| `@WebMvcTest` | Web layer only — controllers, filters, `@ControllerAdvice`. No service/repo beans. | Faster | Testing controller logic in isolation with mocked services |

In production teams, `@WebMvcTest` is preferred for controller-layer tests because it is faster and more focused. `@SpringBootTest` is used for true end-to-end or database integration tests. In this lesson we use `@SpringBootTest` to test the full stack.

### Setting Up the Integration Test

Create `CustomerControllerTest.java` in the corresponding test folder:

- Source: `src/main/java/com/ntu/sg/simple_crm/controller/CustomerController.java`
- Test: `src/test/java/com/ntu/sg/simple_crm/controller/CustomerControllerTest.java`

Add the following static imports at the top of the file — without these, `status()`, `content()`, and `jsonPath()` will not be recognised:

```java
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
```

All test methods in this section go **inside** the `CustomerControllerTest` class body.

```java
@SpringBootTest
@AutoConfigureMockMvc
@Transactional
public class CustomerControllerTest {

  @Autowired
  private MockMvc mockMvc;

  @Autowired
  private ObjectMapper objectMapper;
}
```

- `@SpringBootTest` — loads the full Spring application context
- `@AutoConfigureMockMvc` — auto-wires the `MockMvc` bean
- `@Transactional` — each test runs inside a transaction that is automatically **rolled back** after the test completes, keeping the database clean between tests
- `ObjectMapper` — converts Java objects to JSON strings (provided by Jackson)

> 📖 **Self Reading — Why `@Transactional` matters:** Without it, every test that writes data to the database leaves that data behind. Tests start polluting each other — a record created in test 1 affects the count in test 2. `@Transactional` on the test class rolls back every write after each test, giving each test a clean slate without any manual cleanup or reset scripts.

### Understanding the Test Structure

Every MockMvc test follows the same pattern:

**`RequestBuilder`** — an object that represents the HTTP request you want to send, built using `MockMvcRequestBuilders`:

```java
MockMvcRequestBuilders.get("/customers")     // GET request
MockMvcRequestBuilders.post("/customers")    // POST request
MockMvcRequestBuilders.put("/customers/1")   // PUT request
MockMvcRequestBuilders.delete("/customers/1") // DELETE request
```

**`mockMvc.perform(request)`** — sends the request. Like hitting Send in Postman.

**`.andExpect()`** — each call is one assertion, chained together:

```java
.andExpect(status().isOk())                                    // HTTP 200
.andExpect(content().contentType(MediaType.APPLICATION_JSON))  // response is JSON
.andExpect(jsonPath("$.id").value(1))                          // id field equals 1
```

> 📖 **Self Reading — JsonPath:** JsonPath is a query language for JSON. The `$` represents the root of the JSON response. So `$.id` means "get the `id` field from the root of the JSON object" and `$.size()` means "get the size of the JSON array." For example, if the API returns `{"id": 1, "firstName": "John"}`, then `jsonPath("$.firstName").value("John")` asserts that the `firstName` field equals `"John"`.

### Test Get Customer by ID

This test verifies that sending a GET request to `/customers/1` returns a `200 OK` response with the correct customer data.

```java
@Test
@DisplayName("Get customer by Id")
public void getCustomerByIdTest() throws Exception {
  // Step 1: Build a GET request to /customers/1
  RequestBuilder request = MockMvcRequestBuilders.get("/customers/1");

  // Step 2: Perform the request and assert
  mockMvc.perform(request)
      .andExpect(status().isOk())
      .andExpect(content().contentType(MediaType.APPLICATION_JSON))
      .andExpect(jsonPath("$.id").value(1));
}
```

### Test Get All Customers

This test verifies that sending a GET request to `/customers` returns a `200 OK` response with a JSON array of all customers.

> ⚠️ Note: This test asserts that 4 customers are returned. This depends on the `DataLoader` preloading exactly 4 customers. If you change the DataLoader, update this value accordingly.

```java
@Test
public void getAllCustomersTest() throws Exception {
  RequestBuilder request = MockMvcRequestBuilders.get("/customers");

  mockMvc.perform(request)
      .andExpect(status().isOk())
      .andExpect(content().contentType(MediaType.APPLICATION_JSON))
      .andExpect(jsonPath("$.size()").value(4));
}
```

> 📖 **Self Reading — Fragile assertions:** Hardcoding `4` here is a common source of brittle tests in real projects. If someone adds a customer to the DataLoader, this test breaks with no obvious reason. A more resilient alternative is `jsonPath("$.size()").value(org.hamcrest.Matchers.greaterThan(0))` — asserting that results exist without depending on an exact count.

### Test Valid Customer Creation

This test verifies that sending a valid POST request to `/customers` creates a real customer record in the database and returns `201 Created` with the saved customer data. Because `@Transactional` is on the test class, the record is automatically rolled back after the test — the database is left clean.

```java
@Test
public void validCustomerCreationTest() throws Exception {
  // Step 1: Create a Customer object
  Customer newCustomer = Customer.builder()
      .firstName("Clint").lastName("Barton")
      .email("clint@avengers.com").contactNo("12345678")
      .jobTitle("Special Agent").yearOfBirth(1975)
      .build();

  // Step 2: Convert the Java object to a JSON-formatted String
  // objectMapper.writeValueAsString() produces: {"firstName":"Clint","lastName":"Barton",...}
  // It is a String in Java, but formatted as JSON text
  String newCustomerAsJSON = objectMapper.writeValueAsString(newCustomer);

  // Step 3: Build the POST request
  // .contentType(MediaType.APPLICATION_JSON) tells Spring to treat the string as JSON
  // and deserialize it back into a Customer object on the controller side
  RequestBuilder request = MockMvcRequestBuilders.post("/customers")
      .contentType(MediaType.APPLICATION_JSON)
      .content(newCustomerAsJSON);

  // Step 4: Perform and assert
  mockMvc.perform(request)
      .andExpect(status().isCreated())
      .andExpect(content().contentType(MediaType.APPLICATION_JSON))
      .andExpect(jsonPath("$.id").exists())           // verify an id was assigned
      .andExpect(jsonPath("$.firstName").value("Clint"))
      .andExpect(jsonPath("$.lastName").value("Barton"));
}
```

> 📖 **Self Reading — Why `$.id` exists() instead of a specific value:** Asserting an exact ID value (e.g. `.value(5)`) is a brittle pattern — it breaks the moment the DataLoader or test execution order changes. In production test suites, assert that the field *exists* and has a valid value, not that it equals a specific number.

> 📖 **Self Reading — JSON is always text on the wire:** This is exactly what happens in real HTTP communication. Postman does the same thing — it serializes your JSON body to a string, sets the Content-Type header, and sends it. `ObjectMapper` is doing programmatically what Postman does for you visually.

### Test Invalid Customer Creation

This test verifies that sending a POST request with invalid data returns `400 Bad Request`. The validation annotations from Lesson 3.17 (`@NotBlank`, `@Email`) reject the request before it even reaches the service layer.

```java
@Test
public void invalidCustomerCreationTest() throws Exception {
  // Step 1: Create a Customer object with invalid fields
  // firstName and lastName are blank — violates @NotBlank
  // email is not a valid email format — violates @Email
  Customer invalidCustomer = Customer.builder()
      .firstName("  ")
      .lastName("  ")
      .email("not-a-valid-email")
      .contactNo("12345678")
      .jobTitle("Manager")
      .yearOfBirth(1990)
      .build();

  // Step 2: Convert to JSON
  String invalidCustomerAsJSON = objectMapper.writeValueAsString(invalidCustomer);

  // Step 3: Build the request
  RequestBuilder request = MockMvcRequestBuilders.post("/customers")
      .contentType(MediaType.APPLICATION_JSON)
      .content(invalidCustomerAsJSON);

  // Step 4: Perform and assert
  mockMvc.perform(request)
      .andExpect(status().isBadRequest())
      .andExpect(content().contentType(MediaType.APPLICATION_JSON));
}
```

This test is important because it confirms your validation layer is working correctly end-to-end. Unit tests cannot catch this — only an integration test that sends a real HTTP request through the full stack can confirm that `@NotBlank` and `@Email` are correctly wired and rejecting bad input at the controller level.

---

END
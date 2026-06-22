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

> **Instructor note:** Constructor injection (instead of `@Autowired` on a field) is the current industry standard. It makes dependencies explicit, enables immutability with `final`, and — critically — makes unit testing straightforward because you can construct the class directly without a Spring context.

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

Create a `DemoService.java`:

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

Create a corresponding `DemoServiceTest.java` in `src/test/java`. There are 3 steps in writing a unit test — this is known as the **Arrange-Act-Assert** pattern (also called Given-When-Then):

```java
public class DemoServiceTest {

  @Test
  public void testAdd() {
    // 1. SETUP
    DemoService demoService = new DemoService();
    int expectedResult = 8;

    // 2. EXECUTE
    int actualResult = demoService.add(3, 5);

    // 3. ASSERT
    assertEquals(expectedResult, actualResult, "3 + 5 should be 8");
  }

  @Test
  public void testSubtract() {
    // 1. SETUP
    DemoService demoService = new DemoService();
    int expectedResult = 2;

    // 2. EXECUTE
    int actualResult = demoService.subtract(5, 3);

    // 3. ASSERT
    assertEquals(expectedResult, actualResult, "5 - 3 should be 2");
  }
}
```

Run the test by clicking the green arrow next to the test method, or run `mvn test` in the terminal.

Now try introducing a bug:

```java
public int add(int a, int b) {
    return a * b; // wrong operation
}
```

Run the test again — it should fail, demonstrating that the test caught the regression.

Once you are comfortable, tests can be written more concisely:

```java
@Test
public void testAdd() {
  DemoService demoService = new DemoService();
  assertEquals(8, demoService.add(3, 5), "3 + 5 should be 8");
}
```

Notice we are not using dependency injection here. We are testing without spinning up the Spring context, which means no beans are available — but this also means tests run much faster.

### Test Naming Conventions

Consistent test naming helps teams understand what failed and why without reading the test body. A widely adopted convention in production codebases is:

```
methodName_scenario_expectedBehaviour
```

For example:

| Style | Example |
|---|---|
| Simple (common in tutorials) | `testCreateCustomer` |
| Descriptive (production standard) | `createCustomer_validInput_returnsCreatedCustomer` |
| BDD style | `givenValidCustomer_whenCreateCustomer_thenReturnSavedCustomer` |

You will see both styles in practice. The descriptive style is preferred on team projects because the test name itself acts as documentation.

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

Run `mvn surefire-report:report` and a HTML report will be generated at `target/site/surefire-report.html`.

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

### Prepare the Customer Class

Before writing service tests, add the following Lombok annotations to the `Customer` class if not already present:

```java
@Builder          // enables the builder pattern for creating Customer objects in tests
@EqualsAndHashCode // required for assertEquals() to compare Customer objects by value
public class Customer {
  // ...
}
```

### Mocking with Mockito

Create `CustomerServiceImplTest.java` in the corresponding test folder.

Convention: test files mirror the source folder structure.
- Source: `src/main/java/sg/edu/ntu/simplecrm/service/CustomerServiceImpl.java`
- Test: `src/test/java/sg/edu/ntu/simplecrm/service/CustomerServiceImplTest.java`

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

> **Instructor note — common mistake:** `@InjectMocks` must target the **concrete class** (`CustomerServiceImpl`), not the interface (`CustomerService`). Mockito creates an instance of the concrete class and injects the mocks — it cannot instantiate an interface. Students often write `CustomerService customerService` here and get a confusing error.

This means we can test the service layer without spinning up the Spring context or touching the database.

### Test Create Customer

```java
@Test
public void testCreateCustomer() {

  // 1. SETUP
  Customer customer = Customer.builder()
      .firstName("Clint").lastName("Barton")
      .email("clint@avengers.com").contactNo("12345678")
      .jobTitle("Special Agent").yearOfBirth(1975)
      .build();

  when(customerRepository.save(customer)).thenReturn(customer);

  // 2. EXECUTE
  Customer savedCustomer = customerService.createCustomer(customer);

  // 3. ASSERT
  assertEquals(customer, savedCustomer, "The saved customer should be the same as the new customer");
  verify(customerRepository, times(1)).save(customer);
}
```

- `when(...).thenReturn(...)` — tells Mockito what to return when a specific method is called
- `verify(...)` — confirms the mocked method was called the expected number of times

### Test Get Customer

```java
@Test
public void testGetCustomer() {
  // 1. SETUP
  Customer customer = Customer.builder()
      .firstName("Clint").lastName("Barton")
      .email("clint@avengers.com").contactNo("12345678")
      .jobTitle("Special Agent").yearOfBirth(1975)
      .build();

  Long customerId = 1L;
  when(customerRepository.findById(customerId)).thenReturn(Optional.of(customer));

  // 2. EXECUTE
  Customer retrievedCustomer = customerService.getCustomer(customerId);

  // 3. ASSERT
  assertEquals(customer, retrievedCustomer);
}
```

### Test Get Customer Not Found

```java
@Test
public void testGetCustomerNotFound() {
  Long customerId = 1L;
  when(customerRepository.findById(customerId)).thenReturn(Optional.empty());

  assertThrows(CustomerNotFoundException.class, () -> customerService.getCustomer(customerId));
}
```

---

## Part 5: Integration Testing with MockMvc

Unit tests validate individual components in isolation. Integration tests validate how components work together — the full request/response cycle from controller through service to repository.

Spring provides `MockMvc` to simulate HTTP requests without starting a real server.

### `@SpringBootTest` vs `@WebMvcTest`

Before writing integration tests, it's worth understanding the two main annotations:

| Annotation | What it loads | Speed | Use when |
|---|---|---|---|
| `@SpringBootTest` | Full application context — all beans, datasource, security, AI config, etc. | Slower | Testing the full stack end-to-end |
| `@WebMvcTest` | Web layer only — controllers, filters, `@ControllerAdvice`. No service/repo beans. | Faster | Testing controller logic in isolation with mocked services |

In production teams, `@WebMvcTest` is preferred for controller-layer tests because it is faster and more focused. `@SpringBootTest` is used for true end-to-end or database integration tests.

> **Instructor note:** A common production gotcha — `@SpringBootTest` loads **everything**, including Spring AI and OpenAI configuration. If your test environment (e.g. CI/CD pipeline) does not have the `OPENAI_API_KEY` set, the context will fail to load and **all integration tests will fail before a single test even runs**. The error looks like a configuration failure, not a test failure — which confuses developers. The fix is either to provide a dummy key via `@TestPropertySource` or to use `@WebMvcTest` which skips the AI beans entirely.

For this lesson we use `@SpringBootTest` to test the full stack. Make sure your `OPENAI_API_KEY` environment variable is set, or add this to your test class:

```java
@TestPropertySource(properties = "spring.ai.openai.api-key=test-key")
```

### Setting Up the Integration Test

Create `CustomerControllerTest.java` in the corresponding test folder.

```java
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

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
- `@Transactional` — each test runs inside a transaction that is **rolled back** after the test completes, keeping the database clean between tests
- `ObjectMapper` — used to convert Java objects to JSON (provided by Jackson)

> **Instructor note — why `@Transactional` matters:** Without it, every test that writes data to the database leaves that data behind. Tests start polluting each other — a record created in test 1 affects the count in test 2, or an ID auto-incremented in test 1 causes test 3 to fail because it expected a different ID. This is one of the most common causes of "tests pass individually but fail when run together." `@Transactional` on the test class rolls back every write after each test, giving each test a clean slate.

### Test Get Customer

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

**JsonPath** allows us to query the JSON response:

```java
jsonPath("$.id")        // returns the id field
jsonPath("$.firstName") // returns the firstName field
jsonPath("$.lastName")  // returns the lastName field
```

### Test Get All Customers

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

> **Instructor note — fragile assertions:** Hardcoding `4` here is a common source of brittle tests in real projects. If someone adds a customer to the DataLoader, this test breaks with no obvious reason why. A more resilient alternative is `jsonPath("$.size()").value(org.hamcrest.Matchers.greaterThan(0))` — asserting that results exist without depending on an exact count. Use this as a discussion point about test design trade-offs.

### Test Valid Customer Creation

> ⚠️ Note: With `@Transactional` added to the test class, each test rolls back after completion, so auto-incremented IDs will not accumulate across tests. However, the starting ID still depends on what the DataLoader seeded. If your DataLoader seeds 4 customers, the first new customer will get `id` 5. Update this value if your DataLoader changes.

```java
@Test
public void validCustomerCreationTest() throws Exception {
  // Step 1: Create a Customer object
  Customer newCustomer = Customer.builder()
      .firstName("Clint").lastName("Barton")
      .email("clint@avengers.com").contactNo("12345678")
      .jobTitle("Special Agent").yearOfBirth(1975)
      .build();

  // Step 2: Convert to JSON
  String newCustomerAsJSON = objectMapper.writeValueAsString(newCustomer);

  // Step 3: Build the request
  RequestBuilder request = MockMvcRequestBuilders.post("/customers")
      .contentType(MediaType.APPLICATION_JSON)
      .content(newCustomerAsJSON);

  // Step 4: Perform and assert
  mockMvc.perform(request)
      .andExpect(status().isCreated())
      .andExpect(content().contentType(MediaType.APPLICATION_JSON))
      .andExpect(jsonPath("$.id").exists())           // resilient: just verify an id was assigned
      .andExpect(jsonPath("$.firstName").value("Clint"))
      .andExpect(jsonPath("$.lastName").value("Barton"));
}
```

> **Instructor note:** We changed `.andExpect(jsonPath("$.id").value(5))` to `.andExpect(jsonPath("$.id").exists())`. Asserting the exact ID value is a brittle pattern — it breaks the moment the DataLoader or test execution order changes. In production test suites, assert that the field *exists* and has a valid value, not that it equals a specific number.

### Test Invalid Customer Creation

```java
@Test
public void invalidCustomerCreationTest() throws Exception {
  // Step 1: Create a Customer object with invalid fields
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

---

END
# Assignment (Optional)

## Brief

Create unit tests and integration tests for a Spring Boot project using JUnit, Mockito, and MockMvc.

1. **Unit Testing for Service Layer**
   - Use your existing project (e.g., simple-crm, EmployeeManagementSystem, or any previous project)
   - Create a test class for your Service implementation (e.g., `CustomerServiceImplTest` or `EmployeeServiceImplTest`)
   - Add necessary annotations: `@ExtendWith(MockitoExtension.class)`
   - Mock the repository using `@Mock`
   - Inject mocks into the service using `@InjectMocks`
   - Write unit tests for the following methods:
     - **Create method** - Test creating a new entity
       - Mock the repository's save() method
       - Assert the returned entity matches the input
       - Verify the save() method was called once
     - **Get by ID method** - Test retrieving an entity by ID
       - Mock the repository's findById() to return an Optional with the entity
       - Assert the returned entity is correct
     - **Get by ID Not Found** - Test retrieving a non-existent entity
       - Mock the repository's findById() to return Optional.empty()
       - Use assertThrows() to verify custom exception is thrown
   - Run tests with `mvn test` and ensure all tests pass

2. **Integration Testing for REST Controller**
   - Create a test class for your Controller (e.g., `CustomerControllerTest`)
   - Add annotations: `@SpringBootTest` and `@AutoConfigureMockMvc`
   - Autowire `MockMvc` and `ObjectMapper`
   - Write integration tests for the following endpoints:
     - **GET all entities** (e.g., GET /customers)
       - Build a GET request using MockMvcRequestBuilders
       - Assert status is 200 OK
       - Assert content type is JSON
       - Assert the size of the returned array
     - **POST create entity with valid data**
       - Create a valid entity object
       - Convert it to JSON using ObjectMapper
       - Build a POST request with the JSON content
       - Assert status is 201 Created
       - Assert the returned JSON contains expected values
     - **POST create entity with invalid data**
       - Create an entity with blank/invalid required fields
       - Convert to JSON and send POST request
       - Assert status is 400 Bad Request
   - Run tests and verify all integration tests pass
   - Generate HTML test report using `mvn surefire-report:report`


## References
- Java: https://docs.oracle.com/javase/
- Spring Boot: https://docs.spring.io/spring-boot/docs/current/reference/html/
- PostgreSQL: https://www.postgresql.org/docs/
- OWASP: https://cheatsheetseries.owasp.org/
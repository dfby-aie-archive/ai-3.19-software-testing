# Self Studies: Software Testing: Unit and Integration Testing

## Overview

This lesson introduces automated testing in Java — one of the most important skills for building production-ready applications. The self-study materials below will help you arrive with a clear understanding of JUnit, Mockito, and MockMvc so you can follow the code-along confidently. Pay particular attention to how mocking works — it is the most conceptually new idea in this lesson.

**Estimated Prep Time:** 60–80 minutes

---

## Task 1: Spring Boot Unit Testing and Integration Testing with JUnit and Mockito

This video series covers everything in the lesson — JUnit test structure, Mockito mocking, and MockMvc integration testing — all within a Spring Boot application. Watch the first video to get started and work through the rest of the series at your own pace.

**Watch:** Spring Boot Unit Testing and Integration Testing with JUnit and Mockito
🎬 https://www.youtube.com/watch?v=jqwZthuBmZY&list=PL82C6-O4XrHcg8sNwpoDDhcxUCbFy855E

> 📌 The link above opens the first video in the playlist. **Watch the entire series** for full coverage of all topics in this lesson.

**Then read:** Lesson 3.19 — All Parts

**Guiding Questions:**
- What are the 3 steps of the Arrange-Act-Assert pattern?
- What is the difference between `@Mock` and `@InjectMocks`?
- What does `when().thenReturn()` do in Mockito?
- What is the difference between unit testing and integration testing?
- What does `@SpringBootTest` load that a plain `@ExtendWith(MockitoExtension.class)` test does not?

---

## Active Engagement Strategies

- As you watch the videos, pause after each test method and try to write it yourself from memory before continuing
- After watching, try to identify which parts of `simple-crm` (controller, service, repository) would be tested with unit tests vs integration tests — and why
- Before class, add `@Builder` and `@EqualsAndHashCode` to your `Customer` class — these are required for the service layer tests to work

---

## Additional Reading Material

- [JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/)
- [Mockito Documentation](https://site.mockito.org/)
- [Testing in Spring Boot — Baeldung](https://www.baeldung.com/spring-boot-testing)
- [Arrange-Act-Assert Pattern](https://automationpanda.com/2020/07/07/arrange-act-assert-a-pattern-for-writing-good-tests/)
- [MockMvc Guide — Baeldung](https://www.baeldung.com/integration-testing-in-spring)
---
mode: 'agent'
tools: [ 'changes', 'codebase', 'editFiles', 'problems', 'search' ]
description: 'JUnit 5 unit testing best practices'
applyTo: "**/*Test.java, **/*Test.kt"
---

# JUnit 5 Standards

## Project Setup

- Java tests: `src/test/java`
- Kotlin tests: `src/test/kotlin`
- Run: `bazel test`

## Test Structure

- Class naming: `CalculatorTest` for `Calculator` class
- Method naming:
    - **Kotlin**: Use one of:
        - Backtick strings for readable names
            - `` `createSubscriptionService when creating a new subscription` ``
            - `` `it should return apn when apn with given name exists` ``
        - `testMethodNameShouldExpectedBehaviorWhenScenario`
    - **Java**: `testMethodNameShouldExpectedBehaviorWhenScenario`
- Use Arrange-Act-Assert pattern
- Annotations: `@Test`, `@BeforeEach`, `@AfterEach`, `@BeforeAll`, `@AfterAll`, `@DisplayName`

## Standard Tests

- One behavior per test
- Independent and idempotent

## Parameterized Tests

- `@ParameterizedTest` with:
- `@ValueSource` - literals
- `@MethodSource` - factory method returning Stream/Collection
- `@CsvSource` - inline CSV
- `@CsvFileSource` - CSV file
- `@EnumSource` - enum constants

## Assertions

- **Kotlin**: Use either
    - Kotest infix assertions
        - `result shouldBe expected`
        - `result shouldNotBe null`
        - `list shouldContain element`
    - Or assertj assertions (preferred)
        - `assertThat(result).isEqualTo(expected)`
        - `assertThat(result).isNotNull`
        - `assertThat(list).contains(element)`
- **Java**: JUnit `assertEquals`, `assertTrue`, `assertNotNull`
- `assertAll` - group related assertions
- `assertThrows`, `assertDoesNotThrow` - exception testing

## Mocking

- **Java**: Mockito with `@Mock`, `@InjectMocks`
- **Kotlin**: Mockk

## Organization

- `@Tag` - categorize tests (`"fast"`, `"integration"`, `"intenvIntegration"`)
- `@Nested` - group related tests
- `@Disabled("reason")` - skip tests
- `@Order` with `@TestMethodOrder` - control execution order when necessary

## Bazel Test Organization

- Separate test libraries from test targets:
    - `kt_jvm_library(name = "foo_tests", testonly = 1)` - test code
    - `java_junit5_tests(name = "small_tests")` - test execution
- Use `glob2pkg()` to convert file patterns to test classes
- Split by size: `small_tests` (unit), `large_tests` (integration)
- Exclude patterns: `*IntegrationTest.*`, `*DaoTest.*`, `*ManualTest.*`
- Integration test tags: `["cpu:N", "no-remote-exec"]`
- Integration test data: `data = ["intenv.yml"]`

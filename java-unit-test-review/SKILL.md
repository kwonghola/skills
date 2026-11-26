name: java-unit-test-review
description: Review existing Java unit tests (JUnit 5/6, Mockito, AssertJ) for unused preconditions and dead fixtures, mock data realism, mocking hygiene, and assertion quality. Use when the user asks to review Java unit tests, audit a test diff or PR, find unused fixtures or redundant setup/preconditions, check whether mocks or test fixtures are realistic, spot over-mocked tests or weak/tautological assertions, or verify boundary coverage without rewriting code automatically.
Java Unit Test Review
Audit existing Java unit tests for unused preconditions and dead fixtures, mocking discipline, realistic mock data, and meaningful assertions. This skill is advisory by default: report concrete findings and proposed fixes without editing test files unless the user explicitly asks to apply changes.
When to Use
￼ Reviewing Java unit test files or pull request diffs (JUnit 5/6, Mockito, AssertJ)
￼ Detecting unused preconditions, dead fixtures, bloated ⁠@BeforeEach⁠ setup, and redundant stubs
￼ Auditing mock data quality, test fixtures, and stubbed payloads
￼ Checking whether tests suffer from over-mocking or weak, tautological assertions
￼ Verifying boundary cases, error paths, and determinism in existing test suites
Steps
1. Adapt to Existing Project Conventions
Before flagging style issues, inspect the project's existing test setup (⁠pom.xml⁠, ⁠build.gradle⁠, ⁠CONTRIBUTING.md⁠, or sibling ⁠*Test.java⁠ files) to detect the JUnit version, assertion library (AssertJ vs JUnit Jupiter), naming style, and fixture builders. Follow documented team conventions over generic defaults. If no project context is provided, state that you are applying standard Java testing practices.
2. Audit Unused Preconditions and Dead Fixtures
Trace every setup statement, fixture, and precondition against what the test actually executes and asserts:
￼ Shared fixture bloat (⁠@BeforeEach⁠ / ⁠@BeforeAll⁠): Flag fields, test objects, or stubs initialized in lifecycle hooks that are unused by some or all test methods (General Fixture anti-pattern). Recommend moving scenario-specific setup into the test or focused helper that actually needs it.
￼ Unused input fields and builder properties: Compare populated fields on input DTOs, entities, or builders against the production code path exercised by that test. Flag fields that are never read or cannot influence the asserted outcome (excluding fields strictly required to satisfy constructor invariants or validation).
￼ Unused mocks, spies, and test resources: Flag ⁠@Mock⁠, ⁠@Spy⁠, ⁠@Captor⁠, ⁠@TempDir⁠ files, test constants, or local setup variables declared or initialized without being referenced or interacted with.
￼ Unexercised stubs and redundant preconditions: Flag ⁠when(...).thenReturn(...)⁠, ⁠given(...).willReturn(...)⁠, ⁠doReturn(...)⁠, or ⁠lenient()⁠ stubs whose branch is never reached, plus unnecessary ⁠Assumptions.assumeTrue(...)⁠ or guard preconditions that do not affect the scenario.
￼ Trace before flagging: When the production class under test is available, trace the executed branch to confirm a field or precondition is genuinely unused. If production code is not provided, label field-level findings as Needs verification against the class under test.
3. Audit Mock Data Realism and Fixture Quality
Inspect all test inputs, fixtures, and stubbed return values (⁠when(...).thenReturn(...)⁠, ⁠given(...).willReturn(...)⁠):
￼ Placeholder and magic values: Flag meaningless literals (⁠"test"⁠, ⁠"foo"⁠, ⁠"abc"⁠, ⁠123⁠, magic HTTP status codes) when domain-representative values or named constants clarify business intent and catch validation bugs.
￼ Mirror-stubbing (circular tests): Flag stubs whose return data is copied verbatim into the assertion without the class under test transforming, filtering, or validating it. Such tests only prove Mockito returned what it was told to return.
￼ Unrealistic or incomplete fixtures: Flag mocked entities missing required fields, invariants, or edge-case shapes (empty lists, null optionals, boundary decimals, timezone-aware timestamps) that real collaborators produce.
￼ Noisy inline setup: Flag repetitive multi-line object construction across tests that obscures which fields actually matter for the scenario. Recommend test data builders, factory methods, or Java records with clear defaults and explicit per-test overrides.
4. Audit Mocking Scope and Mockito Hygiene
Verify that mocks isolate boundaries rather than replacing the logic under test:
￼ Boundary-only mocking: Mock collaborators across architectural boundaries (repositories, HTTP clients, ⁠Clock⁠, message senders). Flag mocks of value objects, DTOs, records, collections, ⁠String⁠, or the class under test itself.
￼ Over-mocking: When a test mocks multiple collaborators, list them and check what real production code path remains. If stubbing eliminates the core logic, recommend using real domain instances, a lightweight hand-written fake for stateful interactions, or moving the test to a slice/integration test.
￼ Strict stubbing and unused stubs: Prefer ⁠@ExtendWith(MockitoExtension.class)⁠ with default strictness. Flag unnecessary stubs or unjustified ⁠Strictness.LENIENT⁠ overrides that hide dead test setup.
￼ Fragile mocking APIs: Flag ⁠mockStatic⁠, ⁠mockConstruction⁠, and deep stubbing (⁠RETURNS_DEEP_STUBS⁠) as design smells; suggest injecting a thin collaborator interface instead.
￼ Overly broad matchers: Flag ⁠any()⁠ or ⁠anyString()⁠ in stubs or verifications when the specific argument value is critical to correctness; suggest exact matchers or ⁠@Captor⁠ / ⁠ArgumentCaptor⁠ with value assertions.
5. Audit Assertion Strength (Mental Mutation Test)
For every test method, run a mental mutation test: if the key business condition, calculation, or filter in the production code were inverted or broken, would this test fail?
￼ Missing or tautological assertions: Flag tests with no assertions, assertions that can never fail given the setup, or ⁠try/catch⁠ blocks that swallow ⁠AssertionError⁠ or exceptions.
￼ Weak presence checks: Flag bare ⁠assertNotNull⁠, ⁠isPresent()⁠, or ⁠hasSize(1)⁠ when the actual field values, ordering, or mapping should be verified. Prefer AssertJ fluent assertions (⁠containsExactly⁠, ⁠extracting⁠, ⁠usingRecursiveComparison⁠, ⁠assertThatThrownBy⁠) or JUnit Jupiter ⁠assertAll⁠ and ⁠assertThrowsExactly⁠.
￼ Testing implementation instead of behavior: Flag ⁠verify(mock, times(n))⁠ checks on internal helper calls or read-only queries when asserting on the returned result or observable state would be stronger and less brittle. Reserve ⁠verify⁠ for genuine side effects (saving, publishing, sending) and verify the captured payload, not just invocation count.
￼ Loose exception checks: Flag ⁠assertThrows(Exception.class, ...)⁠ or checks that ignore exception type, error code, or message content.
￼ Crammed unrelated assertions: Flag single test methods verifying multiple independent behaviors where a failure hides which requirement broke.
6. Check Boundary Coverage, Determinism, and Structure
￼ Boundaries (RIGHT-BICEP / CORRECT): Note missing high-value cases such as null inputs, empty collections, zero/negative/max values, duplicate keys, and collaborator failures.
￼ Determinism: Flag hidden dependencies on ⁠Instant.now()⁠, ⁠LocalDate.now()⁠, unseeded ⁠Random⁠, default locale/timezone, ⁠Thread.sleep⁠, or shared mutable static fields across tests.
￼ Control flow in tests: Flag ⁠if⁠, ⁠switch⁠, or loops inside test bodies; recommend explicit assertions or ⁠@ParameterizedTest⁠ (⁠@ValueSource⁠, ⁠@CsvSource⁠, ⁠@MethodSource⁠).
￼ Clarity: Check for clear Given-When-Then separation and scenario-outcome test names.
7. Format the Review Output
Group findings by file. Skip categories that have no issues. For each finding, report:
￼ Severity: ⁠High⁠ (false confidence / misses bugs), ⁠Medium⁠ (brittle / weak coverage / misleading dead setup), or ⁠Low⁠ (readability / maintainability)
￼ Location: File name, test method name, and line number (if available)
￼ Category: ⁠Unused Preconditions & Fixtures⁠, ⁠Mock Data⁠, ⁠Mocking Hygiene⁠, ⁠Assertion Quality⁠, ⁠Boundaries & Determinism⁠, or ⁠Structure⁠
￼ Problem: Concrete explanation of why the test is weak, misleading, or redundant
￼ Suggested Fix: Specific rewrite or short Java snippet
End with a brief summary of the top 1 to 3 systemic patterns to address across the test file. Do not edit files unless the user explicitly confirms they want the fixes applied.
Gotchas
￼ Never invent a missing business requirement to criticize a test; if the spec or production code is not provided, state what cannot be verified and focus on issues visible in the test code itself.
￼ Do not flag fixture fields or files as definitely unused if they may be required by reflection, serialization, Bean Validation, or framework-loaded test resources without verifying the production code first.
￼ Do not recommend splitting a test solely because it is long; split only when it verifies unrelated behaviors that should fail independently.
￼ Do not suggest adding AssertJ or other libraries if the project only uses plain JUnit Jupiter; match the project's existing assertion library first.
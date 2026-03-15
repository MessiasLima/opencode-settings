---
name: test-fixtures
description: Use this skill when writing or updating Kotlin unit tests, especially when creating sample data objects or mocks. It enforces the use of the `fixture()` extension method pattern to make tests succinct and reduce code duplication. Make sure to use this whenever instantiating domain models, entities, or DTOs in test files.
---

# Kotlin Test Fixtures Pattern

When writing or updating unit tests in Kotlin, NEVER instantiate data models, entities, or DTOs directly within the test body (e.g., `val user = User("id", "name")`). Instead, use the `fixture()` pattern.

## The Pattern

The fixture pattern involves creating an extension function on the class's `companion object` that provides default values for all properties. This allows tests to only specify the properties they care about for a specific test case, reducing boilerplate and test brittleness.

### 1. The Model
Ensure the target class has a `companion object`.
```kotlin
data class Session(
    val accessToken: String,
    val refreshToken: String?,
    // ...
) {
    companion object // Required for the extension function
}
```

### 2. The Fixture Definition
Create a file named `[ClassName]Fixture.kt` in the appropriate test source set (e.g., `src/commonTest/...` or `src/test/...`) using the exact same package structure as the model.

```kotlin
package dev.appoutlet.foliary.data.authentication.model

fun Session.Companion.fixture(
    accessToken: String = "default_access_token",
    refreshToken: String? = "default_refresh_token", // Note: type matches exactly, but default is NOT null
    // ...
) = Session(
    accessToken = accessToken,
    refreshToken = refreshToken,
)
```

### 3. Usage in Tests
Use the fixture in your tests, overriding only the values that matter for the specific test scenario using named arguments.

```kotlin
@Test
fun `should process active session`() = runTest {
    // Only override what matters for this test
    val session = Session.fixture(accessToken = "specific_token_for_test")

    dao.insert(session)
    // ...
}
```

## Fixture Rules

1. **Complete Parameter Mapping:** All constructor parameters of the model MUST have a corresponding method argument in the `fixture()` function.
2. **Exact Types:** The method arguments must respect the *exact* type of the constructor argument (e.g., if the constructor takes a `String?`, the fixture argument must be `String?`).
3. **Strict Default Values:** Default values of the function parameters should *always* be provided.
   - **Do NOT set `null`** as a default argument, even though the type might be nullable. Always provide a valid, sensible default value.
   - **Do NOT use empty lists/collections** as default arguments. Collection arguments should always have at least one item.
   - *Why?* This makes testing predictable. Tests that need `null` or empty lists must explicitly ask for them.
4. **Nested Models:** When the argument type is another domain model or custom object, create the corresponding fixture method for this new model and use its fixture as the default value (e.g., `user: User = User.fixture()`).
5. **External or Generated Classes:** When the sample object is from an external library or a generated class, and we cannot add a `companion object` to it, the fixture function should be a regular function named `fun [className]Fixture()` (e.g., `fun userResponseFixture()`). This function should be created alongside the test class that uses it. All other fixture rules (defaults, exact types, etc.) still apply in this situation.
6. **Check First:** Always check for existing fixtures before creating new ones. Search for `[ClassName]Fixture.kt` or `fun [ClassName].Companion.fixture`.
7. **Companion Object:** If the model class lacks a `companion object` (and it's a class we own), add an empty `companion object` to the model class first so the extension function can be attached.
8. **Named Arguments:** When calling `fixture()` with overrides in a test, always use named arguments to make it clear which fields are being customized.
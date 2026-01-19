# Kotlin Coding Standards

## File Organization

- One top-level class per file (with related helper classes if needed)
- Keep files focused and under 1000 lines when possible
- Order: package, imports, top-level declarations, classes

```kotlin
package com.example.myapp.feature

import android.content.Context
import androidx.lifecycle.ViewModel

private const val MAX_RETRY_COUNT = 3

class FeatureViewModel : ViewModel() { ... }
```

## Naming Conventions

### Classes and Objects
- `PascalCase` for class names
- Use descriptive names that explain purpose
- Suffix with type descriptor when helpful: `UserRepository`, `LoginViewModel`

```kotlin
class UserProfileActivity : AppCompatActivity()
object NetworkConfig
sealed class Result<out T>
data class User(val id: String, val name: String)
```

### Functions and Properties
- `camelCase` for function and property names
- Use descriptive, verb-based names for functions
- Boolean properties should read naturally: `isVisible`, `hasData`, `canRetry`

```kotlin
fun fetchUserData() { ... }
fun isValidEmail(email: String): Boolean { ... }

val userName: String
var isLoading: Boolean = false
```

### Constants
- `UPPER_SNAKE_CASE` for compile-time constants
- Use `const val` for primitive types and strings
- Use `val` for object instances

```kotlin
const val MAX_RETRY_COUNT = 3
const val API_BASE_URL = "https://api.example.com"

val DEFAULT_TIMEOUT = Duration.ofSeconds(30)
```

### Packages
- Use lowercase, no underscores
- Use reverse domain notation: `com.example.myapp`
- Group by feature, not layer: `com.example.myapp.user` (not `.ui`, `.data`)

## Kotlin Idioms

### Null Safety
- Leverage Kotlin's null safety system
- Use `?` for nullable types
- Use `?.` for safe calls, `?:` for Elvis operator
- Avoid `!!` unless absolutely certain non-null

```kotlin
// Good
val length = name?.length ?: 0
val user = repository.getUser() ?: return

// Avoid
val length = name!!.length  // Only if you're 100% certain
```

### Data Classes
- Use data classes for simple data holders
- Let compiler generate `equals()`, `hashCode()`, `toString()`, `copy()`

```kotlin
data class User(
    val id: String,
    val name: String,
    val email: String?
)

// Usage
val updatedUser = user.copy(name = "New Name")
```

### When Expression
- Prefer `when` over long if-else chains
- Use sealed classes for exhaustive when

```kotlin
when (result) {
    is Success -> handleSuccess(result.data)
    is Error -> handleError(result.exception)
    is Loading -> showLoading()
}
```

### Extension Functions
- Use extension functions to add utility methods
- Keep extensions focused and discoverable

```kotlin
fun String.isValidEmail(): Boolean {
    return android.util.Patterns.EMAIL_ADDRESS.matcher(this).matches()
}

fun Context.showToast(message: String) {
    Toast.makeText(this, message, Toast.LENGTH_SHORT).show()
}
```

### Scope Functions
- `let`: Transform nullable values, execute code block
- `run`: Execute block and return result
- `with`: Call multiple methods on same object
- `apply`: Configure object properties
- `also`: Perform side effects

```kotlin
// let - transform nullable
val userName = user?.let { it.firstName + " " + it.lastName }

// apply - configure object
val intent = Intent(this, TargetActivity::class.java).apply {
    putExtra("key", value)
    flags = Intent.FLAG_ACTIVITY_NEW_TASK
}

// also - side effects (logging, etc.)
val user = createUser().also {
    log("User created: ${it.id}")
}
```

### Collections
- Use Kotlin collection APIs (map, filter, fold, etc.)
- Prefer immutable collections by default
- Use sequences for large data or chained operations

```kotlin
val activeUsers = users
    .filter { it.isActive }
    .sortedBy { it.name }
    .take(10)

// Use sequence for performance with large datasets
val result = largeList.asSequence()
    .filter { it.isValid() }
    .map { it.transform() }
    .toList()
```

## Properties and Backing Fields

### Custom Getters/Setters
```kotlin
class Rectangle(val width: Int, val height: Int) {
    val area: Int
        get() = width * height

    var isSquare: Boolean
        get() = width == height
        set(value) {
            if (value) {
                height = width
            }
        }
}
```

### Backing Fields
```kotlin
var counter: Int = 0
    set(value) {
        if (value >= 0) {
            field = value
        }
    }
```

### Late Initialization
```kotlin
// For non-null properties that can't be initialized in constructor
lateinit var viewModel: MyViewModel

// For lazy initialization
val database: Database by lazy {
    DatabaseBuilder.build(context)
}
```

## Functions

### Default Parameters
```kotlin
fun connect(
    host: String,
    port: Int = 8080,
    timeout: Duration = Duration.ofSeconds(30)
) { ... }

// Usage
connect("example.com")
connect("example.com", port = 443)
```

### Named Arguments
```kotlin
// Good - improves readability
createUser(
    name = "John Doe",
    email = "john@example.com",
    age = 30
)
```

### Single Expression Functions
```kotlin
fun double(x: Int): Int = x * 2

fun isValidUser(user: User): Boolean =
    user.name.isNotBlank() && user.email.isValidEmail()
```

### Extension Functions
```kotlin
fun List<Int>.average(): Double {
    if (isEmpty()) return 0.0
    return sum().toDouble() / size
}
```

## Classes and Inheritance

### Primary Constructor
```kotlin
class User(
    val id: String,
    val name: String,
    private val email: String
) {
    init {
        require(name.isNotBlank()) { "Name cannot be blank" }
    }
}
```

### Secondary Constructors
```kotlin
class User(val id: String, val name: String) {
    constructor(id: String) : this(id, "Unknown")
}
```

### Sealed Classes
Use for restricted hierarchies:

```kotlin
sealed class Result<out T> {
    data class Success<T>(val data: T) : Result<T>()
    data class Error(val exception: Throwable) : Result<Nothing>()
    object Loading : Result<Nothing>()
}
```

### Object Declarations (Singletons)
```kotlin
object NetworkConfig {
    const val BASE_URL = "https://api.example.com"
    const val TIMEOUT_SECONDS = 30
}
```

### Companion Objects
```kotlin
class User(val id: String, val name: String) {
    companion object {
        const val DEFAULT_ROLE = "user"

        fun create(id: String, name: String): User {
            return User(id, name)
        }
    }
}
```

## Coroutines

### Launch vs Async
```kotlin
// launch - fire and forget
viewModelScope.launch {
    repository.saveData(data)
}

// async - return result
val deferred = viewModelScope.async {
    repository.fetchData()
}
val result = deferred.await()
```

### Structured Concurrency
```kotlin
suspend fun loadUserProfile(): UserProfile = coroutineScope {
    val user = async { userRepository.getUser() }
    val posts = async { postRepository.getPosts() }
    val friends = async { friendRepository.getFriends() }

    UserProfile(
        user = user.await(),
        posts = posts.await(),
        friends = friends.await()
    )
}
```

### Error Handling
```kotlin
viewModelScope.launch {
    try {
        val result = repository.fetchData()
        _uiState.value = Success(result)
    } catch (e: Exception) {
        _uiState.value = Error(e)
    }
}
```

## Type Aliases
Use for complex types:

```kotlin
typealias UserId = String
typealias Callback = (Result<User>) -> Unit
typealias UserMap = Map<UserId, User>
```

## Documentation

### KDoc
```kotlin
/**
 * Fetches user data from the remote server.
 *
 * @param userId The unique identifier for the user
 * @param forceRefresh Whether to bypass cache
 * @return User object or null if not found
 * @throws NetworkException if network request fails
 */
suspend fun fetchUser(userId: String, forceRefresh: Boolean = false): User?
```

## Code Style

### Indentation and Spacing
- Use 4 spaces for indentation
- Maximum line length: 100-120 characters
- Use blank lines to separate logical blocks

### Formatting
```kotlin
// Good
class User(
    val id: String,
    val name: String,
    val email: String
)

// Method chaining
val result = data
    .filter { it.isValid }
    .map { it.transform() }
    .sortedBy { it.priority }
```

## Android-Specific Best Practices

### ViewBinding
```kotlin
class MainActivity : AppCompatActivity() {
    private lateinit var binding: ActivityMainBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)

        binding.button.setOnClickListener {
            // Handle click
        }
    }
}
```

### ViewModel
```kotlin
class UserViewModel(
    private val repository: UserRepository
) : ViewModel() {

    private val _uiState = MutableStateFlow<UiState>(UiState.Loading)
    val uiState: StateFlow<UiState> = _uiState.asStateFlow()

    fun loadUser(userId: String) {
        viewModelScope.launch {
            _uiState.value = UiState.Loading
            try {
                val user = repository.getUser(userId)
                _uiState.value = UiState.Success(user)
            } catch (e: Exception) {
                _uiState.value = UiState.Error(e.message ?: "Unknown error")
            }
        }
    }
}
```

### Dependency Injection (Hilt/Koin)
```kotlin
@HiltViewModel
class UserViewModel @Inject constructor(
    private val repository: UserRepository,
    private val savedStateHandle: SavedStateHandle
) : ViewModel()
```

## Testing

### Unit Tests
```kotlin
class UserViewModelTest {
    @Test
    fun `loadUser updates state to Success when repository returns user`() = runTest {
        // Given
        val mockRepository = mockk<UserRepository>()
        coEvery { mockRepository.getUser(any()) } returns testUser
        val viewModel = UserViewModel(mockRepository)

        // When
        viewModel.loadUser("user-123")

        // Then
        assertEquals(UiState.Success(testUser), viewModel.uiState.value)
    }
}
```

## Common Anti-Patterns to Avoid

- **Using `!!` excessively**: Defeats null safety
- **Ignoring scope functions**: Makes code less idiomatic
- **Mutable collections everywhere**: Prefer immutable by default
- **Not using data classes**: Manually writing equals/hashCode
- **Blocking main thread**: Use coroutines for async work
- **Memory leaks in coroutines**: Use structured concurrency
- **Ignoring lifecycle**: In Android, respect Activity/Fragment lifecycle

## Performance Tips

- Use `sequence` for large collections with multiple operations
- Use `inline` functions for higher-order functions
- Prefer `object` for stateless utilities (avoids allocation)
- Use `const val` for compile-time constants
- Be cautious with reflection (slow on Android)

## Security

- Never hardcode secrets or API keys
- Validate all user inputs
- Use encrypted SharedPreferences for sensitive data
- Sanitize data before display (prevent injection)
- Keep dependencies updated

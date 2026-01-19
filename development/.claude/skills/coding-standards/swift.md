# Swift Coding Standards

## File Organization

- One primary type per file (with related helper types if needed)
- Keep files focused and under 1000 lines when possible
- Order: imports, type definitions, extensions

```swift
import Foundation
import UIKit

class UserProfileViewController: UIViewController {
    // Implementation
}

// MARK: - UITableViewDelegate
extension UserProfileViewController: UITableViewDelegate {
    // Delegate methods
}
```

## Naming Conventions

### Types
- `PascalCase` for types (classes, structs, enums, protocols)
- Use descriptive names that explain purpose
- Protocol names should describe behavior or capability

```swift
class UserProfileViewController: UIViewController { }
struct UserCredentials { }
enum NetworkError: Error { }
protocol DataSourceProvider { }
```

### Functions and Variables
- `camelCase` for functions, variables, and properties
- Use descriptive names that explain intent
- Boolean properties should read naturally: `isVisible`, `hasData`, `canRetry`

```swift
func fetchUserData() { }
func isValidEmail(_ email: String) -> Bool { }

var userName: String
var isLoading = false
let maxRetryCount = 3
```

### Constants
- Use `camelCase` for constants
- Group related constants in enums or structs

```swift
// Good - namespaced constants
enum Constants {
    static let maxRetryCount = 3
    static let apiBaseURL = "https://api.example.com"
    static let defaultTimeout: TimeInterval = 30
}

// Or using struct
struct NetworkConfig {
    static let timeout: TimeInterval = 30
    static let maxRetries = 3
}
```

### Enumerations
- Use `camelCase` for enum cases
- Leverage associated values for contextual data

```swift
enum Result<T> {
    case success(T)
    case failure(Error)
}

enum NetworkError: Error {
    case invalidURL
    case noConnection
    case serverError(statusCode: Int)
}
```

## Swift Idioms

### Optionals
- Use optionals to represent absence of value
- Prefer optional binding over force unwrapping
- Use optional chaining and nil coalescing

```swift
// Good - optional binding
if let user = optionalUser {
    print(user.name)
}

// Good - guard for early exit
guard let user = optionalUser else {
    return
}

// Good - optional chaining and nil coalescing
let userName = user?.name ?? "Unknown"

// Avoid - force unwrapping (use only when guaranteed non-nil)
let name = user!.name  // Only if you're 100% certain
```

### Value Types (Struct) vs Reference Types (Class)
- **Prefer structs** for simple data models
- Use classes when you need:
  - Inheritance
  - Reference semantics
  - Deinitializers

```swift
// Good - struct for simple data
struct User {
    let id: String
    let name: String
    let email: String
}

// Good - class when reference semantics needed
class UserSession {
    var currentUser: User?
    var isAuthenticated = false
}
```

### Protocol-Oriented Programming
- Favor composition over inheritance
- Use protocol extensions for default implementations

```swift
protocol Identifiable {
    var id: String { get }
}

protocol Nameable {
    var name: String { get }
}

struct User: Identifiable, Nameable {
    let id: String
    let name: String
}

// Protocol extension for default behavior
extension Identifiable {
    func isSame(as other: Self) -> Bool where Self: Equatable {
        return self.id == other.id
    }
}
```

### Enums with Associated Values
```swift
enum LoadingState<T> {
    case idle
    case loading
    case loaded(T)
    case failed(Error)
}

// Usage with switch
switch state {
case .idle:
    showIdleState()
case .loading:
    showLoadingIndicator()
case .loaded(let data):
    displayData(data)
case .failed(let error):
    showError(error)
}
```

### Property Wrappers
```swift
@propertyWrapper
struct Clamped<Value: Comparable> {
    private var value: Value
    private let range: ClosedRange<Value>

    var wrappedValue: Value {
        get { value }
        set { value = min(max(range.lowerBound, newValue), range.upperBound) }
    }

    init(wrappedValue: Value, _ range: ClosedRange<Value>) {
        self.range = range
        self.value = min(max(range.lowerBound, wrappedValue), range.upperBound)
    }
}

// Usage
struct Game {
    @Clamped(0...100) var health = 100
}
```

## Properties and Methods

### Computed Properties
```swift
struct Rectangle {
    var width: Double
    var height: Double

    var area: Double {
        width * height
    }

    var perimeter: Double {
        2 * (width + height)
    }
}
```

### Property Observers
```swift
class ViewController: UIViewController {
    var isLoading = false {
        didSet {
            updateLoadingIndicator()
        }
    }

    var data: [Item] = [] {
        willSet {
            print("About to set \(newValue.count) items")
        }
        didSet {
            tableView.reloadData()
        }
    }
}
```

### Lazy Properties
```swift
class DataManager {
    lazy var database: Database = {
        let db = Database()
        db.configure()
        return db
    }()
}
```

### Type Properties and Methods
```swift
struct Math {
    static let pi = 3.14159

    static func square(_ value: Double) -> Double {
        value * value
    }
}
```

## Functions

### Default Parameters
```swift
func connect(
    host: String,
    port: Int = 8080,
    timeout: TimeInterval = 30,
    useTLS: Bool = true
) {
    // Implementation
}

// Usage
connect(host: "example.com")
connect(host: "example.com", port: 443)
```

### Multiple Return Values
```swift
func parseUser(_ data: Data) -> (user: User?, error: Error?) {
    // Implementation
    return (user, nil)
}

// Better - use Result type
func parseUser(_ data: Data) -> Result<User, Error> {
    // Implementation
    return .success(user)
}
```

### Throwing Functions
```swift
enum ValidationError: Error {
    case invalidEmail
    case passwordTooShort
}

func validateCredentials(email: String, password: String) throws {
    guard email.contains("@") else {
        throw ValidationError.invalidEmail
    }

    guard password.count >= 8 else {
        throw ValidationError.passwordTooShort
    }
}

// Usage
do {
    try validateCredentials(email: email, password: password)
} catch ValidationError.invalidEmail {
    showError("Invalid email")
} catch ValidationError.passwordTooShort {
    showError("Password too short")
} catch {
    showError("Unknown error")
}
```

## Error Handling

### Result Type
```swift
func fetchUser(id: String) -> Result<User, NetworkError> {
    // Implementation
}

// Usage
switch fetchUser(id: "123") {
case .success(let user):
    print("Got user: \(user.name)")
case .failure(let error):
    print("Error: \(error)")
}

// Or with if-case
if case .success(let user) = fetchUser(id: "123") {
    print("Got user: \(user.name)")
}
```

### Custom Error Types
```swift
enum NetworkError: Error {
    case invalidURL
    case noConnection
    case unauthorized
    case serverError(statusCode: Int, message: String?)
}

extension NetworkError: LocalizedError {
    var errorDescription: String? {
        switch self {
        case .invalidURL:
            return "The URL is invalid"
        case .noConnection:
            return "No internet connection"
        case .unauthorized:
            return "Unauthorized access"
        case .serverError(let code, let message):
            return message ?? "Server error with code \(code)"
        }
    }
}
```

## Collections

### Arrays
```swift
var numbers = [1, 2, 3, 4, 5]

// Functional operations
let doubled = numbers.map { $0 * 2 }
let evens = numbers.filter { $0 % 2 == 0 }
let sum = numbers.reduce(0, +)

// Safe subscripting
let first = numbers.first  // Optional
let element = numbers[safe: 10]  // Custom extension
```

### Dictionaries
```swift
var userScores: [String: Int] = [:]

// Safe access
let score = userScores["user123"] ?? 0

// Update
userScores["user123", default: 0] += 10
```

### Sets
```swift
let uniqueIDs: Set<String> = ["id1", "id2", "id3"]

// Set operations
let combined = set1.union(set2)
let common = set1.intersection(set2)
let different = set1.symmetricDifference(set2)
```

## Closures

### Trailing Closure Syntax
```swift
// Good
UIView.animate(withDuration: 0.3) {
    view.alpha = 0
}

// Multiple trailing closures (Swift 5.3+)
UIView.animate(withDuration: 0.3) {
    view.alpha = 0
} completion: { finished in
    view.removeFromSuperview()
}
```

### Capturing Values
```swift
class ViewController: UIViewController {
    func setupHandler() {
        // Avoid retain cycle with weak self
        service.fetchData { [weak self] result in
            guard let self = self else { return }
            self.handleResult(result)
        }
    }
}
```

### Escaping Closures
```swift
func performAsync(completion: @escaping (Result<Data, Error>) -> Void) {
    DispatchQueue.global().async {
        // Perform work
        completion(.success(data))
    }
}
```

## Concurrency (Swift Concurrency / async-await)

### Async Functions
```swift
func fetchUser(id: String) async throws -> User {
    let url = URL(string: "https://api.example.com/users/\(id)")!
    let (data, _) = try await URLSession.shared.data(from: url)
    return try JSONDecoder().decode(User.self, from: data)
}

// Usage
Task {
    do {
        let user = try await fetchUser(id: "123")
        print("Got user: \(user.name)")
    } catch {
        print("Error: \(error)")
    }
}
```

### Structured Concurrency
```swift
func loadUserProfile() async throws -> UserProfile {
    async let user = fetchUser()
    async let posts = fetchPosts()
    async let friends = fetchFriends()

    return try await UserProfile(
        user: user,
        posts: posts,
        friends: friends
    )
}
```

### Actors
```swift
actor Counter {
    private var value = 0

    func increment() {
        value += 1
    }

    func getValue() -> Int {
        value
    }
}

// Usage
let counter = Counter()
await counter.increment()
let value = await counter.getValue()
```

## Extensions

### Organize Code with Extensions
```swift
// MARK: - UITableViewDataSource
extension UserViewController: UITableViewDataSource {
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return users.count
    }

    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        // Implementation
    }
}

// MARK: - Private Methods
private extension UserViewController {
    func setupUI() {
        // Setup code
    }

    func handleError(_ error: Error) {
        // Error handling
    }
}
```

### Add Utility Methods
```swift
extension String {
    var isValidEmail: Bool {
        let pattern = "[A-Z0-9a-z._%+-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,64}"
        return range(of: pattern, options: .regularExpression) != nil
    }
}

extension Array {
    subscript(safe index: Int) -> Element? {
        indices.contains(index) ? self[index] : nil
    }
}
```

## Memory Management

### ARC Basics
- Swift uses Automatic Reference Counting
- Be aware of strong reference cycles
- Use `weak` and `unowned` to break cycles

### Weak vs Unowned
```swift
class ViewController: UIViewController {
    var onComplete: (() -> Void)?

    func setupCallback() {
        // weak - use when reference might become nil
        service.fetchData { [weak self] result in
            guard let self = self else { return }
            self.handle(result)
        }

        // unowned - use when reference will never be nil during closure lifetime
        service.fetchData { [unowned self] result in
            self.handle(result)  // Crashes if self is deallocated
        }
    }
}
```

## Documentation

### Documentation Comments
```swift
/// Fetches user data from the remote server.
///
/// - Parameters:
///   - userID: The unique identifier for the user
///   - forceRefresh: Whether to bypass the cache
/// - Returns: The user object if found
/// - Throws: `NetworkError` if the request fails
func fetchUser(userID: String, forceRefresh: Bool = false) async throws -> User {
    // Implementation
}
```

### MARK Comments
```swift
class UserViewController: UIViewController {
    // MARK: - Properties
    private var users: [User] = []

    // MARK: - Lifecycle
    override func viewDidLoad() {
        super.viewDidLoad()
    }

    // MARK: - Public Methods
    func refreshData() {
        // Implementation
    }

    // MARK: - Private Methods
    private func setupUI() {
        // Implementation
    }
}
```

## SwiftUI Basics

### Views
```swift
struct UserProfileView: View {
    @StateObject private var viewModel = UserProfileViewModel()

    var body: some View {
        VStack(alignment: .leading, spacing: 16) {
            Text(viewModel.userName)
                .font(.title)

            Text(viewModel.email)
                .font(.subheadline)
                .foregroundColor(.secondary)
        }
        .padding()
        .task {
            await viewModel.loadUser()
        }
    }
}
```

### State Management
```swift
struct ContentView: View {
    @State private var count = 0
    @StateObject private var viewModel = ViewModel()
    @ObservedObject var sharedData: SharedData
    @EnvironmentObject var settings: AppSettings
    @Binding var isPresented: Bool

    var body: some View {
        // View implementation
    }
}
```

## Testing

### Unit Tests
```swift
import XCTest
@testable import MyApp

class UserViewModelTests: XCTestCase {
    var viewModel: UserViewModel!
    var mockRepository: MockUserRepository!

    override func setUp() {
        super.setUp()
        mockRepository = MockUserRepository()
        viewModel = UserViewModel(repository: mockRepository)
    }

    func testLoadUser_UpdatesStateToSuccess_WhenRepositoryReturnsUser() async {
        // Given
        let expectedUser = User(id: "123", name: "John")
        mockRepository.userToReturn = expectedUser

        // When
        await viewModel.loadUser(id: "123")

        // Then
        XCTAssertEqual(viewModel.state, .success(expectedUser))
    }
}
```

## Code Style

### Access Control
- Use `private` by default
- Expose only what's necessary
- Use `fileprivate` when needed within same file
- Use `internal` for module-wide access
- Use `public` or `open` for framework APIs

```swift
public class APIClient {
    private let session: URLSession
    private let baseURL: URL

    public init(baseURL: URL) {
        self.baseURL = baseURL
        self.session = URLSession.shared
    }

    public func fetchUser(id: String) async throws -> User {
        // Implementation
    }
}
```

### Type Inference
```swift
// Good - let Swift infer types when obvious
let name = "John"
let count = 10
let users: [User] = []  // Specify when needed for clarity

// Avoid unnecessary type annotations
let name: String = "John"  // Redundant
```

## Common Anti-Patterns to Avoid

- **Force unwrapping with `!`**: Use optional binding instead
- **Massive view controllers**: Break into smaller components
- **Retain cycles**: Use `[weak self]` in closures
- **Ignoring Result type**: Prefer Result over optional + error parameter
- **Not using guard**: Guard statements make code cleaner with early exits
- **Ignoring access control**: Always use appropriate access levels
- **Mutable state everywhere**: Prefer immutable let over var
- **Force casting with `as!`**: Use optional casting `as?` or guard

## Best Practices

- Prefer `let` over `var` (immutability)
- Use guard for early exits
- Leverage Swift's type system
- Use extensions to organize code
- Write tests for business logic
- Use MARK comments to organize large files
- Prefer structs for simple data models
- Use protocols for abstraction and testability
- Handle errors explicitly (don't ignore them)
- Use Swift's modern concurrency features (async/await, actors)

## Performance Tips

- Use lazy properties for expensive initialization
- Use value types (structs) when possible (copy-on-write optimization)
- Avoid capturing strong references in closures unnecessarily
- Use `@autoclosure` for expensive default parameters
- Profile before optimizing (use Instruments)
- Use `@inlinable` for performance-critical generic code

## Security

- Never hardcode secrets or API keys
- Validate all user inputs
- Use Keychain for sensitive data storage
- Sanitize data before display
- Keep dependencies updated
- Use App Transport Security (ATS)

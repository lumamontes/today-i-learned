# kotlin-with-spring-boot-for-api-usage

Kotlin is a modern, statically-typed programming language that runs on the JVM and is fully interoperable with Java. When combined with Spring Boot, it provides a powerful and concise way to build REST APIs.

## Why Kotlin for APIs?

- **Conciseness**: Less boilerplate code compared to Java
- **Null safety**: Built-in null safety reduces NullPointerExceptions
- **Coroutines**: Native support for asynchronous programming
- **Interoperability**: Can use existing Java libraries seamlessly
- **Modern syntax**: Data classes, extension functions, and more

## Spring Boot + Kotlin Setup

### Dependencies

```kotlin
dependencies {
    implementation("org.springframework.boot:spring-boot-starter-web")
    implementation("org.springframework.boot:spring-boot-starter-data-jpa")
    implementation("com.fasterxml.jackson.module:jackson-module-kotlin")
    implementation("org.jetbrains.kotlin:kotlin-reflect")
    implementation("org.jetbrains.kotlin:kotlin-stdlib-jdk8")
}
```

### Application Class

```kotlin
@SpringBootApplication
class Application

fun main(args: Array<String>) {
    runApplication<Application>(*args)
}
```

## Key Kotlin Features in Spring Boot

### Data Classes

Perfect for DTOs and entities:

```kotlin
@Entity
data class User(
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    val id: Long? = null,
    val name: String,
    val email: String,
    val createdAt: LocalDateTime = LocalDateTime.now()
)
```

### Null Safety

Kotlin's null safety works great with Spring's optional values:

```kotlin
@GetMapping("/users/{id}")
fun getUser(@PathVariable id: Long): ResponseEntity<User> {
    val user = userRepository.findById(id)
    return if (user != null) {
        ResponseEntity.ok(user)
    } else {
        ResponseEntity.notFound().build()
    }
}
```

### Extension Functions

Add utility functions to existing classes:

```kotlin
fun String.isValidEmail(): Boolean {
    return this.contains("@") && this.contains(".")
}

// Usage
if (email.isValidEmail()) {
    // ...
}
```

### Coroutines for Async Operations

Kotlin coroutines provide a cleaner alternative to CompletableFuture:

```kotlin
@GetMapping("/users")
suspend fun getUsers(): List<User> {
    return userRepository.findAll()
}
```

## REST Controller Example

```kotlin
@RestController
@RequestMapping("/api/users")
class UserController(
    private val userService: UserService
) {
    @GetMapping
    fun getAllUsers(): List<UserDTO> {
        return userService.findAll()
    }
    
    @GetMapping("/{id}")
    fun getUser(@PathVariable id: Long): ResponseEntity<UserDTO> {
        return userService.findById(id)
            ?.let { ResponseEntity.ok(it) }
            ?: ResponseEntity.notFound().build()
    }
    
    @PostMapping
    fun createUser(@RequestBody @Valid userDTO: UserDTO): ResponseEntity<UserDTO> {
        val created = userService.create(userDTO)
        return ResponseEntity.status(HttpStatus.CREATED).body(created)
    }
    
    @PutMapping("/{id}")
    fun updateUser(
        @PathVariable id: Long,
        @RequestBody @Valid userDTO: UserDTO
    ): ResponseEntity<UserDTO> {
        return userService.update(id, userDTO)
            ?.let { ResponseEntity.ok(it) }
            ?: ResponseEntity.notFound().build()
    }
    
    @DeleteMapping("/{id}")
    fun deleteUser(@PathVariable id: Long): ResponseEntity<Void> {
        return if (userService.delete(id)) {
            ResponseEntity.noContent().build()
        } else {
            ResponseEntity.notFound().build()
        }
    }
}
```

## Repository Pattern

```kotlin
interface UserRepository : JpaRepository<User, Long> {
    fun findByEmail(email: String): User?
    fun existsByEmail(email: String): Boolean
}
```

## Service Layer

```kotlin
@Service
class UserService(
    private val userRepository: UserRepository
) {
    fun findAll(): List<UserDTO> {
        return userRepository.findAll().map { it.toDTO() }
    }
    
    fun findById(id: Long): UserDTO? {
        return userRepository.findById(id)
            .map { it.toDTO() }
            .orElse(null)
    }
    
    fun create(userDTO: UserDTO): UserDTO {
        val user = userDTO.toEntity()
        return userRepository.save(user).toDTO()
    }
    
    fun update(id: Long, userDTO: UserDTO): UserDTO? {
        return userRepository.findById(id)
            .map { existing ->
                val updated = existing.copy(
                    name = userDTO.name,
                    email = userDTO.email
                )
                userRepository.save(updated).toDTO()
            }
            .orElse(null)
    }
    
    fun delete(id: Long): Boolean {
        return if (userRepository.existsById(id)) {
            userRepository.deleteById(id)
            true
        } else {
            false
        }
    }
}
```

## Error Handling

```kotlin
@ControllerAdvice
class GlobalExceptionHandler {
    
    @ExceptionHandler(EntityNotFoundException::class)
    fun handleNotFound(ex: EntityNotFoundException): ResponseEntity<ErrorResponse> {
        return ResponseEntity
            .status(HttpStatus.NOT_FOUND)
            .body(ErrorResponse(ex.message ?: "Resource not found"))
    }
    
    @ExceptionHandler(MethodArgumentNotValidException::class)
    fun handleValidation(ex: MethodArgumentNotValidException): ResponseEntity<ErrorResponse> {
        val errors = ex.bindingResult.fieldErrors
            .associate { it.field to (it.defaultMessage ?: "") }
        return ResponseEntity
            .status(HttpStatus.BAD_REQUEST)
            .body(ErrorResponse("Validation failed", errors))
    }
}
```

## Benefits Over Java

1. **Less boilerplate**: Data classes eliminate getters/setters/constructors
2. **Safer code**: Null safety prevents many runtime errors
3. **More expressive**: Extension functions and higher-order functions
4. **Better async**: Coroutines are more intuitive than CompletableFuture
5. **Interoperable**: Can still use Java libraries and frameworks

Links:

- [Kotlin Official Documentation](https://kotlinlang.org/docs/home.html)
- [Spring Boot with Kotlin Guide](https://spring.io/guides/tutorials/spring-boot-kotlin/)
- [Kotlin Coroutines Guide](https://kotlinlang.org/docs/coroutines-guide.html)


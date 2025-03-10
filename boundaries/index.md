#### BOUNDARIES

Boundaries in Clean Code refer to the separation between different parts of a system—such as your code and external systems (e.g., libraries, APIs, databases)—to ensure loose coupling, maintainability, and testability. In Go, a language that values simplicity and explicitness, these principles translate naturally into interfaces, dependency inversion, and careful management of external dependencies

##### `Separate Your Code from External Systems`
Clean Code advises isolating your business logic from third-party libraries or external systems. This reduces dependency on external implementations and makes your code more adaptable to change. In Go, use interfaces to define what your code needs, not how it’s implemented.

```go
    type Notifier interface {
        SendNotification(message string) error
    }

    type EmailNotifier struct{}

    func (e EmailNotifier) SendNotification(message string) error {
        fmt.Println("Sending email:", message)
        return nil
    }

    func NotifyUser(n Notifier, message string) {
        err := n.SendNotification(message)
        if err != nil {
            log.Println("Failed to send notification:", err)
        }
    }
```

**By depending on an interface (Notifier) instead of a concrete type, we allow for different implementations (e.g., SMS, Push notifications) without modifying NotifyUser.**

##### `Dependency Inversion for Flexibility`
Boundaries should invert control so your code dictates the contract (via interfaces), not the external system. This aligns with the Dependency Inversion Principle and keeps your code in charge. Instead of hardcoding dependencies, inject them via constructors.

```go
    package main

    import "fmt"

    // Define what we need from a database
    type DataStore interface {
        Save(data string) error
    }

    // Business logic uses the interface
    type Service struct {
        store DataStore
    }

    func (s *Service) Process(data string) error {
        return s.store.Save(data)
    }

    // External database (e.g., mock for now)
    type MockDB struct{}

    func (m *MockDB) Save(data string) error {
        fmt.Println("Saved to mock DB:", data)
        return nil
    }

    func main() {
        service := Service{store: &MockDB{}}
        service.Process("example data")
    }
```
The Service doesn’t depend on a specific database—it works with any DataStore implementation, making it easy to swap or mock during testing.

***Another Example***
```go 
    type OrderService struct {
        repo OrderRepository
    }

    func NewOrderService(repo OrderRepository) *OrderService {
        return &OrderService{repo: repo}
    }

    func (s *OrderService) ProcessOrder(orderID string) error {
        order, err := s.repo.GetOrder(orderID)
        if err != nil {
            return fmt.Errorf("order retrieval failed: %w", err)
        }
        fmt.Println("Processing order:", order)
        return nil
    }
```

Here, OrderService depends on the OrderRepository interface rather than a concrete implementation, allowing easy substitution and testing.


##### `Separate Core Business Logic from External Dependencies`

Clean Code recommends wrapping third-party libraries in your own abstractions to limit their impact on your codebase. This prevents external API changes from rippling through your system.

```go
    package main

    import (
        "fmt"
        "net/http" // External library
    )

    // Our abstraction
    type HTTPClient interface {
        Get(url string) (string, error)
    }

    // Wrapper around the standard library's http.Client
    type MyHTTPClient struct {
        client *http.Client
    }

    func (c *MyHTTPClient) Get(url string) (string, error) {
        resp, err := c.client.Get(url)
        if err != nil {
            return "", err
        }
        defer resp.Body.Close()
        return "response", nil // Simplified for brevity
    }

    // Business logic uses our interface
    type Fetcher struct {
        client HTTPClient
    }

    func (f *Fetcher) FetchData(url string) (string, error) {
        return f.client.Get(url)
    }

    func main() {
        fetcher := Fetcher{client: &MyHTTPClient{client: &http.Client{}}}
        data, err := fetcher.FetchData("https://example.com")
        if err != nil {
            fmt.Println("Error:", err)
            return
        }
        fmt.Println("Data:", data)
    }
```  

If http.Client changes, only MyHTTPClient needs adjustment—not the entire codebase.

```go
    type PaymentProcessor interface {
        Charge(amount float64, userID string) error
    }

    type CheckoutService struct {
        payment PaymentProcessor
    }

    func (c *CheckoutService) Checkout(userID string, amount float64) error {
        return c.payment.Charge(amount, userID)
    }
```

This ensures that switching from one payment provider to another requires minimal changes.


##### `Avoid Passing External Types Through Your Code`

Don’t let third-party types leak into your core logic. Convert them to your own types at the boundary to maintain control and clarity.

```go
    package main

    import (
        "fmt"
        "time"
    )

    // Our own type, not time.Time
    type Timestamp struct {
        value int64
    }

    // Boundary conversion
    func NewTimestamp(t time.Time) Timestamp {
        return Timestamp{value: t.Unix()}
    }

    // Business logic uses our type
    type Event struct {
        time Timestamp
    }

    func (e Event) Display() {
        fmt.Println("Event time:", e.time.value)
    }

    func main() {
        now := time.Now()
        event := Event{time: NewTimestamp(now)}
        event.Display()
    }
```

Here, time.Time is converted to Timestamp at the boundary, keeping Event independent of the standard library’s implementation.

##### Define Clear API Boundaries
APIs should expose clear contracts and avoid leaking internal details.

```go
    type User struct {
        ID    string `json:"id"`
        Name  string `json:"name"`
        Email string `json:"email"`
    }

    func GetUserHandler(w http.ResponseWriter, r *http.Request) {
        user := User{ID: "123", Name: "John Doe", Email: "john@example.com"}
        json.NewEncoder(w).Encode(user)
    }
```

- The API response follows a structured format (json tags).
- Internal implementation details (e.g., database models) are not exposed directly.

##### `Keep Boundaries Thin and Focused`
Boundary code (adapters, wrappers) should be minimal, doing only translation or delegation. Avoid putting business logic in boundary layers to maintain separation of concerns.

```go
    package main

    import "fmt"

    // Thin boundary layer
    type UserRepo interface {
        FindByID(id string) (string, error)
    }

    type SQLRepo struct{} // Pretend this talks to a real DB

    func (s *SQLRepo) FindByID(id string) (string, error) {
        return "user_" + id, nil // Mock implementation
    }

    // Core logic stays separate
    type UserService struct {
        repo UserRepo
    }

    func (s *UserService) GetUser(id string) (string, error) {
        return s.repo.FindByID(id)
    }

    func main() {
        service := UserService{repo: &SQLRepo{}}
        user, _ := service.GetUser("123")
        fmt.Println(user)
}
```

SQLRepo is a thin adapter; UserService contains the actual logic.

***Another Example***

```go
    type UserRepository interface {
        FindUser(id string) (*User, error)
    }

    type SQLUserRepository struct {
        db *sql.DB
    }

    func (s *SQLUserRepository) FindUser(id string) (*User, error) {
        row := s.db.QueryRow("SELECT id, name, email FROM users WHERE id = ?", id)
        user := &User{}
        err := row.Scan(&user.ID, &user.Name, &user.Email)
        return user, err
    }
```
By isolating database access in a repository layer, the rest of the application does not depend on SQL details.

##### `Encapsulate Third-Party Dependencies`
Wrap external libraries in an abstraction to avoid tight coupling.

```go
    type Cache interface {
        Get(key string) (string, error)
        Set(key, value string, ttl time.Duration) error
    }

    type RedisCache struct {
        client *redis.Client
    }

    func (r *RedisCache) Get(key string) (string, error) {
        return r.client.Get(context.Background(), key).Result()
    }
```
Now, replacing Redis with another caching mechanism won’t require changes throughout the codebase.

--- 

Bad ❌

```go
    package main

    import (
        "database/sql"
        "fmt"
        "log"
        _ "github.com/go-sql-driver/mysql"
    )

    // User model representing a user entity
    type User struct {
        ID    string
        Name  string
        Email string
    }

    // UserService directly interacts with the database (bad practice)
    type UserService struct {
        db *sql.DB
    }

    // FindUser fetches a user from the database
    func (s *UserService) FindUser(id string) (*User, error) {
        row := s.db.QueryRow("SELECT id, name, email FROM users WHERE id = ?", id)
        user := &User{}
        err := row.Scan(&user.ID, &user.Name, &user.Email)
        if err != nil {
            return nil, fmt.Errorf("user not found: %w", err)
        }
        return user, nil
    }

    func main() {
        // Initialize DB connection (not ideal inside main)
        db, err := sql.Open("mysql", "user:password@tcp(localhost:3306)/dbname")
        if err != nil {
            log.Fatal("Failed to connect to database:", err)
        }
        defer db.Close()

        // Directly passing db to UserService (tight coupling)
        userService := &UserService{db: db}

        user, err := userService.FindUser("123")
        if err != nil {
            log.Println("Error:", err)
        } else {
            fmt.Println("User:", user)
        }
    }
```

Good Code ✅

```go
    package main

    import (
        "database/sql"
        "errors"
        "fmt"
        "log"
        _ "github.com/go-sql-driver/mysql"
    )

    // User model representing a user entity
    type User struct {
        ID    string
        Name  string
        Email string
    }

    // Define an interface for user repository (boundary)
    type UserRepository interface {
        FindUser(id string) (*User, error)
    }

    // Concrete implementation using SQL
    type SQLUserRepository struct {
        db *sql.DB
    }

    // FindUser fetches a user from the database
    func (r *SQLUserRepository) FindUser(id string) (*User, error) {
        row := r.db.QueryRow("SELECT id, name, email FROM users WHERE id = ?", id)
        user := &User{}
        err := row.Scan(&user.ID, &user.Name, &user.Email)
        if err != nil {
            if errors.Is(err, sql.ErrNoRows) {
                return nil, fmt.Errorf("user not found: %w", err)
            }
            return nil, fmt.Errorf("database error: %w", err)
        }
        return user, nil
    }

    // Business logic layer (decoupled from database)
    type UserService struct {
        repo UserRepository
    }

    // Constructor for UserService
    func NewUserService(repo UserRepository) *UserService {
        return &UserService{repo: repo}
    }

    // GetUser handles the business logic of retrieving a user
    func (s *UserService) GetUser(id string) (*User, error) {
        return s.repo.FindUser(id)
    }

    func main() {
        // Initialize DB connection
        db, err := sql.Open("mysql", "user:password@tcp(localhost:3306)/dbname")
        if err != nil {
            log.Fatal("Failed to connect to database:", err)
        }
        defer db.Close()

        // Inject repository into service (dependency inversion)
        userRepo := &SQLUserRepository{db: db}
        userService := NewUserService(userRepo)

        // Fetch user
        user, err := userService.GetUser("123")
        if err != nil {
            log.Println("Error:", err)
        } else {
            fmt.Println("User:", user)
        }
    }
```
Conclusion
In Go, Clean Code boundaries leverage the language’s lightweight interfaces and explicit design to create a system that’s modular, testable, and resilient to external changes—perfectly aligning with the ethos of both Clean Code and Go.
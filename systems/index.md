#### SYSTEMS
A "system" here refers to the broader application or service, composed of various components working together. The key takeaway is that good system design hinges on separating concerns, managing dependencies, and deferring decisions until necessary—all principles that align well with Go's simplicity and pragmatism.

Also the idea is that good system design is not just about writing clean code at the micro level but also structuring components in a way that they work well together. When developing software, we must consider boundaries, dependency management, and modularization while ensuring that the system remains adaptable.

##### `Separation of Concerns`
One of the most crucial ideas in this chapter is separating concerns. Systems should be divided into well-defined components that handle specific responsibilities. In Go, this is done by structuring code into packages.  

 Example: A web application can be structured into different layers:
```go
/project
  ├── main.go
  ├── handlers/
  │   ├── user.go
  │   ├── product.go
  ├── services/
  │   ├── user_service.go
  │   ├── product_service.go
  ├── repository/
  │   ├── user_repo.go
  │   ├── product_repo.go
  ├── models/
  │   ├── user.go
  │   ├── product.go

  ```
- Handlers: Handle HTTP requests.
- Services: Contain business logic.
- Repositories: Handle data persistence (DB operations).
- Models: Represent data structures.

This ensures loose coupling and makes it easier to replace implementations without affecting the entire system.

##### `Managing Dependencies in Go`
Systems should manage dependencies explicitly to avoid tight coupling. Go’s lack of a built-in dependency injection framework encourages explicit dependency passing, which aligns with Martin’s advice.

```go

    package main

    import "fmt"

    type Datastore interface {
        Save(data string) error
    }

    type MemoryStore struct{}

    func (m *MemoryStore) Save(data string) error {
        fmt.Println("Saved:", data)
        return nil
    }

    type App struct {
        store Datastore
    }

    func NewApp(store Datastore) *App {
        return &App{store: store}
    }

    func (a *App) Process(data string) error {
        return a.store.Save(data)
    }

    func main() {
        store := &MemoryStore{}
        app := NewApp(store)
        app.Process("example data")
    }
```
The Datastore interface allows swapping implementations (e.g., from memory to a database) without changing App.

```go
    package services

    import "project/repository"

    // UserService depends on UserRepository but uses an interface
    type UserService struct {
        repo repository.UserRepository
    }

    func NewUserService(repo repository.UserRepository) *UserService {
        return &UserService{repo: repo}
    }

    func (s *UserService) GetUser(id int) (repository.User, error) {
        return s.repo.FindByID(id)
    }
```
This allows flexibility—any implementation of UserRepository can be swapped without modifying UserService.

##### `Building Systems Gradually`
An incremental approach to system design rather than trying to plan everything upfront (a common mistake in software architecture). In Go:
- Start with working software.
- Refactor as needed while keeping tests in place.
- Add complexity only when required.

Go’s simplicity allows incremental design without unnecessary abstraction.

##### `Taming Complexity with Facades`
When systems grow large, it’s crucial to hide complexity behind simpler interfaces. Instead of exposing raw implementations, provide a high-level facade.

Example: Facade for a Notification System

```go
    package notifications

    import (
        "fmt"
    )

    // EmailService handles email notifications
    type EmailService struct{}

    func (e *EmailService) SendEmail(to, message string) {
        fmt.Println("Sending email to:", to, "Message:", message)
    }

    // SMSService handles SMS notifications
    type SMSService struct{}

    func (s *SMSService) SendSMS(to, message string) {
        fmt.Println("Sending SMS to:", to, "Message:", message)
    }

    // NotificationFacade provides a unified interface
    type NotificationFacade struct {
        email *EmailService
        sms   *SMSService
    }

    func NewNotificationFacade() *NotificationFacade {
        return &NotificationFacade{
            email: &EmailService{},
            sms:   &SMSService{},
        }
    }

    func (n *NotificationFacade) SendNotification(to, message, method string) {
        switch method {
        case "email":
            n.email.SendEmail(to, message)
        case "sms":
            n.sms.SendSMS(to, message)
        default:
            fmt.Println("Invalid method")
        }
    }
```
This allows the system to grow without exposing its internal workings to every part of the codebase.


##### `Scaling Up with Modularity`
Systems should scale by breaking into smaller, independent modules. In Go, this aligns with the package system. Each package should have a single responsibility:

```go
    // logger/logger.go
    package logger

    import "log"

    type Logger struct {
        l *log.Logger
    }

    func New() *Logger {
        return &Logger{l: log.Default()}
    }

    func (l *Logger) Info(msg string) {
        l.l.Println("INFO:", msg)
    }

    // service/service.go
    package service

    import "path/to/logger"

    type Service struct {
        log *logger.Logger
    }

    func New(log *logger.Logger) *Service {
        return &Service{log: log}
    }

    func (s *Service) Run() {
        s.log.Info("Service running")
    }
```
Here, logger and service are separate packages, promoting modularity and reusability.

##### `Avoid Over-Engineering`
- Go’s philosophy of simplicity resonates with Martin’s warning against premature optimization or over-complicated frameworks. Don’t build a sprawling system upfront—start with what’s needed and refactor as requirements evolve.
Example: Instead of a complex configuration loader, use Go’s flag or a simple struct


##### `Testing at Different Levels`
The importance of testing at multiple levels. In Go, testing can be handled via unit tests and integration tests.

Example: Writing a Unit Test for the UserService

```go
    package services_test

    import (
        "errors"
        "project/repository"
        "project/services"
        "testing"

        "github.com/stretchr/testify/assert"
    )

    // Mock repository
    type MockUserRepository struct{}

    func (m *MockUserRepository) FindByID(id int) (repository.User, error) {
        if id == 1 {
            return repository.User{ID: 1, Name: "John Doe"}, nil
        }
        return repository.User{}, errors.New("user not found")
    }

    func TestGetUser(t *testing.T) {
        mockRepo := &MockUserRepository{}
        userService := services.NewUserService(mockRepo)

        user, err := userService.GetUser(1)
        assert.Nil(t, err)
        assert.Equal(t, "John Doe", user.Name)

        _, err = userService.GetUser(2)
        assert.NotNil(t, err)
    }
```
- The MockUserRepository simulates database behavior.
- The test ensures GetUser correctly fetches users.

This adheres to Separation of Concerns, making the service easy to test.

clean systems emerge from deliberate separation of concerns, thoughtful dependency management, and a focus on modularity—all of which Go supports naturally. By using factory functions like NewX(), passing dependencies explicitly, and organizing code into packages, you can build systems in Go that are both clean and scalable. Start small, keep it simple, and let the system grow organically as needs arise—principles that echo both Clean Code and Go’s design ethos.
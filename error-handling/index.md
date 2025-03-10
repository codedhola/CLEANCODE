#### ERROR HANDLING

Error handling is essential in programming since things can go wrong, and it’s our responsibility to manage failures effectively. However, in many codebases, error handling overwhelms the logic, making it hard to understand what the code actually does. While handling errors is crucial, it should not obscure the program’s intent. This chapter presents techniques for writing clean and robust code that manages errors gracefully without compromising readability. 

##### `Return Errors Explicitly`
Go does not use exceptions; instead, errors are explicit return values. Functions should return errors as part of their signature.

```go
    func divide(a, b float64) (float64, error) {
        if b == 0 {
            return 0, fmt.Errorf("cannot divide by zero")
        }
        return a / b, nil
    }

    func readFile(path string) (string, error) {
        data, err := os.ReadFile(path)
        if err != nil {
            return "", err
        }
        return string(data), nil
    }
```

##### `Check Errors at the Call Site`
Don’t clutter low-level functions with complex error logic—return errors up the call stack to where they can be meaningfully handled. This keeps functions focused on their primary responsibility (Single Responsibility Principle).

```go
    func processFile(path string) error {
        content, err := readFile(path)
        if err != nil {
            return fmt.Errorf("failed to process file: %w", err)
        }
        // Process content
        return nil
    }
```

##### `Use Sentinel Errors Sparingly`
Sentinel errors are predefined error values that can be checked with errors.Is().

```go
    var ErrNotFound = errors.New("not found")

    func findUser(id string) (User, error) {
        return User{}, ErrNotFound
    }

    user, err := findUser("123")

    if errors.Is(err, ErrNotFound) {
        fmt.Println("User does not exist")
    }
```

**Avoid returning raw sentinel errors in multiple places as it can lead to tight coupling.**

##### `Provide Context with Wrapped Errors`
When propagating errors, add context to make them more informative without losing the original error. Go’s fmt.Errorf with %w or errors.Wrap (from external libraries) helps trace the error’s origin, aligning with Clean Code’s emphasis on meaningful feedback.

```go
    func saveData(data string) error {
        err := writeToDB(data)
        if err != nil {
            return fmt.Errorf("saveData: database write failed: %w", err)
        }
        return nil
    }
```

#####  `Define Custom Error Types for Clarity`
For complex cases, use custom error types.

```go
    type ValidationError struct {
        Field string
        Msg   string
    }

    func (e *ValidationError) Error() string {
        return fmt.Sprintf("validation failed: %s - %s", e.Field, e.Msg)
    }

    func validateAge(age int) error {
        if age < 18 {
            return &ValidationError{"Age", "must be 18 or older"}
        }
        return nil
    }
```

##### `Centralized Error Handling`
Instead of handling errors at every level, propagate them and handle them centrally.

```go
    func main() {
        if err := startServer(); err != nil {
            log.Fatalf("Server failed: %v", err)
        }
    }
```
**Using log.Fatalf() or returning an error from main() is preferred over panic().**


##### Avoid Returning nil for Errors Unnecessarily
Clean Code discourages misleading or ambiguous states. In Go, always return an error if something goes wrong—don’t return nil to mask issues. Similarly, avoid returning nil alongside a non-nil error.

```go
    // Bad: Confusing return
    func badExample() (string, error) {
        return "", nil // Misleading if an error occurred
    }

    // Good: Clear intent
    func goodExample() (string, error) {
        if someCondition {
            return "", errors.New("condition not met")
        }
        return "success", nil
    }
```

##### `Conclusion`
By following these Clean Code principles, Go programs become more maintainable, predictable, and easy to debug.


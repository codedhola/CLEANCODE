#### UNIT TESTS

Unit Tests are as critical as production code—perhaps even more so—because they ensure your
code works as intended and remains maintainable over time. In Go, where simplicity and pragmatism
reign, this translates to writing clear, concise, and effective tests using the language’s
built-in testing package.

##### `The Three Laws of TDD (Test-Driven Development)`
-   `First Law`: You may not write production code until you have written a failing unit test.
In Go, this means starting with a test like:
```go
    package mathutils

    import (
        "testing"
    )

    func TestAdd(t *testing.T) {
        if Add(2, 2) != 4 {
            t.Errorf("Add(2, 2) = %d; want 4", Add(2, 2))
        }
    }
```
`Only then do you implement func Add(a, b int) int { return a + b }.`

- `Second Law`: You may not write more of a unit test than is sufficient to fail (and not compiling is failing).
Keep tests minimal at first—don’t overbuild before seeing the failure.

- `Third Law`: You may not write more production code than is sufficient to pass the current failing test.
In Go’s iterative style, this keeps your Add function lean and focused.

Following these laws in Go ensures a tight feedback loop, leveraging go test to catch issues early.



##### `Clean Tests Are Readable and Focused`
Good unit tests in Go should read like a story, not a puzzle. Use descriptive test names
(e.g., TestAddTwoPositiveNumbers) and avoid clutter. Go’s t.Errorf provides clear failure
messages, so make them meaningful:

```go
    func TestDivideByZero(t *testing.T) {
        _, err := Divide(10, 0)
        if err == nil {
            t.Error("Divide(10, 0) should return an error")
        }
    }
```

Each test should have one assertion (or a single "concept") to keep it simple—Go’s philosophy of clarity shines here.


##### `F.I.R.S.T. Principles for Tests`
- `Fast`: Go tests should run quickly with go test ./.... Slow tests discourage frequent runs.
- `Independent`: Tests shouldn’t rely on each other. Avoid shared state; use fresh variables or structs per test:
```go
    func TestUserCreation(t *testing.T) {
        user := NewUser("Alice")
        if user.Name != "Alice" {
            t.Errorf("Expected name 'Alice', got %s", user.Name)
        }
    }
```
- `Repeatable`: Tests should pass consistently, whether run locally or in CI. No flaky network calls!
- `Self-Validating`: No manual checking—go test should tell you pass or fail.
- `Timely`: Write tests before or alongside code, not as an afterthought.


##### `Tests Enable Refactoring`
In Go, where interfaces and simplicity encourage flexible design, unit tests act as a safety net. You can refactor func ProcessData(data []byte) into smaller functions, confident that go test will catch regressions. Without tests, even Go’s type system can’t save you from subtle bugs.

##### `Keep Tests Clean`
Dirty tests—full of duplication or complexity—are a liability. Use Go’s table-driven tests to stay DRY:

```go
    func TestMultiply(t *testing.T) {
        tests := []struct {
            a, b, want int
        }{
            {2, 3, 6},
            {0, 5, 0},
            {-1, 4, -4},
        }
        for _, tt := range tests {
            if got := Multiply(tt.a, tt.b); got != tt.want {
                t.Errorf("Multiply(%d, %d) = %d; want %d", tt.a, tt.b, got, tt.want)
            }
        }
    }
```
This keeps tests concise and maintainable, aligning with Go’s idiom of simplicity.

##### `Tests Are Code Too`
Treat test files (e.g., math_test.go) with the same care as math.go. Refactor them, keep them organized, and ensure they’re as readable as your Go production code.




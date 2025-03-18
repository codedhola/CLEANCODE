#### CLASSES (Struct)

The chapter focuses on designing clean, maintainable, and well-structured classes, emphasizing principles like the Single Responsibility Principle (SRP), cohesion, and managing complexity. Since Go doesn't have traditional classes but uses structs and methods, I'll translate the concepts accordingly.

##### `Classes(Struct) Should Be Small`
Classes (or in Go, structs with associated methods) should have a limited scope and responsibility. A class that grows too large in terms of methods or responsibilities becomes hard to maintain. The name of a struct should clearly describe its purpose, and if you need "and" or "or" to describe it, it might be doing too much.

Example (Bad - Too Many Responsibilities): 

```go
    package main

    type UserManager struct {
        name     string
        email    string
        password string
    }

    func (u *UserManager) ValidateEmail() bool {
        // Email validation logic
        return true
    }

    func (u *UserManager) HashPassword() string {
        // Password hashing logic
        return "hashed"
    }

    func (u *UserManager) SaveToDatabase() {
        // Database save logic
        println("Saved to DB")
    }

```
This UserManager handles validation, hashing, and persistence—too many responsibilities.

Example (Good - Single Responsibility):

```go

    package main

    type User struct {
        name     string
        email    string
        password string
    }

    type EmailValidator struct{}

    func (e *EmailValidator) Validate(email string) bool {
        // Email validation logic
        return true
    }

    type PasswordHasher struct{}

    func (p *PasswordHasher) Hash(password string) string {
        // Hashing logic
        return "hashed"
    }

    type UserRepository struct{}

    func (r *UserRepository) Save(user User) {
        // Save to DB
        println("Saved to DB")
    }
```
Here, each struct has one clear job, adhering to SRP.

##### `The Single Responsibility Principle (SRP)`
A class/struct should have only one reason to change. If a struct changes for multiple reasons (e.g., UI updates and database changes), it’s a sign it’s doing too much. Split responsibilities into separate structs or packages.
Example:
Instead of one struct handling both formatting and data storage:

bad Code ❌

```go

    package main

    type Report struct {
        data string
    }

    func (r *Report) FormatHTML() string {
        return "<html>" + r.data + "</html>"
    }

    func (r *Report) Save() {
        println("Saved:", r.data)
    }

```
<center>separate them </center>
Good Code✅

```go
    package main

    type Report struct {
        data string
    }

    type ReportFormatter struct{}

    func (f *ReportFormatter) FormatHTML(r Report) string {
        return "<html>" + r.data + "</html>"
    }

    type ReportSaver struct{}

    func (s *ReportSaver) Save(r Report) {
        println("Saved:", r.data)
    }
```

##### `Cohesion`
Classes/structs should be cohesive—methods should operate on the struct’s fields, and most fields should be used by most methods. Low cohesion (e.g., methods using only a subset of fields) suggests the struct should be split.
- A class(Struct) should have one clear purpose.
- If a class(Struct) has too many responsibilities, split it into smaller ones.

Example (Low Cohesion): ❌
```go
    type OrderProcessor struct {}

    func (o *OrderProcessor) ValidateOrder() {}
    func (o *OrderProcessor) CalculateDiscount() {}
    func (o *OrderProcessor) SaveToDB() {}
    func (o *OrderProcessor) SendEmail() {}
```

Good example ✅

```go
    type Order struct {
        items []Item
    }

    func (o *Order) CalculateTotal() float64 { /* logic */ return 0.0 }

    type OrderValidator struct {}

    func (v *OrderValidator) Validate(order Order) bool { return true }

    type OrderRepository struct {}

    func (r *OrderRepository) Save(order Order) { /* logic */ }

    type EmailNotifier struct {}

    func (e *EmailNotifier) SendOrderConfirmation(order Order) { /* logic */ }
```
Solution: Each class has a single responsibility.


##### `Isolating from Change`
Dependencies should be minimized and abstracted. Use interfaces to decouple structs from concrete implementations.
Example (Tight Coupling): BAD CODE ❌

```go
    package main

    type Order struct {
        db *Database // Concrete dependency
    }

    func (o *Order) Save() {
        o.db.Save("order data")
    }

    type Database struct{}

    func (d *Database) Save(data string) {
        println("Saved:", data)
    }

```

GOOD CODE ✅

```go
    package main

    type DataStore interface {
        Save(data string)
    }

    type Order struct {
        store DataStore // Abstract dependency
    }

    func (o *Order) Save() {
        o.store.Save("order data")
    }

    type Database struct{}

    func (d *Database) Save(data string) {
        println("Saved:", data)
    }
```

##### `Open-Closed Principle (OCP)`
A class should be open for extension, closed for modification.

BAD CODE ❌

```go
    type PaymentProcessor struct {}

    func (p *PaymentProcessor) Process(paymentType string) {
        if paymentType == "CreditCard" {
            // process credit card
        } else if paymentType == "PayPal" {
            // process PayPal
        }
    }
```
👉 Problem: If a new payment method is introduced, this class must be modified.


GOOD CODE ✅

```go
    type PaymentMethod interface {
        ProcessPayment()
    }

    type CreditCard struct {}

    func (c CreditCard) ProcessPayment() { /* logic */ }

    type PayPal struct {}

    func (p PayPal) ProcessPayment() { /* logic */ }

    func ProcessPayment(method PaymentMethod) {
        method.ProcessPayment()
    }
```
👉 Solution: New payment types can be added without modifying existing code.

##### `Keep Classes Small & Focused`
- Fewer than 500 lines is ideal.
- Follow single responsibility principle.
- Break big classes into smaller ones if necessary.
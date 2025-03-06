#### OBJECT AND DATA STRUCTURES

In clean code, objects and data structures serve different purposes. Objects encapsulate behavior and hide implementation details, while data structures expose state and lack behavior. Understanding the difference helps in writing maintainable and flexible Go code.

- `Objects Hide Data and Expose Behavior`: Objects encapsulate data and expose only the necessary behavior through methods. This enforces abstraction and prevents direct data manipulation.

##### Example: Object Oriented Approach in GO✅

```go
    package main

    import "fmt"

    // User struct hides internal data
    type User struct {
        name string
        age  int
    }

    // Constructor function for User
    func NewUser(name string, age int) *User {
        return &User{name: name, age: age}
    }

    // Method to get the name
    func (u *User) GetName() string {
        return u.name
    }

    // Method to increment age
    func (u *User) IncrementAge() {
        u.age++
    }

    func main() {
        user := NewUser("Alice", 25)
        fmt.Println("User:", user.GetName()) // Encapsulated access
        user.IncrementAge()                  // Controlled modification
    }
```

##### ✅ Why is this good?
	•	Data is private (name and age are not exposed directly).
	•	Methods control access, preventing unintended modifications.
	•	The implementation can change without affecting other code.

- `Data Structures Expose Data and Lack Behavior`: Data structures store state but do not contain behavior. This makes them useful for passing structured information without restricting access.

##### Example: Struct as a Data Structure

```go
    package main

    import "fmt"

    // Plain struct exposing fields
    type UserData struct {
        Name string
        Age  int
    }

    func main() {
        user := UserData{Name: "Bob", Age: 30}
        fmt.Println("User:", user.Name, "Age:", user.Age) // Direct access
    }
```
#####  ✅ Why is this useful?
	•	Simple data storage without unnecessary methods.
	•	Best for transferring data between functions or APIs.
	•	No abstraction, making it easy to serialize (e.g., JSON).


| Scenario | Use Objects | Use Data Structures |
| -------- | ----------- | ------------------- |
Encapsulation needed? |	✅ Yes – Hide data, expose behavior	| ❌ No – Expose data directly
Behavior needed? | ✅ Yes – Methods enforce rules | ❌ No – Just store values
Flexible implementation? | ✅ Yes – Can modify without breaking API| ❌ No – Any change affects usage
Used in APIs? |	❌ No – APIs should use DTOs	 | ✅ Yes – Easier serializationScenario	Use Objects	Use Data Structures


- `Data Transfer Objects (DTOs) for APIs`: APIs often return data structures (DTOs) instead of full objects to separate internal logic from external consumers. 


```go
    package main

    import (
        "encoding/json"
        "fmt"
    )

    // DTO for API response
    type UserDTO struct {
        Name string `json:"name"`
        Age  int    `json:"age"`
    }

    func main() {
        user := UserDTO{Name: "Eve", Age: 28}
        jsonData, _ := json.Marshal(user) // Convert struct to JSON
        fmt.Println(string(jsonData))     // Output: {"name":"Eve","age":28}
    }
```
##### ✅ Why use DTOs?
	•	Keeps internal objects separate from API responses.
	•	Makes data serialization easier.
	•	Reduces tight coupling between API and logic.

`Key Notes`  

✅ Use objects to encapsulate behavior and hide internal data.
✅ Use data structures for transferring raw data without logic.
✅ APIs should expose DTOs, not full objects.
✅ Choose based on flexibility, maintainability, and abstraction needs.
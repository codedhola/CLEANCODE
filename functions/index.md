#### FUNCTIONS

###### Functions are the first line of organization in any program. Writing them well is important.

###### * The first rule of functions is that they should be small. The second rule of functions is that they should be smaller than that. 
for example consider this code below 

```go
    package main

    import (
        "fmt"
        "io/ioutil"
        "net/http"
        "strings"
    )

    func main() {
        url := "https://jsonplaceholder.typicode.com/posts/1"
        resp, err := http.Get(url)
        if err != nil {
            fmt.Println("Error fetching data:", err)
            return
        }
        defer resp.Body.Close()

        body, err := ioutil.ReadAll(resp.Body)
        if err != nil {
            fmt.Println("Error reading response body:", err)
            return
        }

        content := string(body)
        fmt.Println("Fetched Content:", content)

        upperContent := strings.ToUpper(content)
        fmt.Println("Uppercase Content:", upperContent)

        words := strings.Fields(upperContent)
        wordCount := len(words)
        fmt.Println("Word Count:", wordCount)

        err = ioutil.WriteFile("output.txt", []byte(upperContent),0644)
        if err != nil {
            fmt.Println("Error writing to file:", err)
            return
        }
        fmt.Println("Content successfully written to file.")
    }

```

Now compare with the refactored modular one 

```go
    package main

    import (
        "fmt"
        "io/ioutil"
        "net/http"
        "strings"
    )

    func fetchContent(url string) (string, error) {
        resp, err := http.Get(url)
        if err != nil {
            return "", fmt.Errorf("error fetching data: %w", err)
        }
        defer resp.Body.Close()

        body, err := ioutil.ReadAll(resp.Body)
        if err != nil {
            return "", fmt.Errorf("error response body:%w",err)
        }

        return string(body), nil
    }

    func processContent(content string) string {
        return strings.ToUpper(content)
    }

    func countWords(content string) int {
        words := strings.Fields(content)
        return len(words)
    }

    func writeToFile(filename, content string) error {
        return ioutil.WriteFile(filename, []byte(content), 0644)
    }

    func main() {
        url := "https://jsonplaceholder.typicode.com/posts/1"

        content, err := fetchContent(url)
        if err != nil {
            fmt.Println(err)
            return
        }
        fmt.Println("Fetched Content:", content)

        upperContent := processContent(content)
        fmt.Println("Uppercase Content:", upperContent)

        wordCount := countWords(upperContent)
        fmt.Println("Word Count:", wordCount)

        if err := writeToFile("output.txt", upperContent); err != nil {
            fmt.Println("Error writing to file:", err)
            return
        }
        fmt.Println("Content successfully written to file.")
    }

```

As you can see made the program more simple and readable, the seperation is concise and also each function are reusable

## Some rules to follow about functions

- Keep blocks in if, else, and loops to one line, ideally a function call. This keeps functions small and improves readability. Functions should also avoid deep nesting, keeping indentation at one or two levels for clarity.
- One clear purpose: function should do only one thing and do it well (Single Responsibility Principle).
- Name the function exactly what its doing and also be consistent in naming
- Avoid deep nesting: If-else and loops should be extracted into separate functions when they get too complex.
- The best functions have zero arguments, followed by one or two. Avoid three or more
- Single-argument functions can represent events, altering system state (e.g., passwordAttemptFailedNtimes(int attempts)). Use them carefully with clear naming.

---

- Avoid flag arguments. Passing a boolean makes a function do multiple things, complicating readability. Instead, split it into separate functions for clarity  
example check code below  

`BAD CODE❌`
```go

package main

import "fmt"

    func render(isSuite bool) {
        if isSuite {
            fmt.Println("Rendering suite...")
        } else {
            fmt.Println("Rendering single test...")
        }
    }

    func main() {
        render(true) 
        render(false) 
    }
```

`GOOD CODE✅`
```go

    package main

    import "fmt"

    func renderSuite() {
        fmt.Println("Rendering suite...")
    }

    func renderSingleTest() {
        fmt.Println("Rendering single test...")
    }

    func main() {
        renderSuite()      
        renderSingleTest()
    }
```


- Have no Side-Effects

 `* Spot the Side Effect in this Code`
```go
    package main

    type UserValidator struct {
        cryptographer *Cryptographer
    }

    func (uv *UserValidator) CheckPassword(username, password string) bool {
        user := (&UserGateway{}).FindByName(username)
        if user != &UserNull {
            codedPhrase := user.GetPhraseEncodedByPassword()
            phrase := uv.cryptographer.Decrypt(codedPhrase, password)
            if phrase == "Valid Password" {
                session.Initialize() 
                return true
            }
        }
        return false
    }

```

notice how its hiding in there `session.Initialize()` instead this should be handled outside of the function

```go
    package main

    type UserValidator struct {
        cryptographer *Cryptographer
    }

    func (uv *UserValidator) CheckPassword(username, password string) (bool, error) {
        user := (&UserGateway{}).FindByName(username)
        if user == &UserNull {
            return false, nil
        }

        codedPhrase := user.GetPhraseEncodedByPassword()
        phrase := uv.cryptographer.Decrypt(codedPhrase, password)
        if phrase == "Valid Password" {
            return true, nil
        }
        return false, nil
    }

```


```go 

    package main

    import "fmt"

    func main() {
        validator := &UserValidator{cryptographer: &Cryptographer{}}
        isValid, err := validator.CheckPassword("john_doe", "password123")

        if err != nil {
            fmt.Println("Error:", err)
            return
        }

        if isValid {
            session.Initialize() 
            fmt.Println("Password valid:", isValid)
        } else {
            fmt.Println("Invalid password")
        }
    }
```

- A function should either perform an action or return information, but not both. Mixing them causes confusion. example of spliting the function is below 

```go

    package main

    import "fmt"

    func attributeExists(attribute string, data map[string]string) bool {
        _, exists := data[attribute]
        return exists
    }

    func setAttribute(attribute, value string, data map[string]string) {
        data[attribute] = value
    }

    func main() {
        userData := make(map[string]string)

        if attributeExists("username", userData) {
            setAttribute("username", "unclebob", userData)
            fmt.Println("Username set successfully.")
        } else {
            fmt.Println("Attribute does not exist, creating it.")
            setAttribute("username", "unclebob", userData)
        }

        fmt.Println("User Data:", userData)
    }

```

- Prefer Exceptions to returning error codes => error handling with explicit return values is common, but excessive error codes can lead to deeply nested code structures, making it harder to read and maintain.

`BAD COD❌`
```go
    package main

    import "fmt"

    const (
        E_OK    = 0
        E_ERROR = 1
    )

    func deletePage(page string) int {
        if page == "" {
            return E_ERROR
        }
        return E_OK
    }

    func main() {
        page := "home"

        if deletePage(page) == E_OK {
            fmt.Println("Page deleted successfully.")
        } else {
            fmt.Println("Error deleting page.")
        }
    }
```

`GOOD CODE✅`

```GO

    package main

    import (
        "errors"
        "fmt"
    )

    func deletePage(page string) error {
        if page == "" {
            return errors.New("page not found")
        }
        return nil
    }

    func main() {
        page := "home"

        if err := deletePage(page); err != nil {
            fmt.Println("Error:", err)
            return
        }

        fmt.Println("Page deleted successfully.")
    }
```

- Don’t Repeat Yourself (DRY): Avoid duplication in code to improve readability, maintainability, and reduce errors. Refactor repeated logic into reusable methods or structures. Many programming paradigms, like OOP and database normalization, aim to eliminate redundancy.

- Structured Programming: Functions should have a single entry and exit, avoiding multiple returns, break, continue, or goto. However, in small functions, multiple returns can improve readability, while goto should be avoided.
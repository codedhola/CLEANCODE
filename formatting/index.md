#### FORMATTING

Code should be neat, consistent, and well-formatted to reflect professionalism. A messy codebase suggests poor quality. Teams should agree on formatting rules and use automated tools to maintain consistency.

The Importance of Code Formatting: Formatting is essential for clear communication, not just a formality. While functionality may change, readability impacts long-term maintainability. A well-structured coding style ensures future developers can easily understand and modify the code.

###### SOME RULES FOR FORMATTING

 #### **Vertical Formatting in Code**
Vertical formatting refers to the organization of code within a file, ensuring it is readable and structured. A well-formatted file should be small and concise, ideally around 200–500 lines, as smaller files are easier to understand and maintain.
Projects should demonstrate that large systems can be built using small, well-structured files, while some have thousands of lines per file, making them harder to read. While there is no strict rule, keeping files short and focused improves clarity and maintainability.

- **The Newspaper Metaphor for Code**: => A well-structured source file should be like a newspaper article—easy to read top-down. The name should clearly indicate its purpose. The top should provide high-level concepts, while details increase as you go down, ending with low-level functions.
 Just as newspapers have concise articles, code should be modular and organized, avoiding long, disorganized files.

 - **Vertical Openness in Code**: => 
 Code is read top to bottom, and blank lines help separate distinct thoughts, improving readability. Each concept or function should be visually distinct with proper spacing.

<center>Bad code❌</center>  

 ```go
    package main
    import "fmt"
    func greet(){fmt.Println("Hello, World!")}    func add(a, b int) int{return a + b}
    func main(){fmt.Println(add(3, 4))}
 ```

<center>Good code✅</center>

 ```go
    package main

    import "fmt"

    func greet() {
        fmt.Println("Hello, World!")
    }

    func add(a, b int) int {
        return a + b
    }

    func main() {
        greet()
        fmt.Println("Sum:", add(3, 4))
    }
 ```

 - **Vertical Density in Code**: => 
 Vertical density means keeping closely related lines together to improve readability. Unnecessary spacing or comments break the flow, making code harder to follow.

  <center>Bad code❌</center>

  ```go
  package main

    import "fmt"

    // User represents a social media user

    type User struct {
        
        // Number of retweets
        retweets int
        
        // Number of likes
        likes int
    }


    func processUserActivity() {
        
        fmt.Println("Processing user activity...")
    }

    func main() {
        
        processUserActivity()
    }
```
 <center>Good code✅</center>

```go
    package main

    import "fmt"

    type User struct {
        retweets int
        likes    int
    }

    func processUserActivity() {
        fmt.Println("Processing user activity...")
    }

    func main() {
        processUserActivity()
    }
```

- **Dependent Functions in Vertical Order**: =>
Caller functions should be above the callee, keeping the flow natural. This helps readers follow dependencies without searching for function definitions.

```go
    package main

    import "fmt"

    func main() {
        handleUserActivity()
    }

    func handleUserActivity() {
        processRetweets()
        processLikes()
    }

    func processRetweets() {
        fmt.Println("Processing retweets...")
    }

    func processLikes() {
        fmt.Println("Processing likes...")
    }
```

- **Conceptual Affinity in Code**

Functions with similar purposes should be grouped together to improve readability. Even if they don’t call each other, their naming and functionality should keep them close.




 #### **Horizontal Formatting in Code**

 - **Horizontal Openness and Density**: => Horizontal whitespace groups related elements and separates distinct ones, improving readability.

 ```go
    package main

    import (
        "fmt"
        "math"
    )

    func measureLine(line string, lineCount *int, totalChars *int) {
        *lineCount++
        lineSize := len(line)
        *totalChars += lineSize
        recordWidestLine(lineSize)
    }

    func root1(a, b, c float64) float64 {
        return (-b + math.Sqrt(determinant(a, b, c))) / (2 * a)
    }

    func root2(a, b, c float64) float64 {
        return (-b - math.Sqrt(determinant(a, b, c))) / (2 * a)
    }

    func determinant(a, b, c float64) float64 {
        return b*b - 4*a*c
    }

    func recordWidestLine(size int) {
        fmt.Println("Tracking widest line of size:", size)
    }

    func main() {
        lineCount, totalChars := 0, 0
        measureLine("Hello, world!", &lineCount, &totalChars)
        fmt.Println("Roots:", root1(1, -3, 2), root2(1, -3, 2))
    }
```


- **Horizontal Alignment**: => Aligning variables and assignments was once common but hides details, making code harder to read. It encourages vertical scanning, ignoring types and relationships. Long aligned lists often signal a design issue—splitting the class is better than forced formatting. Unaligned, structured code is clearer and easier to maintain.

 <center>Bad code❌</center>

 ```go
    package main

    type Server struct {
        host           string
        port           int
        isSecure       bool
        requestsServed int
        maxConnections int
    }

    func NewServer(
        host string,
        port int, 
        isSecure bool
        ) *Server {
        return &Server{
            host:           host,
            port:           port,
            isSecure:       isSecure,
            requestsServed: 0,
            maxConnections: 100,
        }
    }
```

 <center>Good code✅</center>

 ```go
    package main

    type Server struct {
        host string
        port int
        isSecure bool
        requestsServed int
        maxConnections int
    }

    func NewServer(host string, port int, isSecure bool) *Server {
        return &Server{
            host: host,
            port: port,
            isSecure: isSecure,
            requestsServed: 0,
            maxConnections: 100,
        }
    }
 ```

- **Indentation**

Indentation visually represents a hierarchy in code, making scopes clear. Classes, methods, and blocks are indented progressively to reflect structure. Programmers rely on this to quickly navigate and understand code. Without proper indentation, code becomes unreadable.

 <center>Bad code❌</center>

 ```go
    package main
    import "fmt"
    func reverseString(s string)string{r:=[]rune(s)
    for i,j:=0,len(r)-1;i<j;i,j=i+1,j-1{r[i],r[j]=r[j],r[i]}
    return string(r)}
    func main(){fmt.Println(reverseString("hello"))}

```

<center>Good code✅</center>

```go
    package main

    import "fmt"

    func reverseString(word string) string {
        reverse := []rune(word)

        for i, j := 0, len(r)-1; i < j; i, j = i+1, j-1 {
            reverse[i], reverse[j] = reverse[j], reverse[i]
        }

        return string(r)
    }

    func main() {
        fmt.Println(reverseString("hello"))
    }
```

- **Team Rules**

In a team, consistency matters—the team, not individuals, decides formatting rules. Agreeing on braces, indentation, and naming conventions ensures a uniform codebase. Formatting should be encoded in the IDE to maintain consistency across files, making the code easier to read and understand. Avoid a mix of styles that adds unnecessary complexity.





#### COMMENTS

##### `DON'T COMMENT BAD CODE, REWRITE IT`

Comments Are a Necessary Evil: Helpful comments clarify code, but excessive or outdated ones create confusion. Ideally, code should be self-explanatory, reducing the need for comments. Misleading comments are worse than none—truth lies in the code itself. Prioritize writing clear, expressive code over relying on comments.

###### SOME EXAMPLE PLACES WHERE COMMENTS CAN BE USEFUL ARE THUS
- Legal Comments: Some coding standards require legal comments like copyright and license statements at the start of source files. These are necessary but can be hidden by IDEs to avoid clutter.

- Informative Comments: Comments can clarify return values or patterns but should be avoided if function names or code structure can convey the same meaning. Instead of relying on comments, prefer meaningful names or refactor code for clarity.

- Clarification Comments: Comments can help explain obscure arguments or return values, especially in code you can’t modify. However, they risk being incorrect. Always prioritize clear code over comments, and if comments are necessary, ensure they are accurate.

- Warning Comments: Comments can alert programmers to potential issues, like long-running tests or non-thread-safe code. While better solutions may exist, a well-placed warning can prevent mistakes and unnecessary optimizations.

- TODO Comments: TODOs remind developers of pending tasks but shouldn’t justify leaving bad code. Modern IDEs track them, but they should be reviewed and removed regularly to keep the code clean.


###### BAD COMMENTS

- Mumbling => A comment should clearly explain intent, not leave ambiguity. Here’s the bad example with a vague comment:

```go
package main

import (
	"fmt"
	"os"
)

// ❌: unclear who loads defaults and when
    func loadProperties() {
        propertiesPath := "config.properties"
        file, err := os.Open(propertiesPath)
        if err != nil {
            // No properties file means all defaults are loaded
            return
        }
        defer file.Close()

        // Load properties from file...
        fmt.Println("Properties loaded from file")
    }

```

- Misleading Comments: A poorly written comment can create false expectations. If it’s unclear or inaccurate, it misguides developers, leading to confusion and debugging issues. Ensure comments are precise and easier to understand than the code itself.

- Noise Comments: Comments that restate the obvious add no value and should be avoided. Instead of writing redundant explanations, let clear code and meaningful names speak for themselves. Unnecessary comments create clutter, get ignored, and may become misleading over time.

```go 
    package main

    import "fmt"

    // Default constructor (Unnecessary)
    type User struct{}

    // The user's name (Obvious)
    var name string

    // Returns the user's name (Redundant)
    func getName() string {
        return name
    }

    // Handles errors (Unhelpful comment)
    func startSending() {
        defer func() {
            if r := recover(); r != nil {
                // Give me a break!
                fmt.Println("Recovered from error:", r)
            }
        }()
        panic("Something went wrong")
    }

    func main() {
        fmt.Println(getName())
        startSending()
    }
```

- Commented-out code clutters the codebase, making it harder to maintain. Instead of leaving it in, use version control (Git) to track old changes and delete unnecessary code.
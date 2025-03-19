#### CONCURRENCY

Concurrency introduces significant complexity to software, primarily because it involves multiple threads or processes executing simultaneously, potentially leading to issues like race conditions, deadlocks, and resource contention. Clean concurrent code requires careful design, discipline, and an understanding of both the benefits and pitfalls of parallelism.

#### `Notes`
- Concurrency vs. Parallelism: 
    - Concurrency refers to the composition of independently executing tasks that may or may not run at the same time.
    - Parallelism refers to tasks that actually run simultaneously, often on multiple processors.
-  Shared state in concurrent programs can lead to complex bugs, race conditions, and other problems. The general advice is to minimize shared state, as it increases complexity and the potential for errors.

- One approach to simplifying concurrency is to use immutable objects, which cannot be modified after creation. This removes the need for synchronization, as immutable data cannot be changed by multiple threads concurrently.

- Writing thread-safe code is crucial in concurrent programming. Thread-safe operations allow multiple threads to access shared data without causing unexpected behaviors or corrupting the state.

- Synchronization (locks, semaphores) is often necessary for ensuring thread safety but should be used sparingly to avoid unnecessary complexity and performance hits.

- When threads need to communicate, it’s better to use message passing (via channels in languages like Go) instead of shared state. This keeps the components decoupled and avoids the pitfalls of direct state manipulation.

Execusion Models
---

<table border="1">
  <thead>
    <tr>
      <th>Type</th>
      <th>Explanation</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Bound Resources</td>
      <td>Resources with a fixed size or limited quantity used in a concurrent environment. Examples include database connections and fixed-size read/write buffers.</td>
    </tr>
    <tr>
      <td>Mutual Exclusion</td>
      <td>Only one thread can access shared data or a shared resource at a time.</td>
    </tr>
    <tr>
      <td>Starvation</td>
      <td>A thread or group of threads is prevented from progressing for an extended period or indefinitely. For instance, if fast-running threads are always prioritized, slower-running threads may be starved, as there is no limit to how long fast-running threads can continue.</td>
    </tr>
    <tr>
      <td>Deadlock</td>
      <td>Two or more threads are waiting on each other to complete. Each thread holds a resource that the other thread needs, and neither can proceed until it acquires the resource held by the other.</td>
    </tr>
    <tr>
      <td>Livelock</td>
      <td>Threads operating in lockstep, each attempting to perform work but being blocked by the other. Due to this synchronization conflict, the threads continue trying to make progress but are unable to for an extended period, or indefinitely.</td>
    </tr>
  </tbody>
</table>


#### `Key Principles`

###### `Concurrency is Hard`:  
Writing correct concurrent code is inherently difficult due to shared state, timing issues, and non-deterministic behavior. Developers must prioritize clarity and correctness over premature optimization.

###### `Single Responsibility Principle (SRP) in Concurrency`:  
Keep concurrent code separate from non-concurrent code. Encapsulate concurrency-related logic in small, focused units to reduce complexity.

###### `Limit Data Scope`:   
Minimize shared data between threads to avoid race conditions. Use synchronization mechanisms (e.g., locks) judiciously and only where necessary.

###### `Avoid Over-Synchronization`:  
Excessive use of locks can lead to deadlocks or performance bottlenecks. Design systems to minimize contention.

###### `Test Thoroughly`: 
Concurrent code must be tested under various conditions, as bugs may only surface under specific timing scenarios.

###### `Understand Your Tools`: 
Know the concurrency primitives provided by your language or platform (e.g., threads, locks, semaphores) and use them appropriately.

###### `Use Goroutines and Channels`:  
Leverage Go’s built-in concurrency model with goroutines for asynchronous processing and channels for safe communication between threads.



###### `Common Issues`
 - Race Conditions: Occur when multiple threads access shared data without proper synchronization, leading to unpredictable outcomes.
 - Deadlocks: Happen when threads wait indefinitely for resources held by each other.
 - Starvation: A thread is perpetually denied access to a resource it needs.

###### `Practical Advice`
 - Use high-level concurrency constructs (e.g., thread pools, actors, or message passing) when possible, rather than low-level locks.
 - Keep critical sections small and fast to reduce contention.
 - Design for simplicity: Avoid concurrency unless it provides a clear benefit (e.g., performance or responsiveness).


race condition example 

Bad Code ❌
```go

    package main

    import (
        "fmt"
        "sync"
    )

    func main() {
        var counter int
        var wg sync.WaitGroup

        // Launch 100 goroutines that increment the counter
        for i := 0; i < 100; i++ {
            wg.Add(1)
            go func() {
                defer wg.Done()
                counter++ // Race condition: no synchronization
            }()
        }

        wg.Wait()
        fmt.Println("Counter:", counter) // Output varies (e.g., 87, 92, 99)
    }
```
Fix: Use a mutex to protect the shared variable.

Good code ✅ 
```go
    package main

    import (
        "fmt"
        "sync"
    )

    func main() {
        var counter int
        var wg sync.WaitGroup
        var mu sync.Mutex

        for i := 0; i < 100; i++ {
            wg.Add(1)
            go func() {
                defer wg.Done()
                mu.Lock()
                counter++ // Synchronized access
                mu.Unlock()
            }()
        }

        wg.Wait()
        fmt.Println("Counter:", counter) // Always 100
    }
```

Encapsulating Concurrency (SRP)

```go
    package main

    import (
        "fmt"
        "time"
    )

    func worker(id int, jobs <-chan int, results chan<- int) {
        for job := range jobs {
            time.Sleep(100 * time.Millisecond) // Simulate work
            results <- job * 2
        }
    }

    func main() {
        jobs := make(chan int, 10)
        results := make(chan int, 10)

        // Start 3 workers
        for w := 1; w <= 3; w++ {
            go worker(w, jobs, results)
        }

        // Send jobs
        for j := 1; j <= 5; j++ {
            jobs <- j
        }
        close(jobs)

        // Collect results
        for r := 1; r <= 5; r++ {
            fmt.Println("Result:", <-results)
        }
    }
```
This example isolates concurrency (goroutines and channels) from the core logic (doubling a number), adhering to SRP.

Deadlock Example

Bad Code ❌ 

```go

    package main

    import (
        "fmt"
        "sync"
        "time"
    )

    func main() {
        var mu1, mu2 sync.Mutex
        var wg sync.WaitGroup

        wg.Add(2)

        go func() {
            defer wg.Done()
            mu1.Lock()                        // Locks mu1 first
            defer mu1.Unlock()
            time.Sleep(10 * time.Millisecond) // Simulate work
            mu2.Lock()                        // Then tries to lock mu2
            defer mu2.Unlock()
        }()

        go func() {
            defer wg.Done()
            mu2.Lock()                        // Locks mu2 first
            defer mu2.Unlock()
            time.Sleep(10 * time.Millisecond) // Simulate work
            mu1.Lock()                        // Then tries to lock mu1
            defer mu1.Unlock()
        }()

        wg.Wait() // Deadlock: program hangs
        fmt.Println("Done")
    }
```
`Problem`:
- Goroutine 1 locks mu1 and waits for mu2.
- Goroutine 2 locks mu2 and waits for mu1.
- This circular wait causes a deadlock, and the program hangs.


Fix: Avoid circular dependencies by locking resources in a consistent order.

Good Code ✅ 

```go
    package main

    import (
        "fmt"
        "sync"
        "time"
    )

    func main() {
        var mu1, mu2 sync.Mutex
        var wg sync.WaitGroup

        wg.Add(2)

        go func() {
            defer wg.Done()
            mu1.Lock()                        // Locks mu1 first
            defer mu1.Unlock()
            time.Sleep(10 * time.Millisecond) // Simulate work
            mu2.Lock()                        // Then locks mu2
            defer mu2.Unlock()
            fmt.Println("Goroutine 1 completed")
        }()

        go func() {
            defer wg.Done()
            mu1.Lock()                        // Locks mu1 first (same order)
            defer mu1.Unlock()
            time.Sleep(10 * time.Millisecond) // Simulate work
            mu2.Lock()                        // Then locks mu2
            defer mu2.Unlock()
            fmt.Println("Goroutine 2 completed")
        }()

        wg.Wait()
        fmt.Println("Done")
    }
```
`Fix`:
- Both goroutines lock mu1 first, then mu2.
- This consistent order eliminates the circular wait, ensuring one goroutine completes its critical section before the other proceeds.

`Key Difference`
The only structural difference is in the second goroutine:

- > Bad Code: Locks mu2 first, then mu1.
- > Good Code: Locks mu1 first, then mu2, matching the order of the first goroutine.



Clean Code stresses that concurrency should be approached with caution and discipline. In Go, tools like goroutines, channels, and mutexes make it easier to write clean concurrent code, but developers must still design systems to minimize shared state, avoid over-complication, and test rigorously. By following these principles, you can achieve concurrency that is both efficient and maintainable.
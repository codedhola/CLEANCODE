#### MEANINGFUL NAMES

Names are everywhere in software. We name our variables, our functions, our arguments, classes, and packages. We name our source files and the directories that contain them. We name our jar files and war files and ear files. We name and name and name. Because we do so much of it, we’d better do it well. What follows are some simple rules for creating
good names.

###### Always use intention revealing names.

for example 

```go
    // BAD CODE
    var d = 3.14 // pie
    var xy = 365 // Days in year
```

```go
    // GOOD CODE
    var pie = 3.14
    var numOfDaysInYear = 365
```

\* note that the intent of the code is clear, the declared constant is concise of what is going on

#### Avoid Disinformation
Avoid leaving false clues that obscure the meaning of code. We should avoid words whose entrenched meanings vary from our intended meaning.  

Do not refer to a grouping of accounts as an accountList unless it’s actually a List. The word list means something specific to programmers. If the container holding the accounts is not actually a List, it may lead to false conclusions. So accountGroup or bunchOfAccounts or just plain accounts would be better.  
Spelling similar concepts similarly is information. Using inconsistent spellings is dis- information.

Avoid "noise words" usage when naming in other to make meaningful distinctions. for example look at the method below, how would a different author know what to call

```
    getActiveAccount();
    getActiveAccounts();
    getActiveAccountInfo();
```

In the absence of specific conventions, the variable moneyAmount is indistinguishable from money, customerInfo is indistinguishable from customer, accountData is indistinguish- able from account, and theMessage is indistinguishable from message. Distinguish names in such a way that the reader knows what the differences offer.  
Make use of long names and searchable ones if you know they are going to be called mostly in other files E.G. `MAX_CLASSES_PER_STUDENT` is better than `MAX_C_P_S`

**Class Names**: Classes and objects should have noun or noun phrase names like Customer, WikiPage, Account, and AddressParser. Avoid words like Manager, Processor, Data, or Info in the name of a class. A class name should not be a verb.

**Method Names**: Methods should have verb or verb phrase names like postPayment, deletePage, or save. Accessors, mutators, and predicates should be named for their value and prefixed with get, set, and is according to their standard

```go
type User struct {
    Name string
}

func (user User) GetName() string {
    return user.Name
}

func main() {
    user := User{Name: "Coded Hola"}
    var userName = user.GetName()
}
```

#### Dont be cute
This is being too clever for example what is `HolyHandGrenade` is supposed to do? Sure, it’s cute, but maybe in this case `DeleteItems` might be a better name. Choose clarity over entertainment value.

#### Pick one word per concept
Pick one word for one abstract concept and stick with it. For instance, it’s confusing to have fetch, retrieve, and get as equivalent methods of different classes. A consistent lexicon is a great boon to the programmers who must use your code.

#### Dont Pun  
Avoid using the same word for two purposes. Using the same term for two different ideas is essentially a pun. Now let’s say we are writing a new class that has a method that puts its single parameter into a collection. Should we call this method `add`? It might seem consistent because we have so many other add methods, but in this case, the semantics are different, so we should use a name like `insert` or `append` instead. To call the new method add would be a pun

#### Use Solution Domain Names  
Remember that the people who read your code will be programmers. So go ahead and use computer science (CS) terms, algorithm names, pattern names, math terms, and so forth.

#### Use Problem Domain Names  
Use problem domain names when no clear programming term exists for clarity.
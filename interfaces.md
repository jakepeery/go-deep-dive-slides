# Core - Interfaces
---
### What is an Interface type is Go
**Tour of Go Interface Definition:  An interface type is defined as a set of method signatures.**
  - ☢️ In my opinion, Tour of Go does a poor job describing what an interface is and when and how to use them.

**The Interface Type - A Better Definition**

Interface
: An Interface is a type that provides a way to specify the behavior of an object. We use interfaces to create common abstractions that multiple objects can implement. -100 Go Mistakes

  - Interfaces are ...
    - like a contract or a set of expectations applied to a type - enrolled (an object or type can enroll an interface)
    ```
    "If your type can do these things, then it can be used whereever this interface is expected"
    ```
    It doesn't care how your type does those things, just as long as it satisfies the methods on the interface. 
    - helpful in building reusable code across your application
    - Implicit - (No declaration statements like X implements y) - as long as it satisfies the methods, it is considered a type from that interface      


Interface has a two roles: 
1. Interface is a type (contract) - It's statically defined usually at the top of documents along with structs.  When an interface type is defined, 
```go
type Speaker interface {
    Speak() string
}
```
So any type that has a method Speak that returns the value of string is a speaker.

2. Interface as a value - A concrete value can be assigned to an interface type, it becomes a dynamic value that carries both the value and it's original type.
```go
interface value = (concrete type, concrete value)
```
So you can store any type of data in an interface variable which gives extra flexibility but can lead to type issues as it bypasses the static type nature of Go.
```go
var man interface{}
man = "Hello World!"
```
---
### Example of an interface:

##### Gobyexample code - 
```go

package main

import(
    fmt 
)

type Speaker interface {
    Speak() string
}

type Dog struct{}
type Robot struct{}
type Human struct{}


func (d Dog) Speak() string {
    return "Woof!"
}

func (r Robot) Speak() string {
    return "Beep boop"
}

func (h Human) Speak() string {
    return "Hello World!"
}

func MakeItTalk(s Speaker) {
    fmt.Println(s.Speak())
}

func main() {
    d := Dog{}
    r := Robot{}
    h := Human{}

    MakeItTalk(d)
    MakeItTalk(r)
    MakeItTalk(h)
}
```

**When to use interface type (contract)**
  - Common Behavior
    - Use an interface when multiple types share common behavior, allowing you to define a set of methods that different structs can implement. This enables polymorphism, making it easier to write generic and reusable code.
  - Decoupling
    - Interfaces help decouple code by defining dependencies in terms of behavior rather than concrete types. This improves modularity and testability, as components can be swapped without changing the surrounding logic.
  - Restricting behavior
    - Interfaces can restrict behavior by exposing only the necessary methods, even if the underlying type has many more. This enforces encapsulation and ensures consumers interact with a limited and well-defined API.

**When not to use interfaces**
  - When there's only one implementation - (Also don't use it prematurely)
    - If you only have one use case, don't add abstraction by using an interface.  Even if you think that maybe down the road you might have more use cases. Note that in comments but don't implement. It clusters code and makes it much more difficult to follow. 
  - When it hides meaningful behavior
    - Interfaces can obscure functionality of different types if consumers only see a limited method set. This can lead to confusion or inefficiency when developers need access to capabilities that the interface doesn’t expose.
  - When performance is critical
    - Interfaces involve dynamic dispatch, which adds a small runtime cost. In tight loops or performance-critical paths, concrete types and function calls may be more efficient. In order to find the concrete type that the interface is pointing to, the compiler will look for that type using a hash table which adds a little bit more time to the call. 

---

## Interface Pollution and Composition

```
The bigger the interface, the weaker the abstraction. —Rob Pike
```

"...interfaces are made to create abstractions. And the main caveat when programming meets abstractions is remembering that abstractions should be discovered, not created. What does this mean? It means we shouldn’t start creating abstractions in our code if there is no immediate reason to do so."  
— Teiva Harsanyi, 100 Go Mistakes and How to Avoid Them

Interface pollution occurs when an interface grows too large, accumulating many methods. This makes it harder for types to implement the interface, especially if they don't need all the methods. Large interfaces reduce flexibility and weaken abstraction.

### How to avoid interface pollution? - Use interface composition. ###

Interface composition is the practice of building new interfaces by combining smaller, focused interfaces. This encourages the creation of minimal, reusable abstractions and helps keep your code modular and maintainable.

In Go, this is sometimes called creating an "interface of interfaces." Instead of listing all required methods again, you embed existing interfaces into a new interface. The new interface then requires all the methods of the embedded interfaces, so any type that implements all those methods satisfies the composed interface.

This approach allows you to build more complex abstractions from simple, reusable pieces. It's widely used in Go's standard library (for example, `io.ReadWriter` combines `io.Reader` and `io.Writer`). By composing interfaces, you avoid duplication and keep your interfaces focused and easy to implement.

**Key points:**
- Interface composition lets you combine behaviors from multiple interfaces.
- The resulting interface is satisfied by any type that implements all embedded interfaces' methods.
- This keeps interfaces small, focused, and reusable, and helps prevent interface pollution.

For example:
```go
type Speaker interface {
    Speak() string
}

type LegCount interface {
    LegNum() int
}

// Animal combines both Speaker and LegCount
type Animal interface {
    Speaker
    LegCount
}
```
Any type that implements both `Speak()` and `LegNum()` will satisfy the `Animal` interface.

**Benefits of interface composition:**
- Encourages small, focused interfaces (the "interface segregation principle")
- Makes code more modular and reusable
- Allows you to extend interfaces as your application grows

**Use with caution:**
- Over-composing interfaces can make code harder to understand
- Document composed interfaces clearly to avoid confusion

_Abstractions should be discovered, not created. Only compose interfaces when there is a clear need._

---

**Empty interfaces – the `any` type**

One common practice among new Go programmers is to use empty interfaces (`interface{}`), since they can hold values of any type. However, this bypasses one of Go's main benefits: static typing.

Any data type can be assigned to an empty interface. Starting in Go 1.18, the `any` type was introduced as an alias for `interface{}`.

Example:
```go
var mydata interface{}
var anydata any

mydata = 42
mydata = "Cosmo rocks!"
mydata = SillyStruct{}

anydata = 5
anydata = "Still Working"
```

Using empty interfaces obscures the type information of the stored value. To access the underlying value, you must use a type assertion or type switch. Think of it as a "mystery box"—you need to open it to see what's inside.

Empty interfaces are useful when you don't know the type of data in advance, such as when handling data from APIs or external sources.

#### Common use cases for interface variables
> * Handling arbitrary input from sources you don't control and whose structure you don't know ahead of time (e.g., APIs)
> * Generic collections of data in slices or maps that hold multiple types (not recommended in most cases)
> * Communicating with third-party libraries—some libraries accept or return `interface{}` to provide flexibility (e.g., database drivers)
> * Writing middleware or plugins—empty interfaces let you plug in any handler, callback, or data type, especially when defining your own type system or registry

#### Tips for using interfaces
> * Use them sparingly—keep interfaces small and focused (single responsibility)
> * Interfaces are implicit—track usage and methods carefully
> * Use comments to document where types satisfy interfaces, since this is not explicit in Go
> * Interface variables are powerful for passing data without knowing the type, but use with caution to avoid losing type safety

_see [Tour of Go](https://go.dev/tour/methods/9)_
_see [100 Go Mistakes and How to Avoid Them](https://learning.oreilly.com/library/view/100-go-mistakes/9781617299599/OEBPS/Text/02.htm#heading_id_14)_
_see [Go (Golang) Tutorial #22 - Interfaces](https://www.youtube.com/watch?v=lbW-KVdIXaY)_
_see [Go by Example: Interfaces](https://gobyexample.com/interfaces)_
_see [Golang: The Last Interface Explanation You'll Ever Need](https://www.youtube.com/watch?v=SX1gT5A9H-U&t=738s)_
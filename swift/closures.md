# Swift - Closures<!-- omit in toc -->

[Swift Documentation](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/closures)

- [Closure Expressions](#closure-expressions)
- [Trailing Closures](#trailing-closures)
- [Capturing Values](#capturing-values)
  - [Nested Functions](#nested-functions)
- [Closure are Reference Types](#closure-are-reference-types)
- [Escaping Closures](#escaping-closures)
- [Delaying Execution with a Closure](#delaying-execution-with-a-closure)
- [Autoclosures](#autoclosures)

*Closures* are self-contained blocks of functionality that can be passed around and used in your code. Closures can capture and store references to any constants and variables from the context in which they are defined. This is known as *closing over* those constants and variables. Swift handles all of the memory management of capturing for you.

Closures take one of three forms:

- *Global functions* are closures that have a name and do not capture any values.
- *Nested functions* are closures that have a name and can capture values from their enclosing function.
- *Closure expressions* are unnamed closures written in a lightweight syntax that can capture values from their surrounding context.

More about closures:

- [What is a closure?](https://www.hackingwithswift.com/example-code/language/what-is-a-closure)
- [What is a closure?](https://stackoverflow.com/questions/36636/what-is-a-closure)
  - > A closure is a persistent scope which holds on to local variables even after the code execution has moved out of that block. Languages which support closure (such as JavaScript, Swift, and Ruby) will allow you to keep a reference to a scope (including its parent scopes), even after the block in which those variables were declared has finished executing, provided you keep a reference to that block or function somewhere.

## Closure Expressions

Closure expressions are a way to write inline closures in a brief, focused syntax. Closure expression syntax has the following general form:

```swift
{ (parameters) -> return type in
    statements
}
```

The parameters in closure expression syntax can be in-out parameters, but they can’t have a default value. Variadic parameters can be used if you name the variadic parameter. Tuples can also be used as parameter types and return types.

Swift’s closure expressions have a clean, clear style, with optimizations that encourage brief, clutter-free syntax in common scenarios. These optimizations include:

- Inferring parameter and return value types from context
  - It is always possible to infer the parameter types and return type when passing a closure to a function as an inline closure expression. As a result, you never need to write an inline closure in its fullest form when the closure is used as a function argument.
- Implicit returns from single-expression closures
- Shorthand argument names
  - The shorthand argument names `$0`, `$1`, `$2`, and so on.
  - The highest numbered shorthand argument used determines the number of arguments that the closure takes.
- Trailling closure syntax

```swift
// Function as a closure
let names = ["Chris", "Alex", "Ewa", "Barry", "Daniella"]
func backward(_ s1: String, _ s2: String) -> Bool {
    return s1 > s2
}
var reversedNames = names.sorted(by: backward)
// reversedNames is equal to ["Ewa", "Daniella", "Chris", "Barry", "Alex"]

// Closure expression syntax
reversedNames = names.sorted(by: { (s1: String, s2: String) -> Bool in
    return s1 > s2
})

// Inferring type from context
reversedNames = names.sorted(by: { s1, s2 in return s1 > s2 } )

// Implicit returns from single-expression closures
reversedNames = names.sorted(by: { s1, s2 in s1 > s2 } )

// Shorthand argument names
reversedNames = names.sorted(by: { $0 > $1 } )

// Operator methods
reversedNames = names.sorted(by: >)
```

## Trailing Closures

If you need to pass a closure expression to a function as the function’s *final* argument, it can be useful to write the closure expression after the function call’s parentheses. When you use this closure syntax, you don’t write the argument label for the closure as part of the function call.

If a closure expression is provided as the function or method’s only argument and you provide that expression as a trailing closure, you do not need to write a pair of parentheses `()` after the function or method’s name when you call the function.

If a function takes multiple closures, you omit the argument label for the first trailing closure and label the remaining trailing closures.

```swift
func someFunctionThatTakesAClosure(closure: () -> Void) {
    // function body goes here
}

// Here's how you call this function without using a trailing closure:
someFunctionThatTakesAClosure(closure: {
    // closure's body goes here
})

// Here's how you call this function with a trailing closure instead:
someFunctionThatTakesAClosure() {
    // trailing closure's body goes here
}

// trailing closure syntax for the sorted(by:) method
reversedNames = names.sorted() { $0 > $1 }

// ignore the parentheses
reversedNames = names.sorted { $0 > $1 }

// multiple trailing closures
func loadPicture(from server: Server, completion: (Picture) -> Void, onFailure: () -> Void) {
    if let picture = download("photo.jpg", from: server) {
        completion(picture)
    } else {
        onFailure()
    }
}

loadPicture(from: someServer) { picture in
    someView.currentPicture = picture
} onFailure: {
    print("Couldn't download the next picture.")
}
```

## Capturing Values

A closure can capture constants and variables from the surrounding context in which it is defined. The closure can then *refer to* and *modify* the values of those constants and variables from within its body, even if the original scope that defined the constants and variables no longer exists.

### Nested Functions

Nested functions are closures that have a name and can capture values from their enclosing function.

```swift
func makeIncrementer(forIncrement amount: Int) -> () -> Int {
    var runningTotal = 0
    func incrementer() -> Int {
        runningTotal += amount
        return runningTotal
    }
    return incrementer
}

let incrementByTen = makeIncrementer(forIncrement: 10)
incrementByTen() // returns a value of 10

let incrementBySeven = makeIncrementer(forIncrement: 7)
incrementBySeven() // returns a value of 7
```

## Closure are Reference Types

In Swift, functions and closures are reference types. Whenever you assign a function or a closure to a constant or a variable, you are actually setting that constant or variable to be a reference to the function or closure.

```swift
let alsoIncrementByTen = incrementByTen
alsoIncrementByTen() // returns a value of 20

incrementByTen() // returns a value of 30
```

## Escaping Closures

A closure is said to *escape* a function when the closure is passed as an argument to the function, but is called after the function returns. When you declare a function that takes a closure as one of its parameters, you can write `@escaping` before the parameter’s type to indicate that the closure is allowed to escape.

One way that a closure can escape is by being stored in a variable that is defined outside the function.

```swift
var completionHandlers: [() -> Void] = []
func someFunctionWithEscapingClosure(completionHandler: @escaping () -> Void) {
    completionHandlers.append(completionHandler)
}
```

An escaping closure that refers to `self` needs special consideration if `self` refers to an instance of a class.

- Capturing `self` in an escaping closure means that you are capturing a strong reference to `self`, so you must take care to avoid a *strong reference cycle*.
- You can write `self` explicitly when you reference properties or methods of `self` in an escaping closure to confirm that there isn't a strong reference cycle.
- You can also use a *capture list* to break the strong reference cycle.
- If `self` is an instance of a structure or an enumeration, you can always refer to `self` implicitly. Because structures and enumerations are value types, a strong reference cycle is not possible.
  - However,  an escaping closure can't capture a mutable reference to `self` when `self` is an instance of a structure or an enumeration. Structures and enumerations don't allow shared mutability, as discussed in [Structures and Enumerations Are Value Types](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/classesandstructures#Structures-and-Enumerations-Are-Value-Types).

```swift
func someFunctionWithNonescapingClosure(closure: () -> Void) {
    closure()
}
class SomeClass {
    var x = 10
    func doSomething() {
        someFunctionWithEscapingClosure { self.x = 100 } // explicit self
        someFunctionWithNonescapingClosure { x = 200 }   // implicit self
    }
}
let instance = SomeClass()
instance.doSomething()
print(instance.x)
// Prints "200"
completionHandlers.first?()
print(instance.x)
// Prints "100"

// capture list
class SomeOtherClass {
    var x = 10
    func doSomething() {
        someFunctionWithEscapingClosure { [self] in x = 100 } // capture list
        someFunctionWithNonescapingClosure { x = 200 }
    }
}

// no mutable reference to structure or enumeration
struct SomeStruct {
    var x = 10
    mutating func doSomething() {
        someFunctionWithNonescapingClosure { x = 200 } // OK
        someFunctionWithEscapingClosure { x = 100 }    // Error
    }
}
```

## Delaying Execution with a Closure

**A closure lets you delay evaluation**, because the code inside isn’t run until you call the closure. Delaying evaluation is useful for code that has side effects or is computationally expensive.

```swift
var customersInLine = ["Chris", "Alex", "Ewa", "Barry", "Daniella"]
print(customersInLine.count)
// Prints "5"

let customerProvider = { customersInLine.remove(at: 0) } // closure with type () -> String
print(customersInLine.count)
// Prints "5"

print("Now serving \(customerProvider())!")
// Prints "Now serving Chris!"
print(customersInLine.count)
// Prints "4"
```

## Autoclosures

An *autoclosure* is a closure that is automatically created to wrap an *expression* that’s being passed as an argument to a function. It doesn’t take any arguments, and when it’s called, it returns the value of the expression that’s wrapped inside of it.

You can write a function as taking an autoclosure by writing the `@autoclosure` attribute before the parameter’s type. If you want an autoclosure that is allowed to escape, use both the `@autoclosure` and `@escaping` attributes.

It's common to call functions that take autoclosures, but it's not common to implement that kind of function. For example, the `assert(condition:message:file:line:)` function takes an autoclosure for its condition and message parameters; its condition parameter is evaluated only in debug builds and its message parameter is evaluated only if condition is false.

More about autoclosures:

- [autoclosure](https://developer.apple.com/forums/thread/47169)

```swift
// normal closure
var customersInLine = ["Chris", "Alex", "Ewa", "Barry", "Daniella"]
func serve(customer customerProvider: () -> String) {
    print("Now serving \(customerProvider())!")
}
serve(customer: { customersInLine.remove(at: 0) } )
// Prints "Now serving Chris!"

// autoclosure
func serve(customer customerProvider: @autoclosure () -> String) {
    print("Now serving \(customerProvider())!")
}
serve(customer: customersInLine.remove(at: 0)) // no braces
// Prints "Now serving Alex!"

// autoclosure with escaping
var customerProviders: [() -> String] = []
func collectCustomerProviders(_ customerProvider: @autoclosure @escaping () -> String) {
    customerProviders.append(customerProvider)
}
collectCustomerProviders(customersInLine.remove(at: 0))
collectCustomerProviders(customersInLine.remove(at: 0))
print("Collected \(customerProviders.count) closures.")
// Prints "Collected 2 closures."
for customerProvider in customerProviders {
    print("Now serving \(customerProvider())!")
}
// Prints "Now serving Ewa!"
// Prints "Now serving Barry!"
```

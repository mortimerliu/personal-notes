# Error Handling<!-- omit from toc -->

- [Representing and Throwing Errors](#representing-and-throwing-errors)
- [Handling Errors](#handling-errors)
  - [Propagating Errors Using Throwing Functions](#propagating-errors-using-throwing-functions)
  - [Handling Errors Using Do-Catch](#handling-errors-using-do-catch)
  - [Converting Errors to Optional Values](#converting-errors-to-optional-values)
  - [Disabling Error Propagation](#disabling-error-propagation)
- [Specifying Cleanup Actions](#specifying-cleanup-actions)

## Representing and Throwing Errors

- Errors are represented by values of types that conform to the `Error` protocol, usually enumerations
- `Error` is an empty protocol to indicate that a type can be used for error handling
- Use `throw` to throw an error

```swift
enum VendingMachineError: Error {
    case invalidSelection
    case insufficientFunds(coinsNeeded: Int)
    case outOfStock
}

throw VendingMachineError.insufficientFunds(coinsNeeded: 5)
```

## Handling Errors

Four ways to handle errors in Swift:

- Propagate the error from a function to the code that calls that function
- Handle the error using a `do-catch` statement
- Handle the error as an optional value
- Assert that the error will not occur

To identify places where errors can be thrown, use the `try`, `try?`, or `try!` keywords (immediately preceded to the throwing function call)

> Error handling in Swift doesn’t involve unwinding the call stack, a process that can be computationally expensive. As such, the performance characteristics of a throw statement are comparable to those of a return statement.

### Propagating Errors Using Throwing Functions

- Use `throws` keyword to indicate that a function, method, or initializer can throw an error (*throwing function*)
  - Any errors thrown inside a nonthrowing function must be handled inside the function
- The caller of a throwing function must handle errors using one of the four ways
- If an error is propagated to the top-level scope without being handled, an error is thrown at runtime

```swift
func canThrowErrors() throws -> String

// propagate the error with simply calling the throwing function using `try`
func caller() throws { 
    let x = try canThrowErrors()
}
```

### Handling Errors Using Do-Catch

- Use a `do-catch` statement to handle errors by running a block of code
- If an error is not caught by any `catch` clause, the error propagates to the surrounding scope
- More information about pattern matching in [here](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/patterns)

```swift
do {
    try <#expression#>
    <#statement#>
} catch <#pattern 1#> {
    <#statements#>
} catch <#pattern 2#> where <#condition#> {
    <#statements#>
} catch <#pattern 3#>, <#pattern 4#> where <#condition#> {
    <#statements#>
} catch {
    <#statements#> // catch all errors
}

// example
do {
    try vendingMachine.vend(itemNamed: "Candy Bar")
} catch VendinMachineError.invalidSelection {
    print("Invalid Selection.")
} catch is VendingMachineError {
    print("Some Vending Machine Error.")
}
```

### Converting Errors to Optional Values

- use `try?` to convert the result to an optional value

### Disabling Error Propagation

- use `try!` to disable error propagation and wrap the call in a runtime assertion that no error will be thrown
- only use `try!` when you’re sure that an error won’t be thrown; when an error is thrown, the program crashes

## Specifying Cleanup Actions

- use `defer` to write a block of code that is executed **after** all other code in the function, just **before** code execution leaves the current block of code, either by `return` or by throwing an error
- the `defer` block may not contain any code that would transfer control out of the block, such as a `break` or a `return` statement, or by throwing an error
- Deferred actions are executed in reverse order of how they are written in source code (first `defer` block is executed last)

```swift
func processFile(filename: String) throws {
    if exists(filename) {
        let file = open(filename)
        defer {
            close(file)
        }
        while let line = try file.readline() {
            // Work with the file.
        }
        // close(file) is called here, at the end of the scope.
    }
}
```

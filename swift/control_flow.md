# Swift - Control Flow<!-- omit in toc -->

[Swift Documentation](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/controlflow)

- [For-In Loops](#for-in-loops)
- [While Loops](#while-loops)
- [Conditional Statements](#conditional-statements)
  - [If](#if)
  - [Switch](#switch)
- [Control Transfer Statements](#control-transfer-statements)
  - [Continue](#continue)
  - [Break](#break)
  - [Fallthrough](#fallthrough)
- [Labeled Statements](#labeled-statements)
- [Early Exit](#early-exit)
- [Deferred Actions](#deferred-actions)
- [Checking API Availability](#checking-api-availability)

## For-In Loops

- `item` is a **constant** whose value is set to the current element each time the loop iterates
- `items` is a **sequence** of values that the loop iterates over, including arrays, ranges, strings, collections, or any other types that conforms to the [`Sequence`](https://developer.apple.com/documentation/swift/sequence) protocol
  - `0..<60`
  - `stride(from: 0, to: 60, by: 5)`: from 0 to 55
  - `stride(from: 0, through: 60, by: 5)`: from 0 to 60

```swift
for <#item#> in <#items#> {
    <#code#>
}

for (_, value) in <#dictionary#> { // ignore key
    <#code#>
}
```

## While Loops

```swift
while <#condition#> {
    <#code#>
}

repeat {
    <#code#>
} while <#condition#>
```

## Conditional Statements

### If

```swift
if <#condition#> {
    <#code#>
} else if <#condition#> {
    <#code#>
} else {
    <#code#>
}
```

#### If expression<!-- omit in toc -->

- An `if` statement can be used as an expression
- Each branch of the `if` statement must have a value of the same type
- Type need to be specified explicitly if any branch contains values like `nil` that can be used with more than one type
  - This is because Swift checks the type of each branch separately
- An `if` expression can respond to unexpected failures by throwing an error or calling a function like `fatalError(_:file:line:)`

```swift
let weatherAdvice = if isRaining {
    "Bring an umbrella"
} else {
    "Wear sunscreen"
}

let freezeWarning: String? = if temperatureInFahrenheit <= 32 { // explicit type
    "It's going to freeze"
} else {
    nil
}

let freezeWarning = if temperatureInFahrenheit <= 32 {
    "It's going to freeze"
} else {
    nil as String? // explicit type
} 

test { x -> Int? in  // valid with required type context
    guard let x { return nil }
    return x + 1
}
```

### Switch

- A `switch` statement considers a value and compares it against several possible matching patterns and executes the *first* matching case.
- Each `switch` statement must be *exhaustive*
  - Swift only truly verifies that the `switch` statement is exhaustive
    - if you use an enumeration (`enum` types) as the value of the switch statement’s condition
    - or the value is a Boolean value
  - If it’s possible that none of the cases will match, you must provide a `default` case to cover the remaining values
    - `default` case must be the last case in the `switch` statement if present
    - `default` is equivalent to `case _:`
- `switch` statements also have an expression form
- `switch` statements in Swift don’t fall through the bottom of each case and into the next one by default
  - Instead, the entire `switch` statement finishes its execution as soon as the first matching case is completed, without requiring an explicit `break` statement
  - If you need C-style fallthrough behavior, you can opt in to this behavior on a case-by-case basis with the `fallthrough` keyword

```swift
switch <#value#> {
case <#pattern#>:
    <#code#>
case <#pattern#>, <#pattern#>:
    <#code#>
default: // optional
    <#code#>
}

let anotherCharacter: Character = "a"
let message = switch anotherCharacter { // expression form
case "a", "A":
    "The letter A"
default:
    "Not the letter A"
}
```

- Interval matching: value in `switch` statement can be checked for inclusion in an interval
  - `case 0...9:`
  - `case 0..<9:`
- Tuples: can be used in `switch` statement
  - `case (0, 0):`
  - `case (_, 0):`
  - `case (0, _):`
  - `case (-2...2, -2...2):`
- Value binding: `let` or `var` can be used to bind the value or values matched by a particular case to temporary constants or variables
  - `case let (x, y):`
  - `case (let x, 0):`
  - `case (let x, let y):`
- `where` clause: can be used to check for additional conditions
  - `case let (x, y) where x == y:`
  - `case let (x, y) where x == -y:`
- Compound cases:
  - Multiple cases can be handled by a single case, separating the patterns with commas
    - `case 0, 1, 2:`
    - `case 0...9, 20...29:`
  - Compound cases can also include value bindings. All of the patterns of a compound case have to include the same set of value bindings, and each binding has to get a value of the same type from all of the patterns in the compound case
    - `case let (x, y) where x == y, let (x, y) where x == -y:`
    - `case (let distance, 0), (0, let distance):`

## Control Transfer Statements

Control transfer statements change the order in which your code is executed, by transferring control from one piece of code to another. Swift has five control transfer statements:

- `continue`
- `break`
- `fallthrough`
- `return`: covered in [Functions](functions.md)
- `throw`: covered in [Error Handling](error_handling.md)

### Continue

The `continue` statement tells a loop to stop what it is doing and start again at the beginning of the next iteration through the loop.

### Break

The `break` statement ends execution of an entire control flow statement immediately.

- *Unlabelled* `break` statement ends the execution of the `switch` statement or loop statement that it is located in.
- *Unlabelled* `break` can't be used inside of an `if` or `guard` statement.

### Fallthrough

The `fallthrough` statement tells a switch statement to continue execution at the next case, rather than falling out of the switch statement.

- `fallthrough` doesn’t check the case conditions for the switch case that it causes execution to fall into. The fallthrough statement simply causes code execution to move directly to the statements inside the next case (or default case) block, as in C’s standard switch statement behavior.

```swift
let integerToDescribe = 5
var description = "The number \(integerToDescribe) is"
switch integerToDescribe {
case 2, 3, 5, 7, 11, 13, 17, 19:
    description += " a prime number, and also"
    fallthrough
case 4, 6, 8, 10, 12, 14, 16, 18: // fallthrough, no condition check
    description += " an even integer."
default:
    description += " an integer."
}
print(description)
// Prints "The number 5 is a prime number, and also an even integer."
```

## Labeled Statements

A labeled statement is indicated by placing a label on the same line as the statement’s introducer keyword, followed by the statement itself.

You can mark a loop statement or conditional statement with a statement label.

- With a conditional statement, you can use a statement label with the `break` statement to end the execution of the labeled statement.
- With a loop statement, you can use a statement label with the `break` or `continue` statement to end or continue the execution of the labeled statement.

```swift
<#label name#>: while <#condition#> {
    <#statements#>
}

test: if a == 1 { // label if statement
    break test // break can only be used inside if statement with label
}
```

## Early Exit

A `guard` statement executes a set of statements depending on the Boolean value of an expression. You use a `guard` statement to require that a condition must be true in order for the code after the `guard` statement to be executed.

- Unlike an `if` statement, a `guard` statement always has an `else` clause which is executed if the condition is not true.
  - The `else` clause must either call a function marked with the `noreturn` attribute (such as `fatalError(_:file:line:)`) or transfer program control outside the guard statement’s enclosing scope using one of the following statements:
    - `return`
    - `break`
    - `continue`
    - `throw`
- If the condition is true, code execution continues after the `guard` statement’s closing brace.
  - Any variables or constants that were assigned values using an optional binding as part of the condition are available for the rest of the code block that the `guard` statement appears in.

```swift
func greet(person: [String: String]) {
    guard let name = person["name"] else {
        return
    }

    print("Hello \(name)!")

    guard let location = person["location"] else {
        print("I hope the weather is nice near you.")
        return
    }

    print("I hope the weather is nice in \(location).")
}

greet(person: ["name": "John"])
// Prints "Hello John!"
// Prints "I hope the weather is nice near you."
greet(person: ["name": "Jane", "location": "Cupertino"])
// Prints "Hello Jane!"
// Prints "I hope the weather is nice in Cupertino."
```

## Deferred Actions

A `defer` statement is used for executing code just before transferring program control outside of the scope that the `defer` statement appears in.

- A `defer` statement defers execution until the current scope is exited.
- The deferred statements may not contain any code that would transfer control out of the statements, such as a `break` or a `return` statement, or by throwing an error.
- The code inside of the `defer` statement will be executed regardless of how the program leaves the current scope. This behavior can be useful for ensuring that certain cleanup tasks are always performed.
  - That includes code like an early exit from a function, breaking out of a loop, or throwing an error.
  - If the program doesn’t leave the scope before reaching a `defer` statement, the deferred code won’t be executed.
  - If your program stops running due to a runtime error or other catastrophic failure, it won’t always be possible to execute your deferred code.
- The deferred statements are executed in reverse order of how they are specified — that is, the first one you specify is executed last.

```swift
func processFile(filename: String) throws {
    if exists(filename) {
        let file = open(filename)
        defer {
            close(file) // cleanup code
        }
        while let line = try file.readline() {
            // Work with the file.
        }
        // close(file) is called here, at the end of the scope.
    }
}
```

## Checking API Availability

Swift has built-in support for checking API availability, which ensures that you don’t accidentally use APIs that are unavailable on a given deployment target.

- The compiler uses availability information in the SDK to verify that all of the APIs used in your code are available on the deployment target specified by your project. Swift reports an error at compile time if you try to use an API that isn’t available.
- You use an availability condition in an `if` or `guard` statement to conditionally execute a block of code, depending on whether the APIs you want to use are available at runtime.
- `#available` and `#unavailable`

```swift
if #available(iOS 10, macOS 10.12, *) {
    // Use iOS 10 APIs on iOS, and use macOS 10.12 APIs on macOS
} else {
    // Fall back to earlier iOS and macOS APIs
}

@available(iOS 10, macOS 10.12, *)
struct ColorPreference {
    var bestColor = "blue"
}

func chooseBestColor() -> String {
    guard #available(iOS 10, macOS 10.12, *) else {
        return "black"
    }
    let colors = ColorPreference()
    return colors.bestColor
}

if #unavailable(iOS 10) {
    // Use earlier APIs on iOS
} else {
    // Use iOS 10 APIs on iOS
}
```

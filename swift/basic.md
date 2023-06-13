# Swift - Basics<!-- omit from toc -->

[Swift documentations](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/thebasics/)

- [Constants and Variables](#constants-and-variables)
  - [Type Annotations](#type-annotations)
  - [Naming Convention](#naming-convention)
- [Comments](#comments)
- [Semicolons](#semicolons)
- [Basic Types](#basic-types)
  - [Integers](#integers)
  - [Floating-Point Numbers](#floating-point-numbers)
  - [Booleans](#booleans)
  - [Tuples](#tuples)
- [Numeric Literals](#numeric-literals)
- [Numeric Type Conversion](#numeric-type-conversion)
- [Optionals](#optionals)
  - [Forced Unwrapping](#forced-unwrapping)
  - [Optional Binding](#optional-binding)
  - [Implicitly Unwrapped Optionals](#implicitly-unwrapped-optionals)
- [Type Safety and Type Inference](#type-safety-and-type-inference)
- [Error Handling](#error-handling)
- [Assertions and Preconditions](#assertions-and-preconditions)
  - [Assertions](#assertions)
  - [Preconditions](#preconditions)

## Constants and Variables

- Type of variables or constants can't be changed after declaration
- Value of constants can't be changed after declaration

```swift
let maximumNumberOfLoginAttempts = 10
// maximumNumberOfLoginAttempts = 9 // compile-time error
var currentLoginAttempt = 0
// currentLoginAttempt = "a" // compile-time error
var x = 0.0, y = 0.0, z = 0.0
```

### Type Annotations

```swift
var welcomeMessage: String
var red, green, blue: Double
typealias AudioSample = UInt16 
// you can use the alias anywhere you might use the original name
```

### Naming Convention

Constant and variable names can contain *almost* any character, including Unicode characters

- no whitespace characters, mathematical symbols, arrows
- no line- and box-drawing characters
- can't begin with a number

No variables or constants can be declared with same name.

[AVOID] To declare a constant or variable with the same name as a reserved Swift keyword, surround the keyword with backticks (``)

```swift
let π = 3.14159
let 你好 = "你好世界"
let 🐶🐮 = "dogcow"
```

## Comments

```swift
// This is a comment.
/* This is also a comment
but is written over multiple lines. */
/* This is the start of the first multiline comment.
    /* This is the second, nested multiline comment. */
This is the end of the first multiline comment. */
```

## Semicolons

optional unless multiple separate statements on a single line

```swift
let cat = "🐱"; print(cat)
```

## Basic Types

### Integers

Use the `Int` type for all general-purpose integer constants and variables, even if they’re known to be nonnegative.

- Integer constants and variables are immediately interoperable
- match the inferred type for integer literal values

Use other integer types only when they’re specifically needed for the task at hand:

- explicitly sized data from an external source
- performance or memory usage optimization
- catch overflows

Use unsigned interger (`UInt`) *only* when you specifically need an unsigned integer type with the same size as the platform’s native word size.

#### Signed Intergers<!-- omit from toc -->

Get bound for a type:

```swift
let minValue = Int.min
let maxValue = Int.max
```

| type | size | min | max | Java |
| ---- | ---- | --- | --- | ---- |
| `Int8` | 8 bits | -128 | 127 | byte |
| `Int16` | 16 bits | -32768 | 32767 | long |
| `Int32` | 32 bits | -2147483648 | 2147483647 | int |
| `Int64` | 64 bits | -9223372036854775808 | 9223372036854775807 | long |

#### Unsigned Intergers<!-- omit from toc -->

| type | size | min | max | Java |
| ---- | ---- | --- | --- | ---- |
| `UInt8` | 8 bits | 0 | 255 | - |
| `UInt16` | 16 bits | 0 | 65535 | - |
| `UInt32` | 32 bits | 0 | 4294967295 | - |
| `UInt64` | 64 bits | 0 | 18446744073709551615 | - |

#### Int and UInt<!-- omit from toc -->

- On a 32-bit platform, `Int` / `Uint` is the same size as `Int32` / `UInt32`.
- On a 64-bit platform, `Int` / `Uint` is the same size as `Int64` / `Uint64`.

### Floating-Point Numbers

- [Prefered] `Double` represents a 64-bit floating-point number.
  - `Double` has a precision of at least 15 decimal digits
- `Float` represents a 32-bit floating-point number.
  - The precision of `Float` can be as little as 6 decimal digits

### Booleans

Swift provides two Boolean constant values, true and false.

```swift
let someBool: Bool
let orangesAreOrange = true
let turnipsAreDelicious = false
```

### Tuples

- The values within a tuple can be of any type
- and don’t have to be of the same type as each other

```swift
let http404Error = (404, "Not Found")
// http404Error is of type (Int, String)
let (statusCode, statusMessage) = http404Error // decompose tuple
let (justTheStatusCode, _) = http404Error
print("The status code is \(http404Error.0)") // indexing
let http200Status = (statusCode: 200, description: "OK") // named tuple
print("The status code is \(http200Status.statusCode)")
```

## Numeric Literals

- Hexadecimal floating-point literals must end with an exponent of `x`, the base number is multiplied by `2^x`

```swift
let decimalInteger = 17;
let binaryInteger = 0b10001
let octalInteger = 0o21
let hexadecimalInteger = 0x11
let decimalDouble = 12.1875
let exponentDouble = 1.21875e1 // 1.21875 * 10^1
let hexadecimalDouble = 0xC.3p0 // (12 * 16^0 + 3 * 16^-1) * 2^0
let paddedDouble = 000123.456 // prefix zeros are allowed
let oneMillion = 1_000_000
let justOverOneMillion = 1_000_000.000_000_1
```

## Numeric Type Conversion

Because each numeric type can store a different range of values, you must opt in to numeric type conversion on a case-by-case basis (explicit conversion) to prevent hidden conversion errors and helps make type conversion intentions explicit in your code.

```swift
let cannotBeNegative: UInt8 = -1 // compile error
// No implicit conversion
let someNegInt = -1
let someNegUInt: UInt = someNegInt // compile error

let three = 3
let pointOneFourOneFiveNine = 0.14159
let pi = three + pointOneFourOneFiveNine // compile error
let pi = Double(three) + pointOneFourOneFiveNine
// pi equals 3.14159, and is inferred to be of type Double

let anotherPi = 3 + 0.14159
// this works as literal value of 3 has no explicit type in and of itself

// Hidden errors
let negInt = -1
let negUInt = UInt(negInt) // runtime error

let integerPi = Int(pi) // Double to Int
// always truncated: 4.75 -> 4; -3.9 -> -3
```

## Optionals

*Optionals* are used in situations where a value may be absent.

Optional can be declared by adding a question mark after a type.

```swift
var surveyAnswer: String? // surveyAnswer is automatically set to nil
var serverResponseCode: Int? = 404
serverResponseCode = nil
```

### Forced Unwrapping

Optional can be **foreced unwrapped** to a non-optionl by adding an exclamation point (`!`) at the end of the optional's name.

Forced unwrapping an optional set to `nil` will cause runtime error, thus you should always wrap it in an `if` statement.

```swift
convertedNumber: Int? = Int("404")
if convertedNumber != nil {
    print("convertedNumber contains integer value of \(convertedNumber!).") // forced unwrapping
}
```

### Optional Binding

**Optional binding** is used to find out 1) whether an optional contains a value, 2) and if so, to make that value available as a *temporary* constant or variable.

Optional binding can be used with `if` and `while` statements.

```swift
if let <#constantName#> = <#someOptional#> {
   <#statements#>
}
```

If the conversion is successful, the local variabel or constant becomes available for use within the first branch of the if statement. It has already been initialized with the value contained within the optional, and so you don’t use the `!` suffix to access its value.

Optional can be binding to varialbe and changes to the local variable doesn't change the optional.

You can use the same name as the optional for the new constant or variable

```swift
let myNumber = Int(possibleNumber)
// Here, myNumber is an optional integer
if let myNumber = myNumber {
    // Here, myNumber is a non-optional integer
    print("My number is \(myNumber)")
}
// Here, myNumber is an optional integer
```

or even simplier:

```swift
if let myNumber {
    print("My number is \(myNumber)")
}
```

More than one optional bindings and Boolean conditions can be used in a single `if` statement, separated by commas:

```swift
if let firstNumber = Int("4"), let secondNumber = Int("42"), firstNumber < secondNumber && secondNumber < 100 {
    print("\(firstNumber) < \(secondNumber) < 100")
}
```

### Implicitly Unwrapped Optionals

**Implicitly unwrapped optionals** are useful when an optional’s value is confirmed to exist immediately after the optional is first defined and can definitely be assumed to exist at every point thereafter. The primary use of implicitly unwrapped optionals in Swift is during class initialization, as described in [Unowned References and Implicitly Unwrapped Optional Properties](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/automaticreferencecounting#Unowned-References-and-Implicitly-Unwrapped-Optional-Properties).

You write an implicitly unwrapped optional by placing an exclamation point (`String!`) rather than a question mark (`String?`) after the type that you want to make optional.

**An implicitly unwrapped optional is a normal optional** behind the scenes (can be used in optional binding), but can also be used like a non-optional value, without the need to unwrap the optional value each time it’s accessed. If an implicitly unwrapped optional is `nil` and you try to access its wrapped value, you’ll trigger a runtime error.

```swift
let assumedString: String! = "An implicitly unwrapped optional string."
let implicitString: String = assumedString // no need for an exclamation point
```

When you use an implicitly unwrapped optional value, Swift first tries to use it as an ordinary optional value; if it can’t be used as an optional, Swift force-unwraps the value.

```swift
let optionalString = assumedString
// The type of optionalString is "String?" and assumedString isn't force-unwrapped.
```

## Type Safety and Type Inference

Swift is a **type-safe** language and performs type checks during compilation.

Swift uses **type inference** to work out the appropriate type. Type inference enables a compiler to deduce the type of a particular expression automatically when it compiles your code, simply by examining the values you provide.

Type inference is particularly useful when you declare a constant or variable with an initial value.

```swift
let anotherPi = 3 + 0.14159
// anotherPi is also inferred to be of type Double
```

The literal value of 3 has no explicit type in and of itself (it is not of type Int!). See [Type Safety and Type Inference](#type-safety-and-type-inference) for more details

## Error Handling

In contrast to optionals, which can use the presence or absence of a value to communicate success or failure of a function, error handling allows you to determine the underlying cause of failure, and, if necessary, propagate the error to another part of your program.

A function indicates that it can throw an error by including the throws keyword in its declaration.

```swift
func canThrowAnError() throws {
    // this function may or may not throw an error
}
```

Swift automatically propagates errors out of their current scope until they’re handled by a catch clause.

```swift
do {
    try canThrowAnError()
    // no error was thrown
} catch {
    // an error was thrown
}
```

Here’s an example of how error handling can be used to respond to *different* error conditions:

```swift
func makeASandwich() throws {
    // ...
}


do {
    try makeASandwich()
    eatASandwich()
} catch SandwichError.outOfCleanDishes {
    washDishes()
} catch SandwichError.missingIngredients(let ingredients) {
    buyGroceries(ingredients)
}
```

## Assertions and Preconditions

Assertions and preconditions are checks that happen at **runtime**.

You use assertions and preconditions to express the assumptions you make and the expectations you have while coding, so you can include them as part of your code. Assertions help you find mistakes and incorrect assumptions during **development**, and preconditions help you detect issues in **production**.

Unlike the error conditions discussed in [Error Handling](#error-handling) above, **assertions and preconditions aren’t used for recoverable or expected errors**. Because a failed assertion or precondition indicates an invalid program state, there’s no way to catch a failed assertion.

The difference between assertions and preconditions is in when they’re checked: Assertions are checked only in debug builds, but preconditions are checked in both debug and production builds. In production builds, the condition inside an assertion isn’t evaluated. This means you can use as many assertions as you want during your development process, without impacting performance in production.

### Assertions

`assert(_:_:file:line:)` function

```swift
let age = -3
assert(age >= 0, "A person's age can't be less than zero.")
// This assertion fails because -3 isn't >= 0.
```

If the code already checks the condition, you use the `assertionFailure(_:file:line:)` function to indicate that an assertion has failed.

```swift
if age > 10 {
    print("You can ride the roller-coaster or the ferris wheel.")
} else if age >= 0 {
    print("You can ride the ferris wheel.")
} else {
    assertionFailure("A person's age can't be less than zero.")
}
```

### Preconditions

Use a precondition whenever a condition has the potential to be false, but must definitely be true for your code to continue execution.

`precondition(_:_:file:line:)` function

```swift
// In the implementation of a subscript...
precondition(index > 0, "Index must be greater than zero.")
```

You can also call the `preconditionFailure(_:file:line:)` function to indicate that a failure has occurred — for example, if the default case of a switch was taken, but all valid input data should have been handled by one of the switch’s other cases.

> If you compile in unchecked mode `(-Ounchecked)`, preconditions aren’t checked. The compiler assumes that preconditions are always true, and it optimizes your code accordingly. However, the `fatalError(_:file:line:)` function always halts execution, regardless of optimization settings.
> You can use the `fatalError(_:file:line:)` function during prototyping and early development to create stubs for functionality that hasn’t been implemented yet, by writing fatalError("Unimplemented") as the stub implementation. Because fatal errors are never optimized out, unlike assertions or preconditions, you can be sure that execution always halts if it encounters a stub implementation.

# Enumerations<!-- omit from toc -->

- [Summary](#summary)
- [Syntax](#syntax)
- [Switch Statement](#switch-statement)
- [Iterating through all cases](#iterating-through-all-cases)
- [Recursive Enumerations](#recursive-enumerations)

## Summary

- Define a **common type** for a group of related values and enable you to work with those values in a **type-safe** way within your code
- Enumerations are value types
- Enumerations can have:
  - Instance methods
  - Computed properties
  - Initializers
  - Subscripts
  - Extensions
  - Conform to protocols

## Syntax

```swift
// Enumerations without raw values
// The different cases are value in their own right
enum SomeEnum: <Protocol1>, <Protocol2> {
    case case1
    case case2, case3
}

var someEnum = SomeEnum.case1
someEnum = .case2 // shorthand, type inference

// Enumerations with associated values for any type
// Value types can be different for each case
enum SomeEnumWithAssociatedValue: <Protocol1>, <Protocol2> {
    case case1
    case case2(Int, String)
    case case3(String)
    case case4(Int, Int)
}

var someEnumWithAssociatedValue = SomeEnumWithAssociatedValue.case2(1, "Hello")
someEnumWithAssociatedValue = .case3("World")

// Enumerations with raw values 
// raw values are of the same type for all cases
enum SomeEnumWithRawValue: Character, <Protocol1>, <Protocol2> {
    case case1 = "a"
    case case2 = "b", case3 = "c"

enum SomeEnumWithImplicitStringRawValue: String {
    case case1, case2, case3 // raw values are case names
}
enum SomeEnumWithImplicitIntRawValue: Int {
    case case1, case2, case3 // raw values are 0, 1, 2
}
enum SomeEnumWithImplicitIntRawValue2: Int {
    case case1 = 1, case2, case3 // raw values are 1, 2, 3
}

let possibleSomeEnumWithRawValue = SomeEnumWithRawValue(rawValue: "a") // optional
let possibleSomeEnumWithRawValue = SomeEnumWithRawValue(rawValue: "d") // nil
```

## Switch Statement

```swift
// no need for default; swift will check if all cases are exhausted
switch someEnum {
case .case1:
    // do something
case .case2, .case3:
    // do something
} 

// use let/var to extract associated values
switch someEnumWithAssociatedValue {
case .case1:
    // do something
case .case2(let int, var string):
    // do something
case .case3(let string):
    // do something
case let .case4(int1, int2): // shorthand
    // do something
}
```

## Iterating through all cases

- Compiler will provide an implementation for enumerations with no associated values

```swift
enum SomeEnum: CaseIterable {
    case case1
    case case2
    case case3
}

for someEnum in SomeEnum.allCases {
    // do something
}
```

## Recursive Enumerations

- use `indirect` keyword to indicate that the enumeration is recursive, which tells the compiler to insert the necessary layer of indirection

```swift
enum ArithmeticExpression {
    case number(Int)
    indirect case addition(ArithmeticExpression, ArithmeticExpression)
    indirect case multiplication(ArithmeticExpression, ArithmeticExpression)
}

// or

indirect enum ArithmeticExpression {
    case number(Int)
    case addition(ArithmeticExpression, ArithmeticExpression)
    case multiplication(ArithmeticExpression, ArithmeticExpression)
}

// usage
func evaluate(_ expression: ArithmeticExpression) -> Int {
    switch expression {
    case let .number(value):
        return value
    case let .addition(left, right):
        return evaluate(left) + evaluate(right)
    case let .multiplication(left, right):
        return evaluate(left) * evaluate(right)
    }
}
```

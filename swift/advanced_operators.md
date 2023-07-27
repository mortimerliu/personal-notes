# Advanced Operators<!-- omit from toc -->

- [Bitwise Operators](#bitwise-operators)
  - [Shifting Behavior for Unsigned Integers - Logical Shift](#shifting-behavior-for-unsigned-integers---logical-shift)
  - [Shifting Behavior for Signed Integers - Arithmetic Shift](#shifting-behavior-for-signed-integers---arithmetic-shift)
- [Overflow Operators](#overflow-operators)
  - [Value Overflow](#value-overflow)
- [Precedence and Associativity](#precedence-and-associativity)
- [Operator Methods](#operator-methods)
- [Custom Operators](#custom-operators)
  - [Precedence for Custom Infix Operators](#precedence-for-custom-infix-operators)
- [Result Builders](#result-builders)

## Bitwise Operators

Bitwise operators enable you to manipulate the individual raw data bits within a data structure

- bitwise NOT (`~`)
  - inverts all bits in a number
  - is a prefix operator (immediately before its operand without any white space)
- bitwise AND (`&`)
  - compares the bits of two numbers; if both bits are 1, the resulting bit is 1; otherwise, the resulting bit is 0
- bitwise OR (`|`)
  - compares the bits of two numbers; if either bit is 1, the resulting bit is 1; otherwise, the resulting bit is 0
- bitwise XOR (`^`)
  - compares the bits of two numbers; if the bits are different, the resulting bit is 1; otherwise, the resulting bit is 0
- bitwise left shift (`<<`)
  - moves all bits in a number to the left by a specified number of places
- bitwise right shift (`>>`)
  - moves all bits in a number to the right by a specified number of places

### Shifting Behavior for Unsigned Integers - Logical Shift

- Existing bits are moved to the left or right by the requested number of places
- Any bits that are moved beyond the bounds of the integer’s storage are discarded
- Zeros are inserted in the spaces left behind after the original bits are moved to the left or right

```swift
let pink: UInt32 = 0xCC6699
let redComponent = (pink & 0xFF0000) >> 16 // redComponent is 0xCC, or 204
let greenComponent = (pink & 0x00FF00) >> 8 // greenComponent is 0x66, or 102
let blueComponent = pink & 0x0000FF // blueComponent is 0x99, or 153
```

### Shifting Behavior for Signed Integers - Arithmetic Shift

- A signed integer uses its first bit (known as the *sign bit*) to indicate whether the integer is positive or negative
  - 0 means positive
  - 1 means negative
- The remaining bits (known as the *value bits*) store the actual value
- Positive numbers are stored in exactly the same way as for unsigned integers
- Negative numbers are stored by subtracting their absolute value from 2 to the power of n, where n is the number of value bits, which is known as a *two’s complement representation*
  - For example, an 8-bit number has 7 value bits. `-4` is stored as `11111100`, where the remaining 7 bits is `124` (`2^7 - 4`)
- Advantage of two’s complement representation:
  - you can add -1 to -4, simply by performing a standard binary addition of all eight bits (including the sign bit), and discarding anything that doesn’t fit in the eight bits once you’re done
  - the two’s complement representation also lets you shift the bits of negative numbers to the left and right like positive numbers, and still end up doubling them for every shift you make to the left, or halving them for every shift you make to the right. To achieve this, an extra rule is used when signed integers are shifted to the right: When you shift signed integers to the right, apply the same rules as for unsigned integers, but fill any empty bits on the left with the sign bit, rather than with a zero
- Because of the special way that positive and negative numbers are stored, shifting either of them to the right moves them closer to zero. Keeping the sign bit the same during this shift means that negative integers remain negative as their value moves closer to zero.

## Overflow Operators

Arithmetic operations in Swift don't overflow by default. To opt in to overflow behavior (when you specifically want an overflow condition to truncate the number of available bits), use Swift’s overflow operators which all begin with an ampersand (`&`)

- Overflow addition (`&+`)
- Overflow subtraction (`&-`)
- Overflow multiplication (`&*`)

### Value Overflow

- Numbers can overflow in both the positive and negative direction

```swift
var unsignedOverflow = UInt8.max
unsignedOverflow = unsignedOverflow &+ 1 // 0
// UInt8.max: 11111111
// UInt8.max + 1: 100000000
// Truncated to 8 bits: 00000000

var unsignedOverflow = UInt8.min // 0
unsignedOverflow = unsignedOverflow &- 1 // 255

var signedOverflow = Int8.min // -128
signedOverflow = signedOverflow &- 1 // 127
// Int8.min: 10000000
// &-      : 00000001
// =       : 01111111
```

## Precedence and Associativity

- Operator ***precedence*** gives some operators higher priority than others and these operators are applied first
- Operator ***associativity*** defines how operators of the same precedence are grouped together - either grouped from the left, or grouped from the right

More information on [Operator Declarations](https://developer.apple.com/documentation/swift/operator_declarations)

## Operator Methods

- Classes and structures can provide their own implementations of existing operators, this is known as *overloading* the existing operators
- prefix (`-a`) and postfix (`b!`) operators are marked with the `prefix` and `postfix` modifiers
- compound assignment operators combine assignment (`=`) with another operation; mark the left input parameter type as `inout` to enable the parameter to be modified in place
- It isn’t possible to overload the default assignment operator (`=`). Only the compound assignment operators can be overloaded. Similarly, the ternary conditional operator `(a ? b : c)` can’t be overloaded.
- By default, custom classes and structures don’t have an implementation of the equivalence operators (`==` and `!=`). You can implement either by yourself or use synthesized implementations and add conformances to the `Equatable` protocol

```swift
struct Vector2D {
    var x = 0.0, y = 0.0
}

extension Vector2D {
    // infix operator
    static func + (left: Vector2D, right: Vector2D) -> Vector2D {
        return Vector2D(x: left.x + right.x, y: left.y + right.y)
    }
    // prefix operator
    static prefix func - (vector: Vector2D) -> Vector2D {
        return Vector2D(x: -vector.x, y: -vector.y)
    }
    // postfix operator
    static postfix func ++ (vector: inout Vector2D) -> Vector2D {
        vector += Vector2D(x: 1.0, y: 1.0)
        return vector
    }
    // compound assignment operator
    static func += (left: inout Vector2D, right: Vector2D) {
        left = left + right // use the existing + operator
    }
}

extension Vector2D: Equatable {
    static func == (left: Vector2D, right: Vector2D) -> Bool {
        return (left.x == right.x) && (left.y == right.y)
    }
}
```

## Custom Operators

- You can declare and implement your own custom operators in addition to the standard operators provided by Swift. For a list of characters that can be used to define custom operators, see [Operators](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/lexicalstructure#Operators).
- New operators are declared at a global level using the `operator` keyword, and are marked with the `prefix`, `infix` or `postfix` modifiers

```swift
// global scope
prefix operator +++

extension Vector2D {
    // prefix operator
    static prefix func +++ (vector: inout Vector2D) -> Vector2D {
        vector += vector
        return vector
    }
}
```

### Precedence for Custom Infix Operators

- Custom infix operators each belong to a precedence group which defines the operator’s precedence relative to other infix operators as well as the operator’s associativity
- A custom infix operator that’s not explicitly placed into a precedence group is given a default precedence group with a precedence immediately higher than the precedence of the ternary conditional operator
- You don’t specify a precedence when defining a prefix or postfix operator. However, if you apply both a prefix and a postfix operator to the same operand, the postfix operator is applied first.

```swift
infix operator +-: AdditionPrecedence
extension Vector2D {
    static func +- (left: Vector2D, right: Vector2D) -> Vector2D {
        return Vector2D(x: left.x + right.x, y: left.y - right.y)
    }
}
```

## Result Builders

- A result builder is a type you define that adds syntax for creating nested data, like a list or tree, in a natural, declarative way
- The code that uses the result builder can include ordinary Swift syntax, like if and for, to handle conditional or repeated pieces of data.
- Use the `@resultBuilder` attribute to define a result builder type

```swift
rotocol Drawable {
    func draw() -> String
}
struct Line: Drawable {
    var elements: [Drawable]
    func draw() -> String {
        return elements.map { $0.draw() }.joined(separator: "")
    }
}
struct Text: Drawable {
    var content: String
    init(_ content: String) { self.content = content }
    func draw() -> String { return content }
}
struct Space: Drawable {
    func draw() -> String { return " " }
}
struct Stars: Drawable {
    var length: Int
    func draw() -> String { return String(repeating: "*", count: length) }
}
struct AllCaps: Drawable {
    var content: Drawable
    func draw() -> String { return content.draw().uppercased() }
}


// normal way
let name: String? = "Ravi Patel"
let manualDrawing = Line(elements: [
     Stars(length: 3),
     Text("Hello"),
     Space(),
     AllCaps(content: Text((name ?? "World") + "!")),
     Stars(length: 2),
])
print(manualDrawing.draw())
// Prints "***Hello RAVI PATEL!**"

// using result builder
@resultBuilder
struct DrawingBuilder {
    static func buildBlock(_ components: Drawable...) -> Drawable {
        return Line(elements: components)
    }
    // support if-else statements - first branch
    static func buildEither(first: Drawable) -> Drawable {
        return first
    }
    // support if-else statements - second branch
    static func buildEither(second: Drawable) -> Drawable {
        return second
    }
    // support for-in loops
    static func buildArray(_ components: [Drawable]) -> Drawable {
        return Line(elements: components)
    }    
}

// You can apply the @DrawingBuilder attribute to a function’s parameter, which turns a closure passed to the function into the value that the result builder creates from that closure
func draw(@DrawingBuilder content: () -> Drawable) -> Drawable {
    return content()
}
func caps(@DrawingBuilder content: () -> Drawable) -> Drawable {
    return AllCaps(content: content())
}

func makeGreeting(for name: String? = nil) -> Drawable {
    let greeting = draw { // buildBlock(_:)
        Stars(length: 3) 
        Text("Hello")
        Space()
        caps {
            if let name = name {
                Text(name + "!") // buildEither(first:)
            } else {
                Text("World!") // buildEither(second:)
            }
        }
        Stars(length: 2)
    }
    return greeting
}
let genericGreeting = makeGreeting()
print(genericGreeting.draw())
// Prints "***Hello WORLD!**"

let personalGreeting = makeGreeting(for: "Ravi Patel")
print(personalGreeting.draw())
// Prints "***Hello RAVI PATEL!**"

let manyStars = draw { // buildBlock(_:)
    Text("Starts:")
    for length in 1...3 { // buildArray(_:)
        Space()
        Stars(length: length) 
    }
}
```

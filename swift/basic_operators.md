# Swift - Basic Operators<!-- omit from toc -->

[Swift Documentation](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/basicoperators)

- [Basic operators](#basic-operators)
- [Ternary Conditional Operator](#ternary-conditional-operator)
- [Nil-Coalescing Operator](#nil-coalescing-operator)
- [Range Operators](#range-operators)
  - [Closed Range Operator](#closed-range-operator)
  - [Half-Open Range Operator](#half-open-range-operator)
  - [One-Sided Ranges](#one-sided-ranges)
  - [Identity Operators](#identity-operators)

An operator is a special symbol or phrase that you use to check, change, or combine values.

## Basic operators

- assignment operator: `=`
  - Unlike in C and Objective-C, the assignment operator doesn't return a value and thus can't be used in `if` statement.

  ```swift
  let a = 10
  b = a
  let (x, y) = (1, 2)
  ```

- arithmetic operators: `+`, `-`, `*`, `/`, `%`
  - Unlike the arithmetic operators in C and Objective-C, the Swift arithmetic operators don’t allow values to overflow by default.
  - The remainder operator (%) is also known as a modulo operator in other languages. However, its behavior in Swift for negative numbers means that, strictly speaking, it’s a remainder rather than a modulo operation.
    - `a = (b x some multiplier) + remainder`
    - The sign of `b` is ignored for negative values of `b`. This means that `a % b` and `a % -b` always give the same answer

    ```swift
    10 / 3              // integer division: 3
    10.0 / 2.5          // 4.0
    "hello, " + "world" // string concatenation
    -9 % 4              // -1
    ```

- Unary Minus Operator: `-x`
- Unary Plus operator: `+x` return value without any change
- Compound Assignment Operators: combine assignment (=) with another operation, for example `+=`
  - The compound assignment operator doesn't return a value
- Comparison Operators: `==`, `!=`, `>`, `<`, `>=`, `<=`
  - You can compare two tuples if they have the same type and the same number of values. Tuples are compared from left to right, one value at a time, until the comparison finds two values that aren’t equal. Those two values are compared, and the result of that comparison determines the overall result of the tuple comparison. If all the elements are equal, then the tuples themselves are equal.
  - The Swift standard library includes tuple comparison operators for tuples with fewer than seven elements. To compare tuples with seven or more elements, you must implement the comparison operators yourself.

  ```swift
  (3, 1, "zebra") < (3, 2, "apple") // true
  ("blue", false) < ("purple", true)  // Error because < can't compare Boolean values
  ```

- Logical Operators: `!a`, `a && b`, `a || b`
  - Swift has short-circuit evaluation for both `&&` and `||`.
  - **Combining Logical Operators**: The Swift logical operators && and || are left-associative, meaning that compound expressions with multiple logical operators evaluate the leftmost subexpression first.
  - **Explicit Parentheses**: It’s sometimes useful to include parentheses when they’re not strictly needed, to make the intention of a complex expression easier to read.

## Ternary Conditional Operator

The ternary conditional operator `question ? answer1 : answer2` is a shorthand for the code below:

```swift
if question {
    answer1
} else {
    answer2
}
```

## Nil-Coalescing Operator

The nil-coalescing operator (`a ?? b`) unwraps an *optional* `a` if it contains a value, or returns a default value `b` if a is `nil`.

- The expression a is always of an optional type.
- The expression b must match the type that’s stored inside a.

If the value of `a` is non-nil, the value of `b` isn’t evaluated. This is known as *short-circuit evaluation*.

## Range Operators

### Closed Range Operator

The closed range operator `(a...b)` defines a range that runs from `a` to `b`, and includes the values a and b.

- The value of a must not be greater than b.

```swift
let names = ["Anna", "Alex", "Brian", "Jack"]
names[1...2] // ["Alex", "Brian"]
```

### Half-Open Range Operator

The half-open range operator `(a..<b)` defines a range that runs from a to b, but doesn’t include b.

- The value of a must not be greater than b.
- Half-open ranges are particularly useful when you work with zero-based lists.

### One-Sided Ranges

One-sided ranges continue as far as possible in one direction.

- You can’t iterate over a one-sided range that omits a first value, because it isn’t clear where iteration should begin.
- You can iterate over a one-sided range that omits its final value; however, because the range continues indefinitely, make sure you add an explicit end condition for the loop.

```swift
let names = ["Anna", "Alex", "Brian", "Jack"]
names[2...] // ["Brian", "Jack"]
names[...2] // ["Anna", "Alex", "Brian"]
names[..<2] // ["Anna", "Alex"]
```

One-sided ranges can be used in other contexts, not just in subscripts.

```swift
let range = ...5
range.contains(7)   // false
range.contains(4)   // true
range.contains(-1)  // true
```

### Identity Operators

The identity operators `===` and `!==` are used to test whether two object references both refer to the same object instance.

Both operators can only be used with class instances.

- Identical to (`===`)
- Not identical to (`!==`)

#### `===` vs `==`<!-- omit in toc -->

- `===` checks if two instances refer to the same object
- `==` checks if two instances are equal in value
- if `===` is true, `==` is also true; the reverse is not true

```swift
class Person: Equatable {
    var name: String
    init(name: String) {
        self.name = name
    }
    static func == (lhs: Person, rhs: Person) -> Bool {
        lhs.name == rhs.name
    }
}

let p1 = Person(name: "John")
let p2 = Person(name: "John")
let p3 = p1
p1 === p2 // false
p1 == p2  // true

p1 === p3 // true
p1 == p3  // true
```

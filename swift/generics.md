# Generics<!-- omit from toc -->

- [Generic Functions and Type Parameters](#generic-functions-and-type-parameters)
- [Generic Types](#generic-types)
  - [Extending a Generic Type](#extending-a-generic-type)
- [Type Constraints](#type-constraints)
- [Associated Types](#associated-types)
  - [Extending an Existing Type to Specify an Associated Type](#extending-an-existing-type-to-specify-an-associated-type)
  - [Adding Constraints to an Associated Type](#adding-constraints-to-an-associated-type)
  - [Using a Protocol in Its Associated Type's Constraints](#using-a-protocol-in-its-associated-types-constraints)
- [Generic Where Clauses](#generic-where-clauses)
  - [Extensions with a Generic Where Clause](#extensions-with-a-generic-where-clause)
  - [Contextual Where Clauses](#contextual-where-clauses)
  - [Associated Types with a Generic Where Clause](#associated-types-with-a-generic-where-clause)
- [Generic Subscripts](#generic-subscripts)

Write flexible, reusable functions and types that can work with any type, subject to requirements that define a generic type

## Generic Functions and Type Parameters

- *Type Parameters* specify and name a placeholder type, and are written immediately after the function's name, between a pair of matching angle brackets (such as `<T>`)
- Type parameter can be used to define the type of a function's parameters, or as the function's return type, or as a type annotation within the body of the function
- Define multiple type parameters by writing multiple type parameter names within the angle brackets, separated by commas
- Naming convention: either descriptive (such as `Key` and `Value`), or single letters (such as `T`, `U`, and `V`)

```swift
// this makes sure a and b are of the same type
// the actual type of T is determined each time the swapTwoValues(_:_:) function is called
func swapTwoValues<T>(_ a: inout T, _ b: inout T) {
    let temporaryA = a
    a = b
    b = temporaryA
}
```

## Generic Types

- Define a *generic type* (class, structure, or enumeration) that can work with any type
- Define type parameters within angle brackets (`<>`) after the type name
- The type parameters can be used anywhere inside the type definition

```swift
struct Stack<Element> {
    var items: [Element] = []
    mutating func push(_ item: Element) {
        items.append(item)
    }
    mutating func pop() -> Element {
        return items.removeLast()
    }
}
```

### Extending a Generic Type

- When extending a generic type,  the type parameter list from the `original` type definition is available within the extension body

```swift
extension Stack {
    var topItem: Element? {
        return items.isEmpty ? nil : items[items.count - 1]
    }
}
```

## Type Constraints

- Specify that a type parameter must inherit from a specific class, or conform to a particular protocol or protocol composition
- write type constraints by placing a single class or protocol constraint after a type parameter's name, separated by a colon, as part of the type parameter list

```swift
// same syntax for generic types
func someFunction<T: SomeClass, U: SomeProtocol>(someT: T, someU: U) {
    // function body goes here
}

func findIndex<T: Equatable>(of valueToFind: T, in array:[T]) -> Int? {
    for (index, value) in array.enumerated() {
        if value == valueToFind {
            return index
        }
    }
    return nil
}
```

## Associated Types

- *Associated types* give a placeholder name to a type that is used as part of the ***protocol***
- Actual type to use for that associated type is not specified until the **protocol** is adopted

```swift
protocol Container {
    associatedtype Item
    mutating func append(_ item: Item)
    var count: Int { get }
    subscript(i: Int) -> Item { get }
}

struct IntStack: Container {
    // original IntStack implementation
    var items = [Int]()
    mutating func push(_ item: Int) {
        items.append(item)
    }
    mutating func pop() -> Int {
        return items.removeLast()
    }
    // conformance to the Container protocol
    typealias Item = Int // optional due to type inference
    mutating func append(_ item: Int) {
        self.push(item)
    }
    var count: Int {
        return items.count
    }
    subscript(i: Int) -> Int {
        return items[i]
    }
}

// generic type
struct Stack<Element>: Container {
    // original Stack<Element> implementation
    var items: [Element] = []
    mutating func push(_ item: Element) {
        items.append(item)
    }
    mutating func pop() -> Element {
        return items.removeLast()
    }
    // conformance to the Container protocol
    mutating func append(_ item: Element) {
        self.push(item)
    }
    var count: Int {
        return items.count
    }
    subscript(i: Int) -> Element {
        return items[i]
    }
}
```

### Extending an Existing Type to Specify an Associated Type

```swift
extension Array: Container {}
```

### Adding Constraints to an Associated Type

- Add type constraints to an associated type in a protocol to require that conforming types satisfy those constraints

```swift
protocol Container {
    associatedtype Item: Equatable
    mutating func append(_ item: Item)
    var count: Int { get }
    subscript(i: Int) -> Item { get }
}
```

### Using a Protocol in Its Associated Type's Constraints

- A protocol can appear as part of its own requirements

```swift
protocol SuffixableContainer: Container {
    associatedtype Suffix: SuffixableContainer where Suffix.Item == Item
    func suffix(_ size: Int) -> Suffix
}

extension Stack: SuffixableContainer {
    func suffix(_ size: Int) -> Stack {
        var result = Stack()
        for index in (count-size)..<count {
            result.append(self[index])
        }
        return result
    }
    // Inferred that Suffix is Stack.
}
```

## Generic Where Clauses

- A *generic where clause* enables you to require that an associated type must conform to a certain protocol, or that certain type parameters and associated types must be the same
- A generic where clause starts with the `where` keyword, followed by constraints for associated types or equality relationships between types and associated types
- Write a generic where clause after the type parameter list and before the function or type body

```swift
// C1 and C2 could be different container types, but they must hold the same type of items
func allItemsMatch<C1: Container, C2: Container>
        (_ someContainer: C1, _ anotherContainer: C2) -> Bool
        where C1.Item == C2.Item, C1.Item: Equatable {

    // Check that both containers contain the same number of items.
    if someContainer.count != anotherContainer.count {
        return false
    }

    // Check each pair of items to see if they're equivalent.
    for i in 0..<someContainer.count {
        if someContainer[i] != anotherContainer[i] {
            return false
        }
    }

    // All items match, so return true.
    return true
}
```

### Extensions with a Generic Where Clause

- Write a generic where clause for an extension by writing a generic `where` clause right before the opening curly brace of the extension's body

```swift
// isTop(_:) function is only available on stacks whose elements conform to the Equatable protocol
extension Stack where Element: Equatable {
    func isTop(_ item: Element) -> Bool {
        guard let topItem = items.last else {
            return false
        }
        return topItem == item
    }
}

// extension to a protocol
extension Container where Item: Equatable {
    func startsWith(_ item: Item) -> Bool {
        return count >= 1 && self[0] == item
    }
}

// extension to a protocol with where clause to a specific type
extension Container where Item == Double {
    func average() -> Double {
        var sum = 0.0
        for index in 0..<count {
            sum += self[index]
        }
        return sum / Double(count)
    }
}
```

### Contextual Where Clauses

- write a generic where clause as part of a declaration that doesn't have its own generic type constraints, when you are already working in the context of generic types

```swift
// this can be achieved using two extensions as well
extension Container {
    // average() method is only available on containers whose items is of type Int
    func average() -> Double where Item == Int {
        var sum = 0.0
        for index in 0..<count {
            sum += Double(self[index])
        }
        return sum / Double(count)
    }
    // endsWith(_:) method is only available on containers whose items conform to the Equatable protocol
    func endsWith(_ item: Item) -> Bool where Item: Equatable {
        return count >= 1 && self[count-1] == item
    }
}
```

### Associated Types with a Generic Where Clause

- Write a generic where clause for an associated type by including the `where` keyword after the associated type name followed by any type constraints
- For a protocol that inherits from another protocol, write a generic where clause to an inherited associated type by including the `where` clause in the protocol declaration

```swift
protocol Container {
    associatedtype Item
    mutating func append(_ item: Item)
    var count: Int { get }
    subscript(i: Int) -> Item { get }

    associatedtype Iterator: IteratorProtocol where Iterator.Element == Item
    func makeIterator() -> Iterator
}

protocol ComparableContainer: Container where Item: Comparable { }
```

## Generic Subscripts

- A *generic subscript* can include one or more type parameters, and generic where clauses right before the opening curly brace of the subscript's body

```swift
extension Container {
    subscript<Indices: Sequence>(indices: Indices) -> [Item]
        where Indices.Iterator.Element == Int {
            var result: [Item] = []
            for index in indices {
                result.append(self[index])
            }
            return result
    }
}
```

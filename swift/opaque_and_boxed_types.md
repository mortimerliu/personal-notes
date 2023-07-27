# Opaque and Boxed Types<!-- omit from toc -->

- [Opaque Types](#opaque-types)
  - [The Problem That Opaque Types Solve](#the-problem-that-opaque-types-solve)
  - [Returning an Opaque Type](#returning-an-opaque-type)
- [Boxed Protocol Types](#boxed-protocol-types)
- [Opaque Types vs. Boxed Protocol Types](#opaque-types-vs-boxed-protocol-types)

## Opaque Types

### The Problem That Opaque Types Solve

- Exposing detailed information about the implementation of a value's type allows types that aren't meant to be part of the public interface of a module to leak out (i.e., `JointedShape` and `FlippedShape`), making it harder to change the implementation details without breaking clients of the module

```swift
protocol Shape {
    func draw() -> String
}

struct Triangle: Shape {
    var size: Int
    func draw() -> String {
        var result: [String] = []
        for length in 1...size {
            result.append(String(repeating: "*", count: length))
        }
        return result.joined(separator: "\n")
    }
}

struct FlippedShape<T: Shape>: Shape {
    var shape: T
    func draw() -> String {
        let lines = shape.draw().split(separator: "\n")
        return lines.reversed().joined(separator: "\n")
    }
}

struct JoinedShape<T: Shape, U: Shape>: Shape {
    var top: T
    var bottom: U
    func draw() -> String {
        return top.draw() + "\n" + bottom.draw()
    }
}

let smallTriangle = Triangle(size: 3)
let flippedTriangle = FlippedShape(shape: smallTriangle)
// joinedTriangles is of type JoinedShape<Triangle, FlippedShape<Triangle>>
let joinedTriangles = JoinedShape(top: smallTriangle, bottom: flippedTriangle)
```

### Returning an Opaque Type

- An *opaque type* lets the function implementation pick the type for the value it returns in a way that's abstracted away from the code that calls the function
- Use the `some` keyword to indicate that a return type or a variable is opaque
- the function can return any type that conforms to a given protocol or that inherits from a given class
- For a function with an opaque return type returns from multiple places, all the possible return values must have the same type
  - For a generic function, the return type can use the generic type parameter, but it must still be a single type (as opaque types reserve type identity)

```swift
struct Square: Shape {
    var size: Int
    func draw() -> String {
        let line = String(repeating: "*", count: size)
        let result = Array<String>(repeating: line, count: size)
        return result.joined(separator: "\n")
    }
}

// makeTrapezoid() can return any type that conforms to Shape
// the code that calls makeTrapezoid() needs to be written in a general way such that it can work with any Shape
func makeTrapezoid() -> some Shape {
    let top = Triangle(size: 2)
    let middle = Square(size: 2)
    let bottom = FlippedShape(shape: top)
    let trapezoid = JoinedShape(
        top: top,
        bottom: JoinedShape(top: middle, bottom: bottom)
    )
    return trapezoid
}

// opaque types reserve type identity
// trapezoid is still of type JoinedShape<Triangle, JoinedShape<Square, FlippedShape<Triangle>>>
let trapezoid = makeTrapezoid()

// opaque types with generics
func flip<T: Shape>(_ shape: T) -> some Shape {
    return FlippedShape(shape: shape)
}
func join<T: Shape, U: Shape>(_ top: T, _ bottom: U) -> some Shape {
    return JoinedShape(top: top, bottom: bottom)
}

func invalidFlip<T: Shape>(_ shape: T) -> some Shape {
    if shape is Square {
        return shape // Error: return types don't match
    }
    return FlippedShape(shape: shape) // Error: return types don't match
}

func `repeat`<T: Shape>(shape: T, count: Int) -> some Collection {
    return Array<T>(repeating: shape, count: count)
}
```

## Boxed Protocol Types

- A *boxed protocol type* is also called an *existential type*, which menas "there exists a type T such that T conforms to the protocol"
- Use `any` keyword to indicate that a variable is a boxed protocol type
- Can only use members required by the protocol for a variable of a boxed protocol type
- Can use type casting to access members that aren't required by the protocol

```swift
struct VerticalShapes: Shape {
    var shapes: [any Shape] // boxed protocol type, each element can be different type that conforms to Shape
    func draw() -> String {
        return shapes.map { $0.draw() }.joined(separator: "\n\n")
    }
}

let largeTriangle = Triangle(size: 5)
let largeSquare = Square(size: 5)
let vertial = VerticalShapes(shapes: [largeTriangle, largeSquare])
```

Contrast the three types you could use for shapes:

- Using generics, by writing `struct VerticalShapes<S: Shape>` and `var shapes: [S]`, makes an array whose elements are some specific shape type, and where the identity of that specific type is visible to any code that interacts with the array.
- Using an opaque type, by writing `var shapes: [some Shape]`, makes an array whose elements are some specific shape type, and where that specific type’s identify is hidden.
- Using a boxed protocol type, by writing `var shapes: [any Shape]`, makes an array that can store elements of different types, and where those types’ identities are hidden.

## Opaque Types vs. Boxed Protocol Types

Returning an opaque type looks very similar to using a boxed protocol type as the return type of a function, but these two kinds of return type differ in **whether they preserve type identity**. An opaque type refers to one specific type, although the caller of the function isn’t able to see which type; a boxed protocol type can refer to any type that conforms to the protocol.

Using a boxed protocol type as the return type for a function gives you the flexibility to return any type that conforms to the protocol. However, the cost of that flexibility is that some operations aren’t possible on the returned values.

In contrast, opaque types preserve the identity of the underlying type. Swift can infer associated types, which lets you use an opaque return value in places where a boxed protocol type can’t be used as a return value.

```swift
// use boxed protocol type as return type
func protoFlip<T: Shape>(_ shape: T) -> Shape {
    return FlippedShape(shape: shape)
}

// doesn't requre all possible return values to have the same type
func protoFlip<T: Shape>(_ shape: T) -> Shape {
    if shape is Square {
        return shape
    }

    return FlippedShape(shape: shape)
}

let protoFlippedTriangle = protoFlip(smallTriangle)
let sameThing = protoFlip(smallTriangle)
protoFlippedTriangle == sameThing  // Error as `protoFlip(_:)` returns a boxed protocol type

// use opaque type as return type
protocol Container {
    associatedtype Item
    var count: Int { get }
    subscript(i: Int) -> Item { get }
}
extension Array: Container {}

// You can’t use Container as the return type of a function because that protocol has an associated type. You also can’t use it as constraint in a generic return type because there isn’t enough information outside the function body to infer what the generic type needs to be.

// Error: Protocol with associated types can't be used as a return type.
func makeProtocolContainer<T>(item: T) -> Container {
    return [item]
}

// Error: Not enough information to infer C.
func makeProtocolContainer<T, C: Container>(item: T) -> C {
    return [item]
}

// as opaque types reserve the type identity, the underlying type of the opaque container is [T]. In this case, T is Int, so the return value is an array of integers and the Item associated type is inferred to be Int. 
func makeOpaqueContainer<T>(item: T) -> some Container {
    return [item]
}
let opaqueContainer = makeOpaqueContainer(item: 12)
let twelve = opaqueContainer[0] // of type `Int`
```

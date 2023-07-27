# Protocals<!-- omit from toc -->

- [Protocol as Types](#protocol-as-types)
  - [Collections of Protocol Types](#collections-of-protocol-types)
- [Delegation](#delegation)
- [Adding Protocol Conformance with an Extension](#adding-protocol-conformance-with-an-extension)
- [Conditionally Conforming to a Protocol](#conditionally-conforming-to-a-protocol)
- [Declaring Protocol Adoption with an Extension](#declaring-protocol-adoption-with-an-extension)
- [Adopting a Protocol Using a Synthesized Implementation](#adopting-a-protocol-using-a-synthesized-implementation)
- [Protocol Inheritance](#protocol-inheritance)
- [Class-Only Protocols](#class-only-protocols)
- [Protocol Composition](#protocol-composition)
- [Checking for Protocol Conformance](#checking-for-protocol-conformance)
- [Optional Protocol Requirements](#optional-protocol-requirements)
- [Protocol Extensions](#protocol-extensions)

```swift
protocol SomeProtocol {
    // protocol definition goes here
    var mustBeSettable: Int { get set }
    var doesNotNeedToBeSettable: Int { get }
    static var someTypeProperty: Int { get set }

    static func someTypeMethod()
    func random() -> Double

    init(someParameter: Int)
}

struct SomeStructure: FirstProtocol, AnotherProtocol {
    // structure definition goes here
}

class SomeClass: SomeSuperclass, FirstProtocol, AnotherProtocol {
    // class definition goes here
}
```

## Protocol as Types

- The most common way to use a protocol as a type is as a generic constraint
  - any type conforming to the protocol can be used
  - Specific type is chosen by the code that uses the API
- Code with an opaque type works with some type that conforms to a protocol
  - The underlying type is known at compile time
  - The API implementation chooses that type
  - The type's identity is hidden from clients of the API*
  - Using an opaque type let you prevent implementation details of an API from leaking through the layer of abstraction and only guarentees that the type conforms to the protocol
- Code with a boxed protocal type works with any type, chosen at *runtime*, that conforms to the protocal
  - To support this runtime flexibility, Swift adds a level of indirection when necessary - known as a *box*, which has a performance cost.
  - Swift doesn't know the underlying type at compile time, thus you can only access the memebers that are required by the protocol
  - Accessing any other APIs requires a type casting at runtime

### Collections of Protocol Types

- A protocol can be used as the type to be stored in a collection such as an array or a dictionary
- When using in a loop, the item is of the protocol type and not the concrete type
  - Only members of the protocol can be accessed

```swift
let things: [TextRepresentable] = [game, d12, simonTheHamster]
```

## Delegation

- Delegation is a design pattern that enables a class or structure to hand off (or delegate) some of its responsibilities to an instance of another type
- This design pattern is implemented by defining a protocol that encapsulates the delegated responsibilities, such that a conforming type (known as a delegate) is guaranteed to provide the functionality that has been delegated, without needing to know the exact type of the delegate

```swift
protocol DiceGame {
    var dice: Dice { get }
    func play()
}

// delegate protocol to track the progress of a game
// AnyObject: class-only protocol, letting the `SnakesAndLadders` class declare that its delegate must use a weak  to avoid strong reference cycles
protocol DiceGameDelegate: AnyObject {
    func gameDidStart(_ game: DiceGame)
    func game(_ game: DiceGame, didStartNewTurnWithDiceRoll diceRoll: Int)
    func gameDidEnd(_ game: DiceGame)
}

class SnakesAndLadders: DiceGame {
    let finalSquare = 25
    let dice = Dice(sides: 6, generator: LinearCongruentialGenerator())
    var square = 0
    var board: [Int]
    init() {
        board = Array(repeating: 0, count: finalSquare + 1)
        board[03] = +08; board[06] = +11; board[09] = +09; board[10] = +02
        board[14] = -10; board[19] = -11; board[22] = -02; board[24] = -08
    }
    // weak reference
    weak var delegate: DiceGameDelegate? // no need specific type name
    func play() {
        square = 0
        delegate?.gameDidStart(self)
        gameLoop: while square != finalSquare {
            let diceRoll = dice.roll()
            delegate?.game(self, didStartNewTurnWithDiceRoll: diceRoll)
            switch square + diceRoll {
            case finalSquare:
                break gameLoop
            case let newSquare where newSquare > finalSquare:
                continue gameLoop
            default:
                square += diceRoll
                square += board[square]
            }
        }
        delegate?.gameDidEnd(self)
    }
}

class DiceGameTracker: DiceGameDelegate {
    var numberOfTurns = 0
    func gameDidStart(_ game: DiceGame) {
        numberOfTurns = 0
        if game is SnakesAndLadders { // type casting
            print("Started a new game of Snakes and Ladders")
        }
        print("The game is using a \(game.dice.sides)-sided dice")
    }
    func game(_ game: DiceGame, didStartNewTurnWithDiceRoll diceRoll: Int) {
        numberOfTurns += 1
        print("Rolled a \(diceRoll)")
    }
    func gameDidEnd(_ game: DiceGame) {
        print("The game lasted for \(numberOfTurns) turns")
    }
}
```

## Adding Protocol Conformance with an Extension

- Use an extension to add protocol conformance to a type that is declared elsewhere, or even to a type that you imported from a library or framework
- Existing instances of a type automatically adopt and conform to a protocol when that conformance is added to the instance's type in an extension

## Conditionally Conforming to a Protocol

- A *generic* type may be able to satisfy the requirements of a protocol only under certain conditions, such as when the type's generic parameter conforms to the protocol
- The conditions are be specified by generic `where` clauses

```swift
extension Array: TextRepresentable where Element: TextRepresentable {
    var textualDescription: String {
        let itemsAsText = self.map { $0.textualDescription }
        return "[" + itemsAsText.joined(separator: ", ") + "]"
    }
}
```

## Declaring Protocol Adoption with an Extension

- If a type already conforms to all of the requirements of a protocol, but has not yet stated that it adopts that protocol, you can make it adopt the protocol with an empty extension

```swift
struct Hamster {
    var name: String
    var textualDescription: String {
        return "A hamster named \(name)"
    }
}

extension Hamster: TextRepresentable {}
```

## Adopting a Protocol Using a Synthesized Implementation

- Swift can automatically provide the protocol conformance for `Equatable`, `Hashable`, `Comparable`, and `Codable` protocols in many simple cases
- Using synthesized implementations avoids writing boilerplate code

```swift
struct Vector3D: Equatable {
    var x = 0.0, y = 0.0, z = 0.0
}
```

## Protocol Inheritance

- A protocol can inherit one or more other protocols and can add further requirements on top of the requirements it inherits

```swift
protocol InheritingProtocol: SomeProtocol, AnotherProtocol {
    // protocol definition goes here
}
```

## Class-Only Protocols

- limit protocol adoption to class types (and not structures or enumerations) by adding the `AnyObject` protocol to a protocol's inheritance list
- Use a class-only protocol when the behavior defined by that protocol’s requirements assumes or requires that a conforming type has reference semantics rather than value semantics

```swift
protocol SomeClassOnlyProtocol: AnyObject, SomeInheritedProtocol {
    // class-only protocol definition goes here
}
```

## Protocol Composition

- Combine multiple protocols into a single requirement
- Protocol composition behaves as if you defined a temporary local protocol that has the combined requirements of all protocols in the composition
- Protocol composition doesn't define a new protocol type
- Syntax: as many protocols as you need, separated by ampersands (`&`)
  - `SomeProtocol & AnotherProtocol`
  - in addition to list of protocols, can also include one class type which is the required superclass

## Checking for Protocol Conformance

- Use the `is` and `as` (`as?`, `as!`) operators to check for protocol conformance, and to cast to a specific protocol
- The casted type is of the protocol type and not the concrete type

## Optional Protocol Requirements

- Optional requirements are prefixed by the `optional` modifier
- Optional requirements are available so that you can write code that interoperates with Objective-C
- Both the protocol and the optional requirement must be marked with the `@objc` attribute
- `@objc` protocols can be adopted only by classes
- when you use a method or property in an optional requirement, its type automatically becomes an optional
  - A method of type `(Int) -> String` becomes `((Int) -> String)?` (the entire function type is wrapped in the optional, not the method’s return value)
- An optional protocol requirement can be called with optional chaining

```swift
@objc protocol CounterDataSource {
    @objc optional func increment(forCount count: Int) -> Int
    @objc optional var fixedIncrement: Int { get }
}

class Counter {
    var count = 0
    var dataSource: CounterDataSource?
    func increment() {
        if let amount = dataSource?.increment?(forCount: count) {
            count += amount
        } else if let amount = dataSource?.fixedIncrement {
            count += amount
        }
    }
}
```

## Protocol Extensions

- Use protocol extensions to provide a default implementation to method, initializer, subscript, and computed property requirements
  - If a conforming type provides its own implementation of a required method or property, that implementation will be used instead of the one provided by the extension
- Protocol extensions can't make a protocol extend or inherit from another protocol
  - Protocol inheritance is always specified in the protocol declaration itself
- You can also specify constraints that conforming types must satisfy before they can use the protocol extension using generic `where` clauses
  - If a conforming type satisfies the requirements for multiple constrained extensions that provide implementations for the same method or property, Swift uses the implementation corresponding to the most specialized constraints.

```swift
extension RandomNumberGenerator {
    func randomBool() -> Bool {
        return random() > 0.5
    }
}

extension Collection where Element: Equatable {
    func allEqual() -> Bool {
        for element in self {
            if element != self.first {
                return false
            }
        }
        return true
    }
}
```

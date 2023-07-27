# Memory Safety<!-- omit from toc -->

- [Access Conflicts](#access-conflicts)
- [Conflicting Access to In-Out Parameters](#conflicting-access-to-in-out-parameters)
- [Conflicting Access to self in Methods](#conflicting-access-to-self-in-methods)
- [Conflicting Access to Properties](#conflicting-access-to-properties)

## Access Conflicts

- Three characteristics of memory access to consider in the context of conflicting access:
  - whether the access is a read or a write: a write access changes the location in memory while a read access doesn't
  - the duration of the access: either instantaneous or long-term
    - most memory access is instantaneous and by their nature, two instantaneous accesses can't conflict
    - it's possible to run other code in between the start and end of the long term access, which is called *overlap*; a long term access can overlap with other long term accesses or instantaneous accesses
    - long term access appears primarily in code that uses in-out parameters in functions and methods or mutating methods of a structure
  - the location in memory being accessed: what is being accessed (a variable, property, or subscript)
- A conflict occurs when two accesses that meet all of the following conditions:
  - At least one is a write access or a nonatomic access
  - They access the same location in memory
  - Their durations overlap
- An operation is atomic if it uses only C atomic operations; otherwise it’s nonatomic. For a list of those functions, see the stdatomic(3) man page

## Conflicting Access to In-Out Parameters

- A function has long-term write access to all of its in-out parameters
- The write access for an in-out parameter starts after all of the non-in-out parameters have been evaluated and lasts for the entire duration of that function call
- If multiple in-out parameters, the write accesses start in the same order as the parameters appear
- One consequence of this long-term write access is that you can’t access the original variable that was passed as in-out, even if scoping rules and access control would otherwise permit it
- Another consequence is that you can’t pass a single variable as the argument for multiple in-out parameters of the same function
- Because operators are functions, they can also have long-term accesses to their in-out parameters

```swift
var stepSize = 1

func increment(_ number: inout Int) {
    number += stepSize // read access to stepSize
}

increment(&stepSize) // long term write access to stepSize
// Error: conflicting accesses to stepSize

func balance(_ x: inout Int, _ y: inout Int) {
    let sum = x + y
    x = sum / 2
    y = sum - x
}
var playerOneScore = 42
var playerTwoScore = 30
balance(&playerOneScore, &playerTwoScore) // OK
balance(&playerOneScore, &playerOneScore) // Error: conflicting accesses to playerOneScore
```

## Conflicting Access to self in Methods

- A mutating method on a structure has long-term write access to self during the method call

```swift
struct Player {
    var name: String
    var health: Int
    var energy: Int

    static let maxHealth = 10
    mutating func restoreHealth() {
        health = Player.maxHealth
    }
}

extension Player {
    mutating func shareHealth(with teammate: inout Player) {
        balance(&teammate.health, &health)
    }
}

var oscar = Player(name: "Oscar", health: 10, energy: 10)
var maria = Player(name: "Maria", health: 5, energy: 10)
oscar.shareHealth(with: &maria) // OK

oscar.shareHealth(with: &oscar) // Error: conflicting accesses to oscar
```

## Conflicting Access to Properties

- Types like structures, tuples, and enumerations are made up of individual constituent values, such as the properties of a structure or the elements of a tuple
- Because these types are value types, mutating any piece of the value mutates the whole value, meaning long term write access to the whole value

```swift
var playerInformation = (health: 10, energy: 20)
balance(&playerInformation.health, &playerInformation.energy) // Error: conflicting access to properties of playerInformation

var holly = Player(name: "Holly", health: 10, energy: 10)
balance(&holly.health, &holly.energy) // Error: conflicting access to properties of holly
```

- In practice, most access to properties of a structure can overlap safely
- If the variable is local, the compiler can prove that overlapping accesses to stored properties of a structure is safe

```swift
func someFunction() {
    // The compiler can prove that memory safety is preserved because the two stored properties don’t interact in any way.
    var oscar = Player(name: "Oscar", health: 10, energy: 10)
    balance(&oscar.health, &oscar.energy) // OK
}
```

Memory safety is the desired guarantee, but exclusive access is a stricter requirement than memory safety — which means some code preserves memory safety, even though it violates exclusive access to memory. Swift allows this memory-safe code if the compiler can prove that the nonexclusive access to memory is still safe. Specifically, it can prove that overlapping access to properties of a structure is safe if the following conditions apply:

- You’re accessing only stored properties of an instance, not computed properties or class properties.
- The structure is the value of a local variable, not a global variable.
- The structure is either not captured by any closures, or it’s captured only by nonescaping closures.

If the compiler can’t prove the access is safe, it doesn’t allow the access.

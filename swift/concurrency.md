# Concurrency<!-- omit from toc -->

- [Concurrency with Completion Handlers](#concurrency-with-completion-handlers)
- [Defining and Calling Asynchronous Functions](#defining-and-calling-asynchronous-functions)
- [Places allowed to call asynchronous functions](#places-allowed-to-call-asynchronous-functions)
- [Asynchronous Sequences](#asynchronous-sequences)
- [Calling Asynchronous Functions in Parallel](#calling-asynchronous-functions-in-parallel)
  - [`async let` vs `await`](#async-let-vs-await)
- [Tasks and Task Groups](#tasks-and-task-groups)
- [Unstructured Concurrency](#unstructured-concurrency)
- [Task Cancelation](#task-cancelation)
- [Actors](#actors)
- [Sendable Types](#sendable-types)
  - [Non-Sendable](#non-sendable)

> The concurrency model in Swift is built on top of threads, but you don’t interact with threads directly. An asynchronous function can give up the thread that it’s running on, which lets another asynchronous function run on that thread while the first function is blocked. When an asynchronous function resumes, Swift doesn’t make any guarantees about which thread it will run on.

## Concurrency with Completion Handlers

Concurrency with completion handlers is a way to write asynchronous code, but the code tends to be difficult to read and maintain

```swift
import Foundation

// when running this code in a playground
// import PlaygroundSupport
// PlaygroundPage.current.needsIndefiniteExecution = true


struct CatFact: Codable {
    let fact: String
}

func getCatFact(completion: @escaping ((String) -> Void)) {
    let reqURL = URL(string: "https://catfact.ninja/fact")!
    URLSession.shared.dataTask(with: URLRequest(url: reqURL)) { data, response, error in
        guard let factData = data,
              let catFact = try? JSONDecoder().decode(CatFact.self, from: factData) else {
            completion("No cat facts today.")
            return
        }
        completion("Some random fact: " + catFact.fact)
    }.resume()
    print(reqURL)
}


print("start")
getCatFact {
    print($0)
}
print("end")

// start
// https://catfact.ninja/fact
// end
// Some random fact: The technical term for a cat’s hairball is a “bezoar.”
```

## Defining and Calling Asynchronous Functions

An *asynchronous function* is a function that can suspend execution while it waits for something.

- Use the `async` keyword to define an asynchronous function after its parameters and before the return arrow
  - Write `async` before `throws` for a throwing asynchronous function
- Use the `await` keyword to call an asynchronous function (immediately preceding the function call)
  - Use `try` before `await` for a throwing asynchronous function
- Use `await` to indicate *all* places where the code can suspend in an asynchronous function
- When calling an asynchronous function, execution suspends until the function returns

```swift
func listPhotos(inGallery name: String) async throws -> [String] {
    let result = // some async operation
    return result
}

let photoNames = try await listPhotos(inGallery: "Summer Vacation")
```

> The `Task.sleep(until:tolerance:clock:)` method is useful for testing asynchronous code. It waits at least the given number of nano seconds before it returns.

## Places allowed to call asynchronous functions

The possible suspension points in your code marked with `await` indicate that the current piece of code might pause execution while waiting for the asynchronous function or method to return. This is also called **yielding the thread** because, behind the scenes, Swift suspends the execution of your code on the current thread and runs some other code on that thread instead. Because code with `await` needs to be able to suspend execution, only certain places in your program can call asynchronous functions or methods:

- Code in the body of an asynchronous function, method, or property.
- Code in the static `main()` method of a structure, class, or enumeration that’s marked with `@main`.
- Code in an **unstructured child task** as described in [Unstructured Concurrency](#unstructured-concurrency).

## Asynchronous Sequences

- An *asynchronous sequence* is a sequence that can suspend execution while it waits for the next value to be available.
  - Custom asynchronous sequences can be created by defining a type that conforms to the [`AsyncSequence`](https://developer.apple.com/documentation/swift/asyncsequence) protocol.
- An asynchronous sequence can be used in a `for-await-in` loop, which potentially suspends execution at the beginning of each iteration when it waits for the next value to be available.

## Calling Asynchronous Functions in Parallel

- Calling an asynchronous function with `await` runs only one piece of code at a time. while the asynchronous function is running, the caller waits for it to return.
- To call an asynchronous function in parallel, write `async` before `let` when defining a constant and write `await` each time you access the constant.

```swift
async let firstPhotoName = downloadPhoto(named: photoNames[0])
async let secondPhotoName = downloadPhoto(named: photoNames[1])
async let thirdPhotoName = downloadPhoto(named: photoNames[2])

let photos = await [firstPhotoName, secondPhotoName, thirdPhotoName]
```

### `async let` vs `await`

- Call an asynchronous function with `await` when the following code depends on the result of the function.
- Call an asynchronous function with `async let` when the result of the function is needed later in the code, but not immediately.
- Both `async let` and `await` allow other coe to run while they're suspended.
- In both cases, use `await` to mark the possible suspension points

## Tasks and Task Groups

- A *task* is a unit of work that can be run asynchronously.
  - All asynchronous code runs as part of some task.
  - The `aync let` creates a child task
- A [*task group*](https://developer.apple.com/documentation/swift/taskgroup) is a group of tasks. Child tasks can be added to a task group, which gives you more control over priority and cancellation, and lets you create a dynamic number of tasks.
- **Structured concurrency**: tasks are arranged in a hierarchy. Each task in a task group has the same parent task, and each task can have its own child tasks.
  - Structured concurrency let Swift handle some behaviors like propagating cancellation and detect some errors at compile time.

```swift
await withTaskGroup(of: Data.self) {  taskGroup in
    let photoNames = await listPhotos(inGallery: "Summer Vacation")
    for name in photoNames {
        taskGroup.addTask { await downloadPhoto(named: name) }  
    }
}
```

## Unstructured Concurrency

- An *unstructured task* doesn’t have a parent task
- You have complete flexibility to manage unstuctured tasks, in exchange for more responsibility for correctness
- Use `Task.init(priority:operation:)` to create an unstructured task
- Use `Task.detached(priority:operation:)` to create a detached task, which is an unstructured task that's not part of the current actor

```swift
let newPhoto = // some photo data
let handle = Task {
    return await add(newPhoto), toGalleryNamed: "Summer Vacation")
}
let result = await handle.value
```

## Task Cancelation

Swift concurrency uses a cooperative cancellation model. Each task checks whether it has been canceled at the appropriate points in its execution, and responds to cancellation in whatever way is appropriate.

- Throwing an error like `CancellationError`
- Returning `nil` or an empty collection
- Returning the partially completed work

To check for cancellation:

- Call `Task.checkCancellation()` which throws `CancellationError` if the task has been canceled
- Check `Task.isCancelled` property

To cancel a task, call `Task.cancel()`.

## Actors

- Actors let you safely share information between concurrent code.
- Actors are reference types. Unlike classes, actors allow only one taks to access their mutable state at a time.
- Actors are defined with the `actor` keyword.
- When you access a property or method of an actor, you use `await` to mark the potential suspension point.
  - Because only one task can access an actor’s mutable state at a time, the access could be suspended.
- Code that's part of the actor can access the actor's mutable state without `await`.
- Swift guarantees that only code inside an actor can access the actor’s *local state*. This guarantee is known as **actor isolation**.
  - Properties of an actor are part of its local state.

```swift
// Actor definition
actor TemperatureLogger {
    let label: String
    var measurements: [Int]
    private(set) var max: Int // private settable

    init(label: String, measurement: Int) {
        self.label = label
        self.measurements = [measurement]
        self.max = measurement
    }

    func update(measurement: Int) async {
        measurements.append(measurement)
        if measurement > max {
            max = measurement // no await needed
}

// Actor instantiation
let logger = TemperatureLogger(label: "Outdoors", measurement: 25)
print(await logger.max) // potential suspension point
```

## Sendable Types

- A **concurrency domain** is the part of a program that contains mutable state, like variables and properties, inside of a task or an instance of an actor.
- A **sendable type** is a type that can be safely passed between concurrency domains.
  - For example, it can be passed as an argument when calling an actor method or returned as a result of a task.
  - A class that contains mutable properties and doesn’t serialize access to them isn’t sendable.
- A type is marked as sendable by conforming to the [`Sendable`](https://developer.apple.com/documentation/swift/sendable) protocol which has no code requirements but semantic requirements.

There are three ways for a type to be sendable:

- The type is a value type, and its mutable state is made up of only sendable types.
  - A struct with stored properties that are sendable
  - An enum with associated values that are sendable
- The type doesn’t have any mutable state, and its immutable state is made up of only sendable types.
  - A struct or class that has only read-only properties
- The type has code that ensures the safety of its mutable state, like a class that's marked with `@MainActor` or a class that serializes access to its properties on a particular thread or queue.

Some types are always sendable and comformance to `Sendable` is implicit. Because TemperatureReading is a structure that has only sendable properties, and the structure isn’t marked public or @usableFromInline, it’s implicitly sendable.

```swift
struct TemperatureReading { // implicitly Sendable
    var measurement: Int
}

extension TemperatureLogger {
    func addReading(from reading: TemperatureReading) {
        measurements.append(reading.measurement)
    }
}

let logger = TemperatureLogger(label: "Outdoors", measurement: 25)
let reading = TemperatureReading(measurement: 27)
await logger.addReading(from: reading) // only sendable types
```

### Non-Sendable

To explicitly mark a type as not being sendable, overriding an implicit conformance to the Sendable protocol, use an extension:

```swift
struct FileDescriptor { // implicitly Sendable
    let rawValue: CInt
}

@available(*, unavailable) // make the conformance to `Sendable` unavailable
extension FileDescriptor: Sendable { }
```

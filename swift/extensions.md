# Extensions<!-- omit from toc -->

- [Extension Syntax](#extension-syntax)

- Add new functionality to an existing class, structure, enumeration, or protocol type
  - Even for types you do not have access to the original source code (known as retroactive modeling)
- Extensions in Swift can:
  - Add computed instance properties and computed type properties
    - No stored properties or property observers to existing properties
  - Define instance methods and type methods
    - including mutating instance methods
  - Provide new initializers
    - No designated initializers or deinitializers to classes
    - Defualt initializers and memberwise initializers can be called in an extension (if available)
    - If an extension is to add an initializer to a structure that was declared in another module, the new initializer can't access `self` until it calls an initializer from the defining module
  - Define subscripts
  - Define and use new nested types
  - Make an existing type conform to a protocol
  - Extend a protocol to provide implementations of its requirements or add additional functionality that conforming types can take advantage of
- Extensions can add new functionality to a type, but they cannot override existing functionality
- New functionality provided by an extension will be available on all existing instances of that type, even if they were created before the extension was defined

## Extension Syntax

```swift
extension SomeType: SomeProtocol, AnotherProtocol {
    // implementation of protocol requirements goes here
}
```

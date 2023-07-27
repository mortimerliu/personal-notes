# Access Control<!-- omit from toc -->

- [Modules and Source Files](#modules-and-source-files)
- [Access Levels](#access-levels)
- [Guiding Principle of Access Levels](#guiding-principle-of-access-levels)
- [Access Control Syntax](#access-control-syntax)
- [Custom Types](#custom-types)
- [Tuple Types](#tuple-types)
- [Function Types](#function-types)
- [Enumeration Types](#enumeration-types)
- [Nested Types](#nested-types)
- [Subclassing](#subclassing)
- [Constants, Variables, Properties, and Subscripts](#constants-variables-properties-and-subscripts)
- [Getters and Setters](#getters-and-setters)
- [Initializers](#initializers)
  - [Default Initializers](#default-initializers)
- [Protocols](#protocols)
  - [Protocol Inheritance](#protocol-inheritance)
  - [Protocol Conformance](#protocol-conformance)
- [Extensions](#extensions)
  - [Private Members in Extensions](#private-members-in-extensions)
- [Generics](#generics)
- [Type Aliases](#type-aliases)

## Modules and Source Files

- A ***module*** is a single unit of code distribution - a framework or application that is built and shipped as a single unit and that can be imported by another module with Swift’s import keyword
- Each build target (such as an app bundle or framework) in Xcode is treated as a separate module in Swift
- A ***source file*** is a single Swift source code file within a module (in effect, a single file within an app or framework)

## Access Levels

Swift provides five different access levels for entities within your code

- *Open access* and *public access* enable entities to be used within any source file from their defining module, and also in a source file from another module that imports the defining module
  - Open access applies only to classes and classes members
  - Open access differs from public access by allowing code outside the module to subclass and override, indicating that you intend for the class to be subclassed
- *Internal access* enables entities to be used within any source file from their defining module
- *File-private access* restricts the use of an entity to its own defining source file
- *Private access* restricts the use of an entity to the enclosing declaration, and to extensions of that declaration that are in the same file

## Guiding Principle of Access Levels

- No entity can be defined in terms of another entity that has a lower (more restrictive) access level
- Default access level is ***internal*** (with a few exceptions)
- Access level for single-target apps: typically don't need to specify explicit access control levels
- Access level for frameworks: mark the public-facing interface to that framework as open or public so that it can be viewed and accessed by other modules, such as an app that imports the framework
- Access level for unit test targets: a unit test target can access any internal entity, if you mark the import declaration for a product module with the @testable attribute and compile that product module with testing enabled

## Access Control Syntax

- Use the `open`, `public`, `internal`, `fileprivate`, or `private` modifier to specify an explicit access level for an entity

## Custom Types

- The access control level of a type affects the default access level of that type’s members (its properties, methods, initializers, and subscripts)
- If you define a type’s access level as private or file private, the default access level of its members will also be private or file private
- If you define a type’s access level as internal or public, the default access level of the type’s members will be internal
  - By making a public type member internal by default, you can prevent the member to be exposed as part of the type’s public interface by mistake

## Tuple Types

- The access level for a tuple type is the most restrictive access level of all types used in that tuple
- A tuple type’s access level can’t be specified explicitly

## Function Types

- The access level for a function type is calculated as the most restrictive access level of the function’s parameter types and return type
- You must specify the access level explicitly as part of the function’s definition if the function’s calculated access level doesn’t match the contextual default

```swift
// don't compile as the contextual default is `internal`
// but the most restrictive access level of the function's parameter types and return type is `private`
func someFunction() -> (SomeInternalClass, SomePrivateClass) {
    // function implementation goes here
}

// fix by specifying the access level explicitly
private func someFunction() -> (SomeInternalClass, SomePrivateClass) {
    // function implementation goes here
}
```

## Enumeration Types

- The individual cases of an enumeration automatically receive the same access level as the enumeration they belong to
- You can’t specify a different access level for individual enumeration cases
- The types used for any raw values or associated values in an enumeration definition must have an access level at least as high as the enumeration’s access level

## Nested Types

- the access level of a nested type is the same as its containing type, unless the containing type is public (which is default to internal)

## Subclassing

- You can subclass any class that can be accessed in the current access context and that's defined in a module
- You can also subclass any open class that's defined in another module
- A subclass can’t have a higher access level than its superclass
- For classes that are defined in the same module, you can override any class members that is visible in a certain access context
- FOr classes that are defined in another module, you can override any open class member
- An override can make an inherited class member more accessible than its superclass version
- A subclass member can call a superclass member that has lower access permissions than the subclass member, as long as the call to the superclass’s member takes place within an allowed access level context

```swift
public class A {
    fileprivate func someMethod() {}
}

internal class B: A {
    override internal func someMethod() {} {
        super.someMethod()
    }
}
```

## Constants, Variables, Properties, and Subscripts

- A constant, variable, or property can’t be more public than its type
- A subscript can’t be more public than either its index type or return type
- If a constant, variable, property, or subscript makes use of a private type, the constant, variable, property, or subscript must also be marked as private

```swift
private var privateInstance = SomePrivateClass()
```

## Getters and Setters

- Getters and setters for constants, variables, properties, and subscripts automatically receive the same access level as the constant, variable, property, or subscript they belong to
- You can give a setter a lower access level than its corresponding getter, to restrict the read-write scope of that variable, property, or subscript by writing `fileprivate(set)` or `private(set)` before the var, or subscript introducer
  - This rule applies to stored properties as well as computed properties. Even though you don’t write an explicit getter and setter for a stored property, Swift still synthesizes an implicit getter and setter for you to provide access to the stored property’s backing storage. Use fileprivate(set), private(set), and internal(set) to change the access level of this synthesized setter in exactly the same way as for an explicit setter in a computed property.

```swift
struct TrackedString {
    private(set) var numberOfEdits = 0
    var value: String = "" {
        didSet {
            numberOfEdits += 1
        }
    }
}

public struct TrackedString {
    public private(set) var numberOfEdits = 0
    public var value: String = "" {
        didSet {
            numberOfEdits += 1
        }
    }
    public init() {}
}
```

## Initializers

- Custom initializers can be assigned an access level less than or equal to the type that they initialize
- A required initializer must have the same access level as the class it belongs to
- The types of an initializer’s parameters cannot be more private than the initializer’s own access level

### Default Initializers

- A default initializer has the same access level as the type it initializes, unless that type is defined as public (which is default to internal)
- If you want a public type to be initializable with a no-argument initializer when used in another module, you must explicitly provide a public no-argument initializer yourself as part of the type’s definition
- Default memberwise initializers for structure types are considered private/fileprivate if any of the structure’s stored properties are private/fileprivate; otherwise, the initializer has an access level of internal

## Protocols

- If you want to assign an explicit access level to a protocol type, do so at the point that you define the protocol
- This enables you to create protocols that can only be adopted within a certain access context
- The access level of each requirement within a protocol definition is automatically set to the same access level as the protocol
- You can't set a protocol requirement to a different access level than the protocol it supports
- If you define a public protocol, the protocol’s requirements require a public access level for those requirements when they’re implemented. This behavior is different from other types, where a public type definition implies an access level of internal for the type’s members.

### Protocol Inheritance

- the new protocol can have at most the same access level as the protocol it inherits from

### Protocol Conformance

- A type can conform to a protocol with a lower access level than the type itself
  - For example, you can define a public type that can be used in other modules, but whose conformance to an internal protocol can only be used within the internal protocol’s defining module
- The context in which a type conforms to a particular protocol is the minimum of the type’s access level and the protocol’s access level
  - For example, if a type is public, but a protocol it conforms to is internal, the type’s conformance to that protocol is also internal.
- When you write or extend a type to conform to a protocol, you must ensure that the type’s implementation of each protocol requirement has at least the same access level as the type’s conformance to that protocol.
  - For example, if a public type conforms to an internal protocol, the type’s implementation of each protocol requirement must be at least internal.

## Extensions

- You can extend a class, structure, or enumeration in any access context in which the class, structure, or enumeration is available
- Any type members added in an extension have the same default access level as type members declared in the original type being extended
- Alternatively, you can mark an extension with an explicit access level modifier (for example, private extension) to set a new default access level for all members defined within the extension
  - This new default can still be overridden within the extension for individual type members
- You can’t provide an explicit access level modifier for an extension if you’re using that extension to add protocol conformance; the protocol’s own access level is used to provide the default access level for each protocol requirement implementation within the extension

### Private Members in Extensions

Extensions that are in the same file as the class, structure, or enumeration that they extend behave as if the code in the extension had been written as part of the original type’s declaration. Therefore, you can

- Declare a private member in the original declaration, and access that member from extensions in the same file.
- Declare a private member in one extension, and access that member from another extension in the same file.
- Declare a private member in an extension, and access that member from the *original declaration* in the same file.

## Generics

- The access level for a generic type or generic function is the minimum of the access level of the generic type or function itself and the access level of any type constraints on its type parameters

## Type Aliases

- Any type aliases you define are treated as distinct types for the purposes of access control
- A type alias can have an access level less than or equal to the access level of the type it aliases
- This rule also applies to type aliases for associated types used to satisfy protocol conformances.

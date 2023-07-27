# Structures and Classes<!-- omit from toc -->

- [Structures vs Classes](#structures-vs-classes)
- [Definition](#definition)
- [Properties](#properties)
  - [Global and Local Variables](#global-and-local-variables)
- [Property Wrappers](#property-wrappers)
  - [Property Wrapper Initialization](#property-wrapper-initialization)
  - [Projected Value](#projected-value)
- [Methods](#methods)
- [Self Type](#self-type)
- [Subscripts](#subscripts)
- [Inheritance](#inheritance)
- [Initialization](#initialization)
  - [Default Initializers](#default-initializers)
  - [Memberwise Initializers for Structure Types](#memberwise-initializers-for-structure-types)
  - [Initializer Delegation for Value Types](#initializer-delegation-for-value-types)
  - [Initializer for Class Types](#initializer-for-class-types)
    - [Initializer Delegation for Class Types](#initializer-delegation-for-class-types)
    - [Two-Phase Initialization](#two-phase-initialization)
    - [Initializer Inheritance and Overriding](#initializer-inheritance-and-overriding)
    - [Automatic Initializer Inheritance](#automatic-initializer-inheritance)
  - [Failable Initializers](#failable-initializers)
    - [Overriding a Failable Initializer](#overriding-a-failable-initializer)
    - [`init!` Failable Initializer](#init-failable-initializer)
  - [Required Initializers](#required-initializers)
  - [Default Property Values with Closure or Function](#default-property-values-with-closure-or-function)
- [Deinitialization](#deinitialization)
- [Nested Types](#nested-types)

## Structures vs Classes

- Structures are value types
    ![value_type](../assets/sharedStateStruct@2x.png)
- Classes are reference types
    ![reference_type](../assets/sharedStateClass@2x.png)

Common features:

- instance methods
- instance stored/computed properties
- static methods
- staic stored/computed properties
- subscripts
- initializers
- extensions
- protocol conformance

Only classes:

- inheritance
- type casting
- deinitializers
- reference counting

## Definition

```swift
struct SomeStructure {
    // structure definition
    var someProperty: Int = 0
}

class SomeClass {
    // class definition
    var someProperty: Int = 0
}
```

## Properties

- Instance stored properties: struct, class
- Instance computed properties: struct, class, enum
- Property observers can be added to:
  - stored properties you define (for computed properties you define, just add code to the setter)
  - stored/computed properties that you inherit, by overriding the property within a subclass
  - property observers aren't called during initialization; property observers of superclass are called when in a subclass initializer after the superclass initializer has been called
  - property observers are always called for an in-out parameter
  - property observers aren't called if the property is set in the property observer itself
- Static stored properties: struct, class
  - all instances share the same copy of the property
  - always need to provide a default value (as they are not initialized through initializers)
  - are initialized lazily on their first access; guaranteed to be initialized only once even by multiple threads simultaneously
- Static computed properties: struct, class
  - `class` keyword for computed properties to allow subclasses to override

```swift
struct SomeStructure { // or class SomeClass
    let storedConstantProperty: Int
    let storedConstantPropertyWithDefaultValue = 0 // value can be set/modified in initializer
    var storedProperty: Int
    var storedPropertyWithDefaultValue = 0

    // lazy property must be var as its initial value might not be retrieved after initialization;
    // used for properties whose initial values is dependent on external factors or 
    // requires complex or computationally expensive setup
    lazy var lazyProperty: Int = 0 

    var computedProperty: Int {
        get {
            // return some value
        }
        set(newValue) { // newValue is the default name and can be omitted -> set { ... }
            // set some value
        }
    }
    var computedReadOnlyProperty: Int {
        // return some value
    } // computed properties always declared as var

    var observedProperty: Int = 0 {
        willSet(newValue) { // newValue is the default name and can be omitted -> willSet { ... }
            // called before the value is stored
        }
        didSet(oldValue) { // oldValue is the default name and can be omitted -> didSet { ... }
            // called after the value is stored
        }
    }

    static let staticStoredProperty: Int = 0
    static var staticComputedProperty: Int { // always declared as var
        // return some value
    }
}

class SomeClass {
    static let staticStoredProperty: Int = 0
    class let classStoredProperty: Int = 0
    // class properties can be overridden by subclasses; only computed properties
    class var classComputedProperty: Int {
        // return some value
    }
}

// class properties are mutable even if the instance is declared as constant
// because `someClass` only stores a reference to the instance, not the instance itself
// that is, the reference is constant, not the instance
let someClass = SomeClass()
someClass.storedProperty = 1
// this is not allowed for structures as they are value types， even for variable properties
let someStructure = SomeStructure()
someStructure.storedProperty = 1 // error

// access static properties
SomeClass.staticStoredProperty
```

### Global and Local Variables

- Global variables are variables defined outside of any function, method, closure, or type context
- Local variables are variables defined within a function, method, or closure context
- Global and local variables can be computed, or can be stored with observers
- Global variables are always computed lazily, without the `lazy` modifier
- Local variables are never computed lazily
- Property wrapper can be applied to local stored variables but not global variables or computed variables

## Property Wrappers

- To define a property wrapper, define a structure, enumeration, or class that is annotated with the `@propertyWrapper` attribute and defines a `wrappedValue` property
- Property wrappers can only be applied to `var` stored properties

```swift
@propertyWrapper
struct TwelveOrLess {
    private var number = 0
    var wrappedValue: Int {
        get { number }
        set { number = min(newValue, 12) }
    }
}

struct SmallRectangle {
    @TwelveOrLess var number: Int // use property wrapper
    @TwelveOrLess var number2 // type inference: number2 is of the same type as the property wrapper's wrappedValue
}

// property wrapper is equivalent to the following
struct SmallRectangle {
    private var _number = TwelveOrLess()
    var number: Int {
        get { _number.wrappedValue }
        set { _number.wrappedValue = newValue }
    }
}
```

### Property Wrapper Initialization

- To use a property wrapper without providing an initial value for the wrapped property, the property wrapper must define a `init()` initializer
- To set an initial value to the wrapped property at declaration, the property wrapper must define an `init(wrappedValue:)` initializer
- Property wrapper can define initializers that take additional arguments besides `wrappedValue`

```swift
@propertyWrapper
struct TwelveOrLess {
    private var number = 0
    var wrappedValue: Int {
        get { number }
        set { number = min(newValue, 12) }
    }
    init() { // no initial value
        print("init()")
    }
    init(wrappedValue: Int) { // initial value
        print("init(wrappedValue:)")
        self.wrappedValue = wrappedValue
    }
    init(wrappedValue: Int, anotherValue: Int) { // initial value and additional arguments
        print("init(wrappedValue:anotherValue:)")
        self.wrappedValue = wrappedValue
    }
}

struct SmallRectangle {
    @TwelveOrLess var height: Int // init()
    @TwelveOrLess var height2 = 10 // init(wrappedValue:)
    @TwelveOrLess(wrappedValue: 10, anotherValue: 5) var height3 // init(wrappedValue:anotherValue:)
    @TwelveOrLess(anotherValue: 5) var height4 = 10 // equivalent

    // swift default memberwise initializer
    // note height and height3 need to be initialized with an TwelveOrLess instance where height2 and height4 can be initialized with an Int
    init(height: TwelveOrLess, height2: Int, height3: TwelveOrLess, height4: Int)
}
```

### Projected Value

- Property wrappers can define a `projectedValue` property to expose additional functionality through a separate type
- Projected value is accessed by placing a `$` before the property name
- Projected value can return any , or can return `self` to expose the wrapper instance itself

```swift
@propertyWrapper
struct SmallNumber {
    private var number: Int
    private(set) var projectedValue: Bool

    var wrappedValue: Int {
        get { return number }
        set {
            if newValue > 12 {
                number = 12
                projectedValue = true
            } else {
                number = newValue
                projectedValue = false
            }
        }
    }

    init() {
        self.number = 0
        self.projectedValue = false
    }
}
struct SomeStructure {
    @SmallNumber var someNumber: Int
}
var someStructure = SomeStructure()

someStructure.someNumber = 4
print(someStructure.$someNumber) // false

someStructure.someNumber = 55
print(someStructure.$someNumber) // true
```

## Methods

- Instance methods: struct, class, enum
  - use `self` to refer to the current instance within its own instance methods
  - use `mutating` keyword to mark a method that modifies the instance of a value type
  - can assign a completely new instance to the implicit `self` property within the mutating method (or a different case for an enumeration)
  - `mutating` methods can't be called on constants
- Static methods: struct, class, enum
  - use `self` to refer to the type itself within its own static methods

```swift
class SomeClass {
    var someProperty: Int = 0
    func instanceMethod(someParameter: Int) {
        // instance method implementation
        self.someProperty = someParameter
    }
    static func staticMethod() {
        // static method implementation
    }

    class func classMethod() {
        // class method implementation
    } // allow subclasses to override
}

struct SomeStructure {
    var someProperty: Int = 0
    mutating func instanceMethod(someParameter: Int) {
        self.someProperty = someParameter
    }
}
```

## Self Type

- `Self` type refers to the current type without needing to know the type's name at compile time
- In protocol declaration, the `Self` type refers to the type that conforms to the protocol
- In struct, enum, or class declaration, the `Self` type refers to the type introduced by the declaration.
- In the members of a class declaration, the `Self` type can appear only as follows:
  - As the result type in a method
  - As the result type of a read-only subscript
  - As the type of a read-only computed property
  - In the body of a method
- Inside a nested type declaration, the `Self` type refers to the innermost type declaration
- `Self.someStaticMember` is equivalent to `type(of: self).someStaticMember`

```swift
class Superclass {
    func f() -> Self { return self }
}

let x = Superclass()
print(type(of: x.f()))
// Prints "Superclass"

class Subclass: Superclass { }
let y = Subclass()
print(type(of: y.f()))
// Prints "Subclass"

let z: Superclass = Subclass()
print(type(of: z.f()))
// Prints "Subclass"
```

## Subscripts

- Instance subscripts: struct, class, enum
  - Subscripts are shortcuts for accessing the member elements of a collection, list, or sequence
  - multiple subscripts can be defined for a single type; the appropriate subscript overload to use is selected based on the type of index value
  - subscripts can have multiple parameters of any type and return values of any type
  - subscripts can use variadic parameters and default values, but can't use in-out parameters
  - subscripts can be read-write or read-only
- Type subscripts: struct, class, enum
  - use the `static` keyword to indicate that a subscript is a type subscript
  - use the `class` keyword to allow subclasses to override
  
```swift
subscript(index: Int) -> Int {
    get {
        // return an appropriate subscript value here
    }
    set(newValue) { // newValue is the default name and can be omitted -> set { ... }
        // perform a suitable setting action here
    }
}
```

## Inheritance

- Only classes can inherit
- Only single inheritance
- overrides with `override` keyword:
  - override methods: instance/class (not static) methods can be overridden
  - override properties: instance/class (not static) stored and computed properties can be overridden to provide getter/setter or add property observers
    - read-only property can be overridden as read-write property; read-write property can't be overridden as read-only property (such that subclass can be used in places where the superclass is expected)
    - getter must be overridden if setter is overridden
    - property observers can't be added to inherited constant stored properties or read-only computed properties (the value can't be set)
    - property observers can't be added if the setter is overridden at the same time (add codes to the setter instead)
  - override subscripts: instance/class (not static) subscripts can be overridden
  - all `static` properties/methods/subscripts can't be overridden
  - prevent overrides: use `final` keyword to prevent a method, property, subscript, or entire class from being overridden
- Accessing superclass using `super`
  - `super.someMethod()`
  - `super.someProperty`
  - `super[someIndex]`

```swift
class SomeSuperclass {
    var superClassProperty: Int = 0
    func superClassMethod() {
        // method implementation goes here
    }
}

class SomeClass: SomeSuperclass, FirstProtocol, AnotherProtocol {
    // class definition goes here
    override func superClassMethod() {
        // override method implementation goes here
        super.superClassMethod()
    }
}
```

## Initialization

- struct, class, enum can provide one or more initializers
  - Initializers have the same syntax as functions except declared with `init` keyword, don't have function name, and don't have a return type
- **Classes and structures must set all of their stored properties either in an initializer or by assigning a default value**
  - optional properties are automatically initialized as `nil`

```swift
init() {
    // perform some initialization here
}
```

### Default Initializers

- Structures and classes automatically receive a default initializer if they provide default values for all of their properties and don't provide at least one initializer themselves
  - Optional property types automatically receive a default value of `nil`

### Memberwise Initializers for Structure Types

- Structures automatically receive a memberwise initializer if they don't define any of their own custom initializers
- The memberwise initializer's parameters are the same as the names of the structure's stored properties

### Initializer Delegation for Value Types

- **Initializers for value types (struct, enum) can only delegate to another initializer that they provide themselves** (as they can't inherit)
- Use `self.init` to refer to other initializers from the same value type only from within an initializer
- Note if you define a custom initializer for a value type, you will no longer have access to the default initializer or the memberwise initializer
  - If you want your custom value type to be initializable with the default initializer and memberwise initializer, and also with your own custom initializers, write your custom initializers in an extension rather than as part of the value type’s original implementation.

```swift
struct Rect {
    var origin = Point()
    var size = Size()
    init() {}
    init(origin: Point, size: Size) {
        self.origin = origin
        self.size = size
    }
    init(center: Point, size: Size) {
        let originX = center.x - (size.width / 2)
        let originY = center.y - (size.height / 2)
        self.init(origin: Point(x: originX, y: originY), size: size)
    }
}

// or use extension

struct Rect {
    var origin = Point()
    var size = Size()
    // default initializer init() {}
    // memberwise initializer init(origin: Point, size: Size) {}
}

extension Rect {
    init(center: Point, size: Size) {
        let originX = center.x - (size.width / 2)
        let originY = center.y - (size.height / 2)
        self.init(origin: Point(x: originX, y: originY), size: size)
    }
}
```

### Initializer for Class Types

- All stored properties of a class - including any properties the class inherits from its superclass - must be assigned an initial value during initialization
- Two kinds of initializers for class types:
  - Designated initializers: primary initializers for a class
    - Every class must have at least one designated initializer; can be inherited from superclass
    - Designated initializers must ensure all of the properties introduced by that class are initialized before delegating up to a superclass initializer
    - Designated initializers must delegate up to a superclass initializer before assigning a value to an inherited property
  - Convenience initializers: secondary initializers
    - Convenience initializers has the `convenience` keyword before the `init` keyword
    - Convenience initializers must delegate to another initializer before assigning a value to any property (including properties defined by the same class)
    - Convenience initializers can't be inherited
    - Convenience initializers can't be overridden
    - Convenience initializers can't define any required initializers
- An Initializer can't call any instance methods, read the values of any instance properties, or refer to `self` as a value until after the **first phase** of initialization is complete

#### Initializer Delegation for Class Types

- A designated initializer must call a designated initializer from its *immediate* superclass
- A convenience initializer must call another initializer from the *same* class
- A convenience initializer must ultimately call a designated initializer

That is

- Designated initializers must always delegate up
- Convenience initializers must always delegate across

![initializer_delegation](../assets/initializerDelegation01@2x.png)

#### Two-Phase Initialization

- Phase 1: Each stored property is assigned an initial value by the class that introduced it
  - A designated or convenience initializer is called on the class
  - Memory for a new instance of that class is allocated but not yet initialized
  - A designated initializer for that class confirms that all stored properties introduced by that class have a value; memory is not initialized
  - A designated initializer hands off to a superclass initializer to perform the same task for its own stored properties
  - This continues up the class inheritance chain until the top of the chain is reached
  - Once the top of the chain is reached, and the final class in the chain has ensured that all of its stored properties have a value, the instance's memory is considered to be fully initialized, and phase 1 is complete
- Phase 2: Each class is given the opportunity to customize its stored properties further before the new instance is considered ready for use
  - Working back down from the top of the chain, each designated initializer in the chain has the option to customize the instance further
    - Initializers are now able to access `self` and can modify its properties, call its instance methods, and so on
  - Finally, any convenience initializers in the chain have the option to customize the instance and to work with `self`

#### Initializer Inheritance and Overriding

- Swift subclasses don't inherit their superclass initializers by default
- Swift subclasses can override superclass *designated* initializers to provide an alternative initializer by using the `override` keyword
- Swift subclasses can write a custom initializer that matches a superclass *convenience* initializer, but no `override` keyword is required (as convenience superclass initializers can't be called in subclass)
- Superclass initializers are called by `super.init`
  - If a subclass initializer performs no customization in phase 2 of the initialization process, and the superclass has a synchronous, zero-argument designated initializer, you can omit a call to `super.init()` after assigning values to all of the subclass's stored properties
  - If the superclass's initializer is asynchronous, you need to write `await super.init()` explicitly
- Subclasses can modify inherited variable properties during initialization, but can't modify inherited constant properties

#### Automatic Initializer Inheritance

- Subclasses provide a default value for any new properties introduced by the subclass
- If subclass doesn't define any designated initializers, it automatically inherits all of its superclass designated initializers
- If your subclass provides an implementation of *all* of its superclass designated initializers — either by inheriting them as per rule 1, or by providing a custom implementation as part of its definition — then it automatically inherits all of the superclass convenience initializers.
  - A subclass can implement a superclass designated initializer as a subclass convenience initializer as part of satisfying rule 2

### Failable Initializers

- struct, class, enum can define failable initializers
- Failable initializers return an optional value of the type they initialize
- `init?` keyword and `return nil` to indicate a failable initializer
- Failable and nonfailable initializers can't have the same parameter types and names
- Enumerations with raw values automatically receive a failable initializer, `init?(rawValue:)`
- A failable initializer can delegate across to another failable initializer from the same class/struct/enum
- A subclass failable initializer can delegate up to a superclass failable initializer
- If any failable initializer fails, the entire initialization process fails immediately, and no further initialization code is executed
- A failable initializer can also delegate to a nonfailable initializer
  
```swift
enum TemperatureUnit {
    case kelvin, celsius, fahrenheit
    init?(symbol: Character) {
        switch symbol {
        case "K":
            self = .kelvin
        case "C":
            self = .celsius
        case "F":
            self = .fahrenheit
        default:
            return nil
        }
    }
}

// or enum with raw values
enum TemperatureUnit: Character {
    case kelvin = "K", celsius = "C", fahrenheit = "F"
    // automatically receive a failable initializer, init?(rawValue:)
}
```

#### Overriding a Failable Initializer

- You can override a superclass failable initializer in a subclass, with either a failable or nonfailable initializer
  - If nonfailable, you need to use forced unwrapping to call the superclass failable initializer
- You can't override a superclass nonfailable initializer with a failable initializer

```swift
class Document {
    var name: String?
    init() {}
    init?(name: String) {
        if name.isEmpty { return nil }
        self.name = name
    }
}

class UntitledDocument: Document {
    override init() {
        super.init(name: "[Untitled]")! // forced unwrapping
    }
}
```

#### `init!` Failable Initializer

- define a failable initializer that creates an implicitly unwrapped optional instance of the appropriate type
- `init!` can delegate to `init?` and vice versa
- `init!` can override `init?` and vice versa
- `init` can delegate to `init!` but could trigger a runtime error

### Required Initializers

- Write the `required` modifier before the definition of a class initializer to indicate that every subclass of the class must implement that initializer
- Subclasses must write the `required` modifier as well, `override` keyword is not required when overriding a required designated initializer

### Default Property Values with Closure or Function

- You can use a closure or global function to provide a default property value
- Whenever a new instance of the type that the property belongs to is initialized, the closure or function is called, and its return value is assigned as the property's default value
- The closure or function you assign to the property can't use any instance methods or reference `self` in any way, because the instance for which you are setting a default property value might not exist yet

```swift
class SomeClass {
    let someProperty: Int = {
        // create a default value for someProperty inside this closure
        // someValue must be of the same type as SomeType
        return someValue
    }()
}
```

## Deinitialization

- Only classes can have at most one deinitializers
- use `deinit` keyword to define a deinitializer
- A deinitializer is called immediately before a class instance is deallocated
- Use deinitializers to perform custom cleanup besides memory management provided by ARC
- Superclass deinitializers are inherited by their subclasses, and the superclass deinitializer is called automatically at the end of a subclass deinitializer implementation
- Deinitializers are called automatically, and aren't called manually
- Deinitializers can access all properties of the instance they are called on (as the instance hasn't been deallocated yet)

```swift
deinit { // no paratheses
    // perform the deinitialization
}
```

## Nested Types

- Nest supporting enumerations, classes, and structures within the definition of the type they support
- Refer to nested types by prefixing their names with the name of the type they are nested within

```swift
struct BlackjackCard {


    // nested Suit enumeration
    enum Suit: Character {
        case spades = "♠", hearts = "♡", diamonds = "♢", clubs = "♣"
    }


    // nested Rank enumeration
    enum Rank: Int {
        case two = 2, three, four, five, six, seven, eight, nine, ten
        case jack, queen, king, ace
        struct Values {
            let first: Int, second: Int?
        }
        var values: Values {
            switch self {
            case .ace:
                return Values(first: 1, second: 11)
            case .jack, .queen, .king:
                return Values(first: 10, second: nil)
            default:
                return Values(first: self.rawValue, second: nil)
            }
        }
    }


    // BlackjackCard properties and methods
    let rank: Rank, suit: Suit
    var description: String {
        var output = "suit is \(suit.rawValue),"
        output += " value is \(rank.values.first)"
        if let second = rank.values.second {
            output += " or \(second)"
        }
        return output
    }
}

let heartsSymbol = BlackjackCard.Suit.hearts.rawValue
```

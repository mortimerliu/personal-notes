# Automatic Reference Counting<!-- omit from toc -->

- [Strong Reference Cycles Between Class Instances](#strong-reference-cycles-between-class-instances)
- [Resolving Strong Reference Cycles Between Class Instances](#resolving-strong-reference-cycles-between-class-instances)
  - [Weak References](#weak-references)
  - [Unowned References](#unowned-references)
  - [Unowned Optional References](#unowned-optional-references)
- [Unowned References and Implicitly Unwrapped Optional Properties](#unowned-references-and-implicitly-unwrapped-optional-properties)
- [Strong Reference Cycles for Closures](#strong-reference-cycles-for-closures)
  - [Resolving Strong Reference Cycles for Closures](#resolving-strong-reference-cycles-for-closures)

```swift
class Person {
    let name: String
    init(name: String) {
        self.name = name
        print("\(name) is being initialized")
    }
    deinit {
        print("\(name) is being deinitialized")
    }
}

var reference1: Person?
reference1 = Person(name: "John Appleseed")
// Prints "John Appleseed is being initialized"
reference1 = nil
// Prints "John Appleseed is being deinitialized"
```

## Strong Reference Cycles Between Class Instances

it’s possible to write code in which an instance of a class never gets to a point where it has zero strong references. This can happen if two class instances hold a strong reference to each other, such that each instance keeps the other alive. This is known as a **strong reference cycle**.

```swift
class Person {
    let name: String
    init(name: String) { self.name = name }
    var apartment: Apartment?
    deinit { print("\(name) is being deinitialized") }
}

class Apartment {
    let unit: String
    init(unit: String) { self.unit = unit }
    var tenant: Person?
    deinit { print("Apartment \(unit) is being deinitialized") }
}

var john: Person?
var unit4A: Apartment?

john = Person(name: "John Appleseed")
unit4A = Apartment(unit: "4A")

john!.apartment = unit4A
unit4A!.tenant = john
// strong reference cycle created

john = nil
unit4A = nil
// no deinitializers called
```

## Resolving Strong Reference Cycles Between Class Instances

- Swift provides two ways to resolve strong reference cycles when you work with properties of class type: *weak references* and *unowned references*
  - use a *weak reference* when the other instance has a shorter lifetime - when the other instance can be deallocated first
  - use an *unowned reference* when the other instance has the same lifetime or a longer lifetime

### Weak References

- indicate a weak reference by placing the `weak` keyword before a property or variable declaration, always written as a variable and as an optional type (as the reference can be `nil` at some point in its lifetime)
- because a weak reference does not keep a strong hold on the instance it refers to, it’s possible for that instance to be deallocated while the weak reference is still referring to it
- therefore, ARC automatically sets a weak reference to `nil` when the instance that it refers to is deallocated
  - Property observers aren’t called when ARC sets a weak reference to `nil`

```swift
class Person {
    let name: String
    init(name: String) { self.name = name }
    var apartment: Apartment?
    deinit { print("\(name) is being deinitialized") }
}

class Apartment {
    let unit: String
    init(unit: String) { self.unit = unit }
    weak var tenant: Person?
    deinit { print("Apartment \(unit) is being deinitialized") }
}

var john: Person?
var unit4A: Apartment?

john = Person(name: "John Appleseed")
unit4A = Apartment(unit: "4A")

john!.apartment = unit4A
unit4A!.tenant = john

john = nil
// Prints "John Appleseed is being deinitialized"
```

### Unowned References

- like a weak reference, an *unowned reference* does not keep a strong hold on the instance it refers to
- unlike a weak reference, an unowned reference is assumed to always have a value, thus making a value as unowned doesn’t make it optional and ARC never sets an unowned reference’s value to `nil`
- use an unowned reference only when you are sure that the reference always refers to an instance that has not been deallocated
- Swift also provides unsafe unowned references for cases where you need to disable runtime safety checks — for example, for performance reasons. As with all unsafe operations, you take on the responsibility for checking that code for safety. You indicate an unsafe unowned reference by writing `unowned(unsafe)`. If you try to access an unsafe unowned reference after the instance that it refers to is deallocated, your program will try to access the memory location where the instance used to be, which is an unsafe operation.

```swift
class Customer {
    let name: String
    var card: CreditCard?
    init(name: String) {
        self.name = name
    }
    deinit { print("\(name) is being deinitialized") }
}

class CreditCard {
    let number: UInt64
    unowned let customer: Customer
    init(number: UInt64, customer: Customer) {
        self.number = number
        self.customer = customer
    }
    deinit { print("Card #\(number) is being deinitialized") }
}

var john: Customer?
john = Customer(name: "John Appleseed")
john!.card = CreditCard(number: 1234_5678_9012_3456, customer: john!)

john = nil
// Prints "John Appleseed is being deinitialized"
// Prints "Card #1234567890123456 is being deinitialized"
```

### Unowned Optional References

- you can mark an optional reference to a class as unowned.
- In terms of the ARC ownership model, an unowned optional reference and a weak reference can both be used in the same contexts.
- The difference is that when you use an unowned optional reference, you’re responsible for making sure it always refers to a valid object or is set to nil (where Swift will set a weak reference to nil for you)

```swift
class Department {
    var name: String
    var courses: [Course]
    init(name: String) {
        self.name = name
        self.courses = []
    }
}

class Course {
    var name: String
    unowned var department: Department
    unowned var nextCourse: Course?
    init(name: String, in department: Department) {
        self.name = name
        self.department = department
        self.nextCourse = nil
    }
}

let department = Department(name: "Horticulture")
let intro = Course(name: "Survey of Plants", in: department)
let intermediate = Course(name: "Growing Common Herbs", in: department)
let advanced = Course(name: "Caring for Tropical Plants", in: department)
intro.nextCourse = intermediate
intermediate.nextCourse = advanced
department.courses = [intro, intermediate, advanced]
```

## Unowned References and Implicitly Unwrapped Optional Properties

- For a situation where two properties, both of which are allowed to be `nil`, have the potential to cause a strong reference cycle, use a weak reference
- For a situation where one property that is allowed to be `nil` and another property that cannot be `nil` have the potential to cause a strong reference cycle, use an unowned reference
- For a situation where both properties should always have a value, and neither property should ever be `nil` once initialization is complete, use **an unowned property on one class and an implicitly unwrapped optional property on the other class**
  - this enables both properties to be accessed directly (without optional unwrapping) once initialization is complete

```swift
class Country {
    let name: String
    var capitalCity: City!
    init(name: String, capitalName: String) {
        self.name = name
        // We can use self here because capitalCity is an implicitly unwrapped optional and has a default value of nil
        // thus Country instance is considered to be fully initialized as soon as the Country instance sets its name property within its initializer
        self.capitalCity = City(name: capitalName, country: self)
    }
}

class City {
    let name: String
    unowned let country: Country
    init(name: String, country: Country) {
        self.name = name
        self.country = country
    }
}
```

## Strong Reference Cycles for Closures

- A strong reference cycle can also occur if you assign a closure to a property of a class instance, and the body of that closure captures the instance (e.g. by referring to `self` within the closure)
- This strong reference cycle occurs because closures, like classes, are reference types

```swift
class HTMLElement {
    let name: String
    let text: String?

    // an property of type () -> String
    // asHTML is a lazy property such that it can refer to self within the default closure as the lazy property will not be accessed until after initialization is complete
    lazy var asHTML: () -> String = {
        if let text = self.text {
            return "<\(self.name)>\(text)</\(self.name)>"
        } else {
            return "<\(self.name) />"
        }
    }

    init(name: String, text: String? = nil) {
        self.name = name
        self.text = text
    }
    deinit { print("\(name) is being deinitialized") }
}

var heading: HTMLElement? = HTMLElement(name: "h1")
print(heading!.asHTML()) // triggers the lazy property to be accessed
heading = nil
// no deinitializers called
```

### Resolving Strong Reference Cycles for Closures

- use a *capture list* to break these strong reference cycles
- A capture list defines the rules to use when capturing one or more reference types within the closure’s body
- Declare each captured reference to be a weak or unowned reference rather than a strong reference
- The appropriate choice of weak or unowned depends on the relationships between the different parts of your code
  - define a capture in a closure as an unowned reference when the closure and the instance it captures will always refer to each other, and will always be deallocated at the same time
  - define a capture as a weak reference when the captured reference may become `nil` at some point in the future; weak references are always of an optional type, and automatically become `nil` when the instance they reference is deallocated
  - If the captured reference will never become `nil`, it should always be captured as an unowned reference, rather than a weak reference
- Swift requires you to write `self.someProperty` or `self.someMethod()` (rather than just `someProperty` or `someMethod()`) whenever you refer to a member of `self` within a closure

```swift
lazy var someClosure =  {
        [unowned self, weak delegate = self.delegate]
        (index: Int, stringToProcess: String) -> String in
    // closure body goes here
}

lazy var someClosure = {
    [unowned self, weak delegate = self.delegate] in
    // closure body goes here
}

class HTMLElement {

    let name: String
    let text: String?

    lazy var asHTML: () -> String = {
            [unowned self] in
        if let text = self.text {
            return "<\(self.name)>\(text)</\(self.name)>"
        } else {
            return "<\(self.name) />"
        }
    }

    init(name: String, text: String? = nil) {
        self.name = name
        self.text = text
    }

    deinit {
        print("\(name) is being deinitialized")
    }
}
```

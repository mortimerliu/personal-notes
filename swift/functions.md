# Swift - Functions<!-- omit in toc -->

[Swift Documentation](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/functions)

- [Function Declaration](#function-declaration)
- [Function Parameters](#function-parameters)
  - [Function Argument Labels and Parameter Names](#function-argument-labels-and-parameter-names)
  - [Default Parameter Values](#default-parameter-values)
  - [Variadic Parameters](#variadic-parameters)
  - [In-Out Parameters](#in-out-parameters)
- [Function Return Values](#function-return-values)
  - [Functions With an Implicit Return](#functions-with-an-implicit-return)
- [Function Types](#function-types)
  - [Using Function Types](#using-function-types)
- [Nested Functions](#nested-functions)

## Function Declaration

- Function is declared using the `func` keyword followed by the function name, and optionally one or more named, typed parameters and optionally a return type.
- Every function has a *function name*, which consists of the name of the function and the names of its parameters.
- To call a function, use its name followed by a list of arguments in parentheses. A function's arguments must always be provided **in the same order** and with same type as the function's parameter list.

```swift
func functionName(argumentLabel parameterName: Type, ...) -> ReturnType {
    // function body
}

let returnValue: ReturnType = functionName(argumentLabel: argumentValue, ...)
```

## Function Parameters

Function parameters are constants by default, and cannot be changed within the function body. However, function parameters can be defined as [*in-out* parameters](#in-out-parameters), which can be changed within the function body.

Function can have no parameters:

```swift
func functionName() -> ReturnType { ... }
```

Function can have one or more parameters, separated by commas:

```swift
func functionName(argumentLabel parameterName: Type, ...) -> ReturnType { ... }
```

### Function Argument Labels and Parameter Names

Each function parameter has both an *argument label* and a *parameter name*. The argument label is used when calling the function; each argument is written in the function call with its argument label before it. The parameter name is used in the implementation of the function.

- By default, parameters use their parameter name as their argument label.
- To use a custom argument label, write the argument label before the parameter name, separated by a space.
- If a parameter has an argument label, the argument must be labeled when you call the function.
- If you don't want an argument label for a parameter, write an underscore (`_`) instead of an explicit argument label for that parameter.

```swift
func functionName(parameterName: Type, ...) -> ReturnType { ... } 
let returnValue: ReturnType = functionName(parameterName: argumentValue, ...)

func functionName(argumentLabel parameterName: Type, ...) -> ReturnType { ... }
let returnValue: ReturnType = functionName(argumentLabel: argumentValue, ...)

func functionName(_ parameterName: Type, ...) -> ReturnType { ... }
let returnValue: ReturnType = functionName(argumentValue)
```

### Default Parameter Values

You can define a default value for any parameter in a function by assigning a value to the parameter after that parameter’s type. If a default value is defined, you can omit that parameter when calling the function.

- Place parameters with default values at the end of a function’s parameter list (although this is not required).

```swift
func functionName(parameterName: Type = defaultValue, ...) -> ReturnType { ... }
let returnValue: ReturnType = functionName() // parameterName is set to defaultValue
```

### Variadic Parameters

A variadic parameter accepts *zero* or more values of a specified type. You use a variadic parameter to specify that the parameter can be passed a varying number of input values when the function is called.

- Write variadic parameters by inserting three period characters (`...`) after the parameter’s type name.
- The values passed to a variadic parameter are made available within the function’s body as an array of the appropriate type.
- A function can have multiple variadic parameters. The first parameter that comes after a variadic parameter must have an argument label.

```swift
func functionName(parameterName: Type..., ...) -> ReturnType { ... }
let returnValue: ReturnType = functionName(argumentValue1, argumentValue2, ...)

// example
func arithmeticMean(_ numbers: Double...) -> Double {
    var total: Double = 0
    for number in numbers {
        total += number
    }
    return total / Double(numbers.count)
}
arithmeticMean(1, 2, 3, 4, 5) // 3.0
```

### In-Out Parameters

Function parameters are constants by default. Trying to change the value of a function parameter from within the body of that function results in a compile-time error. If you want a function to modify a parameter’s value, and you want those changes to *persist after the function call has ended*, define that parameter as an *in-out* parameter instead.

- Write an in-out parameter by placing the `inout` keyword right before a parameter’s type.
- You can only pass a variable as the argument for an in-out parameter.
  - An in-out parameter has a value that is passed *in* to the function, is modified by the function, and is passed *back* out of the function to replace the original value.
- You place an ampersand (`&`) directly before a variable’s name when you pass it as an argument to an in-out parameter, to indicate that it can be modified by the function.

```swift
func functionName(parameterName: inout Type, ...) -> ReturnType { ... }
var variableName: Type = value
functionName(parameterName: &variableName, ...)

// example
func swapTwoInts(_ a: inout Int, _ b: inout Int) {
    let temporaryA: Int = a
    a = b
    b = temporaryA
}
var someInt: Int = 3
var anotherInt: Int = 107
swapTwoInts(&someInt, &anotherInt)
print("someInt is now \(someInt), and anotherInt is now \(anotherInt)")
// Prints "someInt is now 107, and anotherInt is now 3"
```

## Function Return Values

Function can have no return values:

  ```swift
  func functionName(parameterName: Type, ...) { ... }  // preferred
  func functionName(parameterName: Type, ...) -> Void { ... } // alternative, type Void, which is an empty tuple ()
  func functionName(parameterName: Type, ...) -> (ReturnType, ...) { ... } // alternative
  ```

Function can have one more return values:

```swift
func functionName(parameterName: Type, ...) -> ReturnType { ... } // return a single value
func functionName(parameterName: Type, ...) -> (ReturnType, ...) { ... } // return multiple values, essentially a tuple
func functionName(parameterName: Type, ...) -> (label1: ReturnType1, label2: ReturnType2, ...) { ... } // tuple with named return values
```

Function can have optional return values:

  ```swift
  func functionName(parameterName: Type, ...) -> ReturnType? { ... } // return an optional value
  func functionName(parameterName: Type, ...) -> (ReturnType?, ...) { ... } // return multiple optional values
  func functionName(parameterName: Type, ...) -> (ReturnType1, ...)? { ... } // return optional tuple
  ```

### Functions With an Implicit Return

If the entire body of a function is a *single* expression, the function implicitly returns that expression. For example, both functions below have the same behavior:

```swift
func greeting1(person: String) -> String {
    return "Hello, " + person + "!"
}

func greeting2(person: String) -> String {
    "Hello, " + person + "!"
}
```

## Function Types

Every function has a specific function type, made up of the parameter types and the return type of the function.

```swift
func functionName(parameterName: Type, ...) -> ReturnType { ... }
// function type: (Type, ...) -> ReturnType

// example
func addTwoInts(_ a: Int, _ b: Int) -> Int {
    return a + b
} // function type: (Int, Int) -> Int

func printHelloWorld() {
    print("hello, world")
} // function type: () -> Void or () -> ()
```

### Using Function Types

You can use a function type just like any other type in Swift.

- Define a constant or variable to be of a function type
- Use a function type as a parameter type or return type in a function

```swift
var mathFunction: (Int, Int) -> Int = addTwoInts
// mathFunction now refers to the addTwoInts(_:_:) function

func printMathResult(_ mathFunction: (Int, Int) -> Int, _ a: Int, _ b: Int) {
    print("Result: \(mathFunction(a, b))")
}
printMathResult(addTwoInts, 3, 5)
// Prints "Result: 8"

func stepForward(_ input: Int) -> Int {
    return input + 1
}
func stepBackward(_ input: Int) -> Int {
    return input - 1
}
func chooseStepFunction(backward: Bool) -> (Int) -> Int {
    return backward ? stepBackward : stepForward
}
var currentValue = 3
let moveNearerToZero = chooseStepFunction(backward: currentValue > 0)
// moveNearerToZero now refers to the stepBackward() function
```

## Nested Functions

All of the functions you have encountered so far in this chapter have been examples of *global functions*, which are defined at a *global scope*. You can also define functions inside the bodies of other functions, known as *nested functions*.

- Nested functions are hidden from the outside world by default, but can still be called and used by their enclosing function. An enclosing function can also return one of its nested functions to allow the nested function to be used in another scope.

```swift
func chooseStepFunction(backward: Bool) -> (Int) -> Int {
    func stepForward(input: Int) -> Int { return input + 1 }
    func stepBackward(input: Int) -> Int { return input - 1 }
    return backward ? stepBackward : stepForward
}

var currentValue = -4
let moveNearerToZero = chooseStepFunction(backward: currentValue > 0)
// moveNearerToZero now refers to the nested stepForward() function
while currentValue != 0 {
    print("\(currentValue)... ")
    currentValue = moveNearerToZero(currentValue)
}
print("zero!")
// -4...
// -3...
// -2...
// -1...
// zero!
```

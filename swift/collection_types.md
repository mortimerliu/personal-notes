# Swift - Collection Types

[Swift documentations](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/collectiontypes)

- [Swift - Collection Types](#swift---collection-types)
  - [Summary](#summary)
  - [Mutability of Collections](#mutability-of-collections)
  - [Arrays](#arrays)
    - [Creating Arrays](#creating-arrays)
    - [Accessing and Modifying an Array](#accessing-and-modifying-an-array)
  - [Sets](#sets)
    - [Hash Values for Set Types](#hash-values-for-set-types)
    - [Creating Sets](#creating-sets)
    - [Accessing and Modifying a Set](#accessing-and-modifying-a-set)
    - [Set Operations](#set-operations)
  - [Dictionaries](#dictionaries)
    - [Creating an Empty Dictionary](#creating-an-empty-dictionary)
    - [Accessing and Modifying a Dictionary](#accessing-and-modifying-a-dictionary)

## Summary

Swift provides three primary collection types, known as arrays, sets, and dictionaries, for storing collections of values.

- Arrays are ordered collections of values.
- Sets are unordered collections of unique values.
- Dictionaries are unordered collections of key-value associations.

## Mutability of Collections

- If you create an array, a set, or a dictionary, and assign it to a *variable*, the collection that is created will be *mutable*.
- If you assign an array, a set, or a dictionary to a *constant*, that collection is *immutable*, and its size and contents cannot be changed.
- It's good practice to create immutable collections in all cases where the collection does not need to change.

## Arrays

- An array stores values of the same type in an ordered list.
- Array type written in full as `Array<Element>`, and in short as `[Element]`, where `Element` is the type of values the array is allowed to store. The short form is preferred.

### Creating Arrays

```swift
var someInts: [Int] = [] // empty array literal
var someInts = [Int]() // empty array initializer
var someInts = Array<Int>() // initializer syntax
let someInts = [1, 2, 3] // immutable array
var threeDoubles = Array(repeating: 0.0, count: 3) // create an array with default value
var anotherThreeDoubles = Array(repeating: 2.5, count: 3)
var sixDoubles = threeDoubles + anotherThreeDoubles
var shoppingList: [String] = ["Eggs", "Milk"] // array literal
var shoppingList = ["Eggs", "Milk"] // type inference

someInts.append(3)
someInts = [] // type is still [Int] because of type inference
```

### Accessing and Modifying an Array

- Use subscript syntax to access an array's individual elements
- `count` property to find out the number of items in an array
- `isEmpty` property as a shortcut for checking whether the count property is equal to 0
- `append(_:)` method to add a new item at the end of an array
- `+=` operator to append another array's contents to the end of an array
- `insert(_:at:)` method to insert a new item into the array at a specified index
- `remove(at:)` method to remove and return an item from the array at a specified index
- `removeLast()` method to remove and return the final item in the array
- `removeAll()` method to remove all of the items in the array
- `removeAll(keepingCapacity:)` method to remove all of the items in the array, without clearing the memory that the array occupies
- `enumerated()` method to iterate over the array, the method returns a tuple composed of an integer and the item
- `first` property to access the first item in a nonempty array and returns an optional value

```swift
var shoppingList = ["Eggs", "Milk"]
shoppingList.count // 2
shoppingList.isEmpty // false
shoppingList.append("Flour") // ["Eggs", "Milk", "Flour"]
shoppingList += ["Baking Powder"] // ["Eggs", "Milk", "Flour", "Baking Powder"]
shoppingList += ["Chocolate Spread", "Cheese", "Butter"] // ["Eggs", "Milk", "Flour", "Baking Powder", "Chocolate Spread", "Cheese", "Butter"]
shoppingList[0] // "Eggs"
shoppingList[4...6] // ["Chocolate Spread", "Cheese", "Butter"]
shoppingList[4...6] = ["Bananas", "Apples"] // ["Eggs", "Milk", "Flour", "Baking Powder", "Bananas", "Apples"]
shoppingList.insert("Maple Syrup", at: 0) // ["Maple Syrup", "Eggs", "Milk", "Flour", "Baking Powder", "Bananas", "Apples"]
let mapleSyrup = shoppingList.remove(at: 0) // "Maple Syrup"
let apples = shoppingList.removeLast() // "Apples"
shoppingList.removeAll() // []
shoppingList.removeAll(keepingCapacity: true) // []
for (index, value) in shoppingList.enumerated() {
  print("Item \(index + 1): \(value)")
}
```

## Sets

- A set stores distinct values of the same type in a collection with no defined ordering.
- Set type written in full as `Set<Element>`, where `Element` is the type of values the set is allowed to store. No shorthand form is available.

### Hash Values for Set Types

- A type must be hashable in order to be stored in a set.
  - You can use any type that conforms to the [`Hashable`](https://developer.apple.com/documentation/swift/hashable) protocol in a set or as a dictionary key.

### Creating Sets

```swift
var letters = Set<Character>()
letters.insert("a")
letters = [] // type is still Set<Character> because of type inference
var favoriteGenres: Set<String> = ["Rock", "Classical", "Hip hop"] // create a set with an array literal
var favoriteGenres: Set = ["Rock", "Classical", "Hip hop"] // type inference
```

### Accessing and Modifying a Set

- `count` property to find out the number of items in a set
- `isEmpty` property as a shortcut for checking whether the count property is equal to 0
- `insert(_:)` method to add a new item into a set
- `remove(_:)` method to remove a specific item from a set and return an *optional* value that contains the removed item if it existed in the set or `nil` if not
- `removeAll()` method to remove all of the items from a set
- `contains(_:)` method to check whether a set contains a particular item
- `sorted()` method to iterate over the values of a set in a specific order

```swift
var favoriteGenres: Set<String> = ["Rock", "Classical", "Hip hop"]
favoriteGenres.count // 3
favoriteGenres.isEmpty // false
favoriteGenres.insert("Jazz") // ["Rock", "Classical", "Hip hop", "Jazz"]

if let removedGenre = favoriteGenres.remove("Rock") { // optional binding
  print("\(removedGenre)? I'm over it.")
} else {
  print("I never much cared for that.")
} // "Rock? I'm over it."

favoriteGenres.contains("Funk") // false

for genre in favoriteGenres {
  print("\(genre)")
} // "Classical", "Hip hop", "Jazz" // order is not defined

for genre in favoriteGenres.sorted() {
  print("\(genre)")
} // "Classical", "Hip hop", "Jazz" // order is defined
```

### Set Operations

- `union(_:)` method to create a new set with the values of two existing sets
- `intersection(_:)` method to create a new set with only the values common to both existing sets
- `subtracting(_:)` method to create a new set with values not in the specified set
- `symmetricDifference(_:)` method to create a new set with values in either set, but not both
- `isSubset(of:)` method to check whether all of the values of a set are contained in the specified set
- `==` operator to check whether two sets contain all of the same values
- `isSuperset(of:)` method to check whether a set contains all of the values in a specified set
- `isDisjoint(with:)` method to check whether two sets have no values in common
- `isStrictSubset(of:)` method to check whether a set is a subset but not equal to a specified set
- `isStrictSuperset(of:)` method to check whether a set is a superset but not equal to a specified set

```swift
let oddDigits: Set = [1, 3, 5, 7, 9]
let evenDigits: Set = [0, 2, 4, 6, 8]
let singleDigitPrimeNumbers: Set = [2, 3, 5, 7]

oddDigits.union(evenDigits).sorted() // [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
oddDigits.intersection(evenDigits).sorted() // []
oddDigits.subtracting(singleDigitPrimeNumbers).sorted() // [1, 9]
oddDigits.symmetricDifference(singleDigitPrimeNumbers).sorted() // [1, 2, 9]

let houseAnimals: Set = ["🐶", "🐱"]
let farmAnimals: Set = ["🐮", "🐔", "🐑", "🐶", "🐱"]
let cityAnimals: Set = ["🐦", "🐭"]

houseAnimals.isSubset(of: farmAnimals) // true
farmAnimals.isSuperset(of: houseAnimals) // true
farmAnimals.isDisjoint(with: cityAnimals) // true
```

## Dictionaries

- A dictionary stores associations between keys of the same type and values of the same type in a collection with no defined ordering.
- Dictionary type written in full as `Dictionary<Key, Value>`, and in short as `[Key: Value]`, where `Key` is the type of value that can be used as a dictionary key, and `Value` is the type of value that the dictionary stores for those keys. Shorthand form is preferred.
- Dictionary keys must be hashable

### Creating an Empty Dictionary

```swift
avr namesOfIntegers: [Int: String] = [:] // empty dictionary literal
var namesOfIntegers = [Int: String]() // empty dictionary initializer
namesOfIntegers[16] = "sixteen" // ["sixteen": 16]
namesOfIntegers = [:] // type is still [Int: String] because of type inference

var airports: [String: String] = ["YYZ": "Toronto Pearson", "DUB": "Dublin"] // dictionary literal
var airports = ["YYZ": "Toronto Pearson", "DUB": "Dublin"] // type inference
```

### Accessing and Modifying a Dictionary

- subscript syntax to access the value for a particular key and return an *optional* value
- subscript syntax to assign a new value for a particular key or to change the value for an existing key
- `count` property to find out the number of items in a dictionary
- `isEmpty` property as a shortcut for checking whether the count property is equal to 0
- `updateValue(_:forKey:)` method to set or update the value for a particular key and return an *optional* value that contains the old value for that key if one existed
- `removeValue(forKey:)` method to remove a key-value pair from a dictionary and return an *optional* value that contains the removed value if one existed
- `removeAll()` method to remove all key-value pairs from a dictionary
- `keys` property to retrieve a collection containing all of the keys in a dictionary
- `values` property to retrieve a collection containing all of the values in a dictionary

```swift
var airports = ["YYZ": "Toronto Pearson", "DUB": "Dublin"]
airports.count // 2
airports.isEmpty // false
airports["LHR"] = "London" // add a new key-value pair
airports["LHR"] = "London Heathrow" // update the value for an existing key
airports["LHR"] = nil // remove a key-value pair

if let oldValue = airports.updateValue("Dublin Airport", forKey: "DUB") { // optional binding
  print("The old value for DUB was \(oldValue).")
} // "The old value for DUB was Dublin."

if let airportName = airports["DUB"] { // optional binding
  print("The name of the airport is \(airportName).")
} else {
  print("That airport is not in the airports dictionary.")
} // "The name of the airport is Dublin Airport."

airports["APL"] = "Apple International"
airports["APL"] = nil // APL has now been removed from the dictionary

if let removedValue = airports.removeValue(forKey: "DUB") { // optional binding
  print("The removed airport's name is \(removedValue).")
} else {
  print("The airports dictionary does not contain a value for DUB.")
} // "The removed airport's name is Dublin Airport."

for (airportCode, airportName) in airports {
  print("\(airportCode): \(airportName)")
} // "YYZ: Toronto Pearson" "LHR: London Heathrow"

for airportCode in airports.keys {
  print("Airport code: \(airportCode)")
} // "Airport code: YYZ" "Airport code: LHR"

for airportName in [String](airports.values).sorted() { // convert to array
  print("Airport name: \(airportName)")
} // "Airport name: London Heathrow" "Airport name: Toronto Pearson"
```

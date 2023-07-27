# Type Casting<!-- omit from toc -->

- [Summary](#summary)
- [Defining a Class Hierarchy](#defining-a-class-hierarchy)
- [Checking Type](#checking-type)
- [Downcasting](#downcasting)
- [Type Casting for Any and AnyObject](#type-casting-for-any-and-anyobject)

## Summary

- Check the type of an instance: `is` operator
- Treat that instance as a different superclass or subclass from somewhere else in its own **class hierarchy**: `as` operator
- Check whether a type conforms to a protocol

## Defining a Class Hierarchy

```swift
class MediaItem {
    var name: String
    init(name: String) {
        self.name = name
    }
}

class Movie: MediaItem {
    var director: String
    init(name: String, director: String) {
        self.director = director
        super.init(name: name)
    }
}

class Song: MediaItem {
    var artist: String
    init(name: String, artist: String) {
        self.artist = artist
        super.init(name: name)
    }
}

let library = [
    Movie(name: "Casablanca", director: "Michael Curtiz"),
    Song(name: "Blue Suede Shoes", artist: "Elvis Presley"),
    Movie(name: "Citizen Kane", director: "Orson Welles"),
    Song(name: "The One And Only", artist: "Chesney Hawkes"),
    Song(name: "Never Gonna Give You Up", artist: "Rick Astley")
] 
// type of library is inferred to be `[MediaItem]`
// Elements of the array are still `Movie` and `Song` instances behind the scenes
// If iterate over the contents of this array, the items returned are typed as `MediaItem`, and not as `Movie` or `Song`
```

## Checking Type

- Use the `is` operator to check whether an instance is of a certain subclass type
- Returns `true` if an instance is of that subclass type

```swift
for item in library {
    if item is Movie {
        print("Movie: \(item.name)") // only able to access `name`, not `director` as `item` is of type `MediaItem`
    } else if item is Song {
        print("Song: \(item.name)")
    }
}
```

## Downcasting

- A constant or variable of a certain class type may actually refer to an instance of a subclass behind the scenes
- Use `as?` or `as!` to downcast the constant or variable to a subclass type (as downcasting can fail)

```swift
for item in library {
    if let movie = item as? Movie {
        print("Movie: \(movie.name), dir. \(movie.director)")
    } else if let song = item as? Song {
        print("Song: \(song.name), by \(song.artist)")
    }
}
```

## Type Casting for Any and AnyObject

- `Any` can represent an instance of any type at all, including function types, and optional types
  - If you really do need to use an optional value as an `Any` value, you can use the as operator to explicitly cast the optional to `Any` to avoid warning
- `AnyObject` can represent an instance of any class type
- Use `Any` and `AnyObject` only when you explicitly need the behavior and capabilities they provide
- Use the `is` and `as` operators in a switch statement to discover the specific type of a constant or variable that is known only to be of type `Any` or `AnyObject`

```swift
var things: [Any] = [] // array can store values of any type

things.append(0)
things.append(0.0)
things.append(42)
things.append(3.14159)
things.append("hello")
things.append((3.0, 5.0))
things.append(Movie(name: "Ghostbusters", director: "Ivan Reitman"))
things.append({ (name: String) -> String in "Hello, \(name)" })

for thing in things {
    switch thing {
    case 0 as Int:
        print("zero as an Int")
    case 0 as Double:
        print("zero as a Double")
    case let someInt as Int:
        print("an integer value of \(someInt)")
    case let someDouble as Double where someDouble > 0:
        print("a positive double value of \(someDouble)")
    case is Double:
        print("some other double value that I don't want to print")
    case let someString as String:
        print("a string value of \"\(someString)\"")
    case let (x, y) as (Double, Double):
        print("an (x, y) point at \(x), \(y)")
    case let movie as Movie:
        print("a movie called \(movie.name), dir. \(movie.director)")
    case let stringConverter as (String) -> String:
        print(stringConverter("Michael"))
    default:
        print("something else")
    }
}

let optionalNumber: Int? = 3
things.append(optionalNumber)        // Warning
things.append(optionalNumber as Any) // No warning
```

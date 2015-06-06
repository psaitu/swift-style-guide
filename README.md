# The Swift Style Guide

This guide is based on the following sources:

- [The Swift Programming Language](https://developer.apple.com/library/prerelease/ios/documentation/Swift/Conceptual/Swift_Programming_Language)
- [Github Swift style guide](https://github.com/github/swift-style-guide)
- [Ray Wenderlich Swift style guide](https://github.com/raywenderlich/swift-style-guide)

### Purpose of the style guide

This guide should help you to improve your Swift code style, its readability, consistency and simplicity. It's not a manifest, it doesn’t tell you what to do (though sometimes it may tell you what not to do). It's a set of hints about subjectively best practices that might help you and your team decrease number of programmers errors.

## Table of contents

* [Spacing](#spacing)
* [Comments](#comments)
* [Code organization](#code-organization)
* [Naming](#naming)
* [Closures](#closures)
* [Types](#types)
* [Mutability – `let` over `var`](#mutability---let-over-var)
* [Optionals](#optionals)
* [Static vs dynamic code](#static-vs-dynamic-code)
* [Implicit getters](#implicit-getters)
* [Access control](#access-control)
* [Explicit references to `self`](#explicit-references-to-self)
* [Value types over reference types](#value-types-over-reference-types)
* [Forbidden](#forbidden)

## Spacing

Indent code with tabs, not spaces. End files with an empty line.

Vertical spaces should be used in long methods to separate its name from implementation. You may also want to use vertical spaces to separate logic within a function. Shorter methods (one or two lines) don't need such spacing.

## Comments

Use comments to describe why something is written as it is, or working like it does. Remember that code should be self-documenting, so use comments only if necessary.

If you decide to add comments, keep them up-to-date. Unmaintained comments should be removed.

## Code organization

Source files should have the following organization.

```swift
// 1. imports

import MoneyKit

// 2. classes, structs, enums

class Wallet {

    // 2.1. public, internal properties

    let cards: [Card]
    private(set) var cash: Cash
    
    // 2.2. private properties
    
    unowned private let owner: Person
    
    // 2.3. initializers
    
    init(cash: Cash, cards: [Card], owner: Person)
    
    // 2.4. public, internal functions
    
    func affordsTransaction(transaction: Transaction) -> Bool
    
    // 2.5. private functions
    
    private func cardWithSuffiecientCash(cash: Cash) -> Card?
    
}

// 3. extensions, protocol implementations

extension Wallet: Printable {

    var description: String {
        return "\(owner.name) has \(cash) cash and \(cards.count) cards"
    }

}
```

Such organization helps others to reach important content earlier. It also saves time, confusion and improves readability.

## Naming

Names should be meaningful and compact, written in camelCase. Try to ask yourself whether the name of a type sufficiently explains its behavior. Meaningful naming is very important to other developers because they define some expectations about their own roles.

It is strongly misadvised to name suffix your types with words like `Manager`, `Helper` or `Utility` because they're meaningless and their role can be easily misinterpreted.

### Generic types

Names of classes, structs, enums, enum cases, typealiases, protocols and generic types should be capitalized. Generic types' names should start with letter `T`, when `U`, `V` and so on.

### Functions and arguments

Function names should be as descriptive and meaningful as possible. Try to express its intent in the name, by keeping it compact at the same time.

Arguments should also be descriptive. Remember that you can use argument labels, which may be more meaningful to a user.

The first argument's name can be a part of the function's name...

```swift
func convertPoint(point: Point, fromView view: View) -> Point
```

... or its label can be explicitly required.

```swift
func convert(#point: Point, toView view: View) -> Point
```

Use default values for arguments where a function expects any value or some specific value most of the time. If a particular argument is not required for a function, it's good to make it optional and `nil` by default.

## Closures

If the last argument of a function is a closure, use trailing closure syntax. 

Trailing closure syntax should be used if a function accepts a closure as its last argument. If it's its only argument, parentheses may be ommited. Unused closure arguments should be replaced with `_` (or fully ommited if no arguments are used). Argument types should be inferred.

```swift
func executeRequest<T>(request: Request<T>, completion: (T, Error?) -> Void)

executeRequest(someRequest) { (result, _) in /* ... */ }
```

Use implicit `return` in one-line closures with clear context.

```swift
let numbers = [1, 2, 3, 4, 5]
let even = filter(numbers) { $0 % 2 == 0 }
```

Also, remember that global functions are closures and sometimes it's convenient to pass a function name as a closure.

```swift
func isPositive(number: Int) -> Bool

let numbers = [-1, 2, 3, -4, 5]
let positive = filter(numbers, isPositive)
```

## Types

Try to use native Swift types before you come up with your own. Every type can be extended, so sometimes instead of introducing new types, it's convenient to extend or alias existing ones.

Remember that Objective-C classes that have native Swift equivalents are not automatically bridged, e.g. `NSString` is not implicitly bridged to `String` in the following example.

```swift
func lowercase(string String) -> String

let string: NSString = /* ... */

lowercase(string) // compile error
lowercase(string as String) // no error
```

Types should be inferred whenever possible. Don't duplicate type identifier if it can be resolved in compile time:

```swift
// preferred

let name = "John Appleseed"
let planets = [.Mars, .Saturn]
let colors = ["red": 0xff0000, "green": 0x00ff00]
```

```swift
// not preferred

let name: String = "Amanda Smith"
let planets: [Planet] = [.Venus, .Earth]
let colors: [String: UInt32] = ["blue": 0x0000ff, "white": 0xffffff]
```

Also, associate colon with type identifier.

```swift
// preferred

class VideoArticle: Article
let events: [Timestamp: Event]
```

```swift
// not preferred

class VideoArticle : Article
let events : [Timestamp : Event]
```

Typealiases should be short and meaningful.

```swift
// preferred

typealias MoneyAmount = Double
```

```swift
// not preferred

typealias Money = Double
```

## Mutability – `let` over `var`

It's safer to assume that a variable is immutable, thus it's highly recommended to declare values as constants, using `let`. Immutable constants ensure their values will never change, which results in less error-prone code.

Mutable `var` variables should only be used when necessary, e.g. when you're absolutely sure you will be changing their values in the future.

## Optionals

Force unwrapping should be avoided as much as possible. Implicitly unwrapped optionals lead to less safe code and can cause unwanted crashes. Use optional chaining or `if-let` bindings to unwrap optional values.

```swift
let user: User? = findUserById(123)

if let user = user {
    println("found user \(user.name) with id \(user.id)")
}
```

Unwrapping several optionals in nested `if-let` statements is forbidden, as it leads to "pyramid of doom". Swift allows you to unwrap multiple optionals in one statement.

```swift
let name: String?
let age: Int?

if let name = name, age = age where age >= 13 {
    /* ... */
}
```

However, implicitly unwrapped optionals can sometimes be useful. They may be used in unit tests, where system under test should never be `nil`. There's no point executing rest of the tests if one of them fails.

```swift
var sut: SystemUnderTest!

beforeEach {
    sut = /* ... */
}

afterEach {
    sut = nil
}

it("should behave as expected") {
    sut.run()
    expect(sut.running).to(beTrue())
}
```

## Static vs. dynamic code

Static code is code which logic and control from can be resolved at compile-time. Swift compiler is able to optimize predictable code to work better and faster. Try to make use of this feature and write as much static code as possible.

On the other hand, dynamic code's control flow is resolved at run-time, which means it's not predictable and, as a result, can't be optimized by the compiler. Avoid using `@dynamic` and `@objc` attributes.

## Implicit getters

Read-only computed properties don't need an explicit getter, thus it can be ommited. This also applies to read-only subscripts.

```swift
struct Person {

    let height: Float
    let weight: Float

    var bmi: Float {
        return weight / (height * height)
    }
    
}
```

## Access control

Access control modifiers should be specified on a top-level scope.

```swift
public extension String {
    var uppercaseString: String
}
```

Use access control modifiers only if necessary. Pay attention to variables. Use `private` or `private(set)` appropriately. Don't add modifiers if they are are already a default.

```swift
// preferred

extension String {
    var uppercaseString: String
}
```

```swift
// not preferred

internal extension String {
    var uppercaseString: String
}
```

## Explicit references to `self`

Explicit references to `self` should only take place in closures and when the language requires it.

```swift
struct Person {
    
    let firstName: String
    let lastName: String
    
    var fullName: String {
        return "\(firstName) \(lastName)"
    }

    init(firstName: String, lastName: String) {
        self.firstName = firstName
        self.lastName = lastName
    }

}
```

## Value types over reference types

Value types, such as structs, enums and tuples are usually simpler than reference types (classes) and they're always passed by copying. This means they're independent thread-safe instances, which makes code simpler and safer. In addition, `let` and `var` work as expected.

Value types are great for representing **data** in the app.

```swift
struct Country {
    let name: String
    let capital: City
}
```

On the other hand, reference types, such as classes, are passed by referencing the same mutable instance in memory. There are several cases when classes should be preferred. The first case emerges when subclassing current Objective-C classes. The second one is when you need to use reference types with mutable state, or if you need to perform some actions on deinitialization.

Reference types are great to represent **behavior** in the app.

```swift
class FileStream {
    let file: File
    var currentPosition: StreamPosition
}
```

Keep in mind that inheritance is not a sufficient argument to use classes. You should try to compose your types using protocols.

```swift
// preferred

protocol Polygon {
    var numberOfSides: Int { get }
}

struct Triangle: Polygon {
    let numberOfSides = 3
}
```

```swift
// not preferred

class Polygon {
    let numberOfSides: Int
    init(numberOfSides: Int)
}

class Trinagle: Polygon {
    init() {
        super.init(numberOfSides: 3)
    }
}
```

## Forbidden

Types should never have prefixes because their names are already implicitly mangled and prefixed by their module name.

Semicolons are obfuscative and should never be used. Statements can be distributed in different lines.

Rewriting standard library functionalities should never take place. Your code will most probably be less optimized and more confusing to other developers.

# The Swift Style Guide

This guide is based on the following sources:

- [The Swift Programming Language](https://developer.apple.com/library/prerelease/ios/documentation/Swift/Conceptual/Swift_Programming_Language)
- [Github Swift style guide](https://github.com/github/swift-style-guide)
- [Ray Wenderlich Swift style guide](https://github.com/raywenderlich/swift-style-guide)

## Purpose of the style guide

This guide should help you to improve your Swift code style, its readability, consistency and simplicity. It's not a manifest, it doesn’t tell you what to do (though sometimes it may tell you what not to do). It's a set of hints about subjectively best practices that might help you and your team decrease number of programmers errors.

## Table of contents

* [Spacing](#spacing)
* [Comments](#comments)
* [Code organization](#code-organization)
* [General naming](#general-naming)
* [Functions naming and arguments](#functions-naming-and-arguments)
* [Closures](#closures)
* [Types](#types)
* [Mutability - let vs var](#mutability---let-vs-var)
* [Optionals](#optionals)
* [Static vs Dynamic](#static-vs-dynamic)
* [Implicit getters on read-only computed properties and subscripts](https://github.com/netguru/swift-style-guide#implicit-getters-on-read-only-computed-properties-and-subscripts)
* [Specify access control](#specify-access-control)
* [Refer to self only when it’s required and necessary](#refer-to-self-only-when-its-required-and-necessary)
* [Value semantics over reference semantics](#value-semantics-over-reference-semantics)
* [Forbidden](#forbidden)

## Spacing

Indent code with tabs, not spaces. Remember about ending file with new line.

Vertical spaces should be used in long methods to separate its name from implementation, what improves readability. You may also want to use vertical spaces to separate logic within a function. Shorter methods (one or two lines) don't need such spacing.

## Comments

Use comments to describe why something is written as it is, or working like it does. Remember that code should be self-documenting, so use comments only if necessary. If you decide to add comments, keep them up-to-date. Unmaintained comments should be removed.

## Code organization

Preferable code organization within a file:

```swift
class Wallet {
    
    private(set) var cash: Double
    let cards: [Card]?
    
    private personID: Identification
    
    init(cash: Double, cards: [Card], identification: Identification) {
        self.cash = cash
        self.cards = cards
        personID = identification
    }
    
    func canAffordTransaction(transaction: Transaction) -> Bool
    
    private func cardForCashAmount(amount: Double) -> Card
}
    
extension Wallet: Printable {

    var description: String {
        return personID.firstName + " " + personID.lastName + " has " + String(cash)
    }
}
```

Such organization helps others to reach important content earlier. It saves time, confusion and improves readability.

## General naming

Names of classes, structs, enums, enum cases, typealiases, protocols and generic types should be capitalized. Generic types' names should start with letter `T`, when `U`, `V` and so on.

Names should be meaningful and compact, written in camel case manner. Try to ask yourself whether the name of a type sufficiently explains its behavior. Meaningful naming is very important to other developers because they define some expectations about their own roles.

It is strongly misadvised to name suffix your types with words like `Manager`, `Helper` or `Utility` because they're meaningless and their role can be easily misinterpreted.

## Function and argument naming

Function names should be as descriptive and meaningful as possible. Try to express its intent in the name, by keeping it compact at the same time.

Arguments should also be descriptive. Remember that you can use argument labels, which may be more meaningful to a user.

The first argument's name can be a part of the function's name:

```swift
func convertPoint(point: CGPoint, toCoordinateSystem system: CoordinateSystem) -> CGPoint
```

Or its label can be explicitly required:

```swift
func convert(#point: CGPoint, toCoordinateSystem system: CoordinateSystem) -> CGPoint
```

Use default values for arguments where a function expects any value or some specific value most of the time. If a particular argument is not required for a function, it's good to make it optional and `nil` by default.

```swift
func describe(#operation: Operation, includingOperationError error: NSError? = nil) -> String
```

## Closures

If the last argument of a function is a closure, use trailing closure syntax. 

Trailing closure syntax should be used if a function accepts a closure as its last argument. If it's its only argument, parentheses may be ommited. Unused closure arguments should be replaced with `_` (or fully ommited if no arguments are used). Argument types should be inferred.

```swift
func perform<T>(#operation: Operation<T>, completion: (T, NSError) -> Void)

perform(#operation: operation) { (result, _) in
    ...
}
```

Use implicit `return` in one-line closures with clear context.

```swift
let users: [User] = ...

users.filter { $0.name != nil }
```

Also, remember that global functions are closures and sometimes it's convenient to pass a function name as a closure:

```swift
func isPositive(number: Int) -> Bool {
    return number > 0
}

let numbers = [-1, 2, 3, -4]

let positive = numbers.filter(isPositive) // [2, 3]
```

## Types

Try to use native Swift types before you come up with your own. Every type can be extended, so sometimes instead of introducing new types, it's convenient to extend or alias existing ones.

Remember that Objective-C classes that have native Swift equivalents are not automatically bridged, e.g. `NSString` is not implicitly bridged to `String` in the following example:

```swift
func inverse(string: String)

let string: NSString = ...

inverse(string) // compile error
inverse(string as String) // no error 
```

Types should be inferred whenever possible. Don't duplicate type identifier if it can be resolved in compile time:

```swift
let name = "text"
var guides = [Guide]()
var addressBook = [String: String]()

// not preferred

let name: String = "text"
var guides: [Guide] = []
var addressBook: [String: String] = [:]
```

Also, associate colon with type identifier:

```swift
class MyView: UIView
let color: UIColor

// not preferred

class MyView : UIView
let color : UIColor
```

Typealiases should be short and meaningful:

```swift
typealias MoneyAmount = Double

struct Item {
    let price: MoneyAmount
}

// not preferred

typealias Money = Double

struct Item {
    let price: Money
}
```

## Mutability – `let` vs `var`

It's safer to assume that a variable is immutable, thus it's highly recommended to declare values as constants, using `let`. Immutable constants ensure their values will never change, which results in less error-prone code.

Mutable `var` variables should only be used when necessary, e.g. when you're absolutely sure you will be changing their values in the future.

## Optionals

Force-unwrapping should be avoided because it leads to less safe code and can cause unwanted crashes.

```swift
let result: Result<String>?

result?.print()

// not preferred

result!.print()
```

Implicitly unwrapped optionals should also be avoided. However, they can be useful in unit tests, where system under test should never be `nil`. There's no point executing the rest of the tests if one of them is written badly.

```swift
var sut: System!

beforeEach {
    sut = System()
}

afterEach {
    sut = nil
}

it ("should be running") {
    sut.start()
    expect(sut.isRunning).to(beTrue())
}
```

Unwrap optional value when it makes sense, e.g. when you need to make a bunch of operations on it:

```swift
var button: UIButton?

if let button = button {
    button.backgroundColor = UIColor.blackColor()
    button.layer.cornerRadius = 5.0
    button.layer.borderWidth = 1.0
}
```

Otherwise use optional chaining because it's clear and safe:

```swift
button?.backgroundColor = UIColor.blackColor()
```

Unwrapping several optionals in nested `if-let` statements is forbidden. Swift allows you to unwrap multiple optionals in one statement:

```swift
var name: String?
var birthDate: NSDate?

if let name = name, birthDate = birthDate {
    ...
}
```

Functions should have optional return type if they may return `nil`:

```swift
func annotationOfType(type: AnnotationType) -> Annotation?
```

## Static vs Dynamic

Static code is a code which state can be resolved in compile time. Swift compiler is able to optimize code that it knows how will work before executing it. Try make use of this feature and write as much static code as it can be, so it will perform better.

Dynamic code is a code which state is resolved in runtime what means that it can’t be optimized by compiler since it doesn’t know the result or behavior of such code at compile time. Avoid using dynamic code like `@objc class`, unless you really need this feature.

Try also to minimize dynamic dispatch as much as possible, since it can decrease performance as well:

```swift
protocol Generator {
    var seed: Int { get }
    func next() -> Int
}

class Uniform: Generator {
    let seed = 5
    func next() -> Int { ... }
}

struct Exponential: Generator {
    let seed = 3
    func next() -> Int { ... }
}

func random(generator: Generator, inRange range: (Int, Int)) -> Int { 
    let next = generator.next()
    ...
}
```

Instead of:

```swift

class Generator {
    let seed: Int
    
    init(seed: Int) {
        self.seed = seed
    }
    
    randomInRange(range: (Int, Int)) -> Int
}

class Uniform: Generator {
    
    init() {
        super.init(seed: 5)
    }
}

class Exponential: Generator {
    
    init() {
        super.init(seed: 3)
    }
}
```

Please take a look at [Value semantics over reference semantics](#value-semantics-over-reference-semantics) section too.

## Implicit getters on read-only computed properties and subscripts

Read-only computed properties doesn’t need to have getter defined explicitly. It can be omitted. This also applies to read-only subscripts:

```swift
class Person {
    let height: Float
    var weight: Float

    var bmi: Float {
        return weight / (height * height)
    }
}
```

## Specify access control

Access control should be specified on a top-level of scope:

```swift
public extension Person {
    func jump()
}
```

Use access modifiers for functions, types and members only if it is necessary. Pay attention on variable access. Use private or private(set) appropriately. Don’t add access modifiers if they are already a default.

```swift
extension Person {
    func jump()
}
```

Not preferred:

```swift
internal extension Person {
    func jump()
}
```

## Refer to self only when it’s required and necessary

Refer to self only in closures or when context requires it:

```swift
UIView.animation(0.25) {
    self.containerView.alpha = 0.0
}
```

```swift
init(title: String) {
    self.title = title
}
```

## Value semantics over reference semantics

Value types are structs, enums and tuples. They are usually simpler than reference types and they are always being copied on assignment, when passing as arguments or in initialization. This means that values are independent instances what makes code simpler and safer, because they can’t be changed by other part of the app, like some other thread. Also let and var keywords work as expected.

Value types are great for representing **data** in the app.

On the other hand reference types are classes. There are several cases when you would like or should to use class. The first case is when you want to subclass Objective-C class, such as `UIView`. You will also have to use reference types to have shared, mutable state, or if you need a deinit.

Object types are great to represent **behavior** in the app.

Remember that inheritance is not a good reason to use `class`, try with `protocol` instead (composition).

## Forbidden

`Class`, `struct`, `enum`, functions and typealiases should never have prefixes, because they are automatically prefixed by the name of module they’re contained in.

Semicolons are obfuscating and they should never be used. Statements can be separated by dropping each one to separate lines.

Rewriting Swift standard library functionalities should never take place. Your code probably won’t be faster and can be confusing to other developers.

# The Swift Style Guide

This guide is based on the following sources:
- [The Swift Programming Language](https://developer.apple.com/library/prerelease/ios/documentation/Swift/Conceptual/Swift_Programming_Language/TheBasics.html#//apple_ref/doc/uid/TP40014097-CH5-ID309)
- [Github Swift style guide](https://github.com/github/swift-style-guide)
- [Ray Wenderlich Swift style guide](https://github.com/raywenderlich/swift-style-guide)

## Purpose of the style guide

This guide should help you improve your Swift code style, its readability, consistency and simplicity. The guide is not a manifest, it doesn’t tell you what to do (except some parts when it tells what not to do). It is a set of hints about subjectively good style and practices that might help you and your team decrease number of programmers errors.

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

Vertical spaces should be used in long methods to separate its name from implementation, what improves readability. You also may want to use vertical spaces to separate sub responsibilities within a function. Shorter methods (one or two lines) don’t need such spacing.

## Comments

Use comments to describe why is something written as it is, or work as it works. Remember that code should be self-documenting, so use comments only if necessary. If you decide to add a comments, keep them up-to-date. Comments that was not maintained should be deleted.

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

This organization helps other programmers read what is important first. It saves time, confusion and improves readability.

## General naming

Names of `class`, `struct`, `enum`, enums cases, `typealias`, `protocol` and generic types’ names should be capitalized. Moreover, generic types’ names should preferably start with `T` letter, then `U`, `V` and so on. 

All names should be written in camel case manner and should be meaningful and compact. Try to ask yourself if name explains the behavior or role of class, struct etc. Correct naming is very important for other people working with your code, because it defines some expectations about it. 

It is strongly misadvised to name `class`, `struct`, etc.mon with keywords like `Manager`, `Helper` or `Utility`, because they’re meaningless and its role can be easily misunderstood.

## Functions naming and arguments

Functions names should be as much descriptive and meaningful as possible. Try to express the intent of a function in its name keeping it compact at the same time. 

Functions’ arguments should be descriptive. Remember that you can use shorter arguments names to use in function implementation while in invocation they can be more meaningful for user. The first argument’s name can be a part of a function name:

```swift
func convertPoint(point: CGPoint, toCoordinateSystem system: CoordinateSystem) -> CGPoint
```

or its name can be explicitly required:

```swift
func convert(#point: CGPoint, toCoordinateSystem system: CoordinateSystem) -> CGPoint
```

Use default values for arguments in situations when function expects any value, so it can be omitted, or most of the time argument would have some specific value. If function doesn’t require a particular value in argument, it is good to make this argument optional and make it implicitly `nil`.

```swift
func describe(#operation: Operation, includingOperationError error: NSError? = nil) -> String
```

## Closures

Trailing closures should be added after parenthesis of a function. If function doesn’t take any arguments besides closure, parenthesis can be omitted and closure can be added right after function name. Unused closure arguments should be replaced with `_`. Parameters types should be omitted.

```swift
func perform(#operation: Operation, completion: (Bool) -> Void)

perform(#operation: operation) { _ in
    ...
}
```

When context of a closure is clear, implicit return can be applied.

```swift
let users: [User] = ...

users.filter { $0.name != nil }
```

Remember that functions can be used in the same way as closures and sometimes it is convenient to pass function as a closure.

## Types

Try to use native types before you will come up with your one. Every type in Swift can be extended, so instead of introducing new types it is convenient to try to extend existing ones. 

Remember that Objective-C classes that has their native equivalents in Swift are not automatically bridged. For example, `NSString` won’t be automatically bridged to `String`. You have to be explicit in the type:

```swift
func inverse(string: String)

let string: NSString = …

inverse(string) // compile error
inverse(string as String) // no error 
```

Types should be inferred, don’t duplicate type identifier if it can be resolved in compile time:

```swift
let name = “text”
var guides = [Guide]()
var addressBook = [String: String]()
```

Not preferred:

```swift
let name: String = "text"
let guides: [Guide] = []
let addressBook: [String: String] = [:]
```

Also associate colon with type identifier.

Typealiases should be short but meaningful:

```swift
typealias MoneyAmount = Double

struct Item {
    let price: MoneyAmount
}
```

Not preferred:

```swift
typealias Money = Double

struct Item {
    let price: Money
}
```


## Mutability - `let` vs `var`

It’s safer to assume that values are immutable, thus it’s highly recommended to declare values with `let`. Immutable values assures the state of a value what in consequence result in less error prone code. 

`var` can be used only if their necessary, for example you are absolutely sure value will change in the future.

## Optionals

Force-unwrapping should be avoid, because it leads to less safer code and can cause unwanted crashes.

```swift
let result: Result<String>?

result?.print()
```

instead of:

```swift
result!.print()
```

Implicitly unwrapped optionals should also be avoided, however thay can be useful in applications like unit tests, where the system under test should never be `nil`. There’s no point in executing rest of tests if one of them is written badly, so it actually saves you time. 

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

Unwrap optional value when it makes sense, for example you will need to make bunch of operations on it. 

```swift
var button: UIButton?

if let button = button {
    button.backgroundColor = UIColor.blackColor()
    button.layer.cornerRadius = 5.0
    button.layer.borderWidth = 1.0
}
```

Otherwise use optional chaining, because it’s clear and safe:

```swift
button?.backgroundColor = UIColor.blackColor()
```

Unwrapping several optionals in nested if statements (pyramid of doom) is forbidden. Swift allows unwrapping of multiple optionals in one if statement:

```swift
var name: String?
var birthDate: NSDate?

if let name = name, birthDate = birthDate {
    ...
}
```

Functions should have optional return type if they may return `nil`.

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

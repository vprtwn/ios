# Swift Style Guide

#### Prefer `let` over `var` wherever possible

Use `let foo = …` over `var foo = …` wherever possible (and when in doubt). Only use `var` if you absolutely have to (i.e. you *know* that the value might change, e.g. when using the `weak` storage modifier).

_Rationale:_ The intent and meaning of both keywords is clear, but *let-by-default* results in safer and clearer code.

A `let`-binding guarantees and *clearly signals to the programmer* that its value is supposed to and will never change. Subsequent code can thus make stronger assumptions about its usage.

#### Avoid force-unwrapping optionals

If you have an identifier `foo` of type `FooType?` or `FooType!`, don't force-unwrap it to get to the underlying value (`foo!`) if possible.

Instead, use an `if let`:

```swift
if let foo = foo {
    // Use unwrapped `foo` value in here
} else {
    // If appropriate, handle the case where the optional is nil
}
```

Or use optional chaining:

```swift
// Call the function if `foo` is not nil. If `foo` is nil, ignore we ever tried to make the call
foo?.callSomethingIfFooIsNotNil()
```

_Rationale:_ Explicit `if let`-binding of optionals results in safer code. Force unwrapping is more prone to runtime crashes.

#### Avoid using implicitly unwrapped optionals

Where possible, use `let foo: FooType?` instead of `let foo: FooType!` if `foo` may be nil.

_Rationale:_ Explicit optionals result in safer code. Implicitly unwrapped optionals have the potential of crashing at runtime.

#### Prefer implicit getters on read-only properties and subscripts

When possible, omit the `get` keyword on read-only computed properties and
read-only subscripts.

**For example:**
```swift
var myGreatProperty: Int {
	return 4
}

subscript(index: Int) -> T {
    return objects[index]
}
```

**Not:**
```swift
var myGreatProperty: Int {
	get {
		return 4
	}
}

subscript(index: Int) -> T {
    get {
        return objects[index]
    }
}
```

#### Always specify access control explicitly for top-level definitions

Top-level functions, types, and variables should always have explicit access control specifiers:

```swift
public var whoopsGlobalState: Int
internal struct TheFez {}
private func doTheThings(things: [Thing]) {}
```

However, definitions within those can leave access control implicit, where appropriate:

```swift
internal struct TheFez {
	var owner: Person = Joshaber()
}
```

_Rationale:_ It's rarely appropriate for top-level definitions to be specifically `internal`, and being explicit ensures that careful thought goes into that decision. Within a definition, reusing the same access control specifier is just duplicative, and the default is usually reasonable.

#### When specifying a type, always associate the colon with the identifier

When specifying the type of an identifier, always put the colon immediately
after the identifier, followed by a space and then the type name.

```swift
class SmallBatchSustainableFairtrade: Coffee { ... }

let timeToCoffee: NSTimeInterval = 2

func makeCoffee(type: CoffeeType) -> Coffee { ... }
```

When specifying the type of a dictionary, always put the colon immediately
after the key type, followed by a space and then the value type.

```swift
let capitals: [Country: City] = [ Sweden: Stockholm ]
```

#### Only explicitly refer to `self` when required

When accessing properties or methods on `self`, leave the reference to `self` implicit by default:

```swift
private class History {
	var events: [Event]

	func rewrite() {
		events = []
	}
}
```

Only include the explicit keyword when required by the language—for example, in a closure, or when parameter names conflict:

```swift
extension History {
	init(events: [Event]) {
		self.events = events
	}

	var whenVictorious: () -> () {
		return {
			self.rewrite()
		}
	}
}
```

_Rationale:_ This makes the capturing semantics of `self` stand out more in closures, and avoids verbosity elsewhere.

#### Use `final` by default
Classes should start as `final`, and only be changed to allow subclassing if a valid need for inheritance has been identified. Even in that case, as many definitions as possible within the class should be final as well, following the same rules.

_Rationale:_ 
* Composition is usually preferable to inheritance
* Using `final` allows the compiler to eliminate dynamic dispatch.
  * Note that a `private` declaration allows the compiler to infer the `final` keyword.

#### Prefer composition over inheritance
Use protocols instead of subclassing where possible. 

_Rationale:_
* Classes provide implicit sharing, which is often undesirable. 
* Inheritance can be intrusive and add unnecessary complexity. 
* Using classes can result in weaker type relationships.

#### Prefer value types over reference types
Unless you require functionality that can only be provided by a class (like identity or deinitializers), implement a `struct` or `enum` instead.

Note that inheritance is (by itself) usually not a good reason to use classes, because polymorphism can be provided by protocols, and implementation reuse can be provided through composition.

Using a class may make sense when:
* copying or comparing instances doesn't make sense (e.g. Window)
* instance lifetime is tied to external effects (e.g. TemporaryFile)
* instances are just "sinks" – write-only conduits to external state (e.g. CGContext)

#### Use `guard` instead of `if` for early exits
Use `guard` to transfer control out of scope if a condition is not met.

**For example:**
```swift
guard let location = person["location"] else { return }
doSomething(location)
```

**Not:**
```swift
let location = person["location"] 
if location == nil { 
  return 
}
doSomething(location!)
```

#### Avoid global constants
Move the constants to an enum, and consider making the enum a nested type

**For example:**
```swift
extension UIImage {
  enum AssetIdentifier: String {
    case Isabella = "Isabella"
    case William = "William"
  }

  convenience init(assetIdentifier: AssetIdentifier) { ... }
}

let image = UIImage(assetIdentifier: .Isabella)
```

Rationale:
* strongly-typed > stringly-typed
* doesn't pollute global namespace
* compiler enforces uniqueness of enum cases
* allows APIs to be non-failable

#### Avoid global functions
Use protocol extensions instead of generic global functions.

**For example:**
```swift
extension CollectionType {
  func countIf(match: Element -> Bool) -> Int {
    ...
  }
}
```

**Not:**
```swift
func countIf<T: CollectionType>(collection: T,
                                match: T.Generator.Element -> Bool) -> Int {
   ...
}
```

#### Use `if case` to pattern match against a single case

**For example:**
```swift
if case .MyEnumCase(let value) = bar() where value != 42 {
  doThing(value)
}
```
**Not:**
```swift
switch bar() {
case .MyEnumCase(let value) = bar() where value != 42:
  doThing(value)

default: break
}
```


#### Use pattern matching to filter `for...in` loops
**For example:**
```swift
for value in mySequence where value != "" {
  doThing(value)
}

for case .MyEnumCase(let value) in enumValues {
  doThing(value)
}

for case let value? in optionals {
  doThing(value)
}
```

**Not:**
```swift
for value in mySequence {
  if value != "" {
    doThing(value)
  }
}
```

#### Use `defer` to guarantee execution when exiting scope
**For example:**
```swift
func processSale(json: AnyObject) throws {
  delegate?.didBeginReadingSale()
  defer { delegate?.didEndReadingSale() }

  guard let buyer = json["buyer"] as? NSDictionary {
    throw DataError.MissingBuyer
  }
  guard let price = json["price"] as? Int {
    throw DataError.MissingPrice
  }
  return Sale(buyer, price)
}
```

**Not:**
```swift
func processSale(json: AnyObject) throws {
  delegate?.didBeginReadingSale()

  guard let buyer = json["buyer"] as? NSDictionary {
    delegate?.didEndReadingSale()
    throw DataError.MissingBuyer
  }
  guard let price = json["price"] as? Int {
    delegate?.didEndReadingSale()
    throw DataError.MissingPrice
  }
  delegate?.didEndReadingSale()
  return Sale(buyer, price)
}
```

## References
* [Github's Swift Style Guide](https://github.com/github/swift-style-guide)
* [WWDC 2015](https://developer.apple.com/videos/wwdc/2015/)


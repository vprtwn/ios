# Swift Style Guide

#### Use `final` by default
Classes should start as `final`, and only be changed to allow subclassing if a valid need for inheritance has been identified. Even in that case, as many definitions as possible within the class should be final as well, following the same rules.

Rationale: 
* Composition is usually preferable to inheritance/
* `final` allows the compiler to eliminate dynamic dispatch.
  * Note that a `private` declaration allows the compiler to infer the `final` keyword.

#### Prefer composition over inheritance
Use protocols instead of subclassing where possible. 

Rationale:
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

##### Value types should always implement `Equatable`
By definition, value types should be equatable by value.

#### Use `guard` instead of `if` for early exits
Use `guard` to transfer control out of scope if a condition is not met.
```swift
guard let location = person["location"] else { return }
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
* [Github Swift Style Guide](https://github.com/github/swift-style-guide)


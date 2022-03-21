# Swift 5.+ updates

## Swift 5.0

### Result type

```swift
  enum NetworkError: Error {
      case badURL
  }

  Result<Int, NetworkError>
```

### Raw strings
Added the ability to create raw strings, where backslashes and quote marks are interpreted as those literal symbols rather than escapes characters or string terminators. This makes a number of use cases more easy, but regular expressions in particular will benefit.

To use raw strings, place one or more # symbols before your strings, like this:

```swift
  let str = ##"My dog said "woof"#gooddog"##
```

### Dynamically callable types
SE-0216 adds a new @dynamicCallable attribute to Swift, which brings with it the ability to mark a type as being directly callable.

```swift
  @dynamicCallable
  struct RandomNumberGenerator {
      func dynamicallyCall(withArguments args: [Int]) -> Double {
          let numberOfZeroes = Double(args[0])
          let maximum = pow(10, numberOfZeroes)
          return Double.random(in: 0...maximum)
      }
  }

  let random = RandomNumberGenerator()
  let result = random(0)
```
### Handling future enum cases

```swift
  func showNew(error: PasswordError) {
    switch error {
    case .short:
        print("Your password was too short.")
    case .obvious:
        print("Your password was too obvious.")
    @unknown default:
        print("Your password wasn't suitable.")
    }
  }
```
## Swift 5.1

### Implicit returns from single-expression functions

```swift
  func double(_ number: Int) -> Int {
    number * 2
  }
```

### Universal Self
SE-0068 expands Swift’s use of Self so that it refers to the containing type when used inside classes, structs, and enums.
This is particularly useful for dynamic types, where the exact type of something needs to be determined at runtime.

```swift
  class ImprovedNetworkManager {
    class var maximumActiveRequests: Int {
        return 4
    }

    func printDebugData() {
        print("Maximum network requests: \(Self.maximumActiveRequests).")
    }
  }
```

### Opaque return types
SE-0244 introduces the concept of opaque types into Swift. 
An opaque type is one where we’re told about the capabilities of an object without knowing specifically what kind of object it is.

```swift
  protocol Fighter { }
  struct XWing: Fighter { }

  func launchOpaqueFighter() -> some Fighter {
    return XWing()
  }
```

From the caller’s perspective that still gets back a Fighter, which might be an XWing, a YWing, or something else that conforms to the Fighter protocol. 
But *from the compiler’s perspective* it knows exactly what is being returned, so it can make sure we follow all the rules correctly.

### Static and class subscripts
SE-0254 adds the ability to mark subscripts as being static, which means they apply to types rather than instances of a type.

```swift
  public enum NewSettings {
    private static var values = [String: String]()

    public static subscript(_ name: String) -> String? {
        get {
            return values[name]
        }
        set {
            print("Adjusting \(name) to \(newValue ?? "nil")")
            values[name] = newValue
        }
    }
  }

  NewSettings["Captain"] = "Gary"
  NewSettings["Friend"] = "Mooncake"
  print(NewSettings["Captain"] ?? "Unknown")
```

## Swift 5.2

### Key Path Expressions as Functions
SE-0249 introduced a marvelous shortcut that allows us to use keypaths in a handful of specific circumstances.

```swift
  struct User {
    let name: String
    let age: Int
    let bestFriend: String?

    var canVote: Bool {
        age >= 18
    }
  }

  let voters = users.filter(\.canVote)
  let bestFriends = users.compactMap(\.bestFriend)
```

### Callable values of user-defined nominal types
SE-0253 introduces statically callable values to Swift, which is a fancy way of saying that you can now call a value directly if its type implements a method named callAsFunction().
You don’t need to conform to any special protocol to make this behavior work; you just need to add that method to your type.

```swift
  struct StepCounter {
    var steps = 0

    mutating func callAsFunction(count: Int) -> Bool {
        steps += count
        print(steps)
        return steps > 10_000
    }
  }

  var steps = StepCounter()
  let targetReached = steps(count: 10)
```

## Swift 5.3

### Multi-pattern catch clauses
SE-0276 introduced the ability to catch multiple error cases inside a single catch block, which allows us to remove some duplication in our error handling.

```swift
  enum TemperatureError: Error {
      case tooCold, tooHot
  }

  do {
    let result = try checkReactorOperational()
    print("Result: \(result)")
  } catch TemperatureError.tooHot, TemperatureError.tooCold {
      print("Shut down the reactor!")
  } catch {
      print("An unknown error occurred.")
  }
```

### Synthesized Comparable conformance for enums
SE-0266 lets us opt in to Comparable conformance for enums that either have no associated values or have associated values that are themselves Comparable. 
This allows us to compare two cases from the same enum using <, >, and similar.

```swift
  enum WorldCupResult: Comparable {
    case neverWon
    case winner(stars: Int)
  }

  let americanMen = WorldCupResult.neverWon
  let americanWomen = WorldCupResult.winner(stars: 4)
  let japaneseMen = WorldCupResult.neverWon
  let japaneseWomen = WorldCupResult.winner(stars: 1)

  let teams = [americanMen, americanWomen, japaneseMen, japaneseWomen]
  let sortedByWins = teams.sorted()
  print(sortedByWins)
```

## Swift 5.4

### Multiple variadic parameters in functions
SE-0284 introduced the ability to have functions, subscripts, and initializers use multiple variadic parameters as long as all parameters that follow a variadic parameter have labels. Before Swift 5.4, you could only have one variadic parameter in this situation.

```swift
  func summarizeGoals(times: Int..., players: String...) {
    let joinedNames = ListFormatter.localizedString(byJoining: players)
    let joinedTimes = ListFormatter.localizedString(byJoining: times.map(String.init))

    print("\(times.count) goals where scored by \(joinedNames) at the follow minutes: \(joinedTimes)")
  }

  summarizeGoals(times: 18, 33, 55, 90, players: "Dani", "Jamie", "Roy")
```

## Swift 5.5

### Async/await + Actors
SE-0296 introduces asynchronous (async) functions into Swift, allowing us to run complex asynchronous code almost is if it were synchronous. 
This is done in two steps: marking async functions with the new async keyword, then calling them using the await keyword, similar to other languages such as C# and JavaScript.

To see how async/await helps the language, it’s helpful to look at how we solved the same problem previously. 
Completion handlers are commonly used in Swift code to allow us to send back values after a function returns, but they had tricky syntax as you’ll see.

```swift
  // TODO: - Replace link with Swift code
  https://www.hackingwithswift.com/articles/233/whats-new-in-swift-5-5
```
## Swift 5.6

### Introduce existential any
SE-0335 introduces a new any keyword to mark existential types, and although that might sound esoteric please don’t skip ahead: this one is a big change, and is likely to break a lot of Swift code in future versions.

```swift
  protocol Vehicle {
    func travel(to destination: String)
  }

  struct Car: Vehicle {
      func travel(to destination: String) {
          print("I'm driving to \(destination)")
      }
  }

  let vehicle = Car()
  vehicle.travel(to: "London")

  func travel<T: Vehicle>(to destinations: [String], using vehicle: T) {
      for destination in destinations {
          vehicle.travel(to: destination)
      }
  }

  travel(to: ["London", "Amarillo"], using: vehicle)

  let vehicle2: Vehicle = Car()
  vehicle2.travel(to: "Glasgow")

  func travel2(to destinations: [String], using vehicle: Vehicle) {
      for destination in destinations {
          vehicle.travel(to: destination)
      }
  }

  let vehicle3: any Vehicle = Car()
  vehicle3.travel(to: "Glasgow")

  func travel3(to destinations: [String], using vehicle: any Vehicle) {
      for destination in destinations {
          vehicle.travel(to: destination)
      }
  }
```

### Type placeholders
SE-0315 introduces the concept of type placeholders, which allow us to explicitly specify only some parts of a value’s type so that the remainder can be filled in using type inference.

In practice, this means writing _ as your type in any place you want Swift to use type inference, meaning that these three lines of code are the same:

```swift
  let score1 = 5
  let score2: Int = 5
  let score3: _ = 5

  var results3: [_: [Int]] = [
      "Cynthia": [],
      "Jenny": [],
      "Trixie": [],
  ]

  struct Player<T: Numeric> {
    var name: String
    var score: T
  }

  func createPlayer() -> _ {
      Player(name: "Anonymous", score: 0)
  }
```

### Unavailability condition

```swift
  if #unavailable(iOS 15) {
    // Code to make iOS 14 and earlier work correctly
  }
```


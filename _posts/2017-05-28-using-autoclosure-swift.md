---
title: Using @autoclosure when designing Swift APIs
date: 2017-05-28 19:02:00 -0300
categories: [Swift]
tags: [language features, api design, Swift5.0]     # TAG names should always be lowercase
---

Swift's `@autoclosure` attribute enables you to define an argument that automatically gets wrapped in a closure. It's primarily used to defer execution of a (potentially expensive) expression to when it's actually needed, rather than doing it directly when the argument is passed.

Let's explain this in more detail using the following code example. In this example, we've created a `debugLog` method and a `Person` struct which we're going to print out:

```swift
struct Person: CustomStringConvertible {
    let name: String

    var description: String {
        print("Asking for Person description")
        return "Person name is \(name)"
    }
}

let isDebuggingEnabled: Bool = false

func debugLog(_ message: String) {
    // You could replace this in projects with #if DEBUG
    if isDebuggingEnabled {
        print("[DEBUG] \(message)")
    }
}

let person = Person(name: "Bernie")
debugLog(person.description)
```

Even though we disabled debugging, the `Person` struct is still asked for its description. This is because the message argument of `debugLog` is directly computed.

We can solve this by making use of a closure:

```swift
let isDebuggingEnabled = false

func debugLog(_ message: () -> String) {
    if isDebuggingEnabled {
        print("[DEBUG] \(message())")
    }
}

let person = Person(name: "Bernie")
debugLog({ person.description })
```

The `message()` closure is only called when debugging is enabled. You can see that we now need to pass in a closure argument to the `debugLog` method which doesn't look so nice.

We can improve this code by making use of the `@autoclosure` keyword:

```swift
func debugLog(_ message: @autoclosure () -> String) {
    ...
}
let person = Person(name: "Bernie")
debug(person.description)
```

The logic within the `debugLog` method stays the same and still has to work with a closure. However, on the implementation level, we can now pass on the argument as if it were a normal expression. It looks both clean and familiar while we did optimize our debug logging code.


Another example of when this is used in the Swift standard library is the `assert` function. Since asserts are only triggered in debug builds, there's no need to evaluate the expression that is being asserted in a release build.
This is where `@autoclosure` comes in:

```swift
func assert(_ expression: @autoclosure () -> Bool, 
            _ message: @autoclosure () -> String) {
    guard isDebug else {
        return
    }

    // Inside assert we can refer to expression as a normal closure
    if !expression() {
        assertFailure(message())
    }
}

```

> I'm paraphrasing the implementation of `assert` a bit above, the actual implementation can be found here.


The nice thing about `@autoclosure` is that it has no effect on the call site. If `assert` was implemented using "normal" closures you'd have to use it like this:

```swift
assert({ someCondition() }, { "Hey, it failed!" })

```

But now, you can just call it like you would any function that takes non-closure arguments:

```swift
assert(someCondition(), "Hey it failed!")
```

Let's take a look at how we can use `@autoclosure` in our own code, and how it enables us to design some pretty nice APIs.

## Inlining assignments

One thing that `@autoclosure` enables is to inline expressions in a function call. This enables us to do things like passing assignment expressions as an argument. Let's take a look at an example where this can be useful.

On iOS, you normally define view animations using this API:

```swift
UIView.animate(withDuration: 0.25) {
    view.frame.origin.y = 100
}
```

With `@autoclosure`, we could write an `animate` function that automatically creates an animation closure and executes it, like this:

```swift
func animate(_ animation: @autoclosure @escaping () -> Void,
            duration: TimeInterval = 0.25) {
    UIView.animate(withDuration: duration, animations: animation)
}
```

Now, we can simply perform our animation with a simple function call without any extra `{}` syntax:
```swift
animate(view.frame.origin.y = 100)
```

Using the above technique, we can really reduce the verbosity of our animation code, without sacrificing readability or expressiveness.

## Passing errors as expressions

Another situation that I find `@autoclosure` very useful in is when writing utilities that deal with errors. For example, let's say we want to add an extension on `Optional` that enables us to unwrap it using a `throwing` API. That way we can require the optional to be non-`nil`, or else throw an error, like this:

```swift
extension Optional {
    func unwrapOrThrow(_ errorExpression: @autoclosure () -> Error) throws -> Wrapped {
        guard let value = self else {
            throw errorExpression()
        }

        return value
    }
}
```

Similar to how `assert` is implemented, we only evaluate the error expression when needed, rather than having to do it for every time we attempt to unwrap an optional. We can now use our `unwrapOrThrow` API like this:
```swift
let name = try argument(at: 1).unwrapOrThrow(ArgumentError.missingName)
```

## Type inference using default values

The final use case for `@autoclosure` that I've found is when extracting an optional value from a dictionary, a database, or `UserDefaults`.

Normally, when extracting a value from an untyped dictionary and providing a default value, you'd have to write something like this:

```swift
let coins = (dictionary["numberOfCoins"] as? Int) ?? 100
```

That's kind of hard to read, and has a lot of syntax cruft with the casting and the `??` operator. With `@autoclosure`, we can define an API that enables us to write the same expression like this instead:

```swift
let coins = dictionary.value(forKey: "numberOfCoins", defaultValue: 100)
```

Above, we can see that the default value is both used for when a value was missing, but also enables Swift to do type inference on the value, without us having to specify the type or perform casting. Pretty neat.

```swift
extension Dictionary where Value == Any {
    func value<T>(forKey key: Key, defaultValue: @autoclosure () -> T) -> T {
        guard let value = self[key] as? T else {
            return defaultValue()
        }
        return value
    }
}
```

> Again, we use `@autoclosure` to avoid having to evaluate the default value every time this method is called.

## Conclusion 

Reducing verbosity is always something that needs to be done with careful consideration. Our goal should always be to write expressive, easy to read code, so we need to make sure that we don't remove important information from the call site when designing low verbosity APIs.

I think when used in appropriate situation, `@autoclosure` is a great tool for doing just that. Dealing with expressions, instead of just values, enables us to reduce verbosity and cruft, while also potentially gaining better performance.

Thanks for reading!



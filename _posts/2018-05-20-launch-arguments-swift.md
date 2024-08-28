---
title: Launch arguments in Swift
date: 2018-05-20 19:07:00 -0300
categories: [Swift]
tags: [scripting, debugging, ui testing, swift5.0, launch arguments]     # TAG names should always be lowercase
---

Launch arguments are probably most commonly used as input to command line tools.

While Swift can be a great language for creating command line tools, let's take a look at how we can also use the power of launch arguments when working on, debugging, and testing an iOS app.

## Parsing 

Swift provides two built-in ways to parse launch arguments. The first one is through the `CommandLine` API, which enables easy access to information passed when launching the app (even on platforms like iOS that don't ship with a terminal). Using `CommandLine.arguments` we get an array of strings representing each argument, which we here use to determine whether a new implementation of a `ProfileViewController` should be used:

```swift
func makeProfileViewController() -> UIViewController {
    if CommandLine.arguments.contains("-new-profile") {
        // If the "-new-profile" arguments was passed, then return
        // an instance of the new implementation.
        return ProfileViewController()
    }

    // Fall back to the old implementation, which is what we
    // still use in production.
    return ProfileLegacyViewController()
}

```

While `CommandLine` provides a simple way to get access to the "raw" arguments, it doesn't actually provide any sophisticated parsing capabilities. To gain access to a bit more powerful features, we can actually use `UserDefaults` to parse launch arguments.

The fact that `UserDefaults` contains all arguments passed on the command line - and can do basic conversion to types like `Bool`, `Int` and `Double` - is a bit of a hidden gem. Here we're using that feature to enable us to override how many articles that should be loaded when performing a network request:

```swift
func loadArticle(then handler: @escaping (Result<[Article]>) -> Void) {
    // The UserDefaults API automatically parses types of like integers
    // doubles from strings passed as launch arguments. In this 
    // case we'll treat 0 as "no limit".
    let limit = UserDefaults.standard.integer(forKey: "article-limit")
    let endpoint = Endpoint.articleList(limit: limit)

    dataLoader.loadData(from: endpoint) { result in
        handler(result.decoded())
    }
}

```

## Passing

Now that we have ways to parse command line arguments in an iOS app, we also need a way to pass them.

When running and debugging, this is done through Xcode, by adding any arguments that we want to pass to our app's scheme. An easy way to access this option in Xcode is press ⌘⌥R, selecting "Arguments" and adding our arguments under "Arguments Passed On Launch".

So what can launch arguments be useful for? Let's take a look at three use cases - debugging, working on a new feature and UI testing.

## Debug actions

Launch arguments can provide a simple and quick way to perform common actions when debugging an app.

For example, let's say that we're working on an app that performs a large number of network request, and that we've had reports from users that the app performs poorly when used on slow networks.

To debug this problem, we might want to add artificial delays to all network requests, so that we can observe how our app behaves under this type of conditions. While iOS provides the Link conditioner tool, which is an excellent option for on-device debugging, when initially working on a fix we might want quicker iteration times by using the simulator.

To make that happen, we can use a launch arguments to add delays to all network requests, like this:

```swift

class DataLoader {
    private let session: URLSession
    private let userDefaults: UserDefaults

    func loadData(from endpoint: Endpoint,
                 then handler: @escaping (Result<Data>) -> Void) {
        let url = endpoint.url
        let task = makeTask(for: url, handler: handler)

        // If the app is running in debug mode, add a delay
        // according to the "network-delay" launch argument
        #if DEBUG
        let delay = userDefaults.double(forKey: "network-delay")
        DispatchQueue.main.asyncAfter(deadline: .now() + delay,
                                    execute: task.resume)
        #else
        task.resume()
        #endif
    }
}
```

Important to note above is that we wrap our delaying code in a `#if DEBUG` compiler condition, to prevent this code from accidentally shipped to the App Store. That way, our debug code won't even be compiled when doing a release build.

Similarly, we can add launch arguments to control other types of debug actions in our app - for example;

navigating to a given screen when the app launches, whether an in-app purchase has been unlocked, or simulating data from things like the Health app.

## Overriding feature flags

Another situation when launch arguments can come in handy is working on a new feature that hasn't been fully shipped yet. Like we took a look at in "Feature flags in Swift", using feature flags can be a great way to enable features to be gradually rolled out to users, and to perform experiments and A/B testing.

However, when working on an app that relies heavily on feature flags, it can be a bit time consuming and tricky to get the app into the exact state that's needed in order to be able to work on a specific feature. To solve that problem, we can introduce launch arguments that let us add local overrides to feature flags that normally get their value dynamically from our server:

```swift

func enableSearchIfNeeded() {
    var override: Bool?
    let key = "search"

    // We first  check if the argument was actually passed, before
    // asking UserDefaults to covert it into a Bool, otherwise, 
    // the dynamic value will never be used.
    #if DEBUG
    if userDefaults.value(forKey: key) != nil {
        override = userDefaults.bool(forKey: key)
    }
    #endif

    if override ?? featureFlags.searchEnabled {
        enableSearch()
    }
}

```

While the above code is specific to the Search feature, we could quite easily generalize it to be applicable to any feature flag, and call it in a shared code path - for example, when we load our values from the server.

## Setting state

Often when debugging or testing an app, we need to put it into a specific state, in order to either reproduce a bug or to be able to use a certain feature. This is something that is often repetitive (and a bit annoying) to have to do over and over again when working on something - so let's automate it using launch arguments!

First up, let's add an easy way to completely reset our app. This could of course be done by manually uninstalling the app, but it'd be a lot easier to simply pass a `-reset` argument when running the app to have it launch in a blank state. To do that, let's add a `resetIfNeeded()` method that we call from our `AppDelegate` when the app launches, like this:

```swift

extension AppDelegate {
    func resetIfNeeded() {
        guard CommandLine.arguments.contains("-reset") else {
            return
        }

        // We can reset our user defaults by removing the persistance 
        // for our app's bundle identifier
        let defaultsName = Bundle.main.bundleIdentifier!
        userDefaults.removePersistentDomain(forName: defaultName)

        // Reset any caching mechanisms, databases, etc.
        cache.reset()
        database.reset()
    }
}

```

Similarly, it's also really convenient to not only be able to reset the app, but also enable it to be put into a specific initial state.
Let's say we're building an app that contains a list of user contacts. Using a launch argument, we can provide a quick way to pre-populate our contacts database with a given list of names (in this case we use a comma-separated list as our input format):

```swift

extension ContactsManager {
    func addNamesFromCommandLine() {
        guard let argument = userDefaults.string(forKey: "contacts") else {
            return
        }

        let names = argument.components(separatedBy: ",")
        names.forEach(add)
    }
}

```

## UI testing

Both being able to completely reset our app's state, and to pre-populated its data, is not only useful for debugging - it also enables us to easily setup a specific state when doing UI testing.

Like we took a look at in "Getting stated with Xcode UI testing in Swift" and "UI testing analytics code in Swift", we can add launch arguments to `XCUIApplication` that can then be read by our app, providing an (otherwise missing) channel of communication between our tests and our app. Here we use both the `-reset` and `-contacts` launch arguments to setup an initial state for a test that verifies that we can remove a contact from the contact list:

```swift

func testRemovingContact() {
    // Setup and launch the app
    let app = XCUIApplication()
    app.launchArguments = ["-reset", "-contacts", "John, Mary"]
    app.launch()

    // Verify that we initially are displaying 2 contacts
    XCTestAssertEqual(app.tables.cells.count, 2)

    // Swipe the cell for a given contact and tap the "Remove" button
    let cell = app.tables.cells["Contact-John"]
    cell.swipeLeft()
    cell.buttons["Remove"].tap()

    // Verify that we no only have 1 contact left
    XCTAssertEqual(app.tables.cells.count, 1)
}

```

Using the above technique to set a specific initial state for our UI tests is a great way to combat flakiness and to speed up to the overall execution time of our test suite, since we don't have to always perform a lot of setup in each test and can instead just jump to the feature we want to verify.

## Containment

One concern when doing all of the above, is how we've essentially now scattered a lot of debugging and testing code all over our normal app code.

While adding code specifically for debugging is not necessarily a bad thing (our code base is kind of our "digital work place", at all), it would be better if we could contain it all in one place instead of spreading it all over our app. That way we'd be more in control over what debug code we actually have, and it becomes much easier to prevent such code from accidentally making its way into a release build.

One way to do is that is to move all code related to launch arguments into a dedicated type. As an example, here's how we could move our actions for resetting, delaying network requests and adding mocked contacts into a single, contained `LaunchArgumentsHandler`:

```swift

struct LaunchArgumentsHandler {
    let userDefaults: UserDefaults
    let contactsManager: ContactsManager
    let dataLoader: DataLoader
    let cache: Cache
    let database: Database

    func handle() {
        resetIfNeeded()
        addNetworkDelayIfNeeded()
        addContactsIfNeeded()
    }

    private func resetIfNeeded() {
        guard CommandLine.arguments.contains("-reset") else {
            return
        }

        let defaultsName = Bundle.main.bundleIdentifier!
        userDefaults.removePersistentDomain(forName: defaultsName)

        cache.reset()
        database.reset()
    }

    private func addNetworkDelayIfNeeded() {
        let delay = userDefaults.double(forKey: "network-delay")

        guard delay > 0 else {
            return
        }

        // We've abstracted the delaying of data loader tasks
        // into an "executor" closure, leaving our production
        // code free of any delaying code.
        dataLoader.taskExecutor = { task in
            DispatchQueue.main.asyncAfter(deadline: .now() + delay,
                                            execute: task.resume)
        }
    }

    private func addContactsIfNeeded() {
        guard let arguments = userDefaults.string(forKey: "contacts") else {
            return
        }

        let name = argument.components(separated: ",")
        names.forEach(contactManager.add)
    }
}

```

We can now either surround the above `LaunchArgumentsHandler` declaration with the same `#if DEBUG` condition as before, or if we're using separate Xcode targets for our debug/staging builds and release builds - we can simply exclude the `LaunchArgumentsHandler.swift` file from our production target.

By doing that, we now both get a much cleaner overview over what launch argument actions that are available, and we'll get a compilation error when doing a release build in case we're accidentally using any debug code where it doesn't belong.

All we have to do now is to call `LaunchArgumentsHandler` after we've set up our app (for example in our `AppDelegate`) and surround that call with `#if DEBUG` to have it compile correctly under all conditions.

## Conclusion

Launch arguments can provide an easy way to set up really useful debugging and mocking actions that can help speed up our development and testing. By adding launch arguments we can quickly get into a state that we desire, or reset our app completely on each launch, removing the need to manually set these things up every time we run the app.

While there's definitely a risk associated with adding these type of debugging actions - since we are introducing more code paths and possible states to our app - keeping the amount of launch arguments low and containing all code dealing with them in a single place really helps mitigating that risks. Like with many things, it becomes a balancing act of risks vs reward, and picking some key launch arguments (and clean up old ones) can definitely make that balance tip in our favor.

What do you think? Do you use launch arguments to speed up your development and testing, or is it something you'll try out? Let me know.

Thanks for reading!
---
title: Feature flags in Swift
date: 2018-03-25 19:02:00 -0300
categories: [Swift]
tags: [architecture, feature flags, maintenance, swift5.0]    # TAG names should always be lowercase
---

When developing new features for an app, it can be really useful to have some form of mechanism to gradually roll out new implementation & functionality, instead of having to launch to every single user at once. Not only can this help "de-risk" the launch of a big change (if something breaks, we can always roll back), it can also help us gather feedback on a partially finished feature, or perform experiments using techniques like A/B testing.

Feature flags can act as such a mechanism. They essentially allow us to gate certain parts of our code base off under certain conditions, either at compile time or at runtime. Let's take a look at a few different ways that feature flags can be used in Swift.

## Conditional compilation

When working on a code base there are multiple strategies we can use when it comes to dealing with features that are a work in progress. We can -for example - use something like feature branches, and use version control to keep a feature that's under development completely separated from our main `master` branch.
Once the feature is ready to be released, we simply merge it in and ship it.

However, there are some big advantages to instead continuously integrate new implementation and features into our main branch. It lets us detect bugs and problems earlier, it saves us the pain of having to solve a massive number of merge conflicts if the two branches have diverged a lot, and it can let us ship a pre-release version of a new feature internally or to beta testers.

But we still need some way to remove code that shouldn't be shipped live to the App Store. One way of doing so is to use compiler flags, which lets us mark a code block so that is only get compiled in if the flag has been set.
Let's say our app is currently using Core Data, and we want to keep it that way in production(for now), while still being able to try out a new solution - like Realm. To do that, we can use a `DATABASE_REALM` compiler flag, that we only add for builds that we want to use Realm in (for example for beta builds). We can then tell the compiler to check that flag when building our app, like this:

```swift

class DataBaseFactory {
    func makeDatabase() -> Database {
        #if DATABASE_REALM
        return RealmDatabase()
        #else
        return CoreDataDatabase()
        #endif
    }
}

```

To toggle the above flag on or off, we can simply open up our target's build settings in Xcode and add or remove `DATABASE_REALM` under _Swift Compiler - Custom Flags > Active Compilation Conditions_. This is especially useful for features that are still under active development, which lets the developers working on such a feature easily turn it on locally without affecting production builds.

## Static flags

Conditional compilation is super useful when you want to completely remove the code for a new implementation from an app. But sometimes that's either not needed or not practical and in those cases defining feature flags in code instead can be a much better option.

```swift
struct FeatureFlags {
    static let searchEnabled = false
    static let maximumNumberOfFavorites = 10
    static let allowLandscapeMode = true
}

```

> As you can see above, flags can also be super useful in order to tweak an existing feature, not only to roll out brand new ones. Using the above `maximumNumberOfFavorites` property we can easily experiment with how many favorites a user can have, to find a value that we think will strike the right balance.

With the above `FeatureFlags` type in place, we can now place checks in code paths where we'd activate a give feature. Here's an example method that conditionally activates the search feature, which gets called from the `viewDidLoad()` method of a `ListViewController`:

```swift
extension ListViewController {
    func addSearchIfNeeded() {
        // If the search feature shouldn't be enabled, we simply return
        guard FeatureFlags.searchEnabled else {
            return
        }

        let resultsVC = SearchResultsViewController()
        let searchVC = UISearchController(searchResultsController: resultsVC)

        searchVC.searchResultsUpdater = resultsVC
        navigationItem.searchController = searchVC
    }
}
```

The benefit of static flags is that they, just like compiler flags, are quite easy to setup and integrate. However, they don't let us modify the value of our app has been compiled. To be able to do that, we need to start using runtime flags.

## Runtime flags

Adding the option to configure our app's feature flags at runtime can be a bit of a "double edged sword". On one hand, it can enable us to perform A/B testing by changing the value of a given flag for a certain percentage of our user base, and on the other hand it can make our app more difficult to maintain & debug - since the code paths it'll end up using are not fully determined at compile time.

Runtime flags are often loaded from some form of backend system, and could potentially (depending on the app's architecture) even be included in the response the app receives as part of logging a user in (otherwise it's common to have a `/feature_flags` endpoint or similar that the app queries at launch). Optionally, we could also enable flags to be tweaked in the app itself using some form of debug UI.

Regardless of how we load the values from our feature flags, we'll want to update our `FeatureFlags` type to use instance properties instead of static ones. That way we can load the values for our flags and then transform them into a `FeatureFlags` instance, which we'll then inject whenever needed. Our flags type now looks like this:

```swift
struct FeatureFlags {
    let searchEnable: Bool
    let maximumNumberOfFavorites: Int
    let allowLandscapeMode: Bool
}
```

To be able to transform an instance from a serialized format, we'll also add an initializer that takes a dictionary. That way we can either create our feature flags from a JSON backend response, or from values stored locally in the app (for example from a cache):

```swift
extension FeatureFlags {
    init(dictionary: [String: Any]) {
        searchEnabled = dictionary.value(for: "search", default: false)
        maximumNumberOfFavorites = dictionary.value(for: "favorites", default: 10)
        allowLandscapeMode = dictionary.value(for: "landscape", default: true)
    }
}

private extension Dictionary where Key == String {
    func value<V>(for key: Key, default defaultExpression: @autoclosure () -> V) -> V {
        return (self[key] as? V) ?? defaultExpression()
    }
}

```

> The reason we're not using Codable above is that we want to use default values (in case our backend hasn't been updated with a given flag), which is much easier done with a simple `Dictionary` extension.
For more information about `@autoclosure`, which is used above, check more details.

We can now load our feature flags whenever our app launches or a user logs in (depending on if we want our flags to be user-specific), and inject them whenever needed, like this:

```swift

class FavoriteManager {
    private let featureFlags: FeatureFlags

    init(featureFlags: FeatureFlags) {
        self.featureFlags = featureFlags
    }

    func canUserAddMoreFavorites(_ user: User) -> Bool {
        let maxCount = featureFlags.maximumNumberOfFavorites
        return user.favorites.count < maxCount

    }
}

```

We now have a lot more freedom when it comes to togging certain features on and off, or tweaking values that determine part of our app's logic. We _could_ also enable our flags to be mutated while our app is running (and add some form of observation API to react to changes), but personally I rarely think adding that much complexity is worth it. Just loading the values once and setting up the app after that helps keep things simple and avoids introducing tricky corner cases.

## Conclusion

Using feature flags can be key when it comes to being able to quickly iterate on an app, especially as its team grows and the code base changes with a much higher velocity. By being able to conditionally enable certain features or tweak their behavior, we can usually integrate our code quicker into our main branch and still keep shipping our app.

Feature flags can also add a bit of complication to our setup, especially when runtime flags are used. It can be come harder to reproduce bugs that only occur when a certain combination of flags are on, and testing all of our app's potential code paths can quickly become much more complicated and time consuming.

If you haven't used feature flags before, I suggest to start simple (perhaps with compiler flags or static ones), and work your way from there. Like with most tools, they might require you to adopt your workflow a bit, especially if the project doesn't currently use much automated testing (which makes using feature flags much less risky).
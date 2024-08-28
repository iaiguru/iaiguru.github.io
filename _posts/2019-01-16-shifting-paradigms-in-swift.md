---
title: Shifting paradigms in Swift
date: 2019-01-16 19:02:00 -0300
categories: [SwiftUI]
tags: [swiftui, architecture, swift5.0]     # TAG names should always be lowercase
---

Every year during WWDC, we in the Apple developer community are presented with a whole suite of new tools, APIs and technologies that we can use to further improve our apps along with the systems that they run on. While most of those changes tend to be quite slow and steady - a sort of natural evolution happening over a long period of time - this year, things have turned out a bit different.

The introduction of technologies like SwiftUI (Apple's new declarative UI framework), Catalyst(iOS apps running on the Mac), and Combine (a built in "Rx-like" reactive data library) might very well be considered the beginning of a "new era" of app development for Apple's platform. That might sound a bit hyperbolic - but I think it's fair to say that Apple haven't attempted to make this big of a leap when it comes to their developer tools since the introduction of Swift in 2014.

So what does that mean for us third party developers, and how can we prepare ourselves for undergoing a major paradigm shift over the next few years - as we move from imperative to declarative, from Objective-C to Swift, and from delegates to subscribers?

## Gradual adoption, gradual mastery

When presented with any kind of technological transition - whether that's from frame-based layouts to Auto Layout, from 32-bit to 64-bit apps, or from UIkit to SwiftUI - it can be tempting to use that transition as an opportunity to start over from a completely clean slate. Out with the old legacy code, and in with the beautiful,shiny new APIs.

_"We rewrote our app from the ground up"_ is something that you often hear companies of all sizes proudly announce in blog posts, conference talks, and even in customer-facing marketing - and both developers and customers alike often get genuinely excited when hearing sentences like that. It makes an app update seem like something fresh and brand new, rather than just an incremental upgrade.

However, while full rewrites do have their metrics - and might be warranted if a code base has truly gone beyond the point of on return - they often turn out to be way less appealing in practice then they are in theory, as old bugs are replaced by new bugs, and various subtitles and handling of edge cases are missed when writing the new implementation.

Especially when undergoing major paradigm shifts, gradual adoption instead lets us ease into using all of the new technologies, pattern and tools that have just been introduced - in a way that both lets us keep leveraging our existing code base, and lets us keeping shipping our app to our users on a regular basis.

Gradually adopting new technologies also lets us dip our toes in the water before completely diving in - giving us a way to gradually master the new APIs and conventions as we're starting to use them.
After all, there's no real rush when it comes to adopting new tools and frameworks - it's not like the existing frameworks and code that we've been shipping will stop working overnight.

## Mix and match

While gradual adoption might sound great on paper, actually getting it done in practice can be much less straightforward. The key is often to find a nice way to "mix and match" our existing code and functionality with the new code and patterns that we're introducing.

As an example, when it comes to new framework like SwiftUI, Apple has - thankfully - already considered this, and is offering full interoperability between SwiftUI and UIKit. So if we wanted to start easing our way into adopting SwiftUI, by building a single screen using it - say a view for rendering in app promotions - then we could do so by wrapping our new SwiftUI `PromotionView` in an instance of `UIHostingController`, like this:

```swift
let vc = UIHostingController(rootView: PromotionView())
```

Since `UIHostingController` is just a regular UIKit view controller - it can be presented, embedded as a child, or pushed onto a navigation controller - all while being powered by SwiftUI under the hood.

When a framework has a clear backward compatibility story, that's most often a great sign - as it shows that the authors didn't only focus on building a great new set of tools, but also on how those tools will be integrated into existing projects, which usually makes for much more complete API.

Even better is when new tools and frameworks don't only offer backward compatibility, but forward compatibility as well. Again using SwiftUI as an example, an `UIView` can quite easily be made SwiftUI-compatible, by wrapping it in an implementation of the `UIViewPresentable` protocol.
Here's an example that wraps an existing `ProfileView`, which is a `UIView` subclass used to render a user's profile:

```swift
struct ProfileSwiftUIView: UIViewPresentable {
    let user: User

    func makeUIView(context: Context) -> ProfileView {
        return ProfileView()
    }

    func updateUIView(_ view: ProfileView, context: Context) {
        view.nameLabel.text = user.name
        view.imageView.image = user.image
    }
}
```

Both of the above two techniques are clear example of how new developer paradigms - like SwiftUI - can be implemented piece by piece, rather than through full rewrites, given that the right tooling has been put in place.
Gradual adoption isn't always possible, but when it is, it lets us both use new technologies to build new features - while still making full use of our existing code as well - kind of giving us the best of both worlds.

## Parallel implementation

Since paradigm shifts usually take place over quite a long period of time(years often), simply being able to build new features using new tools an APIs isn't always enough. We might not be prepared to drop support for old OS versions, or we might not be completely confident that a certain new tool is the right fit for what we're trying to build.

In both of those cases, using multiple, parallel implementations of the same feature can be an option. While it's an option that certainly has a cost, it does give us a ton of additional flexibility - as we can start replacing the implementation details of a given feature using new technologies, all without either abandoning existing users, existing code.

Let's take a look at how that could be done in practice, again using SwiftUI as an example. Say we wanted to start experimenting with using SwiftUI to build one of our app's features - to learn it, and to figure out how our code base could best make use of it - while still keep shipping our UIKit-based implementation, for now.

To do that, we could use the _factory pattern_ to create an abstraction that hides which UI framework that's currently used to implement our feature - an article reading screen in this case:

```swift
protocol ArticleViewControllerFactory {
    func makeViewController(for article: Article) -> UIViewController
}
```

Using the above protocol , we could now easily change which UI framework  that our feature will use, without changing any other part of our code base. We could start by wrapping our existing UIKit-based `ArticleViewController` in a factory that simply creates ne instance of it, by passing in its required dependencies - like this:

```swift
struct ArticleUIKitViewControllerFactory: ArticleViewControllerFactory {
    let navigator: ArticleNavigator
    let imageLoader: ImageLoader

    func makeViewController(for article: Article) -> UIViewController {
        return ArticleViewController(
            article: article, 
            navigator: navigator,
            imageLoader: imageLoader)
    }
}
```
Similarly, we could create another `ArticleViewControllerFactory` implementation that instead creates an instance of our SwiftUI-based `ArticleView` - and then wraps that in a `UIHostingController`, which is then returned. To be able to keeping shipping our app to users using iOS 12 and below, we'll also mark this factory implementation as being only available on iOS 13:

```swift
@available(iOS 13, *)
struct ArticleSwiftUIViewControllerFactory: ArticleViewControllerFactory {
    let navigator: ArticleNavigator
    let imageLoader: ImageLoader

    func makeViewController(for article: Article) -> UIViewController {
        let view = ArticleView(
            navigator: navigator,
            imageLoader: imageLoader
        )
        return UIHostingController(rootVie: view)
    }
}
```

With the above in place, we'd now be able to select which implementation to use based on any number of conditions - for example whether the device we're targeting is already running on iOS 13, and whether we've enable a custom `USE_SWIFT_UI` compiler flag:

```swift
let articleViewControllerFactory: ArticleViewControllerFactory = {
    #if USE_SWIFT_UI
    if #available(iOS 13, *) {
        return ArticleSwiftUIViewControllerFactory(
            navigator: navigator,
            imageLoader: imageLoader
        )
    }
    #endif

    return ArticleUIKitViewControllerFactory(
        navigator: navigator,
        imageLoader: imageLoader
    )
}()
```

Doing something like the above may seem like a lot of unnecessary extra work (why maintain two implementations of the same feature, when we can just have one?) - but it does give us a ton of extra flexibility - and a way to start easing into a new paradigm (and giving us a _"safe place"_ to learn all of its new patterns) while still being able to keep shipping our existing code, just like before.

The above approach can also be a great way to improve the overall architecture and separation of concerns within a code base, since we'd have to make sure that our core services - like `ArticleNavigator` and `ImageLoader` in this example - are completely separated from our UI code, which usually makes for an overall clear structure and easier testing.

## Conclusion

Big paradigm shifts can both be incredibly exciting, but also confusing, and a bit scary. While it's very common to get the feeling that the introduction of major new APIs and technologies suddenly turned all of our existing code into _"tech debt"_ - that's rarely the case, and there's often a way to be found that lets us keep leveraging our existing code's functionality, while also starting to adopt and learn the new paradigm's technologies and patterns.

Thanks for reading!
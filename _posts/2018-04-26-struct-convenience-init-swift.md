---
title: Struct convenience initializers
date: 2018-04-26 20:07:00 -0300
categories: [Swift]
tags: [swift5.0, models, api design]     # TAG names should always be lowercase
---

I love to struct my code using extensions in Swift. One big benefit of doing so when it comes to struct initializers, is that defining a convenience initializer doesn't remove the default one the compiler generates - best of both world!

```swift

struct Article {
    let date: Date
    var title: String
    var text: String
    var comments: [Comment]
}

extension Article {
    init(title: String, text: String) {
        self.init(date: Date(), title: title, text: text, comments: [])
    }
}

let articleA = Article(title: "Best Cupcake Recipe", text: "...")

let articleB = Article(
    date: Date(),
    title: "Best Cupcake Recipe",
    text: "...",
    comments: [
        Comment(user: currentUser, text: "Yep, can confirm!")
    ]
)
```
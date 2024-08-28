---
title: Using SwiftUI in a playground
date: 2019-03-13 19:02:00 -0300
categories: [SwiftUI]
tags: [swiftui, xcode, swift playground, swift5.1]     # TAG names should always be lowercase
---

You can totally start learning and experimenting with SwiftUI in an Xcode playground. Just import `PlaygroundSupport`, and assign a `UIHostingController` as your live view:

```swift
import SwiftUI
import PlaygroundSupport

struct MyView: View {
    var body: some View {
        Text("Hello, world!")
    }
}

let vc = UIHostingController(rootView: MyView())
PlaygroundPage.current.liveView = vc
```


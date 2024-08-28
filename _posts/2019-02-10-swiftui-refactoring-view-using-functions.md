---
title: Refactoring SwiftUI views using functions
date: 2019-02-10 17:02:00 -0300
categories: [SwiftUI]
tags: [swiftui, code structure, functions, swift]     # TAG names should always be lowercase
---

I see lots of SwiftUI comments about how it'll "force" developers to write _"Pyramids of Doom"_ with heavily nested code, which is just as false as MVC forcing developers to build massive view controllers. Here's one example of how nesting can be avoided, by using inline functions:

```swift
struct MyView: View {
    var body: some View {
        func makeVStack() -> some View {
            VStack {
                ForEach(0..<5) { _ in makeHStack() }
            }
        }

        func makeHStack() -> some View {
            HStack {
                Text("Leading")
                Text("Trailing")
            }
        }

        return ZStack {
            Color.gray
            makeVStack()
        }
    }
}

```
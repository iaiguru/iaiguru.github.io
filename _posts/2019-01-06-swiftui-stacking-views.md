---
title: Stacking views in SwiftUI
date: 2019-01-06 17:02:00 -0300
categories: [SwiftUI]
tags: [swiftui, ui development, swift5.1]     # TAG names should always be lowercase
---

SwiftUI ships with 3 main stack types - that enables you to easily align content on either the X, Y, or Z axis. And, since all SwiftUI views are composable, they can also be nested in order to build really complex UIs:

The above will result in a view which content looks like this:

```swift

struct MyView: View {
    var body: some View {
        // A ZStack stacks its child views on top of each other
        // in terms of depth, from back to front.
        ZStack {
            Color.gray
            // A VStack stacks its child views vertically, from top to bottom.
            VStack {
                // The ForEach construct enables you to create
                // multiple views in one go, for example by
                // mapping a collection of elements to views.
                ForEach(0..<5) { _ in
                    // A HStack aligns its child views horizontally,
                    // from the leading to the trailing edge.
                    HStack {
                        Text("Leading")
                        Text("Trailing")
                    }
                }
            }
        }
    }
}

```
---
title: SwiftUI mix and match
date: 2019-01-05 17:02:00 -0300
categories: [SwiftUI]
tags: [swiftui, ui development, swift]     # TAG names should always be lowercase
---

You can definitely mix and match SwiftUI with UIKit/AppKit, which means that you can adopt it gradually. Any SwiftUI hierarchy can be embedded in a view controller, and UIKit views can  be retrofitted with SwiftUI support:

```swift
// You can easily use SwiftUI view hierarchies in UIKit-based
// UIs, by wrapping them in a UIHostingController:
let vc = UIHostingController(rootView: HeaderView())

// For the opposite direction, using UIViews with SwiftUI - just
// create a wrapper that conforms to UIViewRepresentable, which
// acts just like any other SwiftUI view:
struct ProfileSwiftUIView: UIViewRepresentable {
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
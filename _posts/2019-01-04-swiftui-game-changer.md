---
title: SwiftUI is a game changer
date: 2019-01-04 19:02:00 -0300
categories: [SwiftUI]
tags: [swiftui, swift5.1]     # TAG names should always be lowercase
---

There is no doubt in my mind that SwiftUI is a complete game changer. This is all the code you need to define a UI that has:
- A navigation controller
- A table view
- Cell configuration
- Label font setup

```swift
struct ArticleListView: View {
    let articles: [Article]

    var body: some View {
        NavigationView {
            List(articles.identified(by: \.id)) { article in
                VStack(alignment: .leading) {
                    Text(article.title).font(.headline)
                    Text(article.preview).font(.subheadline)
                }
            }.navigationBarTitle(Text("Articles"))
        }
    }
}
```
---
title: Four Metrics Every Mobile Developer should care about
date: 2017-03-08 19:02:00 -0300
categories: [Metrics]
tags: [mobile, metrics]     # TAG names should always be lowercase
---

Slow apps frustrate users, which leads to bad reviews, or customers that swipe left to competition.
Unfortunately, seeing and solving performance issues can be a struggle and time-consuming.

Most developers use profilers within IDEs like Android Studio or Xcode to hunt for bottlenecks and automated performance tests to catch performance regressions in their code during development. However, testing an app before it ships is not enough.

To chat che most frustrating performance issues, you need to explore what's happening on your users's devices. That means visibility into how fast your app starts, duration of HTTP requests, number of slow and frozen frames, how fast your views are loading, and more. With Sentry for Mobile, you can now easily monitor your ReactNative, iOS, and Android app's performance in real-time without additional setup (or accumulating a pile of testing devices).

## Mobile Vitals

We believe there are four metrics every mobile team should track to better understand how their app is performing: Cold starts, warm starts, slow frames, and frozen frames. These four metrics, as a core part of Sentry's performance monitoring, give you the details you need to not only priority critical performance issues but trace the issue down to the root cause to solve them faster.

## Measuring App Start

When a user taps on your app icon, it should start fast. On iOS, Apple recommends your app take at most 400ms to render the first frame. On Android, the Google Play console warns you when a cold start takes longer than 5 seconds or a warm start longer than 2 seconds.

- Cold start: App launched from the first time, after a reboot or update.
- Warm start: App launched at least once and is partially in memory.

The exact definitions differ slightly per platform.

No matter the platform, it is critical that your app starts quickly to provide a delightful user experience. That's why on iOS, Mac Catalyst, tvOS, and Android we track how long your app needs to draw your first frame. We grab this information and add spans for different phases of the app start. 

On Android, it is trickier to hook into the initialization phases of the app start, and therefore we currently add one span from the application launch to the first auto-generated UI transaction. Still this information is very useful and can help you to improve the duration of your app start.

## Slow and Frozen Frames

Unresponsive UI, animation hitches or just jank annoy users and degrade the user experience. Two measurements to track this unwanted experience are slow and frozen frames. A phone or tablet typically renders with 60 frames per second (fps).

The frame rate can also be higher, especially as 120 fps displays are becoming more popular. With 60 fps, every frame has 16.67 ms to render. If your app needs more than 16.67 ms for a frame, it is a slow frame.

Frozen frames are UI frames that take longer than 700 ms. An app that is running smoothly should not experience either. That's why the SDK adds these two measurements to all transactions captured. The detail view of the transaction displays the slow, frozen, and total frames to the right.

Mobile vitals are a core part of Sentry's performance monitoring for mobile and unlocks more ways to spot bottlenecks and speed up your apps.
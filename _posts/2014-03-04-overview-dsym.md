---
title: An overview of dSYM
date: 2014-03-04 18:02:00 -0300
categories: [Apple]
tags: [dsym, ios, debugging]     # TAG names should always be lowercase
---

_From Apple's Documentation:_

https://developer.apple.com/documentation/xcode/building-your-app-to-include-debugging-information


> When Xcode compiles our source code into machine code, it generates a list of **symbols** in our app -  class names, global variables, and methods and function names. **These symbols correspond to the file and line numbers where they're defined; this association creates a debug symbol**, so you can use the debugger in Xcode, or refer to line numbers reported by a crash report. Debug builds of an app place the debug symbols inside the compiled binary file by default, while release builds of an app place the debug symbols in a companion Debug Symbol file (dSYM) to reduce the size of the distributed app.


Let us try to understand the above complex statement in a detailed and simple manner.

When an app crashes, the operating system collects diagnostic information about what the app was doing at the time of crash. One of the most important parts of the crash report are the thread backtraces, which are reported as hexadecimal addresses.

## What is SYMBOLICATION

Now to understand the root cause of our app crashes, firs we need to translate Thread backtraces into human readable language (i.e. function names and line number in our source code that caused the crash). This process is called symbolication.

## Role of dSYM (Debug SYMbol)

Without dSYM the crash report will just show memory addresses of objects and methods. We won't be able to read any crash reports without first re-symbolicating binary files.

dSYM files store the debug symbols for our app. It contains mapping information to decode a stack-trace into readable format.

The purpose of dSYM is to replace symbols in the crash logs with the specific methods names so it will be readable and helpful for debugging the crash.

## Key points regarding dSYM

The dSYM file is created when the application source code is compiled with the **Debugging Information Format** set to `DWARF with dSYM file`

dSYM fils change every time we compile our app having code changes.

The dSYM file contains UUID that links it to a specific Xcode build. The consequence is that symbolication only works if the UUID of the binary that caused a crash matches the UUID of the dSYM that is used for symbolication.

The benefit of using the dSYM is that we don't need to ship our App with its symbols which makes it **difficult to reverse engineer** our app. This also **reduce our App binary size**.

## How to enable dSYM?

1. In Xcode, select your project in the Project Navigator.
2. In the target list, select the target that builds your application.
3. Select the Build Settings tab.
4. In the Build Options section, make sure that the Debugging Information Format is set to `DWARF with dSYM File`.

_Note: It is recommended to set Debug to "DWARF" instead of "DWARF with dSYM File" to reduce the compile time._

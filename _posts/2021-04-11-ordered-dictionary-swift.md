---
title: OrderedDictionary in Swift
date: 2021-04-11 18:02:00 -0300
categories: [Swift]
tags: [swift, ordered dictionary]     # TAG names should always be lowercase
---

Many iOS and macOS applications use `Dictionary` for internal logic. A new Swift package called swift-collections introduces `OrderedDictionary`, a dictionary which keeps track of insertion order. This post presents an overview of `OrderedDictionary` and usage examples:

To use `OrderedDictionary`, first add the `swift-collection` Swift package to your project. Then, import the `OrderedCollections` module:

```swift
import OrderedCollections
```

## OrderedDictionary Examples

### Insert Value for Key

```swift
var oderedDic: OrderedDictionary = [
    "key0": 0,
    "key1": 1
]

orderedDict["key2"] = 2

// orderedDic now contains, in order,
// "key0": 0, "key1": 1, "key2": 2
```

### Get Value for Key

Like a traditional `Dictionary`, `OrderedDictionary` is a key-value store and can retrieve values using a specific key:

```swift
var orderedDict: OrderedDictionary = [
    "key0": 0,
    "key1": 1
]

// Return the value 1
orderedDict["key1"]
```

Additionally, `OrderedDictionary` has an internal order and can retrieve values at a specific position in the order using the elements property:

```swift
// Returns the element at index 0
var element = orderedDict.elements[0]
element.key // "key0"
element.value // 0
```

### Remove Key and Value

There are multiple ways to remove keys and values from an `OrderedDictionary`. One way is to remove a key and value explicitly, either specifying the key directly or the index the key and value are at:

```swift
// Remove a specific key
orderedDict.removeValue(forKey: "key1")

// Remove a key and value at a specific index
orderedDict.remove(at: 2)

```

Another method is to remove keys and values relative to the front and back of the `OrderedDictionary`:

```swift
// Remove keys and values from the front
orderedDict.removeFirst()
orderedDict.removeFirst(2)

// Remove keys and values from the back
orderedDict.removeLast()
orderedDict.removeLast(2)
```

`OrderedDictionary` also includes methods for removing all keys and values, and removing all keys and values that meet some filter criteria:

```swift
// Remove all keys and values
orderedDict.removeAll()

// Filter keys and values
orderedDict.removeAll{ (key, value) -> Bool in
    // Filter criteria
}
```

## Dictionary vs OrderedDictionary

`Dictionary` is an unordered collection of keys and associated values, often used as a key-value store. Like `Dictionary`, `OrderedDictionary` contains keys and associated values and can be used as a key-value store. Unlike `Dictionary`:

### OrderedDictionary Maintains Insertion Order

As the name indicates, `OrderedDictionary` introduces an `elements` property. The `elements` property is an `Array` value, and can be used to iterate over or retrieve keys and values at a specific position in the order.
This means an `OrderedDictionary` can efficiently retrieve a value for a specific key, like a traditional `Dictionary`, and also retrieve a key and value at a specific position, similar to a traditional `Array`.

An important note, reassigning keys to different values does not change the order:

```swift
var orderdDict: OrderedDictionary = [
    "key0": 0,
    "key1": 1,
    "key2": 2
]

// orderedDict contains, in order,
// "key0": 0, "key1": 1, "key2": 2

orderedDict["key1"] = 100

// OrderedDict contains, in order,
// "key0": 0, "key1": 100, "key2": 2
```

## When To Use OrderedDictionary

### Ordered Counters

A counter is often used to determine the number of occurrences unique elements have in a sequence. An ordered counter allows the occurrences of unique elements to be counted, while also preserving the first-seen order:

```swift
var sequence = [
    "a", "b", "a", "c", "b", "b", "b", "b", "a"
]

var orderedCounter: OrderedDictionary<String, Int> = [:]

for item in sequence {
    orderedCounter[item, default: 0] += 1
}

// orderedCounter now contains, in order,
// "a": 3, "b": 4, "c": 1

// Accessing the key "b" returns 4, the number of 
// times "b" occurs in the sequence
orderedCounter["b"]

// Accessing the position 0 returns the element
// "a": 3, introducing "a" occurred first in the sequence
// and occurred a total of 3 times
var element = orderedCounter.element[1]
element.key // "a"
element.value // 3
```

### Random Access To Unique, Ordered Elements

When working with unique sequences, like time-series, it is often useful to access the elements of the unique sequence in order and using a unique identifier.

`OrderedDictionary` provides a type that can do both:

```swift
var timeSeries = [
    ["id", "t0", "value": "0.1"],
    ["id", "t1", "value": "1.1"],
    ["id", "t2", "value": "2.1"],
]

var series = OrderedDictionary<String, Dictionary<String,String>> = [:]

for datapoint in timeSeries {
    series[datapoint["id"]!] = datapoint
}

// Accessing the key "t1" returns the datapoint
// associated with the id "t1"
series["t1"] // ["id": "t1", value: "1.1"]

// Accessing the element at index 2 returns
// the element associated with index 2 in the
// time series
var element = series.elements[2]
element.key // "t2"
element.value // ["id": "t2", "value": "2.1"]

```


That' it!

By using the `swift-collection` package and `OrderedDictionary`, you can take advantage of both `Dictionary` key-value store properties and element ordering in Swift.
---
description: AtCollections hold lists of CItems
---

# Introduction

## What are AtCollections?

AtCollections is a typed, shareable, reactive layer on top of AtRecords.

An `AtCollection<T>` hold a list of `CItem<T>`, where `T` is a type that you define. A `CItem<T>` is a record that wraps your own domain object (a `Todo` class, a `Pet` class, a Dart `Map`, or a plain `String`).&#x20;

Instead of working with raw `AtKey`/`AtValue` pairs, the SDK handles key shapes, per-recipient fan-out, encryption, sync, and event plumbing underneath.

Each collection is scoped to a namespace (like `todos.my_app`) and a type. Items in a collection are owned by the author Atsign that created them and can be shared with other Atsigns.

## Two ways to build a collection

There are two paths depending on what you want to store. Go to these next pages to continue your journey in AtCollections.

| **Collection item type**                                                                           | **When to use**                                          |
| -------------------------------------------------------------------------------------------------- | -------------------------------------------------------- |
| [create-atcollection-from-domain-object.md](create-atcollection-from-domain-object.md "mention")   | You have your own class (`Todo`, `Pet`, `BlogPost`)      |
| [create-atcollection-from-dart-primitive.md](create-atcollection-from-dart-primitive.md "mention") | You just want to store a `Map`, `String`, or `Uint8List` |

There are a bit of tiny things that differ between the two paths, but by the end of each path, both  produce the same `AtCollection<T>` object which will be used to manage your collection of data. Everything after the initial setup (CRUD, sub-collections, read receipts, event streams) works the same way regardless of which path you took.

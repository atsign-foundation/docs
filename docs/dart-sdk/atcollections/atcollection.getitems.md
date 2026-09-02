---
description: >-
  AtCollection.getItems returns a list of CItems, which is helpful to know what
  items exist
---

# AtCollection.getItems

`getItems` fetches every item in the collection as a list.

`getItemsAsStream` is the underlying stream it is built on -- everything else (including [atcollection.get.md](atcollection.get.md "mention") and [atcollection.query.md](atcollection.query.md "mention")) is a thin wrapper over it.

## `AtCollection.getItems`

### Basic usage

Basic usage is to get all items under the collection's namespace.

```dart
import 'package:at_client/at_client.dart';

// assume `todos` is of type AtCollection<Todo>

final List<CItem<Todo>> all = await todos.getItems();
```

{% hint style="info" %}
Items with the same `(owner, id)` across self and recipient copies are deduplicated, and their `sharedWith` sets are unioned automatically. This means something like the `CItem` copy for yourself and the `CItem` copy shared with another Atsign will only be displayed once.
{% endhint %}

### `getItems` under an `owner`

Fetch all items that belong to an Atsign.

```dart
final List<CItem<Todo>> mine = await todos.getItems(owner: '@alice'.toAtsign());
```

For anything beyond simple id/owner filtering, see **AtCollection.query**.

### `getItems` under an `id`

Fetch all items under a certain id. This could mean collection items with this `id` under the `todos` namespace from `@bob`, `@charlie` as well.

```dart
final List<CItem<Todo>> oneId = await todos.getItems(id: 'daily-standup');
```

For more complex queries, see [atcollection.query.md](atcollection.query.md "mention").

## `AtCollection.getItemsAsStream`

`getItems` is actually just `getItemsAsStream().toList()`. There are some powerful and optimal things you can do with `getItemsAsStream`!

The code below is one strong example of `getItemsAsStream`, where we can `print` `CItem<Todo>` information as they come in.&#x20;

```dart
await for (final CItem<Todo> item in todos.getItemsAsStream()) {
  print('[${item.owner}] ${item.id}: ${item.obj.title}');
}
```

This second example shows you how to optimally filter your `CItem` on some criteria. In this example, we are filtering the `Todo` for `done==true` .&#x20;

```dart
final List<CItem<Todo>> done = await todos.getItemsAsStream()
  .where((item) => item.obj.done)
  .toList();
```

Although this could also be done with [atcollection.query.md](atcollection.query.md "mention"), we just wanted to show you the different ways you can use this powerful method.

## Error handling

{% hint style="info" %}
This section won't make sense if you are unfamiliar with [Dart streams](https://dart.dev/libraries/async/using-streams).
{% endhint %}

Sometimes when you call `getItems` or `getItemsAsStream`, there will be a JSON decode error. Maybe something went wrong with the record data (JSON corrupted) or the record format changed and JSON decoding failed. Whichever the scenario is, the question remains: should `getItems/getItemsAsStream` stop and give you all the good items and skip the bad ones _or_ should it throw an error when it immediately finds a bad one? Our answer to this question is **both.** Below, we have some pointers on how to handle them gracefully depending on your use case.

### Stop on first bad record

When using `await getItems()` **or** `await for` with no `onError` supplied, this will stop the action and throw an error on the first bad record. This is good for when you expect no bad collection items.

```dart
// Stop on the first bad collection item (default)
final List<CItem> all = await todos.getItems();
```

### Acknowledge bad collection items and keep going

When using `.listen(onData, onError: ...)` to consume your stream, `onError` will fire for every bad collection item and all of the good collection items flow to the final `List<>` . This is good for when you want to get as many items as you can, while you define how bad items are handled.

```dart
// Log bad records, keep going
todos.getItemsAsStream().listen(
    (CItem item) => render(item),
    onError: (e) => log.warning('skipping bad item: $e'),
);
```

### Silently skip bad records

When using `.handleError` to consume the stream, you silently skip bad collection items. This is good for when you don't care about bad records and want to get as much data as possible.

```dart
// Silently skip bad records 
final List<CItem> tolerant = await todos.getItemsAsStream().handleError((e) { 
    /* swallow */ 
}).toList();
```

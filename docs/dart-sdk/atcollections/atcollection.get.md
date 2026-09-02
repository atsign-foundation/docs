---
description: AtCollection.get is for getting the value of a CItem
---

# AtCollection.get

Use `get` and `getOrNull` to fetch one item by its `owner` and `id` pair.

## `AtCollection.get`

Use `AtCollection.get` to get the value of a collection item. You will need to supply the `id` and `owner`.

The code sample below will get the collection item belonging to the `todos` collection namespace, with the id `daily-standup` and is authored by the Atsign `todos.self`, which could be a value like `@alice`. If it is not found, it will throw `AtKeyNotFoundException`.

```dart
import 'package:at_client/at_client.dart';

// assume todo is of type AtCollection<Todo>

final CItem<Todo> item = await todos.get('daily-standup', todos.self);

// throws AtKeyNotFoundException if DNE
```

## `AtCollection.getOrNull`

If you do not want to deal with `AtKeyNotFoundException` and work with nullables in Dart instead, you can use `getOrNull` which will return a null type if it does not exist.

```dart
final CItem<Todo>? item = await todos.getOrNull('daily-standup', todos.self);
if (item == null) {
  // no such item
}
```

## Get data from others

Reading data that is shared with you works the same way. Instead, pass their Atsign as `owner`:

```dart
final CItem<Todo>? fromBob =
    await todos.getOrNull('daily-standup', '@bob'.toAtsign());
```

## Debug-printing an item

```dart
print(item.prettyString);
```

A multi-line dump (`owner`, `id`, `sharedWith`, `expiresAt`, `availableAt`, `type`, and either the decoded `obj` or a byte length for binary items).

## Errors

| Exception                | Cause                                  |
| ------------------------ | -------------------------------------- |
| `AtKeyNotFoundException` | `get`: no item with this `(id, owner)` |

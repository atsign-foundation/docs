---
description: AtCollection.create is for creating new items, if they don't already exist
---

# AtCollection.create

`AtCollection.create` returns a `CItem` . It will create a record in your atServer, and if one already exists, will throw an error. You can create CItems and either [store data for yourself](atcollection.create.md#storing-data-for-yourself) or [share data with others](atcollection.create.md#sharing-data-with-others).

## Storing data for yourself

Writing with no `sharedWith` keeps the item private to you, the same way a self key does at the [atrecords.md](../../atsign-platform/atserver/atrecords.md "mention") layer. Leaving `sharedWith` empty means you are sharing it with nobody, and it's intended for self-keeping.

If you omit `id`, the SDK generates a random 8-character one for you. It may throw a `StateError` if a collision occurs (after up to 10 retries), which is astronomically low.

```dart
import 'package:at_client/at_client.dart';

final Todo newTodo = Todo('do task #1');

final CItem<Todo> item = await todos.create(obj: newTodo);
```

Otherwise, you may supply your own `id`.

```dart
final Todo newTodo = Todo('do task #2');
final String id = 'daily-standup';

final CItem<Todo> citem = await todos.create(obj: newTodo, id: id);
```

## Sharing data with others

Pass `sharedWith` at creation time and the library writes and encrypts one recipient copy per Atsign, in addition to your own self copy.

```dart
import 'package:at_commons/at_commons.dart';

final Todo newTodo = Todo('do task #3');
final String id = 'very-important-todo';
final Set<Atsign> sharedWithAtsigns = {
  '@bob'.toAtsign(),
  '@carol'.toAtsign(),
};

final CItem<Todo> item = await todos.create(
  obj: newTodo,
  id: id,
  sharedWith: sharedWithAtsigns,
);
```

See [atcollection.update.md](atcollection.update.md "mention") on how to update who it is shared with without replacing the item.

## `AtCollection.create` parameters

### `obj`: the data itself

This is a required field. This will be the data that you write to the collection item itself. If your collection is a `Todo` type, then you will need to pass a `Todo` type to the `obj` parameter.

```dart
final Todo newTodo = Todo('do task #2');
final CItem<Todo> citem = await todos.create(obj: todo);
```

### `id` : a name to your `CItem`

Not supplying an `id` will generate a 8-character id for you. In the event fo a collision, it will retry up to 10 times, and otherwise throw a `StateError` (this is astronomically unlikely).

Supplying your own `id` may look like:

```dart
final Todo newTodo = Todo('do task #2');
final String id = 'daily-standup';

final CItem<Todo> citem = await todos.create(obj: todo, id: id);
```

### `availableAt`: delayed visibility

This will make the collection item appear at a later time to observers of your atServer. Your application does not need to be active. The data gets pushed to the atServer, but the atServer will make it available at a later time.

```dart
final CItem<Todo> item = await todos.create(
  obj: Todo('surprise release'),
  availableAt: DateTime.now().add(const Duration(hours: 1)),
);
```

### `expiresAt`: lifecycle

Defaults to `now + defaultExpiration` (the value you passed to `atClient.collection<T>()`). Mutate `item.expiresAt` and call `update()` to change an existing item's lifecycle. Push `item.expiresAt` forward before rewriting a record you left sitting for longer than its lifetime. See [atcollection.update.md](atcollection.update.md "mention") for this.

```dart
final CItem<Todo> item = await todos.create(
  obj: Todo('one-off reminder'),
  expiresAt: DateTime.now().add(const Duration(days: 1)),
);
```

## Errors

| Exception               | Cause                                                                                         |
| ----------------------- | --------------------------------------------------------------------------------------------- |
| `ArgumentError`         | An `id` containing a `.`; `item.owner` is not you; or `item.expiresAt` is already in the past |
| `StateError`            | `create` with a colliding id                                                                  |
| `CollectionOpException` | A key-level `put` failed. Inspect `.failures` / `.firstFailure` for the per-key breakdown     |

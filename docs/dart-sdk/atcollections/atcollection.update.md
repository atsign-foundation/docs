---
description: AtCollection.update is for updating existing CItems.
---

# AtCollection.update

`AtCollection.update` is for updating existing `CItems`. This is not the correct operation if it does not exist yet. For that, see [atcollection.create.md](atcollection.create.md "mention") and [atcollection.upsert.md](atcollection.upsert.md "mention").

Every `CItem` remembers who it's shared with.&#x20;

* `update` can rewrite the value and recipient copies in one call
* `updateSharedWith` touches only the recipient set without rewriting the self copy.

## `AtCollection.update(item)`

The code below demonstrates basic usage. Here, we assume `item` is an existing in-memory `CItem`, whether from an [atcollection.draft.md](atcollection.draft.md "mention"), [atcollection.getitems.md](atcollection.getitems.md "mention") or from somewhere else.

```dart
import 'package:at_client/at_client.dart';

// assume `item` is an existent `CItem`

item.obj = Todo('updated title'); // 1. overwrite the value
await todos.update(item); // 2. update the existing record
```

`update` persists `item.obj` to the self copy and overwrites every recipient copy.  It rewrites the value and the recipient set. Every recipient gets a fresh copy pushed to them, even if the value didn't change for them.

It throws `StateError` if the item's self-key does not exist yet. Use [atcollection.create.md](atcollection.create.md "mention") for genuinely new items.

## `AtCollection.update(item, unshareWithOthers: false)`

Propose this scenario: you are `@alice` and you have shared a `CItem<Todo>` with `@bob` and `@charlie`. Now you want to share it with `@dave` but you want to leave `@bob`'s and `@charlie`'s items unaffected (because otherwise, their atServers would be pinged and notified of a change, even though the value might just be the same).

To solve this, pass `unshareWithOthers: false` to the `update` function which leave existing recipients' copies alone. This is useful when you're adding someone and don't want others to deceive others of a data change.

```dart
// assume `item` is of type `CItem<Todo>` and is an existent collection item in the atServer
// 1. modify the existing CItem 
item.sharedWith
  ..clear()
  ..addAll({'@dave'.toAtsign()}); // add @dave

// 2. push an update
await todos.update(item, unshareWithOthers: false);
```

## `AtCollection.updateSharedWith(item, newSet)`

When the value itself hasn't changed, `updateSharedWith` is cheaper and has different observable behavior: it does **not** touch the self-key, bump the item's commit-id, or emit a local `CItemUpdated` on this collection's event streams, because from this Atsign's perspective the item's own state hasn't changed:

```dart
await todos.updateSharedWith(item, 
    {'@bob'.toAtsign(), '@charlie'.toAtsign(), '@dave'.toAtsign()});
```

Additive-only mode leaves anyone already sharing this item alone:

```dart
await todos.updateSharedWith(
  item,
  {'@dave'.toAtsign()},
  unshareWithOthers: false,
);
```

## Which one to use

| Situation                                          | Use                                           |
| -------------------------------------------------- | --------------------------------------------- |
| Value changed and recipients may have changed too  | `update()`                                    |
| Adding recipients, leaving existing ones untouched | either method with `unshareWithOthers: false` |
| Only the recipient set changed                     | `updateSharedWith()`                          |

## Errors

| Exception               | Cause                                                                         |
| ----------------------- | ----------------------------------------------------------------------------- |
| `ArgumentError`         | `item.owner` is not you or when adding a recipient to an already-expired item |
| `StateError`            | The item's self-key doesn't exist yet: `create()` it first                    |
| `CollectionOpException` | A key-level put/delete failed. Inspect `.failures`                            |

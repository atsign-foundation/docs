---
description: Use AtCollection.watch
---

# AtCollection.watch

Every `AtCollection` exposes event streams you can subscribe to. Events tell you when items are created, updated, deleted, or become available, without polling.

## Event streams

```dart
import 'package:at_client/at_client.dart';

collection.watch()          // Stream<CEvent> -- everything below
collection.updates          // Stream<CItemUpdated>
collection.deletes          // Stream<CItemDeleted>
collection.readReceipts     // Stream<CReadReceipt>
collection.subUpdates       // Stream<CSubItemUpdated>
collection.subDeletes       // Stream<CSubItemDeleted>
collection.availableEvents  // Stream<CItemAvailable>
```

`AtCollection.watch()` contains all events as one `Stream<CEvent>` (with the exception of `CItemExpiringSoon`; see [#atcollection.expiringsoonevents](atcollection.watch.md#atcollection.expiringsoonevents "mention")). Use conditionals to check which concrete type `CEvent` may be (e.g. `if (event is CItemUpdated)`).

## `EventSource`

Every `AtCollection` picks one of three event sources at construction (defaults to `EventSource.both`):

```dart
final AtCollection<Todo> todos = await atClient.collection<Todo>(
  'todos.my_app',
  const Duration(days: 7),
  fromJson: Todo.fromJson,
  typeTag: 'Todo',
  eventSource: EventSource.data,
);
```

`data` is the recommended choice whenever a real `SyncService` is running. It is the only source that sees writes your own process just made, since a locally-driven write never generates a notification to yourself. Sub-collections built via `subCollection`/`readReceiptsFor` inherit the parent's choice; you cannot mix sources within one collection tree.

## `AtCollection.watch`

`CEvent` is a non-sealed Dart class. New subtypes can arrive in future minor releases, so an exhaustive `switch` is a forward-compatibility risk, so be sure to always include a `default:` branch.

Example usage of `AtCollection.watch`:

```dart
collection.watch().listen((event) {
  switch (event) {
    case CItemUpdated():      onUpdate(event); break;
    case CItemDeleted():      onDelete(event); break;
    case CReadReceipt():      onReceipt(event); break;
    case CItemAvailable():    onAvailable(event); break;
    default:                  break; // unknown / future event
  }
});
```

## `AtCollection.*` event streams

### `AtCollection.updates`

Listen for updates. Event is triggered when a `CItem` is created or updated that concerns the owner Atsign of the collection (`CItems` shared with `collection.owner newly` created or updated).

```dart
collection.updates.listen((CItemUpdated e) {
  print('Item ${e.id} owned by ${e.owner} was created or updated');
});
```

### `AtCollection.deletes`

Listen for when `CItems` are deleted. Events are triggered when a `CItem` in the collection is deleted.

&#x20;`wasExpired` is `true` when the delete was driven by TTL expiry rather than an explicit delete call.

```dart
collection.deletes.listen((CItemDeleted e) {
  if (e.wasExpired) {
    print('Item ${e.id} expired');
  } else {
    print('Item ${e.id} was deleted');
  }
});
```

### `AtCollection.readReceipts`

Event is triggered when another Atsign posts a read receipt for an item the caller owns. Read-receipt events on your own writes are not fired.

```dart
collection.readReceipts.listen((CReadReceipt e) {
  print('${e.from} read item ${e.id} at ${e.readAt}');
});
```

### `AtCollection.subUpdates`

Read more about this in [#listening-for-sub-collection-events](atcollection.subcollection.md#listening-for-sub-collection-events "mention")

### `AtCollection.subDeletes`

Read more about this in [#listening-for-sub-collection-events](atcollection.subcollection.md#listening-for-sub-collection-events "mention")

### `AtCollection.availableEvents`

An event is passed to this stream when a scheduled item's `availableAt` time passes.

```dart
collection.availableEvents.listen((CItemAvailable e) {
  print('${e.id} is now available (scheduled for ${e.availableAt})');
});
```

### `AtCollection.expiringSoonEvents`

An event is passed to this stream before `leadTime` on an item's `expiresAt`. Useful for reminder or alarm UIs that need to nudge the user before a `CItem` disappears.

```dart
// print to the user that `e` is expiring in 5 minutes!
collection.expiringSoonEvents(leadTime: const Duration(minutes: 5))
    .listen((CItemExpiringSoon e) {
  print('${e.id} expires at ${e.expiresAt} (in 5 minutes)');
});
```

Items whose `expiresAt - leadTime` is already in the past at subscription time fire on the next event-loop turn, so a late subscriber does not silently miss them.

`CItemExpiringSoon` events do not flow through `watch()`. Each `expiringSoonEvents(leadTime:)` call creates its own independent stream with its own scheduler, so different subscribers can use different lead times without interfering with each other.

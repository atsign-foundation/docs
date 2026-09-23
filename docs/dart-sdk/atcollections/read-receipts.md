---
description: >-
  Read receipts are useful for when you share data and want to be notified for
  when it is read.
---

# Read receipts

Every item has a built-in read-receipt sub-collection using the reserved `__rr` sub-name. The reader marks an item as read, and the owner can check who has read it.

## Marking a `CItem` as read

You can only mark a `CItem` as read when it's a `CItem` shared with you. You cannot receipt your own items (no-operation happens when you try).

```dart
// assume `incomingItem` is a `CItem` that came from somewhere like from `AtCollection.updates`
await incomingItem.markReadByMe();
```

## Checking who has read an item

```dart
final Set<Atsign> readers = await item.readBy;
```

If you own the `CItem`, you see who read it.&#x20;

If someone else owns the `CItem`, you see your own receipt (confirming you read it).

`readBy`'s first access loads the item's `__rr` sub-collection once, then stays current by listening to this collection's `readReceipts` stream. Later calls are O(1) and do not re-fetch.

## Checking if you already read an item

```dart
// Have I already sent a read receipt for this incoming item?
final bool alreadyRead = await todos.wasMarkedReadByMe(incomingItem);
```

Returns `true` for self-owned items without any I/O (the owner is trivially "caught up" on their own record).

## Listen for read receipts

Event is triggered when a `CItem` that you have shared with another Atsign is read by them.

```dart
todos.readReceipts.listen((CReadReceipt e) {
  print('${e.from} read item ${e.id} at ${e.readAt}');
});
```

`CReadReceipt.owner` and `CReadReceipt.id` identify the parent item being read (not the receipt sub-item itself). `CReadReceipt.from` is the reader. `CReadReceipt.readAt` is the moment the notification was received (not the moment the reader wrote it).

## `AtCollection.readReceiptsFor`

`AtCollection.readReceiptsFor` gives you the read receipts for a particular item (see which Atsigns have read it already).

```dart
final AtCollection<Map<String, dynamic>> receipts =
    todos.readReceiptsFor(item);
```

Example below answers "how many people have read my item?"

```dart
final Stream<int> readerCount = todos.readReceiptsFor(item)
    .query()
    .watch()
    .map((list) => list.length);
```

Or a one-shot check:

```dart
final int count = await todos.readReceiptsFor(item).query().count();
```

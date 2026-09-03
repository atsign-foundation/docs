---
description: AtCollection.upsert is for creating new items or updating existing items
---

# AtCollection.upsert

`upsert` is the idempotent write operation: it creates the item if the id does not exist yet, or overwrites it in place if it does. The operation will execute successfully whether a collection item with the same id existed previously or not.

## When to use `upsert` over `create`

**Use `upsert` whenever re-runnability matters more than catching an accidental id collision.** Reserve `create` for the strict cases; for example when an `id` collision would indicate a bug.

{% hint style="info" %}
Unlike `create`, which can auto-generate a random id, `upsert` always requires an explicit `id`. The method needs a stable key to decide "create" or "overwrite."
{% endhint %}

## `AtCollection.upsert` for self

The code usage is very similar to `create`. Instead of throwing an error if the collection item exists already, it will simply overwrite it.&#x20;

The code sample below shows you how to store data for yourself.

```dart
import 'package:at_client/at_client.dart';

// assume `stats` is an AtCollection<SensorReading>

await stats.upsert(
  id: 'latest-reading',
  obj: SensorReading(temperature: 22.5),
);
```

## `AtCollection.upsert` for others

In `AtCollection.upsert`, you pass `sharedWith` the same way you would on `AtCollection.create`.&#x20;

The code sample below shows you how to pass a `sharedWith` value of type `Set<Atsign>`.&#x20;

```dart
await stats.upsert(
  id: s.timestamp.toString(),
  obj: s,
  sharedWith: {'@bob'.toAtsign(), '@carol'.toAtsign()},
);
```

## Parameters

| Parameter     | Type           | Default                       |
| ------------- | -------------- | ----------------------------- |
| `id`          | `String`       | **Required**                  |
| `obj`         | `T`            | **Required**                  |
| `sharedWith`  | `Set<Atsign>?` | `null` (private to self)      |
| `expiresAt`   | `DateTime?`    | `now + defaultExpiration`     |
| `availableAt` | `DateTime?`    | `null` -- visible immediately |

## Errors

| Exception               | Cause                                                                                     |
| ----------------------- | ----------------------------------------------------------------------------------------- |
| `ArgumentError`         | `id` contains a `.`, or `expiresAt` is already in the past                                |
| `CollectionOpException` | A key-level `put` failed. Inspect `.failures` / `.firstFailure` for the per-key breakdown |

`upsert` never throws `StateError` for a colliding id -- that is the whole point.

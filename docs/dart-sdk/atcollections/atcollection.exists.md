---
description: AtCollection.exists will check if a collection item exists cheaply
---

# AtCollection.exists

## `AtCollection.exists`

Just like [atcollection.get.md](atcollection.get.md "mention"), the mandatory fields to use `AtCollection.exists` are both `id` and `owner`.

```dart
import 'package:at_client/at_client.dart';

final bool there = await todos.exists('daily-standup', todos.self);
```

## Checking items shared with you

The code sample below shows how to check if someone like `@bob` shared data with you under the `daily-standup` id and the corresponding AtCollection namespace.

```dart
final bool bob =
    await todos.exists('daily-standup', '@bob'.toAtsign());
```

For items owned by someone else, `exists` probes your local `cached:` copy. So `exists` and `getOrNull(id, owner) != null` always agree, including while offline.

## How is this cheaper than `AtCollection.getOrNull`

For your own items, `exists` short-circuits against an in-process cache of ids you've already written or read. This skips json decodes and round trips. For items owned by others it still avoids the decode step and checks whether the key is present, not whether the value parses.

Use `getOrNull` when you need the item itself. Use `exists` when you only need to know if it's there or not.

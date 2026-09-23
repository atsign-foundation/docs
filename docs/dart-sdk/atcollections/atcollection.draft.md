---
description: >-
  AtCollection.draft is used to build a CItem first and subsequent various
  operations with it afterwards
---

# AtCollection.draft

`draft` is a preview tool: it lets you see what `id`, `expiresAt`, and `sharedWith` the SDK would assign before you commit anything. It does not feed directly into `create` or `upsert` (those take raw parameters, not a `CItem`), but it is possible to feed into [atcollection.update.md](atcollection.update.md "mention"), which takes a `CItem<T>` directly.

## `AtCollection.draft`

Code below samples basic usage. First we create a draft, and it is stored in `CItem<Todo>`. Then, we can print various things like `item.id` or `item.expiresAt`. Useful for inspecting and debugging before pushing.

```dart
import 'package:at_client/at_client.dart';

final CItem<Todo> item = todos.draft(
  obj: Todo(title: 'write readme', description: 'Draft the docs'),
);

print(item.id);        // e.g. "a7k2m9x1" (random)
print(item.expiresAt); // now + defaultExpiration
```

## Parameters

| Parameter     | Default                                         |
| ------------- | ----------------------------------------------- |
| `obj`         | **Required**                                    |
| `id`          | Random 8-character `[a-z0-9]` string if omitted |
| `sharedWith`  | `{}`                                            |
| `expiresAt`   | `now + defaultExpiration`                       |
| `availableAt` | `null` -- visible immediately                   |

## Errors

| Exception       | Cause               |
| --------------- | ------------------- |
| `ArgumentError` | `id` contains a `.` |

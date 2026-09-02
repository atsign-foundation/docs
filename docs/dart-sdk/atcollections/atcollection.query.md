---
description: AtCollection.query is an easy way to filter and query collections.
---

# AtCollection.query

`Query<T>` is a composable, reusable builder on top of `getItemsAsStream`. Build one up with chained modifiers, store it, pass it around, and terminate with either `get()` (one-shot) or `watch()` (live reactive stream).

Execution always runs against the local synced store. Nothing here makes a network call. End-to-end encryption means the atServer cannot decrypt the records it holds on your behalf, so on-device is the only correct execution model.

## Building a query

Below is a simple example.

Here, we are querying for unfinished todos that are before today's date, sorted by due date, and at most 20 items.

```dart
import 'package:at_client/at_client.dart';

final Query<Todo> overdue = todos.query()
    .where((t) => !t.obj.done) // unfinished
    .where((t) => t.obj.due.isBefore(DateTime.now())) // late
    .orderBy((t) => t.obj.due) // sort by due date
    .limit(20); // at most 20 items

final List<CItem<Todo>> list = await overdue.get();
```

Queries are immutable values. Each chained call returns a new `Query<T>`. Multiple `.where(...)` calls are ANDed together.

## Methods

| Method                         | Effect                                                           |
| ------------------------------ | ---------------------------------------------------------------- |
| `where(predicate)`             | Ad-hoc closure predicate, ANDed with any others                  |
| `wherePath(predicate)`         | Typed, introspectable predicate -- see below                     |
| `orderBy(keyFn, {descending})` | Sets the sort; a second `orderBy` call **replaces** it           |
| `thenBy(keyFn, {descending})`  | Adds a tiebreaker. Throws `StateError` without a prior `orderBy` |
| `limit(n)`                     | Keeps at most `n`, after filter + sort + skip                    |
| `skip(n)`                      | Skips the first `n`, after filter + sort, before `limit`         |
| `get()`                        | One-shot `Future<List<CItem<T>>>`                                |
| `watch()`                      | Live `Stream<List<CItem<T>>>` -- see below                       |
| `count()`                      | `(await get()).length`, spelled explicitly                       |
| `any([predicate])`             | `Future<bool>` -- short-circuits, ignores sort/skip/limit        |
| `first()`                      | First matching item. Throws `StateError` if nothing matches      |
| `firstOrNull()`                | First matching item, or `null`                                   |
| `distinct(keyFn)`              | One-shot fetch, first match per `keyFn` kept                     |
| `groupBy(keyFn)`               | One-shot fetch grouped into `Map<K, List<CItem<T>>>`             |

`fetch()` is a deprecated alias for `get()`. It will be removed in the next minor release; migrate call sites now.

## Examples

One-liners for common tasks in our `Todo` example:

### Example 1

Find non-done `Todos`:

```dart
// assume todos is of type AtCollection<T>
final int total = await todos.query()
    .where((t) => !t.obj.done)
    .count();
```

Check if there are any high priority `Todos`

```dart
// assume todos is of type AtCollection<T>
final bool hasAny = await todos.query()
    .any((t) => t.obj.priority == Priority.high);
```

### Example 3

Get the `Todo` with the closest due date

```dart
final CItem<Todo>? next = await todos.query()
    .where((t) => !t.obj.done)
    .orderBy((t) => t.obj.due)
    .firstOrNull();
```

### Example 4

Group `Todos` by Atsign owner to a `Map<Atsign, List<CItem<Todo>>>`

```dart
final Map<Atsign, List<CItem<Todo>>> byOwner = await todos.query()
    .where((t) => !t.obj.done)
    .groupBy((t) => t.owner);
```

## `PathField`&#x20;

`PathField` is a powerful class provided by the AtCollection toolkit. It uses the same semantics as `where` , but great if you have reusable logic.

1. First, define some `static final PathField<T>` variables in your abstract class.

For example:

```dart
abstract class $Todo {
  static final PathField<bool> done = PathField<bool>(
    path: ['obj', 'done'],
    extract: (item) => (item.obj as Todo).done,
  );
  static final PathField<DateTime> due = PathField<DateTime>(
    path: ['obj', 'due'],
    extract: (item) => (item.obj as Todo).due,
  );
}
```

2. Then use them in queries:

For example:

```dart
final List<CItem<Todo>> overdue = await todos.query()
    .wherePath($Todo.done.eq(false))
    .wherePath($Todo.due.lt(DateTime.now()))
    .get();
```

Another example:

```dart
// Compose within one clause with `.and` / `.or` / `.not`:
final List<CItem<Todo>> urgent = await todos.query()
    .wherePath($Todo.done.eq(false).and($Todo.due.lt(soon)))
    .get();
```

Available comparisons on a `PathField<V>`: `eq`, `neq`, `lt`, `lte`, `gt`, `gte`, `isNull`, `isNotNull`.

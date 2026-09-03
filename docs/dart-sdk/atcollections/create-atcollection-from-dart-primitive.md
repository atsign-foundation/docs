---
description: Learn how to create an AtCollection object that holds Dart primitives
---

# Create AtCollection from Dart primitive

Use dart primitives when you want to create a collection that is based on Dart primitives.

This is Path B from the [introduction.md](introduction.md "mention"). Use this when you just want to store a `Map`, `String`, `Uint8List`, or other type that Dart's `jsonEncode` already handles.&#x20;

Unlike what is done in [create-atcollection-from-domain-object.md](create-atcollection-from-domain-object.md "mention"), you do not need `fromJson` or `typeTag`. The SDK knows how to serialize and deserialize these natively.

## Examples

### Maps

```dart
final AtCollection<Map> maps = await atClient.collection<Map>(
  'maps.my_app',
  const Duration(days: 7),
);
```

### Strings

```dart
final AtCollection<String> strings = await atClient.collection<String>(
  'strings.my_app',
  const Duration(days: 7),
);

await strings.create(
  obj: 'this is just a String',
  sharedWith: otherAtSigns,
);
```

### Lists

```dart
final AtCollection<List<String>> tags = await atClient.collection<List<String>>(
  'tags.my_app',
  const Duration(days: 7),
);
```

### Binary data (Uint8List)

```dart
import 'dart:typed_data';

final AtCollection<Uint8List> binaries = await atClient.collection<Uint8List>(
  'binary.my_app',
  const Duration(days: 7),
);
```

## Untyped (generic) collections

You can create a collection with no type parameter. This gives you an `AtCollection<dynamic>` that accepts any Dart primitive in the same collection. No `fromJson`, `typeTag`, or `registerFactory` call is needed - the SDK tags every primitive as `'n/a'` internally and round-trips them through `jsonDecode` automatically.

```dart
final AtCollection generic = await atClient.collection(
  'mixed.my_app',
  const Duration(days: 7),
);

await generic.create(
  obj: {'key': 'value', 'count': 42},
  sharedWith: otherAtSigns,
);
await generic.create(obj: 'just a string', sharedWith: otherAtSigns);
await generic.create(obj: ['dart', 'flutter'], sharedWith: otherAtSigns);
await generic.create(obj: 42, sharedWith: otherAtSigns);
await generic.create(obj: true, sharedWith: otherAtSigns);

for (final CItem item in await generic.getItems()) {
  print('${item.obj.runtimeType}: ${item.obj}');
}
```

You lose compile-time type safety on `item.obj` (it is `dynamic`), so cast when you read back:

```dart
for (final CItem item in await generic.getItems()) {
  if (item.obj is Map) {
    final Map map = item.obj as Map;
    print('map with ${map.length} entries');
  } else if (item.obj is String) {
    print('string: ${item.obj}');
  }
}
```

You are ready to move onto [atcollection.create.md](atcollection.create.md "mention").

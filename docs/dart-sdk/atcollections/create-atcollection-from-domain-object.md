---
description: Learn how to create an AtCollection from a domain object you define
---

# Create AtCollection from domain object

This is Path A from the **Introduction**. Use this when you have your own class (`Todo`, `Pet`, `BlogPost`) and want a typed collection around it.

Use domain objects when you have your own class that you want to store in a collection

## 1. Define the domain object

There is no abstract class or interface to implement for your domain object class. Your object just needs two things:

* A `toJson()` method that returns `Map<String, dynamic>`
* A `fromJson` factory that takes `Map<String, dynamic>` and returns your type

For example:

```dart
class Todo {
  String title;
  String description;
  bool done;
  DateTime? dueDate;

  Todo({
    required this.title,
    required this.description,
    this.done = false,
    this.dueDate,
  });

  factory Todo.fromJson(Map<String, dynamic> json) {
    return Todo(
      title: json['title'],
      description: json['description'],
      done: json['done'] ?? false,
      dueDate: json['dueDate'] != null
          ? DateTime.parse(json['dueDate'])
          : null,
    );
  }

  Map<String, dynamic> toJson() => {
    'title': title,
    'description': description,
    'done': done,
    if (dueDate != null) 'dueDate': dueDate!.toIso8601String(),
  };
}
```

## 2. Get the collection

Create the AtCollection object using `atClient.collection`. Get your AtClient instance by following: [create-an-atclient-instance.md](../authentication/create-an-atclient-instance.md "mention").

Pass `fromJson` and `typeTag` when creating the collection. This is a mandatory step for domain objects, but not for [create-atcollection-from-dart-primitive.md](create-atcollection-from-dart-primitive.md "mention").

```dart
import 'package:at_client/at_client.dart';

final AtCollection<Todo> todos = await atClient.collection<Todo>(
  'todos.my_app',
  const Duration(days: 7),
  fromJson: Todo.fromJson,
  typeTag: 'Todo',
);
```

`typeTag` is a string literal that gets written into every item's envelope on the wire. When items are rehydrated (read back from the atServer), the SDK uses the `typeTag` to find the right `fromJson` factory.

### Parameters reference

| Parameter                  | Meaning                                                                                                                                                                                                                                                 |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `namespace`                | <p>Fully qualified namepsace. must contain a <code>.</code>. </p><p></p><p>See <a data-mention href="../../atsign-platform/atserver/namespaces.md">namespaces.md</a> for more information.</p><p></p><p>Throws <code>ArgumentError</code> otherwise</p> |
| `defaultExpiration`        | The `expiresAt` applied to items that don't override it                                                                                                                                                                                                 |
| `eventSource`              | Which stream(s) feed events. Defaults to `EventSource.both` — see [events](../events/ "mention")                                                                                                                                                        |
| `cleanupOrphansOnCreation` | Cleans up oprhan collection items before the future completes. Defaults to `false` — see [atcollection.query.md](atcollection.query.md "mention")                                                                                                       |
| `fromJson`                 | Rehydrates `T` from the decoded JSON map. Only needed for custom domain objects. Only required for domain objects.                                                                                                                                      |
| `typeTag`                  | The wire-format identifier for `T`. Required whenever `fromJson` is supplied. Only required for domain objects.                                                                                                                                         |

## Polymorphic collections example

When taking advantage of polymorphism in Dart, there is one small caveat that drifts from the traditional path when creating an AtCollection from a domain object.

When working with an abstract supertype (like `Pet`) and several concrete types (like `Dog` and `Cat`), you will need to register each concrete type separately with `registerFactory` instead of passing `fromJson` to `atClient.collection<T>()`.

Code below shows our supertype `Pet` and two concrete types `Dog` and `Cat`. Both concrete types still need to define `fromJson` and `toJson`

```dart
abstract class Pet {
  final String name;
  Pet({required this.name});
  Map<String, dynamic> toJson();
}

class Dog extends Pet {
  Dog({required super.name});
  factory Dog.fromJson(Map<String, dynamic> json) =>
      Dog(name: json['name']);
  @override
  Map<String, dynamic> toJson() => {'name': name};
}

class Cat extends Pet {
  Cat({required super.name});
  factory Cat.fromJson(Map<String, dynamic> json) =>
      Cat(name: json['name']);
  @override
  Map<String, dynamic> toJson() => {'name': name};
}
```

Then, your code needs to run `AtCollection.registerFactory` for both classes. Notice that we did not specify `fromJson` and `typeTag`, unlike what is done in [#id-1.-define-the-domain-object](create-atcollection-from-domain-object.md#id-1.-define-the-domain-object "mention").

```dart
AtCollection.registerFactory<Dog>(Dog.fromJson, typeTag: 'Dog');
AtCollection.registerFactory<Cat>(Cat.fromJson, typeTag: 'Cat');

final AtCollection<Pet> pets = await atClient.collection<Pet>(
  'pets.my_app',
  const Duration(days: 365),
);
```

You are ready to move onto [Broken link](/broken/pages/BStwQ6cvCSwxv97wlUNT "mention")

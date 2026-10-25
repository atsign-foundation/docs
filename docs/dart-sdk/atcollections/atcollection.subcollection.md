---
description: >-
  AtCollection.subCollection is useful when your data structure has a
  parent-child relationship.
---

# AtCollection.subCollection

## What is a sub-collection?

A sub-collection is an `AtCollection` that belongs to a specific `CItem` in a parent collection. Think of it like a one-to-many relationship: a blog post has many comments, a comment has many replies.

```
posts (AtCollection<Post>)
  └── "hello-world" (CItem<Post>)
        └── comments (AtCollection<Comment>)     <-- sub-collection
              ├── "c1" (CItem<Comment>)
              │     └── replies (AtCollection<Reply>) <-- nested sub-collection
              │           └── "r1" (CItem<Reply>)
              └── "c2" (CItem<Comment>)
```

When instantiating your own sub-collection, you must instantiate each `AtCollection` separately, then they are linked together by namespace.

## `AtCollection.subCollection`

Any `CItem` can parent its own `AtCollection<T>`. This lets you model hierarchies like posts and comments, nested to arbitrary depth. Nesting is bounded only by the Atsign Protocol's 255-character key limit.

## Creating a sub-collection

1. First, create your domain objects. In this case, we'll create `Post` , `Comment`, and `Reply`.

```dart
class Post {
  final String title;
  final String body;
  Post(this.title, this.body);
  Map<String, dynamic> toJson() => {'title': title, 'body': body};
  factory Post.fromJson(Map<String, dynamic> json) =>
      Post(json['title'], json['body']);
}

class Comment {
  final String text;
  Comment(this.text);
  Map<String, dynamic> toJson() => {'text': text};
  factory Comment.fromJson(Map<String, dynamic> json) =>
      Comment(json['text']);
}
```

2. Instantiate your `AtCollection<T>` objects. See [create-atcollection-from-domain-object.md](create-atcollection-from-domain-object.md "mention") for a better explanation on how to do that.

```dart
// 1. Create the parent collection using `AtCollection.collection`
final AtCollection<Post> posts = await atClient.collection<Post>(
  'posts.my_app',
  const Duration(days: 30),
  fromJson: Post.fromJson,
  typeTag: 'Post',
);

// 2. Use `AtCollection.subCollection`
final AtCollection<Comment> comments = posts.subCollection<Comment>(
  parent: post,
  subName: 'comments',
  defaultExpiration: const Duration(days: 30),
  fromJson: Comment.fromJson,
  typeTag: 'Comment',
);

// 3. Write a `Post`
final CItem<Post> post = await posts.create(
  obj: Post('Hello World', 'First post'),
  sharedWith: {'@bob'.toAtsign()},
);

// 4. Write a `Comment` to that `Post`.
await comments.create(
  obj: Comment('Great post!'),
  sharedWith: post.sharedWith,
);
```

The returned sub-collection is a plain `AtCollection<T>`. Everything works on it just like a regular AtCollection.&#x20;

This means **multi-level nesting** works the same way: simply define a sub-collection on the sub-collection.

In practice, with a 15-character application namespace and single-character sub-collection names, the theoretical depth ceiling is 11 levels (root plus 10 nested sub-collections). Each `subCollection` call enforces the budget and throws `ArgumentError` before any I/O if the composed namespace would overflow.

## Sub-collection events

### `AtCollection.subUpdates`

`AtCollection.subUpdates` is a stream of `CSubItemUpdated`. This lets you listen for stream for sub-collection item updates (such as creates or updates).

```dart
posts.subUpdates.listen((e) { // e`` is `CSubItemUpdated`
  print('Sub-update from ${e.owner}: id=${e.id} sub=${e.subName}');
});
```

Note: when you're reacting to a `CSubItemUpdated` event from several levels down and only have its `ancestry`, `getDescendant` walks the whole chain in one call instead of you threading `subCollection` calls by hand. See example code below.

```dart
posts.subUpdates.listen((e) async {
  if (e.subName != 'comments') {
    return;
  }
  final CItem<Comment>? sample = await posts.getDescendant<Comment>(
    ancestry: e.ancestry,
    id: e.id,
    owner: e.owner,
    leafExpiration: const Duration(minutes: 10),
    leafFromJson: Comment.fromJson,
    leafTypeTag: 'Comment',
  );
  if (sample != null) window.add(sample.obj);
});
```

### `AtCollection.subDeletes`

`AtCollection.subDeletes` is a stream of `CSubItemDeleted`. This lets you stream delete events of sub-collection items.

```dart
posts.subDeletes.listen((e) { // `e` is `CSubItemDeleted`
  print('Sub-delete from ${e.owner}: id=${e.id} sub=${e.subName}');
});
```

`getDescendant` returns `null` if any link in the chain is missing (a parent expired before the leaf event arrived, for instance) rather than throwing. It requires every `CAncestor.owner` in `ancestry` to be non-null; `CSubItemDeleted` events carry null owners by design, so for those, cache the most recent `CSubItemUpdated` for the same `(id, subName)` and reuse its ancestry instead.

## Constraints

`subCollection` throws `ArgumentError` when:

* `subName` is empty or contains a `.`
* `subName` is `__rr` -- reserved for the built-in read-receipt sub-collection
* `parent.id` contains a `.`
* the composed namespace would exceed 128 characters

## Cascading deletes

Deleting a parent item without `cascade: true` is blocked if it has self-owned descendants:

```dart
// Throws StateError: item has self-owned descendants.
await posts.delete(post);

// Deletes the post and all of its comments.
await posts.delete(post, cascade: true);
```

Without `cascade: true`, sub-collection items become orphans. You can clean them up later with `cleanupOrphans()`. See [#cascade-true](atcollection.delete.md#cascade-true "mention").

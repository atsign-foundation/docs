---
description: AtCollection.delete to delete a CItem
---

# AtCollection.delete

Only the owner Atsign of the `CItem` may delete the `CItem`, whether it is shared with others or stored for self. If the `CItem` is shared with others, `AtCollection.delete` removes the self copy and every current recipient copy; recipients see their own `CItemDeleted` on the next sync/notification.

## `AtCollection.delete`

### Basic usage

```dart
// assume `posts` is of type `AtCollection<Post>` and `post` is of type `CItem<Post>`
await posts.delete(post);
```

Throws `ArgumentError` if `item.owner` isn't you:

```
You may not delete items owned by other Atsigns
```

### `cascade: false` (default)

To explain `AtCollection.delete`, we will use this subcollection example: `posts` --> `comments` --> `replies`. In this example, we have three AtCollections: `AtCollection<Post>`, `AtCollection<Comment>`, and `AtCollection<Reply>`. Read more about [atcollection.subcollection.md](atcollection.subcollection.md "mention").

```
post                   (id: p1)   namespace: posts.my_app
├── c1  "Great post!"  (id: c1)   namespace: comments.p1.posts.my_app
│   ├── reply "thanks" (id: r1)   namespace: replies.c1.comments.p1.posts.my_app
│   └── reply "+1"     (id: r2)   namespace: replies.c1.comments.p1.posts.my_app
└── c2  "I disagree"   (id: c2)   namespace: comments.p1.posts.my_app
```

`cascade` is set to `false` by default. This is a safety guard: if the item has any self-owned descendants in subcollections, the SDK throws a `StateError` and deletes nothing. You must either delete the descendants first yourself or pass `cascade: true`.

```dart
// Throws StateError if `post` has self-owned comments or replies
try {
  await posts.delete(post);
} catch (error) {
  if (error is StateError) {
    print('We need to delete descendants of `posts` first, or use `cascade: true`');
  }
}
```

To avoid the `StateError`, delete leaf-to-root — deepest descendants first:

```dart
// 1. Delete replies (leaves -- no descendants, never throws)
await replies.delete(r1);
await replies.delete(r2);

// 2. Delete comments (safe now -- their replies are gone)
await comments.delete(c1);
await comments.delete(c2);

// 3. Delete the post (safe now -- its comments are gone)
await posts.delete(post);
```

This is a lot of work, which is why [#cascade-true](atcollection.delete.md#cascade-true "mention")exists.

### `cascade: true`

`cascade: true` deletes the item and its entire owned subtree in one call.

```dart
await posts.delete(post, cascade: true);
```

Using the same `posts` example as explained above, this deletes all 5 items: the post itself, `c1`, `c2`, and the two replies.&#x20;

This works because  the `p1.posts.my_app` namespace is used in all collection namespaces; so it is assumed that items in that collection are descendents of some supercollection. The SDK finds descendants by scanning the keystore for keys whose namespace contains the parent item's ID (`p1.posts.my_app`), so it does not need the child `AtCollection` handles to be constructed.

If this collection is shared with other Atsigns, their cached copies will also be cascade deleted on sync.

## `AtCollection.cleanupOrphans`

`cascade: true` only deletes **self-owned** descendants. In our posts example, let's say `@alice` creates a post and shares it with `@bob`. `@bob` creates his own comment `c1` in the `comments.p1.posts.my_app` namespace. Later, `@alice` deletes the post with `cascade: true`, which deletes all of `@alice`'s descendants, but `@bob`'s comment is owned by `@bob`, not `@alice`. `@bob`'s comment is now an orphan: its parent post no longer exists because `@alice` owned it and also deleted it.

`cleanupOrphans()` sweeps for exactly this case, where items whose parent has been deleted by another Atsign (or on another device).

### Basic usage

`OpResult` is an operation result in the AtCollection toolkit.

```dart
final List<OpResult> results = await comments.cleanupOrphans();
await replies.cleanupOrphans();
```

Each subcollection level must be cleaned up separately.

### `cleanupOrphansOnCreation`

Set `cleanupOrphansOnCreation: true` during the instantiation of your `AtCollection<T>` object. This runs the sweep automatically once, right after construction of the object.

```dart
final AtCollection<Comment> comments = await atClient.collection<Comment>(
  'comments.$postId.posts.my_app',
  const Duration(days: 7),
  fromJson: Comment.fromJson,
  typeTag: 'Comment',
  cleanupOrphansOnCreation: true,
);
```

It is run at most one time, no matter how many times `collection<T>` is called for that namespace.

## Errors

| Exception               | Cause                                                            |
| ----------------------- | ---------------------------------------------------------------- |
| `ArgumentError`         | `item.owner` is not you                                          |
| `StateError`            | Self-owned descendants exist and `cascade` was not set to `true` |
| `CollectionOpException` | A key-level delete failed. Inspect `.failures` / `.firstFailure` |

A `CollectionOpException` from a `cascade: true` delete can mean some descendants were removed and others weren't -- check `.results` for the per-key breakdown rather than assuming the whole subtree failed together.

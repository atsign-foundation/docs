---
description: AtRecords are data records stored in atServers.
icon: books
---

# AtRecords

## What are AtRecords?

AtRecords are composed of two things: [#atkey](atrecords.md#atkey "mention") and [#atvalue](atrecords.md#atvalue "mention").

They are very similar to key-value pairs in a JSON file, but the "key" part follows a strict format (that is abstracted away by [sdks.md](../sdks.md "mention")) and "values" which can be text or binary.

Related Atsign Protocol verbs are `update`, `llookup`, `lookup`, `plookup`, and `delete`. See the [atsign-protocol.md](../atsign-protocol.md "mention") for more information.

{% hint style="info" %}
AtRecords is a new concept in our Atsign Platform docs, and you may find no reference of them in our SDKs. However, you will find lots of references to AtKeys and AtValues!
{% endhint %}

## AtKey :key:

{% hint style="info" %}
This is not to be confused with the [.atKeys file](../atsign/atkeys.md) . Since their names are very similar, we refer to AtKey-AtValue pairs as AtRecords.
{% endhint %}

An AtKey is the identifier half of the "key-value" pair. Similar to the primary key of a tabular database, the AtKey must be a unique string which represents the data.

### Types

There are 5 different AtKey types:

<table><thead><tr><th width="210.39205034291814">Type</th><th>Purpose</th></tr></thead><tbody><tr><td>Public</td><td>Store and share public data which can be seen by anyone. Using a public AtKey indicates it stores public and unencrypted plain text.</td></tr><tr><td>Self</td><td>Store data which can only be seen by the author of the AtKey. Using a self AtKey indicates it stores data intended for self, and contains encrypted data with keys only itself has.</td></tr><tr><td>Shared</td><td>Store and share private data which can only be seen by the owner and intended recipient. Using a shared AtKey indicates it points to data that is encrypted with someone else's key package.</td></tr><tr><td>Private</td><td>Store data which can only be seen by the owner, hidden by default.</td></tr><tr><td>Cached</td><td>Cache shared data from other atSigns for performance and offline mode.</td></tr></tbody></table>

### Anatomy

Below is a diagram of the anatomy of an AtKey. Learn more about key anatomy in the [atsign-protocol.md](../atsign-protocol.md "mention").

```
        @bob : city.address . my_app @ alice
        └─┬─┘  └─────┬─────┘ └──┬───┘  └─┬─┘
    sharedWith      key      namespace  sharedBy
    (recipient)                          (owner)
```

Each part of the AtKey has meaning; some mandatory and some not.

<table><thead><tr><th width="201.15277099609375">Part</th><th>Meaning</th></tr></thead><tbody><tr><td>sharedWith</td><td>This AtKey is shared with this Atsign, always starts with an <code>@</code> symbol</td></tr><tr><td>key</td><td>Mandatory. This is the name you give it</td></tr><tr><td>namespace</td><td>The application namespace it's part of (e.g. <code>my_app</code> or <code>contacts.favourites</code>)</td></tr><tr><td>sharedBy</td><td>Mandatory. The owner/author of this AtKey</td></tr></tbody></table>

### Examples

<table><thead><tr><th width="164.15625">Type</th><th width="237.5799560546875">Example</th><th>Meaning</th></tr></thead><tbody><tr><td>Public AtKey</td><td><code>public:location@alice</code></td><td>A public AtKey with a record id of <code>location</code> shared by <code>@alice</code> (<code>@alice</code> authored this AtKey). This AtKey is prefixed with <code>public:</code> which means it holds public data that any Atsign can access, even without authentication. The AtValue that this key holds is expected to be plain text.</td></tr><tr><td>Private AtKey</td><td><code>privatekey:pk1@alice</code></td><td>A private AtKey with a record id of <code>pk1</code> shared by <code>@alice</code> . No other Atsign can view this key except for <code>@alice</code>.</td></tr><tr><td>Shared AtKey</td><td><code>@bob:phone@alice</code></td><td>A shared AtKey with a record id of <code>phone</code>, shared with <code>@bob</code>, and shared by <code>@alice</code>. The AtValue is expected to be accessible by <code>@bob</code>'s cryptographic keys.</td></tr><tr><td>Internal AtKey</td><td><code>_latestnotificationid.at_skeleton_app@alice</code></td><td><p>An internal AtKey with a record id of <code>_latestnotificationid</code>, namespace of <code>at_skeleton_app</code> and is shared by <code>@alice</code>.</p><p>This does not show up in <code>scan:showHidden:true</code>. Read more about that in <a data-mention href="../atsign-protocol.md">atsign-protocol.md</a>.</p></td></tr><tr><td>Cached AtKey</td><td><code>cached:@bob:phone@alice</code></td><td>A cached AtKey with a record id of <code>phone</code>, shared with <code>@bob</code> and is shared with <code>@alice</code>. This is a key that would be expected to be cached in <code>@bob</code>'s atServer, since bob here will be caching alice's data for faster retrieval.</td></tr></tbody></table>

### Rules

* Length of an atKey should not be more than 240 characters\
  (a limitation of the current implementation of the atServer, not a protocol limitation)
* A maximum of 55 7-bit characters for the atSign (unicode is translated to UTF-7)
* Allowed characters in an entity are: `[\w._,-"']`
* Namespace is mandatory in the current implementation of the protocol\
  i.e entity must follow the notation: `<identifier>.<namespace>`
* Cached atKeys should have a different owner than the current Atsign
* Visibility scope and owner cannot be the same for a shared atKey
* Reserved atKeys cannot be [modified](/broken/pages/Ej3Lr2XTD4miZiFNE5Id) or [notified](/broken/pages/nzMHbToyp0fQoBJds83e)
* For newly created atKeys, the owner must match the current Atsign

### Reserved AtKeys

The following is a list of reserved AtKeys which the atServer requires to function.

* `privatekey:at_pkam_privatekey`
* `privatekey:at_pkam_publickey`
* `public:publickey`
* `privatekey:privatekey`
* `shared_key`
* `privatekey:self_encryption_key`
* `signing_privatekey`
* `public:signing_publickey`
* `privatekey:at_secret`
* `privatekey:at_secret_deleted`

### Reserved namespaces

You cannot give your AtKeys certain namespaces. See our section on [namespaces.md](namespaces.md "mention") regarding [reserved namespaces](namespaces.md).

### AtKey Metadata

Metadata of the atRecord is also stored and describes the following properties of the atValue.

<table data-header-hidden><thead><tr><th width="174.33333333333331"></th><th width="144"></th><th></th></tr></thead><tbody><tr><td><strong>Meta Attribute</strong></td><td><strong>Auto create?</strong></td><td><strong>Description</strong></td></tr><tr><td>availableFrom</td><td>Yes</td><td>A Date and Time derived from the ttb (now + ttb). A Key should be only available after availableFrom.</td></tr><tr><td>ccd</td><td>No</td><td>Indicates if a cached key needs to be deleted when the Atsign owner who has originally shared it deletes it.</td></tr><tr><td>createdBy</td><td>Yes</td><td>Atsign that has created the key</td></tr><tr><td>createdOn</td><td>Yes</td><td>Date and time when the key was created.</td></tr><tr><td>expiresOn</td><td>Yes</td><td>A Date and Time derived from the ttl (now + ttl). A Key should be auto deleted once it expires.</td></tr><tr><td>isBinary</td><td>No</td><td>True if the value is a binary value.</td></tr><tr><td>isCached</td><td>No</td><td>True if the key is cached.</td></tr><tr><td>isEncrypted</td><td>No</td><td>True if the value is encrypted.</td></tr><tr><td>refreshAt</td><td>No</td><td>A Date and Time derived from the ttr. The time at which the key gets refreshed.</td></tr><tr><td>sharedWith</td><td>No</td><td>Atsign of the individual with whom the key has been shared. Can be null if not shared with anyone.</td></tr><tr><td>updatedOn</td><td>Yes</td><td>Date and time when the key was last updated.</td></tr><tr><td>ttb</td><td>No</td><td>Time to birth in milliseconds.</td></tr><tr><td>ttl</td><td>No</td><td>Time to live in milliseconds.</td></tr><tr><td>ttr</td><td>No</td><td>Time in milliseconds after which the cached key needs to be refreshed. A ttr of -1 indicates that the key can be cached forever. ttr of 0 indicates do not refresh. ttr of > 0 will refresh the key. ttr of null indicates the key is impossible to cache, hence, refreshing does not make sense (which has the same effect as a ttr of 0).</td></tr></tbody></table>

## AtValue :ballot\_box:

You can save text or binary values in an atServer.

**While the atServer is suitable for small objects, you should handle large objects by reference.**

For example, to share a large file:

1. Derive a new encryption key.
2. Encrypt the file.
3. Upload the file to a storage location.
4. Notify other Atsigns of the location and the encryption key.

This "by reference" pattern is used in applications like [NoPorts](https://noports.com) to ensure efficient data transfer.

{% hint style="warning" %}
The size of the value saved in an atServer is bound by the atPlatform Protocol's config parameter "maxBufferSize".
{% endhint %}

Each AtValue belongs to one AtKey. Its metadata specifies encryption, binary encoding, and availability.

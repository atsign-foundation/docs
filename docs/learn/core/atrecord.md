---
description: The format atServers use to store and share data.
icon: books
---

# atRecord

{% hint style="info" %}
At Atsign, whenever we mention the word "key" we are normally talking about a cryptographic key.

The main exception being the atKey, this is the "key" of a "key value pair" that makes up every atRecord.&#x20;

It's unfortunate that the word "key" is polymorphic in computer science, and we have tried in the past to move to other words, but we have decided to stick with keys and the developer will have to understand the context of cryptographic key or key value pair (sorry!)
{% endhint %}

atRecords are the data records that are stored by the atServers. We use the common key-value pair format.&#x20;

## atKey

An atKey is the identifier half of the "key-value" pair. Similar to the primary key of a tabular database, the atKey must be a unique string which represents the data.

There are 5 different types of an atKey.

<table><thead><tr><th width="119.33333333333331">Type</th><th>Purpose</th></tr></thead><tbody><tr><td>Public</td><td>Store and share public data which can be seen by anyone.</td></tr><tr><td>Self</td><td>Store data which can only be seen by the owner.</td></tr><tr><td>Shared</td><td>Store and share private data which can only be seen by the owner and intended recipient.</td></tr><tr><td>Private</td><td>Store data which can only be seen by the owner, hidden by default.</td></tr><tr><td>Cached</td><td>Cache shared data from other atSigns for performance and offline mode.</td></tr></tbody></table>

### Structure of an atKey

```
[cached:]<visibility scope>:<record ID><owner’s atSign>
```

#### Cached

A marker for whether it is a cached key or not.

#### Visibility Scope

A marker for who has access to the key.

#### Record ID

A unique string used to represent the atRecord.

#### Owner's atSign

The owner (i.e. creator's) atSign for that particular atRecord. The shared by atSign of an atRecord is synonymous to the owner atSign of an atRecord.

<details>

<summary>Rules for atKeys</summary>

1. Length of an atKey should not be more than 240 characters\
   (a limitation of the current implementation of the atServer, not a protocol limitation)
2. A maximum of 55 7-bit characters for the atSign (unicode is translated to UTF-7)
3. Allowed characters in an entity are: `[\w._,-"']`
4. Namespace is mandatory in the current implementation of the protocol\
   i.e entity must follow the notation: `<identifier>.<namespace>`
5. Cached atKeys should have a different owner than the current atSign
6. Visibility scope and owner cannot be the same for a shared atKey
7. Reserved atKeys cannot be [modified](../sdk/crud-operations.md) or [notified](../sdk/events.md)
8. For newly created atKeys, the owner must match the current atSign

</details>

<details>

<summary>Reserved atKeys</summary>

The following is a list of reserved atIKeys which the atServer requires to function.

**Don't** try to delete or overwrite these keys, the atServer cannot function without them.

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

</details>

<details>

<summary>Example atKeys</summary>

**Public atKey**

1. A `public` atKey with a record id of `location` shared by `@alice`. This atKey typically holds public data that any atSign can access.

`public:location@alice`

2. A `public` atKey with a record id of `publickey` shared by `@bob`. Note that this is a [reserved atKey](atrecord.md#reserved-atids).

`public:publickey@bob`

**Private atKey**

1. A `private` atKey with a record id of `pk1` shared by `@alice`.

`privatekey:pk1@alice`

**Shared atKey**

1. A `shared` atKey with a record id of `phone` shared with `@bob`, shared by `@alice`.

`@bob:phone@alice`

2. A `shared` atKey with the record id of `name`, a namespace of `wavi`, shared with `@alice`, and shared by `@bob`.

`@alice:name.wavi@bob`

**Internal atKey**

1. An `internal` atKey with a record id of `_latestnotificationid`, namespace of `at_skeleton_app` and is shared by `@alice`.

`_latestnotificationid.at_skeleton_app@alice`

**Cached atKey**

1. A `cached` atKey with a record id of `phone`, shared with `@bob`, and is shared with `@alice`.

`cached:@bob:phone@alice`

</details>

## atMetadata

Metadata of the atRecord is also stored and describes the following properties of the atValue.

<table data-header-hidden><thead><tr><th width="174.33333333333331"></th><th width="144"></th><th></th></tr></thead><tbody><tr><td><strong>Meta Attribute</strong></td><td><strong>Auto create?</strong></td><td><strong>Description</strong></td></tr><tr><td>availableFrom</td><td>Yes</td><td>A Date and Time derived from the ttb (now + ttb). A Key should be only available after availableFrom.</td></tr><tr><td>ccd</td><td>No</td><td>Indicates if a cached key needs to be deleted when the atSign user who has originally shared it deletes it.</td></tr><tr><td>createdBy</td><td>Yes</td><td>atSign that has created the key</td></tr><tr><td>createdOn</td><td>Yes</td><td>Date and time when the key has been created.</td></tr><tr><td>expiresOn</td><td>Yes</td><td>A Date and Time derived from the ttl (now + ttl). A Key should be auto deleted once it expires.</td></tr><tr><td>isBinary</td><td>No</td><td>True if the value is a binary value.</td></tr><tr><td>isCached</td><td>No</td><td>True if the key is cached.</td></tr><tr><td>isEncrypted</td><td>No</td><td>True if the value is encrypted.</td></tr><tr><td>refreshAt</td><td>No</td><td>A Date and Time derived from the ttr. The time at which the key gets refreshed.</td></tr><tr><td>sharedWith</td><td>No</td><td>atSign of the user with whom the key has been shared. Can be null if not shared with anyone.</td></tr><tr><td>updatedOn</td><td>Yes</td><td>Date and time when the key has been last updated.</td></tr><tr><td>ttb</td><td>No</td><td>Time to birth in milliseconds.</td></tr><tr><td>ttl</td><td>No</td><td>Time to live in milliseconds.</td></tr><tr><td>ttr</td><td>No</td><td>Time in milliseconds after which the cached key needs to be refreshed. A ttr of -1 indicates that the key can be cached forever. ttr of 0 indicates do not refresh. ttr of > 0 will refresh the key. ttr of null indicates the key is impossible to cache, hence, refreshing does not make sense (which has the same effect as a ttr of 0).</td></tr></tbody></table>

## atValue

Text or binary values can be saved in an atServer.&#x20;

**Small objects are fine to use the atServer but large objects should be used by reference.**

For example, derive a new encryption key, encypt a file, upload that file to location, then notify other atSigns of the location and the encyption key. This is how [atmospherePro ](https://atsign.com/apps/atmospherepro/)works.<br>

{% hint style="warning" %}
The size of the value saved in an atServer is bound by the atProtocol's config parameter "maxBufferSize".
{% endhint %}

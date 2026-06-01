---
description: Learn how to create atKeys for your chosen platform
---

# atKey Reference

{% hint style="warning" %}
atKey

Please note that any reference to the word "atKey" in this document is not associated with cryptographic keys. The atKey is the "key" of the key-value pair that makes up an atRecord.
{% endhint %}

{% hint style="warning" %}
This article explains how to create an atKey in the  atSDK.

If you are unfamiliar with atKeys please read [this](../../atsign-platform/atrecord.md#atid) first.
{% endhint %}

{% tabs %}
{% tab title="Flutter / Dart" %}
## Flutter / Dart

### Package Installation

The [at\_commons](https://pub.dev/packages/at_commons) package contains common elements used in a number of Atsign's Flutter and Dart packages. This package needs to be included in the application in order to create atKeys.

First add the package to your project:

```
flutter pub add at_commons
```

### Usage

See below for how to create the various types of atKeys.

#### Public atKey

To create a public atKey, first use the `AtKey.public` builder to configure it, then call `.build` to create it.

{% code title="AtKey.public signature" %}
```dart
static PublicKeyBuilder public(String key,
    {String? namespace, String sharedBy = ''})
```
{% endcode %}

The `build` method on `PublicKeyBuilder` takes no parameters.

**Example**

`public:phone.wavi@alice`

{% code overflow="wrap" %}
```dart
AtKey myPublicID = AtKey.public('phone', namespace: 'wavi', sharedBy: '@alice').build();
```
{% endcode %}

#### Self atKey

To create a self atkeyD, first use the `AtKey.self` builder to configure it, then call `.build` to create it.

{% code title="AtKey.self signature" %}
```dart
static SelfKeyBuilder self(String key,
    {String? namespace, String sharedBy = ''})
```
{% endcode %}

The `build` method on `SelfKeyBuilder` takes no parameters.

**Example**

`phone.wavi@alice`

{% code overflow="wrap" %}
```dart
AtKey mySelfID = AtKey.self('phone', namespace: 'wavi', sharedBy: '@alice').build();
```
{% endcode %}

#### Shared atKey

To create a shared atKey, first use the `AtKey.shared` builder to configure it, then call `.build` to create it.

{% code title="AtKey.shared signature" %}
```dart
static SharedKeyBuilder shared(String key,
    {String? namespace, String sharedBy = ''})
```
{% endcode %}

The `build` method on `SharedKeyBuilder` takes no parameters.

**Example**

`@bob:phone.wavi@alice`

{% code overflow="wrap" %}
```dart
AtKey sharedKey = (AtKey.shared('phone', 'wavi')
    ..sharedWith('@bob')).build();
```
{% endcode %}

**Caching shared atKeys**

To cache a shared atKey, you can do a [cascade](https://dart.dev/language/operators#cascade-notation) call on `SharedKeyBuilder.cache`.

{% code title="Caching example" %}
```dart
AtKey atKey = (AtKey.shared('phone', namespace: 'wavi', sharedBy: '@alice')
 ..sharedWith('@bob')
 ..cache(1000, true))
 .build();
```
{% endcode %}

### API Docs

You can find the API reference for the entire package available on [pub](https://pub.dev/documentation/at_commons/latest/).

The `AtKey` class API reference is available [here](https://pub.dev/documentation/at_commons/latest/at_commons/AtKey-class.html).
{% endtab %}

{% tab title="C" %}
## C

You can find all these examples also on our [GitHub](https://github.com/atsign-foundation/at_demos/tree/trunk/demos/get_started_c).

### Introduction

There are three kinds of atKeys:

1. [Public atKey](atid-reference.md#public-atkey)
2. [Self atKey](atid-reference.md#self-atkey)
3. [Shared atKey](atid-reference.md#shared-atkey)

[Public atKey](atid-reference.md#public-atkey) holds public (and non-encrypted) data, available for any Atsign to get from you.&#x20;

[Self atKey](atid-reference.md#self-atkey) holds self encrypted data, only available for your own Atsign to get.&#x20;

[Shared atKey](atid-reference.md#shared-atkey) holds encrypted data that is only decipherable by you and the intended recipient.

Every atKey has [metadata](atid-reference.md#metadata), which is free for you to also control (to an extent). Not all metadata should be handled by the developer. Some metadata is managed by the SDK itself. Check out our [documentation](../../atsign-platform/atrecord.md) on metadata to find out which metadata is worth your time handling.

Before running any of the examples, be sure to include `atkey.h`

```c
#include <atclient/atkey.h>
```

### Public atKey

This is how you create a Public atKey.

First, create the atkey struct.

```c
atclient_atkey my_public_atkey;
atclient_atkey_init(&my_public_atkey);
```

Next, call the `atclient_atkey_create_public_key` function.

```c
const char *atkey_key = "phone";
const char *atkey_shared_by = "@jeremy_0";
const char *atkey_namespace = "wavi"; 

if(atclient_atkey_create_public_key(
  &my_public_atkey,
  atkey_key,
  atkey_shared_by,
  atkey_namespace) != 0) {
  // an error occurred
}
```

Don't forget to call `atclient_atkey_free` at the end of the intended life-time of your atkey struct.

```c
atclient_atkey_free(&my_public_atkey);
```

#### Example Application

```c
#include <atclient/atclient.h>
#include <atclient/atclient_utils.h>
#include <atlogger/atlogger.h>
#include <stdlib.h>

#define ATSIGN "@soccer99"

int main()
{
    int exit_code = -1;

    /*
     * Create an atkey struct
     */
    atclient_atkey my_public_atkey;
    atclient_atkey_init(&my_public_atkey);

    const char *atkey_key = "phone";
    const char *atkey_shared_by = ATSIGN;
    const char *atkey_namespace = "wavi"; 

    /*
     * Use the dandy atkey_create_public_key function to populate the struct for you with your desired values.
     */
    if((exit_code = atclient_atkey_create_public_key(&my_public_atkey, atkey_key, atkey_shared_by, atkey_namespace)) != 0) {
        goto exit;
    }

    exit_code = 0;
exit:
{
    atclient_atkey_free(&my_public_atkey);
    return exit_code;
}
}
```

### Self atKey

This is how you create a Self atKey.

First, create the `atclient_atkey struct`

<pre class="language-c"><code class="lang-c"><strong>atclient_atkey my_self_atkey;
</strong>atclient_atkey_init(&#x26;my_self_atkey);
</code></pre>

Next, call the `atclient_atkey_create_self_key` function.

```c
const char *atkey_key = "phone";
const char *atkey_shared_by = "@jeremy_0";
const char *atkey_namespace = "wavi";

if(atclient_atkey_create_self_key(
  &my_self_atkey,
  atkey_key,
  atkey_shared_by,
  atkey_namespace) != 0) {
  // an error occurred
}
```

Don't forget to call `atclient_atkey_free` at the end of the intended life-time of your atkey struct.

```c
atclient_atkey_free(&my_public_atkey);
```

#### Example Application

```c
#include <atclient/atclient.h>
#include <atclient/atclient_utils.h>
#include <atlogger/atlogger.h>
#include <stdlib.h>

#define ATSIGN "@soccer99"

int main()
{
    int exit_code = -1;

    /*
     * Create an atkey struct
     */
    atclient_atkey my_self_atkey;
    atclient_atkey_init(&my_self_atkey);

    const char *atkey_key = "phone";
    const char *atkey_shared_by = ATSIGN;
    const char *atkey_namespace = "wavi";

    /*
     * Use the dandy atkey_create_self_key function to populate the struct for you with your desired values.
     */
    if((exit_code = atclient_atkey_create_self_key(&my_self_atkey, atkey_key, atkey_shared_by, atkey_namespace)) != 0) {
        goto exit;
    }

    exit_code = 0;
exit:
{
    atclient_atkey_free(&my_self_atkey);
    return exit_code;
}
}
```

### Shared atKey

This is how you create a Shared atKey.

First, create the `atclient_atkey` struct.

```c
atclient_atkey my_shared_atkey;
atclient_atkey_init(&my_shared_atkey);
```

Next, call the `atclient_atkey_create_shared_key` function.

```c
const char *atkey_key = "phone";
const char *atkey_shared_by = ATSIGN;
const char *atkey_shared_with = "@soccer99";
const char *atkey_namespace = "wavi";

if(atclient_atkey_create_shared_key(&my_shared_atkey,
  atkey_key,
  atkey_shared_by,
  atkey_shared_with, 
  atkey_namespace) != 0) {
  // an error occurred
}
```

Don't forget to call `atclient_atkey_free` at the end of the intended life-time of your atkey struct.

```c
atclient_atkey_free(&my_public_atkey);
```

#### Example Application

```c
#include <atclient/atclient.h>
#include <atclient/atclient_utils.h>
#include <atlogger/atlogger.h>
#include <stdlib.h>

#define ATSIGN "@soccer99"

int main()
{
    int exit_code = -1;

    /*
     * Create an atkey struct
     */
    atclient_atkey my_shared_atkey;
    atclient_atkey_init(&my_shared_atkey);

    const char *atkey_key = "phone";
    const char *atkey_shared_by = ATSIGN;
    const char *atkey_shared_with = "@soccer99";
    const char *atkey_namespace = "wavi";

    /*
     * Use the dandy atkey_create_shared_key function to populate the struct for you with your desired values.
     */
    if((exit_code = atclient_atkey_create_shared_key(&my_shared_atkey, atkey_key, atkey_shared_by, atkey_shared_with, atkey_namespace)) != 0) {
        goto exit;
    }


    exit_code = 0;
exit:
{
    atclient_atkey_free(&my_shared_atkey);
    return exit_code;
}
}
```

### Metadata

This is how you modify the metadata of any atKey. For the sake of this example, we will modify the metadata of a Shared atKey.

First, create your atKey as usual.

```c
atclient_atkey my_shared_atkey;
atclient_atkey_init(&my_shared_atkey);

const char *atkey_key = "phone";
const char *atkey_shared_by = "@jeremy_0";
const char *atkey_shared_with = "@soccer99";
const char *atkey_namespace = "wavi";

if (atclient_atkey_create_shared_key(&my_shared_atkey, atkey_key, atkey_shared_by, atkey_shared_with, atkey_namespace) != 0)
{
  // an error occurred
}
```

Next, let's create a pointer to the atKey's metadata. We can feel safe reading the `atclient_atkey`'s inner metadata field because we have previously called `atclient_atkey_init`.

```c
atclient_atkey_metadata *metadata = &(my_shared_atkey.metadata);
```

Call a `atclient_atkey_metadata_set_*` function. For the sake of this example, we will call `atclient_atkey_metadata_set_ttl` which sets the atKey's lifespan, specified in milliseconds.

```c
if (atclient_atkey_metadata_set_ttl(metadata, 1 * 1000) != 0)
{
    // an error occurred
}
```

We can check if that was set and is safe to read by using the `atclient_atkey_metadata_is_ttl_initialized(metadata)` function. It is important to check that `ttl` was initialized. If this function returns true, we can feel safe knowing that the the internal initialized bit was set to true previously and that we are not reading garbage data.

```c
if(atclient_atkey_metadata_is_ttl_initialized(metadata)) {
    // metadata->ttl is safe to read and we know it is populated.
    printf("metadata->ttl: %d\n", metadata->ttl);
}
```

#### Example Application

```c
#include <atclient/atclient.h>
#include <atclient/atclient_utils.h>
#include <atlogger/atlogger.h>
#include <stdlib.h>

#define ATSIGN "@soccer99"

#define ATSERVER_HOST "root.atsign.org"
#define ATSERVER_PORT 64

int main()
{
    int exit_code = -1;

    atlogger_set_logging_level(ATLOGGER_LOGGING_LEVEL_DEBUG);

    atclient_atkey my_shared_atkey;
    atclient_atkey_init(&my_shared_atkey);

    const char *atkey_key = "phone";
    const char *atkey_shared_by = ATSIGN;
    const char *atkey_shared_with = "@soccer99";
    const char *atkey_namespace = "wavi";

    if ((exit_code = atclient_atkey_create_shared_key(&my_shared_atkey, atkey_key, atkey_shared_by, atkey_shared_with, atkey_namespace)) != 0)
    {
        goto exit;
    }

    /*
     * Now that the AtKey is properly set up, we can set the metadata of that AtKey like this.
     * Since we called the `atclient_atkey_init` function earlier, we can feel safe knowing that the metadata is also already initialized.
     */
    atclient_atkey_metadata *metadata = &(my_shared_atkey.metadata);

    /*
     * Set the ttl (time to live) of the AtKey. Once this AtKey is put into the atServer, it will only live for 1000 milliseconds.
     */
    if ((exit_code = atclient_atkey_metadata_set_ttl(metadata, 1 * 1000)) != 0)
    {
        goto exit;
    }

    /*
     * Set the ttr (time to refresh) of the AtKey. Once this AtKey is put into the atServer, the recipient of the AtKey will have it refreshed every 1000
     * milliseconds, in case there are any value changes.
     */
    if ((exit_code = atclient_atkey_metadata_set_ttr(metadata, 1 * 1000)) != 0)
    {
        goto exit;
    }

    if(atclient_atkey_metadata_is_ttl_initialized(metadata)) {
        // metadata->ttl is safe to read and we know it is populated.
        atlogger_log("main", ATLOGGER_LOGGING_LEVEL_DEBUG, "metadata->ttl: %d\n", metadata->ttl); // [DEBG] 2024-08-15 01:52:09.596055 | main | metadata->ttl: 1000
    }

    exit_code = 0;
exit:
{
    atclient_atkey_free(&my_shared_atkey); // this _free command will automatically free the metadata as well
    return exit_code;
}
}
```

x
{% endtab %}
{% endtabs %}


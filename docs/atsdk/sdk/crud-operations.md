---
description: How to do basic CRUD operations on an atServer
---

# CRUD Operations

{% tabs %}
{% tab title="Flutter / Dart" %}
In Dart, the AtClient is stored within the AtClientManager. Once an Atsign has been [onboarded](onboarding.md), you will be able to access the AtClientManager for its associated atSign.

### AtClientManager

AtClientManager is a [singleton](https://en.wikipedia.org/wiki/Singleton_pattern) model. When `AtClientManager.getInstance()` is called, it will get the AtClientManager instance for the last onboarded Atsign.

```dart
AtClientManager atClientManager = AtClientManager.getInstance();
```

{% hint style="info" %}
If you need simultaneous access to multiple atClients, you need to create a new [isolate](https://dart.dev/language/concurrency#how-isolates-work) for each additional atClient, and onboard its Atsign within the isolate.

An example of this pattern can be found in [at\_daemon\_server](https://github.com/atsign-foundation/at_services/tree/trunk/packages/at_daemon_server/lib/src/server).
{% endhint %}

### AtClient

As previously mentioned, the AtClientManager stores the actual AtClient itself. You can retrieve the `AtClient` by calling `atClientManager.atClient`.

```dart
AtClient atClient = atClientManager.atClient;
```

#### atKey

Before you can do anything with an atRecord, you need an [atKey](../../atsign-platform/atrecord.md#atidentifier) to represent it.

If you don't know how to create an atKey, please see the [reference](atid-reference.md) first.

{% hint style="info" %}
The following examples use the self atKey `phone.wavi@<current atSign>`

It is up to the developer to modify the atKey according to their use case.
{% endhint %}

#### Creating / Updating Data

To create data the `put` method is used, this method accepts text (`String`) or binary (`List<int>`).

{% code overflow="wrap" %}
```dart
String currentAtSign = atClient.getCurrentAtSign()!;
AtKey myID = AtKey.self('phone', namespace: 'wavi',
                        sharedBy: currentAtSign).build();
String dataToStore = "123-456-7890";
bool res = await atClient.put(myID, dataToStore);
```
{% endcode %}

<pre class="language-dart" data-title="put signature"><code class="lang-dart">Future&#x3C;bool> put(
    AtKey key,
    dynamic value,
    {bool <a data-footnote-ref href="#user-content-fn-1">isDedicated</a> = false,
<strong>    PutRequestOptions? putRequestOptions});
</strong></code></pre>

**Strongly Typed Methods**

Since `put` accepts both `String` or `List<int>` as a value, the typing is dynamic. atClient also contains the strongly typed `putText` which only accepts `String` for the value, or `putBinary` which only accepts `List<int>` for the value.

**To update existing data**

Updating existing data is done by doing a put to the same atKey, this will overwrite any existing data stored in the atRecord.

```dart
List<int> binaryData = [1, 2, 3, 4];
bool res = await atClient.put(myID, binaryData);
```

The bytes `[1, 2, 3, 4]` have now replaced the string `"123-456-7890"`.

#### Reading Data

There are two parts to reading data using the atClient SDK:

1. Scanning for and listing out the atKeys for atRecords that can be retrieved
2. Retrieving the atRecord for a given atKey

**1. Scanning and listing** atKey**s**

There are two methods available for scanning and listing atKeys:

1. `getAtKeys` which provides the list in Class format (i.e. `List<AtKey>`)
2. `getKeys` which provides the list in String format (i.e. `List<String>`)

Both methods accept the same parameters, only the return type is different. So we will show the more commonly used getAtKeys signature:

{% code title="getAtKeys signature" %}
```dart
Future<List<AtKey>> getAtKeys(
    {String? regex,
    String? sharedBy,
    String? sharedWith,
    bool showHiddenKeys = false});
```
{% endcode %}

_regex_

A regular expression used to filter the list of atKeys.

_sharedBy_

Filter the list of atKeys to only include ones shared by a particular Atsign.

_sharedWith_

Filter the list of atKeys to only include ones shared with a particular Atsign.

_showHiddenKeys_

A boolean flag to enable the inclusion of hidden atKeys (default = `false`)

**Usage**

Get all available (non-hidden) atKeys:

```dart
List<AtKey> allIDs = await atClient.getAtKeys();
```

All atKeys which end with ".wavi" in the record identifier part:

```dart
List<AtKey> waviIDs = await atClient.getAtKeys(regex: '^.*\.wavi@.+$');
```

**2. Retrieving atRecords by** atKey

To retrieve an atRecord, you must know the atKey and pass it to the `get` method.

{% code title="get signature" %}
```dart
Future<AtValue> get(
    AtKey key,
    {bool isDedicated = false,
    GetRequestOptions? getRequestOptions});
```
{% endcode %}

Calling this function will return an AtValue, and update the atKey passed to it with any changes to the metadata.

**Usage**

<pre class="language-dart"><code class="lang-dart">AtValue atValue = await atClient.get(myID);
String? text = atValue.<a data-footnote-ref href="#user-content-fn-2">value</a>;
</code></pre>

#### Deleting Data

To delete data, simply call the `delete` method with the atKey for the atRecord to delete.

{% code title="delete signature" %}
```dart
Future<bool> delete(
    AtKey key,
    {bool isDedicated = false});
```
{% endcode %}

**Usage**

```dart
bool res = await atClient.delete(myID);
```

### Additional Features

See [#additional-features](crud-operations.md#additional-features "mention") to learn about synchronization, which supports syncing of a local atServer, allowing CRUD operations to work even if the application has no internet access.

### API Docs

You can find the API reference for the entire package available on [pub](https://pub.dev/documentation/at_client/latest/).

The `AtClient` class API reference is available [here](https://pub.dev/documentation/at_client/latest/at_client/AtClient-class.html).
{% endtab %}

{% tab title="C" %}
## C

You can find all of these examples on our [GitHub](https://github.com/atsign-foundation/at_demos/tree/trunk/demos/get_started_c).

### Table of Contents

* [#introduction](crud-operations.md#introduction "mention")
* [#put-public-atkey](crud-operations.md#put-public-atkey "mention")
  * [#id-1.-create-public-atkey](crud-operations.md#id-1.-create-public-atkey "mention")
  * [#id-2.-call-atclient\_put\_public\_key](crud-operations.md#id-2.-call-atclient_put_public_key "mention")
  * [#example-application](crud-operations.md#example-application "mention")
* [#put-self-atkey](crud-operations.md#put-self-atkey "mention")
  * [#id-1.-create-a-self-atkey](crud-operations.md#id-1.-create-a-self-atkey "mention")
  * [#id-2.-call-atclient\_put\_self\_key](crud-operations.md#id-2.-call-atclient_put_self_key "mention")
  * [#example-application-1](crud-operations.md#example-application-1 "mention")
* [#put-shared-atkey](crud-operations.md#put-shared-atkey "mention")
  * [#id-1.-create-a-shared-atkey](crud-operations.md#id-1.-create-a-shared-atkey "mention")
  * [#id-2.-call-atclient\_put\_shared\_key](crud-operations.md#id-2.-call-atclient_put_shared_key "mention")
  * [#example-application-2](crud-operations.md#example-application-2 "mention")
* [#get-public-atkey](crud-operations.md#get-public-atkey "mention")
  * [#id-1.-create-public-atkey-1](crud-operations.md#id-1.-create-public-atkey-1 "mention")
  * [#id-2.-call-atclient\_get\_public\_key](crud-operations.md#id-2.-call-atclient_get_public_key "mention")
  * [#id-3.-free-value](crud-operations.md#id-3.-free-value "mention")
  * [#example-application-3](crud-operations.md#example-application-3 "mention")
* [#get-self-atkey](crud-operations.md#get-self-atkey "mention")
  * [#id-1.-create-a-self-atkey-1](crud-operations.md#id-1.-create-a-self-atkey-1 "mention")
  * [#id-2.-call-atclient\_get\_self\_key](crud-operations.md#id-2.-call-atclient_get_self_key "mention")
  * [#id-3.-free-value-1](crud-operations.md#id-3.-free-value-1 "mention")
  * [#example-application-4](crud-operations.md#example-application-4 "mention")
* [#get-shared-atkey](crud-operations.md#get-shared-atkey "mention")
  * [#id-1.-create-a-shared-atkey-1](crud-operations.md#id-1.-create-a-shared-atkey-1 "mention")
  * [#id-2.-call-atclient\_get\_shared\_key](crud-operations.md#id-2.-call-atclient_get_shared_key "mention")
  * [#id-3.-free-value-2](crud-operations.md#id-3.-free-value-2 "mention")
  * [#example-application-5](crud-operations.md#example-application-5 "mention")
* [#delete-an-atkey](crud-operations.md#delete-an-atkey "mention")
  * [#id-1.-create-an-atkey](crud-operations.md#id-1.-create-an-atkey "mention")
  * [#id-2.-call-atclient\_delete](crud-operations.md#id-2.-call-atclient_delete "mention")
  * [#example-application-6](crud-operations.md#example-application-6 "mention")
* [#request-options](crud-operations.md#request-options "mention")

### Introduction

In this section, we will learn how to `put`, `get`, and `delete` an atKey.

If you are unfamiliar with the different atKey types, check out our documentation on [atRecords](../../atsign-platform/atrecord.md).

### Put Public atKey

#### 1. Create Public atKey

First, create a public atKey. It is important to note that the `shared_by` Atsign should be the same as the authenticated Atsign in the application.

```c
atclient_atkey my_public_atkey;
atclient_atkey_init(&my_public_atkey);

const char *atkey_key = "phone";
const char *atkey_shared_by = "@jeremy_0";
const char *atkey_namespace = "c_demos";

if (atclient_atkey_create_public_key(&my_public_atkey, atkey_key, atkey_shared_by, atkey_namespace) != 0)
{
  // an error occurred
}
```

#### 2. Call \`atclient\_put\_public\_key\`

Next, simply \*put\* the value into your atServer. Since we are putting a public value into our atServer, no data will be encrypted and this data will be available for any Atsign to get.

We will pass `NULL` into the request\_options and commit\_id parameters because we want to use the default options for now and we don't particularly care about the commit\_id that it returns, but you could receive it if you would like.

This function returns an `int` for error handling, in which a non-zero exit code indicates an error.

```c
const char *atkey_value = "123-456-7890";

if (atclient_put_public_key(&atclient, &my_public_atkey, atkey_value, NULL, NULL) != 0)
{
    // an error occurred
}
```

#### Example Application

```c
#include <atclient/atclient.h>
#include <atclient/atclient_utils.h>
#include <atclient/constants.h>
#include <atlogger/atlogger.h>
#include <stdlib.h>

#define ATSIGN "@soccer99"

int main()
{
    int exit_code = -1;

    atlogger_set_logging_level(ATLOGGER_LOGGING_LEVEL_DEBUG);

    char *atserver_host = NULL;
    int atserver_port = 0;

    atclient_atkeys atkeys;
    atclient_atkeys_init(&atkeys);

    atclient atclient;
    atclient_init(&atclient);

    atclient_atkey my_public_atkey;
    atclient_atkey_init(&my_public_atkey);

    if ((exit_code = atclient_utils_find_atserver_address(ATCLIENT_ATDIRECTORY_PRODUCTION_HOST, ATCLIENT_ATDIRECTORY_PRODUCTION_PORT, ATSIGN, &atserver_host, &atserver_port)) != 0)
    {
        goto exit;
    }

    if ((exit_code = atclient_utils_populate_atkeys_from_homedir(&atkeys, ATSIGN)) != 0)
    {
        goto exit;
    }

    if ((exit_code = atclient_pkam_authenticate(&atclient, atserver_host, atserver_port, &atkeys, ATSIGN)) != 0)
    {
        goto exit;
    }

    const char *atkey_key = "phone";
    const char *atkey_shared_by = ATSIGN;
    const char *atkey_namespace = "c_demos";

    if ((exit_code = atclient_atkey_create_public_key(&my_public_atkey, atkey_key, atkey_shared_by, atkey_namespace)) != 0)
    {
        goto exit;
    }

    const char *atkey_value = "123-456-7890";

    /*
     * atclient_put_self_key lets you put a key-value pair in your atSign's atServer.
     * For our purposes, we will pass `NULL` for the request options and the commit id. 
     * We want to use the default options and we don't want to receive and
     * store the commit id.
     */
    if ((exit_code = atclient_put_public_key(&atclient, &my_public_atkey, atkey_value, NULL, NULL)) != 0)
    {
        goto exit;
    }

    exit_code = 0;
exit:
{
    free(atserver_host);
    atclient_atkeys_free(&atkeys);
    atclient_free(&atclient);
    atclient_atkey_free(&my_public_atkey);
    return exit_code;
}
}
```

### Put Self atKey

#### 1. Create a Self atKey

```c
atclient_atkey my_self_atkey;
atclient_atkey_init(&my_self_atkey);

const char *atkey_key = "phone";
const char *atkey_shared_by = "@jeremy_0";
const char *atkey_namespace = "c_demos";

if (atclient_atkey_create_self_key(&my_self_atkey, atkey_key, atkey_shared_by, atkey_namespace) != 0)
{
    // an error occurred
}
```

#### 2. Call \`atclient\_put\_self\_key\`

This will put a value specially encrypted for your atServer that only the Atsign's atKeys can decrypt.

We will pass `NULL` into the request\_options and commit\_id parameters because we want to use the default options for now and we don't particularly care about the commit\_id that it returns, but you could receive it if you would like.

This function will return an `int` for error handling, in which a non-zero exit code indicates an error.&#x20;

```c
const char *atkey_value = "123-456-7890";

if (atclient_put_self_key(&atclient, &my_self_atkey, atkey_value, NULL, NULL) != 0)
{
  // an error occurred
}
```

#### Example Application

```c
#include <atclient/atclient.h>
#include <atclient/atclient_utils.h>
#include <atclient/constants.h>
#include <atlogger/atlogger.h>
#include <stdlib.h>

#define ATSIGN "@soccer99"

int main()
{
    int exit_code = -1;

    atlogger_set_logging_level(ATLOGGER_LOGGING_LEVEL_DEBUG);

    char *atserver_host = NULL;
    int atserver_port = 0;

    atclient_atkeys atkeys;
    atclient_atkeys_init(&atkeys);

    atclient atclient;
    atclient_init(&atclient);

    atclient_atkey my_self_atkey;
    atclient_atkey_init(&my_self_atkey);

    if ((exit_code = atclient_utils_find_atserver_address(ATCLIENT_ATDIRECTORY_PRODUCTION_HOST, ATCLIENT_ATDIRECTORY_PRODUCTION_PORT, ATSIGN, &atserver_host, &atserver_port)) != 0)
    {
        goto exit;
    }

    if ((exit_code = atclient_utils_populate_atkeys_from_homedir(&atkeys, ATSIGN)) != 0)
    {
        goto exit;
    }

    if ((exit_code = atclient_pkam_authenticate(&atclient, atserver_host, atserver_port, &atkeys, ATSIGN)) != 0)
    {
        goto exit;
    }

    const char *atkey_key = "phone";
    const char *atkey_shared_by = ATSIGN;
    const char *atkey_namespace = "c_demos";

    if ((exit_code = atclient_atkey_create_self_key(&my_self_atkey, atkey_key, atkey_shared_by, atkey_namespace)) != 0)
    {
        goto exit;
    }

    const char *atkey_value = "123-456-7890";

    /*
     * atclient_put_self_key lets you put a key-value pair in your Atsign's atServer.
     * For our purposes, we will pass `NULL` for the request options and the commit id. 
     * We want to use the default options and we don't want to receive and
     * store the commit id.
     */
    if ((exit_code = atclient_put_self_key(&atclient, &my_self_atkey, atkey_value, NULL, NULL)) != 0)
    {
        goto exit;
    }

    exit_code = 0;
exit:
{
    free(atserver_host);
    atclient_atkeys_free(&atkeys);
    atclient_free(&atclient);
    atclient_atkey_free(&my_self_atkey);
    return exit_code;
}
}

```

### Put Shared atKey

#### 1. Create a Shared atKey

```c
atclient_atkey my_shared_atkey;
atclient_atkey_init(&my_shared_atkey);

const char *atkey_key = "phone";
const char *atkey_shared_by = "@jeremy_0";
const char *atkey_shared_with = "@soccer0";
const char *atkey_namespace = "c_demos";
if (atclient_atkey_create_shared_key(&my_shared_atkey, atkey_key, atkey_shared_by, atkey_shared_with, atkey_namespace) != 0)
{
    // an error occurred
}
```

#### 2. Call \`atclient\_put\_shared\_key\`

This function will put our string value into the atServer. Since we are using a Shared atKey, that means only the `shared_by` and `shared_with` Atsign will be able to decrypt this value.&#x20;

We will pass `NULL` into the request\_options and commit\_id parameters because we want to use the default options for now and we don't particularly care about the commit\_id that it returns, but you could receive it if you would like.

This function returns an `int` for error handling, in which a non-zero exit code indicates an error.

```c
const char *atkey_value = "123-456-7890";
if (atclient_put_shared_key(&atclient, &my_shared_atkey, atkey_value, NULL, NULL) != 0)
{
    // an error occurred
}
```

#### Example Application

```c
#include <atclient/atclient.h>
#include <atclient/atclient_utils.h>
#include <atclient/constants.h>
#include <atlogger/atlogger.h>
#include <stdlib.h>

#define ATSIGN "@soccer99"

int main()
{
    int exit_code = -1;

    atlogger_set_logging_level(ATLOGGER_LOGGING_LEVEL_DEBUG);

    char *atserver_host = NULL;
    int atserver_port = 0;

    atclient_atkeys atkeys;
    atclient_atkeys_init(&atkeys);

    atclient atclient;
    atclient_init(&atclient);

    atclient_atkey my_shared_atkey;
    atclient_atkey_init(&my_shared_atkey);

    if ((exit_code = atclient_utils_find_atserver_address(ATCLIENT_ATDIRECTORY_PRODUCTION_HOST, ATCLIENT_ATDIRECTORY_PRODUCTION_PORT, ATSIGN, &atserver_host, &atserver_port)) != 0)
    {
        goto exit;
    }

    if ((exit_code = atclient_utils_populate_atkeys_from_homedir(&atkeys, ATSIGN)) != 0)
    {
        goto exit;
    }

    if ((exit_code = atclient_pkam_authenticate(&atclient, atserver_host, atserver_port, &atkeys, ATSIGN)) != 0)
    {
        goto exit;
    }

    const char *atkey_key = "phone";
    const char *atkey_shared_by = ATSIGN;
    const char *atkey_shared_with = "@soccer0";
    const char *atkey_namespace = "c_demos";

    if ((exit_code = atclient_atkey_create_shared_key(&my_shared_atkey, atkey_key, atkey_shared_by, atkey_shared_with, atkey_namespace)) != 0)
    {
        goto exit;
    }

    const char *atkey_value = "123-456-7890";

    /*
     * atclient_put_self_key lets you put a key-value pair in your Atsign's atServer.
     * For our purposes, we will pass `NULL` for the request options and the commit id. 
     * We want to use the default options and we don't want to receive and
     * store the commit id.
     */
    if ((exit_code = atclient_put_shared_key(&atclient, &my_shared_atkey, atkey_value, NULL, NULL)) != 0)
    {
        goto exit;
    }

    exit_code = 0;
exit:
{
    free(atserver_host);
    atclient_atkeys_free(&atkeys);
    atclient_free(&atclient);
    atclient_atkey_free(&my_shared_atkey);
    return exit_code;
}
}
```

### Get Public atKey

#### 1. Create Public atKey

First, create a public atKey. It is important to note that the `shared_by` Atsign should be the same as the authenticated Atsign in the application.

```c
atclient_atkey my_public_atkey;
atclient_atkey_init(&my_public_atkey);

const char *atkey_key = "phone";
const char *atkey_shared_by = "@jeremy_0";
const char *atkey_namespace = "c_demos";

if (atclient_atkey_create_public_key(&my_public_atkey, atkey_key, atkey_shared_by, atkey_namespace) != 0)
{
  // an error occurred
}
```

#### 2. Call \`atclient\_get\_public\_key\`

We will create a variable named `value` and pass the address to it in our `atclient_get_public_key` function call. The function will allocate memory for us and populate that variable for us.&#x20;

We will pass `NULL` into the request\_options parameter because we want to use the default options for now.

This function returns an `int` for error handling, in which a non-zero exit code indicates an error.

```c
char *value = NULL;
if (atclient_get_public_key(&atclient, &my_public_atkey, &value, NULL) != 0)
{
    // an error occurred
}
```

#### 3. Free \`value\`

Once we are done with the value itself, we should free it to avoid any memory leaks.

```c
free(value);
```

#### Example Application

```c
#include <atclient/atclient.h>
#include <atclient/atclient_utils.h>
#include <atclient/constants.h>
#include <atlogger/atlogger.h>
#include <stdlib.h>

#define ATSIGN "@soccer99"

int main()
{
    int exit_code = -1;

    atlogger_set_logging_level(ATLOGGER_LOGGING_LEVEL_DEBUG);

    char *atserver_host = NULL;
    int atserver_port = 0;

    atclient_atkeys atkeys;
    atclient_atkeys_init(&atkeys);

    atclient atclient;
    atclient_init(&atclient);

    atclient_atkey my_public_atkey;
    atclient_atkey_init(&my_public_atkey);

    /*
     * Let's declare a null pointer which will later be allocated by our `atclient_get_public_key` function. It is our responsibility to free it once we are
     * done with it.
     */
    char *value = NULL;

    if ((exit_code = atclient_utils_find_atserver_address(ATCLIENT_ATDIRECTORY_PRODUCTION_HOST, ATCLIENT_ATDIRECTORY_PRODUCTION_PORT, ATSIGN, &atserver_host, &atserver_port)) != 0)
    {
        goto exit;
    }

    if ((exit_code = atclient_utils_populate_atkeys_from_homedir(&atkeys, ATSIGN)) != 0)
    {
        goto exit;
    }

    if ((exit_code = atclient_pkam_authenticate(&atclient, atserver_host, atserver_port, &atkeys, ATSIGN)) != 0)
    {
        goto exit;
    }

    const char *atkey_key = "phone";
    const char *atkey_shared_by = ATSIGN;
    const char *atkey_namespace = "c_demos";

    if ((exit_code = atclient_atkey_create_public_key(&my_public_atkey, atkey_key, atkey_shared_by, atkey_namespace)) != 0)
    {
        goto exit;
    }

    /*
     * `atclient_get_public_key` will allocate memory for the `value` pointer. It is our responsibility to free it once we are done with it.
     * We will pass `NULL` into the request_options argument for now and just use the default request options.
     */
    if ((exit_code = atclient_get_public_key(&atclient, &my_public_atkey, &value, NULL)) != 0)
    {
        goto exit;
    }

    atlogger_log("3d-get-public-atkey", ATLOGGER_LOGGING_LEVEL_INFO, "value: \"%s\"\n", value);

    exit_code = 0;
exit:
{
    free(atserver_host);
    atclient_atkeys_free(&atkeys);
    atclient_free(&atclient);
    atclient_atkey_free(&my_public_atkey);
    free(value);
    return exit_code;
}
}
```

### Get Self atKey

#### 1. Create a Self atKey

```c
atclient_atkey my_self_atkey;
atclient_atkey_init(&my_self_atkey);

const char *atkey_key = "phone";
const char *atkey_shared_by = "@jeremy_0";
const char *atkey_namespace = "c_demos";

if (atclient_atkey_create_self_key(&my_self_atkey, atkey_key, atkey_shared_by, atkey_namespace) != 0)
{
    // an error occurred
}
```

#### 2. Call \`atclient\_get\_self\_key\`

We will create a variable named `value` and pass the address to it in our `atclient_get_self_key` function call. The function will allocate memory for us and populate that variable for us.&#x20;

We will pass `NULL` into the request\_options parameter because we want to use the default options for now.

This function returns an `int` for error handling, in which a non-zero exit code indicates an error.

```c
char *value = NULL;
if (atclient_get_self_key(&atclient, &my_self_atkey, &value, NULL) != 0)
{
    // an error occurred
}
```

#### 3. Free \`value\`

Once we are done with the value itself, we should free it to avoid any memory leaks.

```c
free(value);
```

#### Example Application

```c
#include <atclient/atclient.h>
#include <atclient/atclient_utils.h>
#include <atclient/constants.h>
#include <atlogger/atlogger.h>
#include <stdlib.h>

#define ATSIGN "@soccer99"

int main()
{
    int exit_code = -1;

    atlogger_set_logging_level(ATLOGGER_LOGGING_LEVEL_DEBUG);

    char *atserver_host = NULL;
    int atserver_port = 0;

    atclient_atkeys atkeys;
    atclient_atkeys_init(&atkeys);

    atclient atclient;
    atclient_init(&atclient);

    atclient_atkey my_self_atkey;
    atclient_atkey_init(&my_self_atkey);

    /*
     * Let's declare a null pointer which will later be allocated by our `atclient_get_public_key` function. It is our responsibility to free it once we are
     * done with it.
     */
    char *value = NULL;

    if ((exit_code = atclient_utils_find_atserver_address(ATCLIENT_ATDIRECTORY_PRODUCTION_HOST, ATCLIENT_ATDIRECTORY_PRODUCTION_PORT, ATSIGN, &atserver_host, &atserver_port)) != 0)
    {
        goto exit;
    }

    if ((exit_code = atclient_utils_populate_atkeys_from_homedir(&atkeys, ATSIGN)) != 0)
    {
        goto exit;
    }

    if ((exit_code = atclient_pkam_authenticate(&atclient, atserver_host, atserver_port, &atkeys, ATSIGN)) != 0)
    {
        goto exit;
    }

    const char *atkey_key = "phone";
    const char *atkey_shared_by = ATSIGN;
    const char *atkey_namespace = "c_demos";

    if ((exit_code = atclient_atkey_create_self_key(&my_self_atkey, atkey_key, atkey_shared_by, atkey_namespace)) != 0)
    {
        goto exit;
    }

    /*
     * `atclient_get_self_key` will allocate memory for the `value` pointer. It is our responsibility to free it once we are done with it.
     * We will pass `NULL` into the request_options argument for now and just use the default request options.
     */
    if ((exit_code = atclient_get_self_key(&atclient, &my_self_atkey, &value, NULL)) != 0)
    {
        goto exit;
    }

    atlogger_log("3E-get-self-atkey", ATLOGGER_LOGGING_LEVEL_INFO, "value: \"%s\"\n", value);

    exit_code = 0;
exit:
{
    free(atserver_host);
    atclient_atkeys_free(&atkeys);
    atclient_free(&atclient);
    atclient_atkey_free(&my_self_atkey);
    free(value);
    return exit_code;
}
}
```

### Get Shared atKey

#### 1. Create a Shared atKey

```c
atclient_atkey my_shared_atkey;
atclient_atkey_init(&my_shared_atkey);

const char *atkey_key = "phone";
const char *atkey_shared_by = "@jeremy_0";
const char *atkey_shared_with = "@soccer0";
const char *atkey_namespace = "c_demos";
if (atclient_atkey_create_shared_key(&my_shared_atkey, atkey_key, atkey_shared_by, atkey_shared_with, atkey_namespace) != 0)
{
    // an error occurred
}
```

#### 2. Call \`atclient\_get\_shared\_key\`

We will create a variable named `value` and pass the address to it in our `atclient_get_shared_key` function call. The function will allocate memory for us and populate that variable for us.&#x20;

We will pass `NULL` into the request\_options parameter because we want to use the default options for now.

This function returns an `int` for error handling, in which a non-zero exit code indicates an error.

```c
char *value = NULL;
if (atclient_get_shared_key(&atclient, &my_self_atkey, &value, NULL) != 0)
{
    // an error occurred
}
```

#### 3. Free \`value\`

Once we are done with the value itself, we should free it to avoid any memory leaks.

```c
free(value)
```

#### Example Application

```c
#include <atclient/atclient.h>
#include <atclient/atclient_utils.h>
#include <atclient/constants.h>
#include <atlogger/atlogger.h>
#include <stdlib.h>

#define ATSIGN "@soccer99"

int main()
{
    int exit_code = -1;

    atlogger_set_logging_level(ATLOGGER_LOGGING_LEVEL_DEBUG);

    char *atserver_host = NULL;
    int atserver_port = 0;

    atclient_atkeys atkeys;
    atclient_atkeys_init(&atkeys);

    atclient atclient;
    atclient_init(&atclient);

    atclient_atkey my_shared_atkey;
    atclient_atkey_init(&my_shared_atkey);

    /*
     * Let's declare a null pointer which will later be allocated by our `atclient_get_public_key` function. It is our responsibility to free it once we are
     * done with it.
     */
    char *value = NULL;

    if ((exit_code = atclient_utils_find_atserver_address(ATCLIENT_ATDIRECTORY_PRODUCTION_HOST, ATCLIENT_ATDIRECTORY_PRODUCTION_PORT, ATSIGN, &atserver_host, &atserver_port)) != 0)
    {
        goto exit;
    }

    if ((exit_code = atclient_utils_populate_atkeys_from_homedir(&atkeys, ATSIGN)) != 0)
    {
        goto exit;
    }

    if ((exit_code = atclient_pkam_authenticate(&atclient, atserver_host, atserver_port, &atkeys, ATSIGN)) != 0)
    {
        goto exit;
    }

    const char *atkey_key = "phone";
    const char *atkey_shared_by = ATSIGN;
    const char *atkey_shared_with = "@soccer0";
    const char *atkey_namespace = "c_demos";

    if ((exit_code = atclient_atkey_create_shared_key(&my_shared_atkey, atkey_key, atkey_shared_by, atkey_shared_with, atkey_namespace)) != 0)
    {
        goto exit;
    }

    /*
     * `atclient_get_self_key` will allocate memory for the `value` pointer. It is our responsibility to free it once we are done with it.
     * We will pass `NULL` into the request_options argument for now and just use the default request options.
     */
    if ((exit_code = atclient_get_shared_key(&atclient, &my_shared_atkey, &value, NULL)) != 0)
    {
        goto exit;
    }

    atlogger_log("3E-get-self-atkey", ATLOGGER_LOGGING_LEVEL_INFO, "value: \"%s\"\n", value);

    exit_code = 0;
exit:
{
    free(atserver_host);
    atclient_atkeys_free(&atkeys);
    atclient_free(&atclient);
    atclient_atkey_free(&my_shared_atkey);
    free(value);
    return exit_code;
}
}
```

### Delete an atKey

#### 1. Create an atKey

First step is to create the atKey that you wish to delete. This can be of any atKey type (public, self, or shared). What is important to note is that you can only delete an atKey that you own (which means that the authenticated Atsign is the same as the shared\_by Atsign). This should be obvious because you can only delete atKeys that have once been created by you. Only the rightful owners of the atKey that was created can delete it.

For the sake of this demo, we will create a Shared atKey.

```c
atclient_atkey my_shared_atkey;
atclient_atkey_init(&my_shared_atkey);

const char *atkey_key = "phone";
const char *atkey_shared_by = "@jeremy_0";
const char *atkey_shared_with = "@soccer0";
const char *atkey_namespace = "c_demos";
if (atclient_atkey_create_shared_key(&my_shared_atkey, atkey_key, atkey_shared_by, atkey_shared_with, atkey_namespace) != 0)
{
    // an error occurred
}
```

#### 2. Call \`atclient\_delete\`

`atclient_delete` will delete the atKey from your atServer.&#x20;

We will pass `NULL` to the `request_options` and `commit_id` parameter because we want to use the default options for now and we do not care about the commit\_id. You can receive the commit\_id if you would like by passing an int pointer.

It is pointless to set any metadata to the atKey when deleting. Metadata is only useful when getting (you can read any metadata that the key possesses) and putting (you can modify the atKey's behavior). When you are deleting, no metadata is used.

This functions returns an `int` for error handling. A non-zero exit code indicates an error.

```c
if (atclient_delete(&atclient, &my_shared_atkey, NULL, NULL) != 0) {
    // an error occurred
}
```

#### Example Application

```c
#include <atclient/atclient.h>
#include <atclient/atclient_utils.h>
#include <atclient/constants.h>
#include <atlogger/atlogger.h>
#include <stdlib.h>

#define ATSIGN "@jeremy_0"

int main()
{
    int exit_code = -1;

    atlogger_set_logging_level(ATLOGGER_LOGGING_LEVEL_DEBUG);

    char *atserver_host = NULL;
    int atserver_port = 0;

    atclient_atkeys atkeys;
    atclient_atkeys_init(&atkeys);

    atclient atclient;
    atclient_init(&atclient);

    atclient_atkey my_shared_atkey;
    atclient_atkey_init(&my_shared_atkey);

    if ((exit_code = atclient_utils_find_atserver_address(ATCLIENT_ATDIRECTORY_PRODUCTION_HOST, ATCLIENT_ATDIRECTORY_PRODUCTION_PORT, ATSIGN, &atserver_host, &atserver_port)) != 0)
    {
        goto exit;
    }

    if ((exit_code = atclient_utils_populate_atkeys_from_homedir(&atkeys, ATSIGN)) != 0)
    {
        goto exit;
    }

    if ((exit_code = atclient_pkam_authenticate(&atclient, atserver_host, atserver_port, &atkeys, ATSIGN)) != 0)
    {
        goto exit;
    }

    const char *atkey_key = "phone";
    const char *atkey_shared_by = ATSIGN;
    const char *atkey_shared_with = "@soccer0";
    const char *atkey_namespace = "c_demos";

    if ((exit_code = atclient_atkey_create_shared_key(&my_shared_atkey, atkey_key, atkey_shared_by, atkey_shared_with, atkey_namespace)) != 0)
    {
        goto exit;
    }

    /*
     * `atclient_delete` will delete this shared atkey from our atServer. It is important to note that only the `shared_by` Atsign can delete the shared atKey
     * When deleting, the `shared_by` Atsign should always be the authenticated Atsign in the `atclient` object.
     * We will pass `NULL` to the request_options and commit_id parameters because we want to use the default request_options and we don't care about the
     * commit_id we get back.
     */
    if ((exit_code = atclient_delete(&atclient, &my_shared_atkey, NULL, NULL) != 0)) {
        goto exit;
    }

    exit_code = 0;
exit:
{
    free(atserver_host);
    atclient_atkeys_free(&atkeys);
    atclient_free(&atclient);
    atclient_atkey_free(&my_shared_atkey);
    return exit_code;
}
}
```

### Request Options

{% hint style="info" %}
Under Construction
{% endhint %}
{% endtab %}
{% endtabs %}





[^1]: This has been deprecated, and will be ignored.

[^2]: If metaData.isBinary is true, then this will be a List\<int>.

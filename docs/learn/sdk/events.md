---
description: How to send and receive real-time messages
---

# Notifications

{% tabs %}
{% tab title="Flutter / Dart" %}
In Dart, the AtClient is stored within the AtClientManager. Once an Atign has been [onboarded](onboarding.md), you will be able to access the AtClientManager for its associated Atsign.

### AtClientManager

AtClientManager is a [singleton](https://en.wikipedia.org/wiki/Singleton_pattern) model. When `AtClientManager.getInstance()` is called, it will get the AtClientManager instance for the last onboarded atSign.

```dart
AtClientManager atClientManager = AtClientManager.getInstance();
```

{% hint style="info" %}
If you need simultaneous access to multiple atClients, you need to create a new [isolate](https://dart.dev/language/concurrency#how-isolates-work) for each additional atClient, and onboard its atSign within the isolate.

An example of this pattern can be found in [at\_daemon\_server](https://github.com/atsign-foundation/at_services/tree/trunk/packages/at_daemon_server/lib/src/server).
{% endhint %}

### AtClient

As previously mentioned, the AtClientManager stores the actual AtClient itself. You can retrieve the `AtClient` by calling `atClientManager.atClient`.

```dart
AtClient atClient = atClientManager.atClient;
```

### Monitor

You can subscribe to a stream of notifications like this:

```dart
AtClient atClient = AtClientManager.getInstance().atClient;
atClient.notificationService
  .subscribe(regex: 'message.$nameSpace@', shouldDecrypt: true)
  .listen((notification) {
      print("Got a message: ${notification.value}");
  });
```

### Notify

You can send (a.k.a. publish) a notification like this:

```dart
AtKey messageKey = (AtKey.shared('message', nameSpace)
    ..sharedWith('@bob')).build();
String message = "Hi bob, how are you?";
await atClient.notificationService.notify(
  NotificationParams.forUpdate(messageKey, value: message),
);
```
{% endtab %}

{% tab title="C" %}
## C

You can find the full code of this example on our [GitHub](https://github.com/atsign-foundation/at_demos/tree/trunk/demos/get_started_c/4a-monitor#4a---monitor).

### Table of Contents

* [#introduction](events.md#introduction "mention")
* [#monitor](events.md#monitor "mention")
  * [#id-1.-include-monitor.h](events.md#id-1.-include-monitor.h "mention")
  * [#id-2.-create-a-monitor-context](events.md#id-2.-create-a-monitor-context "mention")
  * [#id-3.-call-atclient\_monitor\_pkam\_authenticate](events.md#id-3.-call-atclient_monitor_pkam_authenticate "mention")
  * [#id-4.-call-atclient\_monitor\_start](events.md#id-4.-call-atclient_monitor_start "mention")
  * [#id-5.-call-atclient\_monitor\_read](events.md#id-5.-call-atclient_monitor_read "mention")
  * [#id-6.-free-everything](events.md#id-6.-free-everything "mention")
  * [#example-application](events.md#example-application "mention")
* [#notify](events.md#notify "mention")
  * [#id-1.-include-notify.h](events.md#id-1.-include-notify.h "mention")
  * [#id-2.-create-a-shared-atkey](events.md#id-2.-create-a-shared-atkey "mention")
  * [#id-3.-set-up-notify-params](events.md#id-3.-set-up-notify-params "mention")
  * [#id-4.-call-atclient\_notify](events.md#id-4.-call-atclient_notify "mention")
  * [#id-5.-free-everything](events.md#id-5.-free-everything "mention")
  * [#example-application-1](events.md#example-application-1 "mention")

### Introduction

Events is how we send and receive real-time messages in the atPlatform Protocol. The client SDK assists in using the atPlatform Protocol simply and handles all the complex encryption stuff for you.

* `notify` is how we send a real-time message to an Atsign
* `monitor` is how we listen for real-time messages

### Monitor

#### 1. Include \`monitor.h\`

```c
#include <atclient/monitor.h>
```

#### 2. Create a Monitor Context

First, we need to create a separate monitor connection to our atServer. This is similar to creating a atclient context for conducting CRUD operations, but it is important to make a completely separate one so that connection can be clear of any noise and used purely for monitoring for notifications.&#x20;

```css
atclient monitor_client;
atclient_init(&monitor_client);
```

#### 3. Call \`atclient\_monitor\_pkam\_authenticate\`

This is similar to the `atclient_pkam_authenticate` that you know and love. It should use the same `atserver_host`, `atserver_port`, and `atclient_atkeys` in your original atclient context because we are connecting to the same atServer that can be authenticated using the same set of cryptographic atKeys.

This function will return an `int` for error handling in which a non-zero exit code indicates an error.

```
if (atclient_monitor_pkam_authenticate(
  &monitor_client, 
  atserver_host,
  atserver_port,
  &atkeys,
  "@jeremy_0") != 0) {
    // an error occurred
}
```

#### 4. Call \`atclient\_monitor\_start\`

This will begin the monitoring process of the atclient context that you pass. Since we made a completely different connection, this context will be purely for listening for any notifications.

In this example, we pass `.*` for the regex to receive all incoming notifications.

This function will return an `int` for error handling in which a non-zero exit code indicates an error.

```c
if(atclient_monitor_start(&monitor_client, ".*") != 0) {
    // an error occurred
}
```

#### 5. Call \`atclient\_monitor\_read\`

Calling `atclient_monitor_read` will attempt to read the network buffer for any notifications.&#x20;

It may be useful to put this in a loop to constantly be checking for any notifications that may have arrived to your atServer. You will commonly get error code `-26624` because most times, there will be nothing to read!&#x20;

```c
atclient_monitor_response response;
atclient_monitor_response_init(&response);

int exit_code = atclient_monitor_read(&monitor_client, &atclient1, &response, NULL);

// do things with the `response` variable
if (response.type == ATCLIENT_MONITOR_MESSAGE_TYPE_NOTIFICATION) {
  // we should check if this value is populated first, so that we know it is safe for reading.
  if (atclient_atnotification_is_decrypted_value_initialized(&response.notification)) {
    atlogger_log(TAG, ATLOGGER_LOGGING_LEVEL_INFO, "Received notification: %s\n", response.notification.decrypted_value);
  }
}

atclient_monitor_response_free(&response);
```

See the [#example-application-1](events.md#example-application-1 "mention") on how to read the different kinds of values from the `atclient_monitor_response` struct.

#### 6. Free everything

Don't forget to free everything at the end of your application's life time

```c
atclient_free(&monitor_client);
```

Also free the `atclient_monitor_response` struct if you haven't already.

```c
atclient_monitor_response_free(&response);
```

#### Example Application

You can find the full code of this example on our [GitHub](https://github.com/atsign-foundation/at_demos/tree/trunk/demos/get_started_c/4a-monitor#4a---monitor).

```c
#include <atclient/atclient.h>
#include <atclient/atclient_utils.h>
#include <atclient/constants.h>
#include <atclient/monitor.h>
#include <atlogger/atlogger.h>
#include <stdlib.h>

#define TAG "4a-monitor"

#define ATSIGN "@soccer99"

int main()
{
    int exit_code = -1;

    atlogger_set_logging_level(ATLOGGER_LOGGING_LEVEL_DEBUG);

    char *atserver_host = NULL;
    int atserver_port = 0;

    atclient_atkeys atkeys;
    atclient_atkeys_init(&atkeys);

    atclient atclient1;
    atclient_init(&atclient1);

    atclient monitor_client;
    atclient_init(&monitor_client);

    if ((exit_code = atclient_utils_find_atserver_address(ATCLIENT_ATDIRECTORY_PRODUCTION_HOST, ATCLIENT_ATDIRECTORY_PRODUCTION_PORT, ATSIGN, &atserver_host, &atserver_port)) != 0)
    {
        goto exit;
    }

    if ((exit_code = atclient_utils_populate_atkeys_from_homedir(&atkeys, ATSIGN)) != 0)
    {
        goto exit;
    }

    if ((exit_code = atclient_pkam_authenticate(&atclient1, atserver_host, atserver_port, &atkeys, ATSIGN)) != 0)
    {
        goto exit;
    }

    /*
     * Our monitor connection is just an ordinary atclient context, but we will use a completely separate atclient context to manage it.
     * And instead of calling the atclient functions on our atclient monitor context, we will use monitor functions on it instead.
     * In this case, we are using `atclient_monitor_pkam_authenticate` instead of `atclient_pkam_authenticate` even though it's essentially the same function
     * signature (aside from the function name)
     */
    if ((exit_code = atclient_monitor_pkam_authenticate(&monitor_client, atserver_host, atserver_port, &atkeys, ATSIGN)) != 0)
    {
        goto exit;
    }

    if((exit_code = atclient_monitor_start(&monitor_client, ".*")) != 0) {
        goto exit;
    }

    while (true)
    {
        atclient_monitor_response response;
        atclient_monitor_response_init(&response);

        exit_code = atclient_monitor_read(&monitor_client, &atclient1, &response, NULL);

        if (exit_code != 0)
        {
            if (response.type == ATCLIENT_MONITOR_ERROR_READ)
            {
                atlogger_log(TAG, ATLOGGER_LOGGING_LEVEL_ERROR, "Error reading from monitor: %d\n", response.error_read.error_code);
            }
            else if (response.type == ATCLIENT_MONITOR_ERROR_PARSE_NOTIFICATION)
            {
                atlogger_log(TAG, ATLOGGER_LOGGING_LEVEL_ERROR, "Parse notification from monitor: %d\n", exit_code);
            }
            else if (response.type == ATCLIENT_MONITOR_ERROR_DECRYPT_NOTIFICATION)
            {
                atlogger_log(TAG, ATLOGGER_LOGGING_LEVEL_ERROR, "Error response from monitor: %d\n", exit_code);
            } else {
                atlogger_log(TAG, ATLOGGER_LOGGING_LEVEL_ERROR, "Unknown error: %d\n", exit_code);
            }
        }
        else
        {

            if (response.type == ATCLIENT_MONITOR_MESSAGE_TYPE_NOTIFICATION)
            {
                // we should check if this value is populated first, so that we know it is safe for reading.
                if (atclient_atnotification_is_decrypted_value_initialized(&response.notification))
                {
                    atlogger_log(TAG, ATLOGGER_LOGGING_LEVEL_INFO, "Received notification: %s\n", response.notification.decrypted_value);
                }
            }
            else if (response.type == ATCLIENT_MONITOR_MESSAGE_TYPE_DATA_RESPONSE)
            {
                atlogger_log(TAG, ATLOGGER_LOGGING_LEVEL_INFO, "Received data: %s\n", response.data_response); // most times, this is a `data:ok` from a heart beat
            }
            else if (response.type == ATCLIENT_MONITOR_MESSAGE_TYPE_ERROR_RESPONSE)
            {
                atlogger_log(TAG, ATLOGGER_LOGGING_LEVEL_ERROR, "Received error: %s\n", response.error_response);
            }
            else if (response.type == ATCLIENT_MONITOR_MESSAGE_TYPE_NONE)
            {
                atlogger_log(TAG, ATLOGGER_LOGGING_LEVEL_INFO, "No message received\n");
            } else {
                atlogger_log(TAG, ATLOGGER_LOGGING_LEVEL_ERROR, "Unknown message type: %d\n", response.type);
            }
        }

        atclient_monitor_response_free(&response);
    }

    exit_code = 0;
exit:
{
    free(atserver_host);
    atclient_atkeys_free(&atkeys);
    atclient_free(&atclient1);
    atclient_free(&monitor_client);
    return exit_code;
}
}

```

### Notify

#### 1. Include \`notify.h\`

```c
#include <atclient/notify.h>
```

#### 2. Create a Shared atKey

Since we are notifying another Atsign, we have to set up a Shared atKey.&#x20;

This process should be familiar to you if you have already gone through [crud-operations.md](crud-operations.md "mention").

```c
atclient_atkey atkey;
atclient_atkey_init(&atkey);

if(atclient_atkey_create_shared_key(&atkey, "phone", "@jeremy_0", "@soccer99", "c_demos") != 0) {
    // an error occurred
}
```

#### 3. Set up Notify Params

Next, we will set up the parameters before sending the notification. This will essentially hold the instructions on how to send the notification.

The `atKey` and `value` are the two fields that are mandatory in order to send a notification. If you forget to set these two fields, then you will not be allowed to send a notification because the instructions of sending a notification are incomplete.

```c
atclient_notify_params notify_params;
atclient_notify_params_init(&notify_params);

if(atclient_notify_params_set_atkey(&notify_params, &atkey) != 0) {
  // an error occurred
}

const char *value = "Hello! This is a notification!";
if(atclient_notify_params_set_value(&notify_params, value) != 0) {
  // an error occurred
}
```

#### 4. Call \`atclient\_notify\`

Simply call `atclient_notify` .This will send a notification to the `shared_with` Atsign that you specified when creating the Shared atKey which was done in this [step](events.md#id-2.-create-a-shared-atkey).

We will pass `NULL` into the `commit_id` parameter because we don't reallty care about the commt id for now. You can receive it by passing a `int *` if you would like.

```c
if(atclient_notify(&atclient1, &notify_params, NULL) != 0) {
  // an error occurred
}
```

#### 5. Free everything

Don't forget to free everything

```c
atclient_atkey_free(&atkey);
atclient_notify_params_free(&notify_params);
```

#### Example Application

You can find the full code of this example on our [GitHub](https://github.com/atsign-foundation/at_demos/tree/trunk/demos/get_started_c/4b-notify#4b---notify).

```c
#include <atclient/atclient.h>
#include <atclient/atclient_utils.h>
#include <atclient/constants.h>
#include <atclient/notify.h>
#include <atlogger/atlogger.h>
#include <stdlib.h>

#define ATSIGN "@soccer0"

int main()
{
    int exit_code = -1;

    atlogger_set_logging_level(ATLOGGER_LOGGING_LEVEL_DEBUG);

    char *atserver_host = NULL;
    int atserver_port = 0;

    atclient_atkeys atkeys;
    atclient_atkeys_init(&atkeys);

    atclient atclient1;
    atclient_init(&atclient1);

    atclient_atkey atkey;
    atclient_atkey_init(&atkey);

    atclient_notify_params notify_params;
    atclient_notify_params_init(&notify_params);

    if ((exit_code = atclient_utils_find_atserver_address(ATCLIENT_ATDIRECTORY_PRODUCTION_HOST, ATCLIENT_ATDIRECTORY_PRODUCTION_PORT, ATSIGN, &atserver_host, &atserver_port)) != 0)
    {
        goto exit;
    }

    if ((exit_code = atclient_utils_populate_atkeys_from_homedir(&atkeys, ATSIGN)) != 0)
    {
        goto exit;
    }

    if ((exit_code = atclient_pkam_authenticate(&atclient1, atserver_host, atserver_port, &atkeys, ATSIGN)) != 0)
    {
        goto exit;
    }

    /*
     * 1a. Set up atkey in our notify_params struct
     */
    if((exit_code = atclient_atkey_create_shared_key(&atkey, "phone", ATSIGN, "@soccer99", "c_demos")) != 0) {
        goto exit;
    }

    if((exit_code = atclient_notify_params_set_atkey(&notify_params, &atkey)) != 0) {
        goto exit;
    }

    /*
     * 1b. Set up value in our notify_params struct
     */
    const char *value = "Hello! This is a notification!";
    if((exit_code = atclient_notify_params_set_value(&notify_params, value)) != 0) {
        goto exit;
    }

    /*
     * 2. Send the notification
     * We will pass `NULL` for the commit id.
     */
    if((exit_code = atclient_notify(&atclient1, &notify_params, NULL)) != 0) {
        goto exit;
    }

    exit_code = 0;
exit:
{
    free(atserver_host);
    atclient_atkeys_free(&atkeys);
    atclient_free(&atclient1);
    atclient_atkey_free(&atkey);
    atclient_notify_params_free(&notify_params);
    return exit_code;
}
}
```
{% endtab %}
{% endtabs %}


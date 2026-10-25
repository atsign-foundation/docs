---
description: >-
  In this page, we learn about AtClient.NotificationService and how to
  notify/monitor
---

# Notify and monitor

Notify and monitor are  [Atsign Protocol](../../atsign-platform/atsign-protocol.md) concepts. `NotificationService` is the Dart client SDK implementation of that concept. `NotificationService` is how one Atsign tells another that something happened (real-time events). `notify` is like sending and `monitoring` is like reading.

Every `AtClient` has one at `atClient.notificationService`.  For basic use cases (real-time communication), all you really need to know are [#send](notify-and-monitor.md#send "mention") and [#subscribe](notify-and-monitor.md#subscribe "mention"). Have fun!

## How to notify

Use `atClient.notificationService.send` over `atClient.notificationService.notify`. The latter is old and soon to be deprecated.

### `.send`

* `to` is the Atsign you want to send the notification to
* `namespace` is the application namespace that helps with things like filtering and separating notifications from others. See [namespaces.md](../../atsign-platform/atserver/namespaces.md "mention").
* `body` is the payload of your message, typically a JSON encoded string.

```dart
final String notificationId = await atClient.notificationService.send(
  to: '@bob'.toAtsign(),
  namespace: 'updates.my_app',
  body: jsonEncode({'event': 'phone_changed'}),
);
```

#### Caching

By default, a notification is ephemeral. It is delivered and disappears after `expiration`. The recipient does not store it as a key-value entry. However, with `cacheAtRecipient: true` , turns the notification into a real-time event + an AtKey that can be read later using using  `atClient.get` . The sender defines `recipientCacheExpiration` which is the timstamp at which the cached copy will expire. Expiration means deletion of the cached copy.

```dart
await atClient.notificationService.send(
  to: '@bob'.toAtsign(),
  namespace: 'updates.my_app',
  body: jsonEncode(payload),
  cacheAtRecipient: true,
  recipientCacheExpiration: DateTime.now().add(const Duration(days: 1)),
);
```

`cacheAtRecipient: true` without `recipientCacheExpiration` throws `ArgumentError`. The cached copy is written with `ttr: -1` (cache once, never refresh) and a `ttl` derived from the expiry you gave, so a recipient reads it locally until that moment and then loses it. Learn more about `ttr` and `ttl` from [atrecords.md](../../atsign-platform/atserver/atrecords.md "mention").

### `.notify`&#x20;

This is kept around for backwards compatibility. Please use [#send](notify-and-monitor.md#send "mention")instead.

```dart
// notify: tied to a specific AtKey, with priority/strategy/dedup control
final AtKey key = AtKey()
  ..key = 'phone'
  ..sharedWith = '@bob';

final NotificationResult result = await atClient.notificationService.notify(
  NotificationParams.forUpdate(key, value: '+1 555 0100'),
);
```

### `.getStatus`

Check the delivery status of a notification by its ID. Returns a `NotificationResult` with `notificationStatusEnum` set to `NotificationStatusEnum.delivered` or `NotificationStatusEnum.undelivered`.

```dart
final NotificationResult status =
    await atClient.notificationService.getStatus(notificationId);

print(status.notificationStatusEnum); // delivered / undelivered
```

### `.fetch`

Retrieve the full `AtNotification` object for a given notification ID. Returns `NotificationStatus.expired` if the notification no longer exists or has expired.

```dart
final AtNotification n =
    await atClient.notificationService.fetch(notificationId);

print(n.value);
```

## How to monitor

There are various `atClient.notificationService.*` functions to be aware of.

### `.subscribe`

`atClient.notificationService.subscribe` is the most common way to listen for notifications. It returns a `Stream<AtNotification>` which you can interact with in many ways. Check out the official [Dart streams documentation](https://dart.dev/libraries/async/using-streams).

```dart
import 'package:at_client/at_client.dart';

final Stream<AtNotification> stream = atClient.notificationService.subscribe(
  regex: 'my_app',
  shouldDecrypt: true,
);

stream.listen((AtNotification n) {
  print('${n.from} sent ${n.key}: ${n.value}');
});
```

`regex` is an unanchored substring match against the notification key, same rule as everywhere else in the SDK. An unset `regex` receives everything sent to you.

`shouldDecrypt` defaults to **`false`** for backwards compatibility purposes. Most times you want to set this to `true` , or you get the ciphertext in `AtNotification.value`.

### `.subscribeFiltered`

This builds the regex for you from a `namespace`, and only receives notifications from `acceptedSenders` . Omit `acceptedSenders` to receive notifications from any Atsign.

```dart
final Stream<AtNotification> stream =
    atClient.notificationService.subscribeFiltered(
  namespace: 'my_app',
  acceptedSenders: {'@alice'.toAtsign(), '@carol'.toAtsign()},
);
```

### `.currentListenerStateStream`

Listen for updates in your monitor connection.

```dart
atClient.notificationService.currentListenerStateStream.listen((state) {
  print(state); // NotificationListenerState.listening / .notConnected
});

atClient.notificationService.stopListening(); // prints NotificationListenerState.notConnected in the event listener above
atClient.notificationService.startListening(); // prints NotificationListenerState.listening in the event listener above
```

### `.stopListening()`

Stops the monitor connection from listening

```dart
atClient.notificationService.stopListening();
```

### `.startListening`

Starts the monitor connection and listens for incoming notifications. This is turned on by default. Listeners reconnect automatically in case of network loss.

```dart
atClient.notificationService.startListening();
```

### `.listening`

This is similar to the example used in [#currentlistenerstatestream](notify-and-monitor.md#currentlistenerstatestream "mention"), but this way is more convenient.

```dart
// a convenience bool for checking if the current monitor connection is listening
print(atClient.notificationService.listening);
```

### `.lastReceipt`&#x20;

Check the last `DateTime` you received a notification in the monitor connection.

```
print(atClient.notificationService.lastReceipt); // DateTime? of last notification
```

## Errors

| Exception                | Cause                                                                                                    |
| ------------------------ | -------------------------------------------------------------------------------------------------------- |
| `AtKeyException`         | Invalid `NotificationParams.atKey.key`, or invalid metadata                                              |
| `InvalidAtSignException` | Malformed `sharedWith`/`sharedBy` on the key                                                             |
| `AtClientException`      | Encryption keys not found, `strategy: latest` used without a `notifier`, or your atServer is unreachable |

These surface inside `NotificationResult.atClientException` for the callback-based path, or are thrown directly when awaited synchronously.

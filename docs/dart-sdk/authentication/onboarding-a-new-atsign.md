---
description: >-
  Learn how to do the initial onboard for an Atsign in your Dart application
  using AtAuth.onboard
---

# Onboarding a new Atsign

## What is onboarding?

Onboarding is the one-time process of activating an Atsign. An "onboarded" or "activated" Atsign means the `.atKeys` file for the corresponding Atsign and atServer is generated. To onboard the same Atsign again, the Atsign needs to be [reset](../../atsign-platform/atsign/#resetting-your-atsign).

Every later authentication uses [authenticating-an-existing-atsign.md](authenticating-an-existing-atsign.md "mention"). If you are building an application that only uses and assumes already-onboarded Atsigns, then move onto the next section: [authenticating-an-existing-atsign.md](authenticating-an-existing-atsign.md "mention").

## Prerequisites

* A **registered, but not yet activated** Atsign.
* Its **CRAM Key** (also known as the CRAM secret/license key/activation secret)

## The core classes

| Class                  | Description                                                                                                             |
| ---------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| `AtAuth`               | Object that holds onboarding/authentication logic. Contains `AtAuth.onboard` which we will be covering in this section. |
| `AtOnboardingRequest`  | Represents a request to onboard an Atsign                                                                               |
| `AtOnboardingResponse` | Represents a response after `AtAuth.onboard` was executed                                                               |

## Onboarding flows

There are a ton of different ways to parameterize `AtAuth.onboard` either by manipulating `AtOnboardingRequest` or passing different values into the `AtAuth.onboard`'s function.

| Flow                                                                                     | Summary                                                                                                                                                  |
| ---------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [#the-minimal-flow](onboarding-a-new-atsign.md#the-minimal-flow "mention")               | Simple Atsign onboard, use all defaults                                                                                                                  |
| [#custom-keys-path](onboarding-a-new-atsign.md#custom-keys-path "mention")               | Customize the output `.atKeys` file path                                                                                                                 |
| [#custom-root-domain](onboarding-a-new-atsign.md#custom-root-domain "mention")           | Use a custom atDirectory `host:port`                                                                                                                     |
| [#password-protected-file](onboarding-a-new-atsign.md#password-protected-file "mention") | Save `.atKeys` file behind a password                                                                                                                    |
| [#custom-retry-timeout](onboarding-a-new-atsign.md#custom-retry-timeout "mention")       | Implement retry and timeout functionality to `AtAuth.onboard`                                                                                            |
| [#progress-listening](onboarding-a-new-atsign.md#progress-listening "mention")           | Listen for events during the `AtAuth.onboard` process                                                                                                    |
| [#in-memory-keys](onboarding-a-new-atsign.md#in-memory-keys "mention")                   | Save `.atKeys` to memory instead of to a file. Good for ephemeral enrollments. Not recommended for beginners.                                            |
| [#deferred-activation](onboarding-a-new-atsign.md#deferred-activation "mention")         | Do the first part of CRAM onboarding, then delay the creation of cryptographic keys. Good for Atsign developer debugging. Not recommended for beginners. |

### The minimal flow

Copy and paste the code below. Be sure to replace the values of the variables `atsign` and `cramKey` accordingly to the Atsign you registered and the CRAM key which can be fetched from the dashboard.

```dart
import 'dart:io';
import 'package:at_auth/at_auth.dart';
import 'package:at_commons/at_commons.dart';

const String atsign = '@alice'; // the Atsign you want to onboard
const String cramKey = 'abc123...'; // a copyable secret from my.atsign.com/dashboard
final AtAuth atAuth = AtAuth.create();
final AtOnboardingRequest request = AtOnboardingRequest(atsign);
try {
  final response = await atAuth.onboard(request, cramKey);
  if (!response.isSuccessful) exit(1);
} on AtException catch (_) {
  exit(1);
}
exit(0);
```

### Custom keys path

Specify a different output directory using `FileAtKeysIo`.

```dart
import 'package:at_auth/at_auth.dart';

const String atsign = '@alice'; // the Atsign you want to onboard
const String cramKey = 'abc123...'; // a copyable secret from my.atsign.com/dashboard

final AtAuth atAuth = AtAuth.create();
final AtOnboardingRequest request = AtOnboardingRequest(
  '@alice',
  atKeysIo: FileAtKeysIo(
    filePath: (atsign) => '/home/alice/.atsign/keys/${atsign}_key.atKeys',
  ),
);
final AtOnboardingResponse response = await atAuth.onboard(
  request,
  cramKey,
);
```

### Custom root domain

Specify a different atDirectory `host:port`

```dart
import 'package:at_auth/at_auth.dart';
import 'package:at_commons/at_commons.dart' show AtRootDomain;

const String atsign = '@alice';
const String cramKey = 'abc123...';

final AtAuth atAuth = AtAuth.create();
final AtOnboardingRequest request = AtOnboardingRequest(
  '@alice',
  rootDomain: AtRootDomain('root.atsign.org', 64), 
);

final AtOnboardingResponse response = await atAuth.onboard(request, cramKey);
```

### Password-protected file

You can implement password protected `.atKeys` files by passing a `passPhrase` to `FileAtKeysIo`

```dart
import 'package:at_auth/at_auth.dart';

const String atsign = '@alice';
const String cramKey = 'abc123...';

final AtAuth atAuth = AtAuth.create();
final AtOnboardingRequest request = AtOnboardingRequest(
  atsign,
  atKeysIo: FileAtKeysIo(passPhrase: 'a-strong-passphrase'), // encrypts the .atKeys file at rest
);
final AtOnboardingResponse response = await atAuth.onboard(request, cramKey);
```

### Custom retry/timeout

Use `retryOptions` in `AtOnobardingRequest` to offer a good user experience for your users in your Dart application.

```dart
import 'package:at_auth/at_auth.dart';

const String atsign = '@alice';
const String cramKey = 'abc123...';

final AtAuth atAuth = AtAuth.create();
final AtOnboardingRequest request = AtOnboardingRequest(
  atsign,
  retryOptions: RetryOptions(
    maxRetries: 10,
    retryDelay: Duration(seconds: 2),
    overallTimeout: Duration(minutes: 10), // widen for a slow provisioner; defaults to 5 minutes
  ),
);
final AtOnboardingResponse response = await atAuth.onboard(request, cramKey);
```

### Progress listening

Listen for events using `atAuth.progressStream.listen` .

```dart
import 'package:at_auth/at_auth.dart';

const String atsign = '@alice';
const String cramKey = 'abc123...';

final AtAuth atAuth = AtAuth.create();
atAuth.progressStream.listen((event) {
  print('${event.group}: ${event.msg}');
});

final AtOnboardingRequest request = AtOnboardingRequest(atsign);
final AtOnboardingResponse response = await atAuth.onboard(request, cramKey);
```

### In-memory keys

{% hint style="info" %}
This flow is not recommended for beginners. You may lose atKeys if not saved properly. This flow is really only useful for testing purposes.
{% endhint %}

The main use for `InMemoryAtKeysIo` is for saving the cryptographic  `.atKeys`  in memory.

```dart
import 'package:at_auth/at_auth.dart';

const String atsign = '@alice';
const String cramKey = 'abc123...';

final AtAuth atAuth = AtAuth.create();
final AtOnboardingRequest request = AtOnboardingRequest(
  atsign,
  atKeysIo: InMemoryAtKeysIo(), // keys live only for this process; nothing touches disk
);
final AtOnboardingResponse response = await atAuth.onboard(request, cramKey);
```

### Deferred activation

{% hint style="info" %}
This flow is not recommended for beginners. This flow is really only useful for developer purposes or if you want extra logging.\`
{% endhint %}

This flow is useful for when you want to add more logging or developer testing. This flow is generally not recommended because an adequate amount of logging already occurs in onboarding, but is available if need be. For progress listening and logging, it is recommended to use [#progress-listening](onboarding-a-new-atsign.md#progress-listening "mention") flow instead.

```dart
import 'package:at_auth/at_auth.dart';

const String atsign = '@alice';
const String cramKey = 'abc123...';

final AtAuth atAuth = AtAuth.create();
final AtOnboardingRequest request = AtOnboardingRequest(atsign);
final AtOnboardingResponse response = await atAuth.onboard(
  request,
  cramKey,
  autoCompleteActivation: false, // hold off publishing the encryption key and deleting the CRAM key
);

// ...later, once you're ready to finish activation...
await atAuth.completeActivation();
```

## Onboarding again

If you want to test your onboarding flow again with the same Atsign, you must reset your Atsign. Note that this will completely wipe your Atsign's atServer, all of its data, and also render your current set of `.atKeys` incapable of authenticating to its Atsign. See our section on [Resetting your Atsign](../../atsign-platform/atsign/atsign.md#resetting-your-atsign).

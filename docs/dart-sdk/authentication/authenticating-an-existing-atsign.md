---
description: >-
  Learn how to do the initial onboard for an Atsign in your Dart application
  using AtAuth.authenticate
---

# Authenticating an existing Atsign

## What is authentication?

Authentication is what you do every time an already-onboarded (activated) Atsign needs to connect to its atServer. Unlike onboarding, it does not generate new keys or touch the atServer's enrollment state, it reads the existing key material and proves ownership of the Atsign using PKAM.

This is the flow you write in almost every app, for almost every session. It never needs a CRAM key.

If the Atsign has never been activated, use [onboarding-a-new-atsign.md](onboarding-a-new-atsign.md "mention") instead.

## Prerequisites

* An already-onboarded (activated) Atsign
* Its `.atKeys` file

## The core classes

| Class          | Description                                                                        |
| -------------- | ---------------------------------------------------------------------------------- |
| AtAuth         | Object that holds onboarding/authentication logic. Contains `AtAuth.authenticate`. |
| AtAuthRequest  | Represents a request to authenticate an Atsign; passed to `AtAuth.authenticate`    |
| AtAuthResponse | Represents a response after `AtAuth.authenticate` was executed                     |
| AtAuthSession  | The auth→client hand-off object, populated on `response.session` on success        |

## Authentication flows

There are several ways to parameterize `AtAuth.authenticate`, either by manipulating `AtAuthRequest` or by using different `AtKeysIo` implementations.

| Flow                                                                                                                     | Summary                                                                                                                                                            |
| ------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| [#the-minimal-flow](authenticating-an-existing-atsign.md#the-minimal-flow "mention")                                     | Simple Atsign authenticate, use all defaults                                                                                                                       |
| [#custom-keys-path](authenticating-an-existing-atsign.md#custom-keys-path "mention")                                     | Read the `.atKeys` file from a non-default path                                                                                                                    |
| [#custom-root-domain](authenticating-an-existing-atsign.md#custom-root-domain "mention")                                 | Use a custom atDirectory `host:port`                                                                                                                               |
| [#password-protected-file](authenticating-an-existing-atsign.md#password-protected-file "mention")                       | Read a `.atKeys` file saved behind a password                                                                                                                      |
| [#custom-retry-timeout](authenticating-an-existing-atsign.md#custom-retry-timeout "mention")                             | Implement retry and timeout functionality to `AtAuth.authenticate`                                                                                                 |
| [#progress-listening](authenticating-an-existing-atsign.md#progress-listening "mention")                                 | Listen for events during the `AtAuth.authenticate` process                                                                                                         |
| [#in-memory-keys](authenticating-an-existing-atsign.md#in-memory-keys "mention")                                         | Authenticate using `AtKeys` already held in memory, instead of a file. Good for ephemeral enrollments carried over from onboarding. Not recommended for beginners. |
| [#checking-the-atserver-status-first](authenticating-an-existing-atsign.md#checking-the-atserver-status-first "mention") | Validate the atServer is reachable and already activated before attempting authentication                                                                          |

### The minimal flow

Copy and paste the code below. Be sure to replace the value of the variable `atsign` accordingly to the Atsign you already onboarded.

```dart
import 'dart:io';
import 'package:at_auth/at_auth.dart';
import 'package:at_commons/at_commons.dart';

const String atsign = '@alice'; // the Atsign you want to authenticate
final AtAuth atAuth = AtAuth.create();
final AtAuthRequest request = AtAuthRequest(
  atsign,
  atKeysIo: FileAtKeysIo(), // reads ~/.atsign/keys/@alice_key.atKeys by default
);
try {
  final response = await atAuth.authenticate(request);
  if (!response.isSuccessful) exit(1);
} on AtAuthenticationException catch (_) {
  exit(1);
}
exit(0);
```

### Custom keys path

Specify a different input directory using `FileAtKeysIo`.

```dart
import 'package:at_auth/at_auth.dart';

const String atsign = '@alice'; // the Atsign you want to authenticate

final AtAuth atAuth = AtAuth.create();
final AtAuthRequest request = AtAuthRequest(
  atsign,
  atKeysIo: FileAtKeysIo(
    filePath: (atsign) => '/home/alice/.atsign/keys/${atsign}_key.atKeys',
  ),
);
final AtAuthResponse response = await atAuth.authenticate(request);
```

### Custom root domain

Specify a different atDirectory `host:port`.

```dart
import 'package:at_auth/at_auth.dart';
import 'package:at_commons/at_commons.dart' show AtRootDomain;

const String atsign = '@alice';

final AtAuth atAuth = AtAuth.create();
final AtAuthRequest request = AtAuthRequest(
  atsign,
  atKeysIo: FileAtKeysIo(),
  rootDomain: AtRootDomain('root.atsign.org', 64),
);

final AtAuthResponse response = await atAuth.authenticate(request);
```

### Password-protected file

If the `.atKeys` file was saved with a `passPhrase` during onboarding, pass the same `passPhrase` to `FileAtKeysIo` to read it back.

```dart
import 'package:at_auth/at_auth.dart';

const String atsign = '@alice';

final AtAuth atAuth = AtAuth.create();
final AtAuthRequest request = AtAuthRequest(
  atsign,
  atKeysIo: FileAtKeysIo(passPhrase: 'a-strong-passphrase'), // decrypts the .atKeys file at rest
);
final AtAuthResponse response = await atAuth.authenticate(request);
```

### Custom retry/timeout

Use `retryOptions` in `AtAuthRequest` to offer a good user experience for your users in your Dart application.

```dart
import 'package:at_auth/at_auth.dart';

const String atsign = '@alice';

final AtAuth atAuth = AtAuth.create();
final AtAuthRequest request = AtAuthRequest(
  atsign,
  atKeysIo: FileAtKeysIo(),
  retryOptions: RetryOptions(
    maxRetries: 10,
    retryDelay: Duration(seconds: 2),
    overallTimeout: Duration(seconds: 60), // widen for a flaky network; defaults to 30 seconds
  ),
);
final AtAuthResponse response = await atAuth.authenticate(request);
```

### Progress listening

Listen for events using `atAuth.progressStream.listen`.

```dart
import 'package:at_auth/at_auth.dart';

const String atsign = '@alice';

final AtAuth atAuth = AtAuth.create();
atAuth.progressStream.listen((event) {
  print('${event.group}: ${event.msg}');
});

final AtAuthRequest request = AtAuthRequest(atsign, atKeysIo: FileAtKeysIo());
final AtAuthResponse response = await atAuth.authenticate(request);
```

### In-memory keys

{% hint style="info" %}
This flow is not recommended for beginners. It is really only useful when key material already lives in memory. For example, right after onboarding in the same process, or when keys arrive over a channel you control rather than from disk.
{% endhint %}

`InMemoryAtKeysIo` keeps `AtKeys` in a process-local map instead of writing them to disk. The same instance can be reused across an onboard and a later authenticate in the same process:

```dart
import 'package:at_auth/at_auth.dart';

const String atsign = '@alice';
const String cramKey = 'abc123...';

final AtAuth atAuth = AtAuth.create();
final InMemoryAtKeysIo atKeysIo = InMemoryAtKeysIo();

// onboard() populates atKeysIo in memory; nothing touches disk
await atAuth.onboard(
  AtOnboardingRequest(atsign, atKeysIo: atKeysIo),
  cramKey,
);

// later in the same process, authenticate reuses the same in-memory keys
final AtAuthRequest request = AtAuthRequest(atsign, atKeysIo: atKeysIo);
final AtAuthResponse response = await atAuth.authenticate(request);
```

`InMemoryAtKeysIo.read` throws `AtKeysNotInMemoryException` if nothing has been written for the Atsign yet, so the instance must be populated (by `onboard()`, or by calling `atKeysIo.write(atsign, existingAtKeys)` yourself) before it is used to authenticate.

This flow is not recommended because it is best to save keys in a hard disk to avoid catastrophic failures: losing cryptographic keys.

### Checking the atServer status first

`authenticate()` already calls this internally, but calling it yourself lets you tell "this Atsign was never activated" apart from "the keys are wrong" before spending a PKAM round trip. This is useful for a clearer error message in a UI.

```dart
import 'package:at_auth/at_auth.dart';

const String atsign = '@alice';

final AtAuth atAuth = AtAuth.create();
final AtAuthRequest request = AtAuthRequest(atsign, atKeysIo: FileAtKeysIo());

try {
  await atAuth.validateAtServer(request);
} on AtException catch (e) {
  print('Cannot authenticate @alice yet: $e');
  return;
}

final AtAuthResponse response = await atAuth.authenticate(request);
```

## After authenticating

Check `isSuccessful` before touching `session` .

```dart
if (!response.isSuccessful) {
  exit(1);
}
// now `response.session!` is safe to use!
await AtClientManager.getInstance().fromAuthSession(response.session!, preference);
```

Now that we have authenticated, the next step is to obtain an AtClient instance in the next [step](create-an-atclient-instance.md).

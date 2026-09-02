---
description: >-
  Two programs: one to onboard @alice (run once) and one to use the SDK (run any
  time after).
---

# Quickstart

See [prerequisites.md](prerequisites.md "mention") before you start.

## 1. Create the project

```shellscript
dart create quickstart
cd quickstart
dart pub add at_client at_auth at_commons
```

Your `pubspec.yaml` file should look something like this:

```yaml
name: quickstart
description: A sample command-line application.
version: 1.0.0

environment:
  sdk: ^3.13.0

dependencies:
  at_auth: ^3.3.0
  at_client: ^3.14.0
  at_commons: ^5.15.0
  path: ^1.9.0

dev_dependencies:
  lints: ^6.0.0
  test: ^1.25.6
```

## 2. Onboard (run once)

`bin/onboard.dart` - activates `@alice` by exchanging a one-time CRAM secret for permanent keys. This should only be performed once; after that, the `.atHeys` file exists, and this program cannot be run again. This is equivalent to using the [at\_activate](https://pub.dev/packages/at_onboarding_cli) tool to do the initial onboarding of your Atsign.

1. Write this content to `bin/onboard.dart`

```dart
import 'dart:io';

import 'package:at_auth/at_auth.dart';
import 'package:at_commons/at_commons.dart' show AtException, AtRootDomain;
import 'package:at_utils/at_utils.dart' show AtSignLogger;

const String myAtSign = '@alice';

void main() async {
  AtSignLogger.root_level = 'warning';

  stdout.write('Enter CRAM secret for $myAtSign: ');
  final String cramSecret = stdin.readLineSync()!.trim();

  final AtAuth atAuth = AtAuth.create();
  final AtOnboardingRequest request =
      AtOnboardingRequest(myAtSign, rootDomain: AtRootDomain.atsignDomain)
        ..appName = 'quickstart'
        ..deviceName = 'laptop';

  try {
    final response = await atAuth.onboard(request, cramSecret);
    if (!response.isSuccessful) exit(1);
  } on AtException catch (e) {
    stderr.writeln('Onboarding failed: ${e.message}');
    exit(1);
  }

  stdout.writeln(
    'Onboarded $myAtSign -> ~/.atsign/keys/${myAtSign}_key.atKeys',
  );
}
```

2. Change the `myAtSign` string to the Atsign you own

```dart
const String myAtSign = '@alice';
```

3. Run the program:

```shellscript
dart run bin/onboard.dart
```

Sample output

```
Enter CRAM secret for @alice: 03f68b...
Onboarded @alice -> ~/.atsign/keys/@alice_key.atKeys
```

This will write an `.atKeys` file to `~/.atsign/keys/` and onboard (also known as activate) your Atsign.

Ensure you backup your keys! Read our section on [atkeys.md](../../atsign-platform/atsign/atkeys.md "mention") to learn what this file really is.

If you want to reset your Atsign and run this step again, see our section on [Resetting your Atsign](../../atsign-platform/atsign/#resetting-your-atsign).

## 3. Write the quickstart program

`bin/quickstart.dart`  — authenticates with the `.atKeys` file from step 2, writes an encrypted record, shares it with `@bob`, and reads it back. You can run this program as many times as you like.

1. Copy and paste this code into `bin/quickstart.dart`

```dart
import 'dart:io';

import 'package:at_auth/at_auth.dart';
import 'package:at_client/at_client.dart';
import 'package:at_utils/at_utils.dart' show AtSignLogger;

const String myAtSign = '@alice';
const String appNamespace = 'quickstart';

void main() async {
  AtSignLogger.root_level = 'warning';

  final AtAuth atAuth = AtAuth.create();
  final AtAuthRequest authRequest = AtAuthRequest(
    myAtSign,
    atKeysIo: FileAtKeysIo(),
    rootDomain: AtRootDomain.atsignDomain,
  );

  final Directory storage = Directory.systemTemp.createTempSync('quickstart_');
  final AtClientPreference preference = AtClientPreference()
    ..namespace = appNamespace
    ..syncRegex = appNamespace
    ..hiveStoragePath = storage.path
    ..commitLogPath = storage.path;

  try {
    final authResponse = await atAuth.authenticate(authRequest);
    if (!authResponse.isSuccessful) exit(1);
    await AtClientManager.getInstance().setCurrentAtSign(
      myAtSign,
      appNamespace,
      preference,
      atChops: authResponse.atChops,
      atLookUp: authResponse.atLookUp,
    );
  } on AtException catch (e) {
    stderr.writeln('Authentication failed: ${e.message}');
    exit(1);
  }

  final AtClient atClient = AtClientManager.getInstance().atClient;
  final AtCollection<String> notes = await atClient.collection<String>(
    'notes.$appNamespace',
    const Duration(days: 7),
  );

  await notes.create(obj: 'Hello from the Dart SDK');
  await atClient.syncService.waitUntilCaughtUp(
    timeout: const Duration(seconds: 30),
  );

  for (final CItem<String> item in await notes.getItems()) {
    stdout.writeln('${item.id} (owner ${item.owner}): ${item.obj}');
  }

  exit(0);
}

```

2. Replace `@alice` with your Atsign.

```dart
const String myAtSign = '@alice';
```

3. Run the program

```shellscript
dart run bin/quickstart.dart
```

Sample output:

```
u1n5eh4g (owner @alice): Hello from the Dart SDK
```

## 4. Clean up

`bin/cleanup.dart` — a Dart program that will clean up the data left by Step 3 ( [#id-3.-write-the-quickstart-program](quickstart.md#id-3.-write-the-quickstart-program "mention") ).

1. Copy and paste this code into `bin/cleanup.dart`

```dart
import 'dart:io';

import 'package:at_auth/at_auth.dart';
import 'package:at_client/at_client.dart';
import 'package:at_utils/at_utils.dart' show AtSignLogger;

const String myAtSign = '@alice';
const String appNamespace = 'quickstart';

void main() async {
  AtSignLogger.root_level = 'warning';

  final AtAuth atAuth = AtAuth.create();
  final AtAuthRequest authRequest = AtAuthRequest(
    myAtSign,
    atKeysIo: FileAtKeysIo(),
    rootDomain: AtRootDomain.atsignDomain,
  );

  final Directory storage = Directory.systemTemp.createTempSync('cleanup_');
  final AtClientPreference preference = AtClientPreference()
    ..namespace = appNamespace
    ..syncRegex = appNamespace
    ..hiveStoragePath = storage.path
    ..commitLogPath = storage.path;

  try {
    final authResponse = await atAuth.authenticate(authRequest);
    if (!authResponse.isSuccessful) exit(1);
    await AtClientManager.getInstance().setCurrentAtSign(
      myAtSign,
      appNamespace,
      preference,
      atChops: authResponse.atChops,
      atLookUp: authResponse.atLookUp,
    );
  } on AtException catch (e) {
    stderr.writeln('Authentication failed: ${e.message}');
    exit(1);
  }

  final AtClient atClient = AtClientManager.getInstance().atClient;
  final AtCollection<String> notes = await atClient.collection<String>(
    'notes.$appNamespace',
    const Duration(days: 7),
  );

  await atClient.syncService.waitUntilCaughtUp(
    timeout: const Duration(seconds: 30),
  );

  final List<CItem<String>> items = await notes.getItems();
  for (final CItem<String> item in items) {
    await notes.delete(item);
  }

  stdout.writeln('Deleted ${items.length} item(s) from notes.$appNamespace');
  exit(0);
}
```

2. Change the `myAtSign` to the Atsign you activated last step.

```dart
const String myAtSign = '@alice';
```

3. Run the program

```shellscript
dart run bin/cleanup.dart
```

Sample output:

```
Deleted 2 item(s) from notes.quickstart
```

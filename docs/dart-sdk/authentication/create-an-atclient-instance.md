---
description: The goal of authentication is to create a ready-to-use AtClient instance.
---

# Create an AtClient instance

Now that you have properly instantiated an `AtAuth` instance and have either called `.onboard` or `.authenticate`, it is now time to obtain an AtClient instance

## What you need

* A successful `AtOnboardingResponse` or `AtAuthResponse` object, with `response.session` populated. See [onboarding-a-new-atsign.md](onboarding-a-new-atsign.md "mention") or [authenticating-an-existing-atsign.md](authenticating-an-existing-atsign.md "mention") respectively on how to get those and what they are.
* A local storage path to store AtClient data (into  `AtClientPreference.hiveStoragePath` )

## The minimal flow

### From an AtOnboardingResponse success

```dart
import 'dart:io';

import 'package:at_auth/at_auth.dart';
import 'package:at_client/at_client.dart';

void main() async {
  const String atSign = '@alice';
  const String cramKey = 'the-cram-key';
  const String storagePath = '/home/alice/.atsign/storage/hive';

  final AtAuth atAuth = AtAuth.create();
  final AtOnboardingRequest request = AtOnboardingRequest(
    atSign,
    rootDomain: AtRootDomain.atsignDomain,
  );

  final AtOnboardingResponse response = await atAuth.onboard(request, cramKey);
  if (!response.isSuccessful) {
    stderr.writeln('Onboarding failed');
    exit(1);
  }

  final AtClientPreference preference = AtClientPreference()
    ..hiveStoragePath = storagePath;

  final AtClientManager atClientManager = await AtClientManager.getInstance()
      .fromAuthSession(response.session!, preference);

  final AtClient atClient = atClientManager.atClient;
  stdout.writeln('AtClient ready for $atSign');
}
```

### From an AtAuthResponse success

```dart
import 'dart:io';

import 'package:at_auth/at_auth.dart';
import 'package:at_client/at_client.dart';

void main() async {
  const String atSign = '@alice';
  const String storagePath = '/home/alice/.atsign/storage/hive';

  final AtAuth atAuth = AtAuth.create();
  final AtAuthRequest request = AtAuthRequest(atSign, atKeysIo: FileAtKeysIo());

  final AtAuthResponse response = await atAuth.authenticate(request);
  if (!response.isSuccessful) {
    stderr.writeln('Authentication failed');
    exit(1);
  }

  final AtClientPreference preference = AtClientPreference()
    ..hiveStoragePath = storagePath;

  final AtClientManager atClientManager = await AtClientManager.getInstance()
      .fromAuthSession(response.session!, preference);

  final AtClient atClient = atClientManager.atClient;
  stdout.writeln('AtClient ready for $atSign');
}
```

## Next

Put your first record: **Building an AtKey**.

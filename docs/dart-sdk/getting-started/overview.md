---
description: Overview of the Dart SDK
---

# Overview

Atsign's SDK written in Dart is the most mature SDK out of all SDKs offered by the platform. [AtClient 3.x](https://pub.dev/packages/at_client/versions) currently contains the most amount of features and we try our best to uphold backwards compatibility with clients that have existed since 2018.

The Dart SDK is a client-side SDK that allows developers to interact with their atServer and ultimately create secure end-to-end encrypted and identity-backed data exchanges with other Atsigns.

| Where to go next                               | Why                                                                                                               |
| ---------------------------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| [prerequisites.md](prerequisites.md "mention") | First, fulfill the prerequisites so that you can start writing programs in the Dart SDK.                          |
| [quickstart.md](quickstart.md "mention")       | Run a simple program that will onboard and write data to your atServer.                                           |
| [our-packages.md](our-packages.md "mention")   | Get an overview of the packages we host on pub.dev and which packages you will be using in your Dart applications |

## Flutter Applications

Note that the Dart SDK and Flutter SDK section have lots of overlap. You will be writing code that is referenced in both the [Dart SDK](https://app.gitbook.com/s/pQPWQHMckXtXYemavYhN/dart-sdk "mention") section and the [Flutter SDK](https://app.gitbook.com/s/pQPWQHMckXtXYemavYhN/flutter-sdk "mention") section. Dart code such as [atcollections](../atcollections/ "mention") and [events](../events/ "mention") remain the same in both a Dart and Flutter app. However, subtle things like Keychain Management and Onboarding will be different in the [Flutter SDK](https://app.gitbook.com/s/pQPWQHMckXtXYemavYhN/flutter-sdk "mention"). See [Flutter SDK](https://app.gitbook.com/s/pQPWQHMckXtXYemavYhN/flutter-sdk "mention") to learn more.

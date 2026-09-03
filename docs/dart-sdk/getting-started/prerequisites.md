---
description: Three things before you write any code
---

# Prerequisites

## Dart

We require **Dart 3.12 or newer**.

Follow Dart's official documentation on getting the Dart SDK: [https://dart.dev/get-dart](https://dart.dev/get-dart).

Check your dart version via

```
dart --version
```

## A registered Atsign

Purchase an Atsign for as low as $10 from [https://my.atsign.com/go](https://my.atsign.com/go).

A registered Atsign is an Atsign that is tied to an email you own. This means you own the name. In [authentication](../authentication/ "mention"), we cover initial onboarding of an Atsign and subsequent authentications of an Atsign.

You can check if your Atsign is registered if the production atDirectroy has an atServer address connected to that Atsign. See [#how-to-connect-to-the-atdirectory](../../atsign-platform/atdirectory.md#how-to-connect-to-the-atdirectory "mention")

## Networking requirements

The SDK makes **outbound** TLS connections only; nothing listens. No inbound firewall rule is needed.

Other than having access to the Internet, it is recommended to check that your network can access these destinations:

| Destination                 | Purpose                                                                                                                                                            |
| --------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `root.atsign.org:64`        | atDirectory look up (Atsign to atServer address). If your network blocks outbound port 64, atDirectory look up will fail in the SDK.                               |
| `proxy:root.atsign.org:443` | Passing this value into an AtRootDomain parameter will route all traffic through our proxy server in the cloud, in case your only outbound capability is port 443. |


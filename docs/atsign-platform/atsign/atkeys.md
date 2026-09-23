---
description: The file that contains the set of cryptographic keys belonging to an Atsign.
icon: key
---

# atKeys

## What is the atKeys file?

The atKeys file is a set of cryptographic keys that allows devices to authenticate into their atServer and encrypt data for other Atsigns.

This is a file. Not to be confused with "AtKeys" which are the identifiers used in [atrecords.md](../atserver/atrecords.md "mention").

## File Conventions

The name of the file will always follow this format:

```
<@atsign>_key.atKeys
```

Example: `@alice_key.atKeys`

The location of your atKeys files will almost always be automatically generated to this directory on your computer (MacOS, Linux, Windows).

```
$HOME/.atsign/keys/
```

If on a mobile device, you will be manually saving it to a safe location via your mobile device's file system. You will need to manually keep track of this key's location, so be sure to keep it somewhere safe!

## How do I get an atKeys file?

To obtain the .atKeys file, you must either onboard a new Atsign ( [onboarding.md](onboarding.md "mention") ) or enroll an already-onboarded Atsign ( [enrollments.md](enrollments.md "mention") ).

<table><thead><tr><th width="226.30902099609375">What</th><th width="521.6909790039062">When</th></tr></thead><tbody><tr><td><a data-mention href="onboarding.md">onboarding.md</a></td><td>When the Atsign is registered, but hasn't been activated yet. Onboarding generates the initial set of atKeys which is also known as the manager atKeys file.</td></tr><tr><td><a data-mention href="enrollments.md">enrollments.md</a></td><td>When the Atsign is registered and already onboarded, you want to "enroll" a copy when you want revocability and scoped permissions. For example, when you want your Atsign to be used on an untrusted network but have the ability to revoke the atKeys' capabilities in case it is compromised or when you want your friend to only use your Atsign under a certain namespace (like <code>noports</code>).</td></tr></tbody></table>

## Help! I've lost my atKeys file

**Atsign Inc. cannot recover your lost cryptographic keys**. At the time they were generated, only you and your device knows the keys to your atServer.

The only path forward is to [reset your Atsign](atsign.md#resetting-your-atsign) which will result in all data being wiped on your atServer. However, your Atsign (handle) can be reused to re-activate a new atServer.

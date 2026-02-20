---
description: A unique identifier which serves as the address of the atServer
icon: at
---

# Atsign

## What is an Atsign?

An Atsign (e.g. @alice) is simply a resolvable address for an atServer. Anything can have an Atsign, a person, entity, or thing (IoT), even individual songs or videos could have Atsign addresses.

### What's it used for?

An Atsign is used to protect and securely exchange information with other Atsigns without any chance of surveillance, impersonation, or theft of the information by anyone.

### What characters can be used in an Atsign?

An Atsign supports any combination of Unicode UTF-8 characters that are translated to UTF-7 and must have less than characters 55 characters in length. This provides an enormous name space of 10^224 atSigns.

### How do I get an Atsign?

Head to [the registrar site](https://my.atsign.com/go). It is recommended that you login with your email. One email can hold up to 10 free Atsigns and unlimited paid Atsigns.

### How do I generate my associated cryptographic keys?

There are multiple ways to generate the associated `.atKeys` file for an Atsign. Within applications the .atKeys are stored in encrypted keychains that the OS provides.  Using the command line .atKeys files are produced and put in the directory `~/.atsign/keys`.&#x20;

Most applications have a way to export the .atkeys to a file if you need to keep them safe or use them on another device.

* [Using atmospherePro](https://www.youtube.com/watch?v=8xJnbsuF4C8) (3 minute video)
* Dart [at\_onboarding\_cli/at\_activate](https://github.com/atsign-foundation/at_libraries/tree/trunk/packages/at_onboarding_cli#activate_cli) to activate an owned Atsign or [at\_onboarding\_cli/at\_register](https://github.com/atsign-foundation/at_libraries/tree/trunk/packages/at_onboarding_cli#register_cli) to generate a new free Atsign.
* [Java Registration CLI](https://github.com/atsign-foundation/at_java/blob/trunk/getting_started_guide.md)

### Paid Atsigns

You can purchase custom Atsigns from [the registrar site](https://my.atsign.com/go).

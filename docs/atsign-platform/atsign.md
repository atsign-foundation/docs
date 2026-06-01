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

Atsigns start at $10 per year. You can purchase and manage your Atsigns at the [Atsign Registrar](https://my.atsign.com/go). While we no longer offer free Atsigns, existing free accounts remain active under our current terms.&#x20;

### How do I generate my associated cryptographic keys?

There are two ways to generate the `.atKeys` file for an Atsign:

* When using the command line, .atKeys files are generated and stored in the `~/.atsign/keys` directory. [at\_onboarding\_cli](https://github.com/atsign-foundation/at_client_sdk/blob/trunk/packages/at_onboarding_cli/README.md): Use the `at_activate` command to activate an Atsign you have purchased.
* If using an Atsign Platform application, .atKeys are stored in the encrypted keychains provided by the operating system. Most applications allow you to export your .atKeys file for backup or for use on other devices.


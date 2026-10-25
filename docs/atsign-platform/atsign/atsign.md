---
description: Learn about what an Atsign is
icon: at
---

# Atsign

{% hint style="info" %}
The product name "Atsign" is the same as the company name!
{% endhint %}

## What is an Atsign?

An Atsign (e.g. @alice) is a resolvable address for an [atserver.md](../atserver/atserver.md "mention"). Similar to how a handle like @alice or @bob makes people addressable on social applications, our goal for an Atsign is to give addressability to everything on the Internet (people, entities, and things).

## What is an Atsign used for?

An Atsign is used to protect and securely exchange information with other Atsigns without any chance of surveillance, impersonation, or theft of the information by anyone. Since cryptographic keys live only on your edge devices, that means no middle actors (not even Atsign) can decrypt your data.

## How do I get an Atsign?

You can purchase and manage your Atsigns at the [Atsign Registrar](https://my.atsign.com/go). The price of your Atsign will vary given the amount of special characters in the Atsign (emojis, numbers, etc,.).

### How do I get a Free Atsign?

We no longer offer free Atsigns. Existing free Atsigns remain active under our current terms.&#x20;

## What characters can be used in an Atsign?

An Atsign supports any combination of Unicode UTF-8 characters that are translated to UTF-7 and must have less than characters 55 characters in length. This provides an enormous name space of 10^224 atSigns.

## How do I activate my Atsign?

{% hint style="info" %}
Activating and onboarding are the same.
{% endhint %}

There are two ways to onboard/activate an Atsign. The first way is using our CLI tool [at\_activate](onboarding.md#at_activate) and the second way is by using an Atsign application with an onboarding widget (such as [NoPorts Desktop](https://docs.noports.com/reference/noports-desktop-application) or [AtmospherePro](https://play.google.com/store/apps/details?id=com.atsign.atsign_atmosphere_pro\&hl=en_CA)), where an OTP will be sent to your email and providing the OTP to the app will onboard your keys and be saved either to your file system or your keychain.

You can only activate an Atsign if you own the Atsign (it's associated to an email you have access to) and it has not already been activated before. If you need to re-activate an Atsign because you have lost your .atKeys file, check out [#help-ive-lost-my-.atkeys](atkeys.md#help-ive-lost-my-.atkeys "mention").

## How do I enroll my Atsign?

Check out [enrollments.md](enrollments.md "mention"). You can enroll your Atsign in two ways: [at\_activate CLI](enrollments.md#at_activate) or using an Atsign application that supports enrollments, such as [NoPorts Desktop](https://docs.noports.com/reference/noports-desktop-application).

## Resetting your Atsign

If you have lost your [.atKeys file](atkeys.md) or your atServer broke, you have come to the right place. Most people wish to reset their Atsign (also known as resetting atServer) to re-generate a new .atKeys set due to lost keys or they wish to reset all data on their atServer and redo the [onboarding process](onboarding.md) altogether.

Resetting your Atsign will wipe all encrypted and public data and you will no longer be able to authenticate to the same atServer with stale .atKeys.

To reset your Atsign, go to [https://my.atsign.com](https://my.atsign.com/), login with the email address that is tied to that Atsign, then navigate to the manage page of your Atsign and open the Reset dropdown. Type the Atsign to proceed with resetting your atServer. This will make any `.atKeys` file that corresponds to this atServer useless, which includes the master set of keys and APKAM enrollments.

<figure><img src="../../.gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>

Once you reset your Atsign, delete stale versions of the `.atKeys`  file, as they no longer work when authenticating with your atServer. Find them in your `$HOME/.atsign/keys/` directory.

You can also reach out to [support@atsign.com](mailto:support@atsign.com) and we can help you reset your Atsign. Please reach out with the corresponding email that is tied to the relevant Atsign.

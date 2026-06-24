---
description: The file that contains the set of cryptographic keys belonging to an Atsign.
icon: key
---

# atKeys

## What is the atKeys file?

The atKeys file is a set of cryptographic keys that provides devices the ability to authenticate into their AtServer and encrypt data for other Atsigns.

This is a file. Not to be confused with "AtKeys" which are the identifiers used in [atRecords](atrecord.md).

### File Conventions

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

## What is activation?

**Activating** is the one-time, irreversable bootstrapping of a freshly provisioned Atsign. During activation, the client generates a cryptographic identity (a keypair) and registers it with the server as the proof of ownership.&#x20;

The end result of activation is the **.atKeys** file; a portable credential file that represents your ownership of that Atsign. Ensure that you backup this file, as losing it means your Atsign's data becomes unrecoverable!

Activating is also known as "CRAM authentication" or "onboarding."

Activating can be done with the [at\_activate](https://pub.dev/packages/at_onboarding_cli) binary which can be downloaded several ways: from [pub.dev](https://pub.dev/packages/at_onboarding_cli), [NoPorts release archives](https://github.com/atsign-foundation/noports/releases/latest), or from [GitHub source](https://github.com/atsign-foundation/at_client_sdk/tree/trunk/packages/at_onboarding_cli).

## at\_activate

### Installing at\_activate

`at_activate` is Atsign's official activation CLI tool. It contains functions like initial activation (CRAM) and for generating additional namespace-scoped copies (APKAM).

To install it, you must have Dart installed on your system.

1. Ensure you have [Dart](https://dart.dev/get-dart) installed

```
dart --version
```

2. Install [at\_activate](http://pub.dev/packages/at_onboarding_cli) using `dart pub`&#x20;

```
dart pub global activate at_onboarding_cli
```

3. You should be able to run `at_activate` in your terminal

```
at_activate
```

If missing, check `$HOME/.pub-cache/bin` for the at\_activate binary.

### Uninstalling at\_activate

Simply use `dart pub global deactivate`

```
dart pub global deactivate at_onboarding_cli
```

### Initial activation

Ensure you have `at_activate` installed before moving on. Check out [#installing-at\_activate](atkeys.md#installing-at_activate "mention") on how to do that.

1. Use the `onboard` command to onboard your Atsign. Replace `<@atsign>` with your Atsign. Example: `at_activate onboard -a @alice`

```
at_activate onboard -a <@atsign>
```

2. You will then be prompted with a yes/no question. It is very important you backup and keep this file safe, once it is generated in the next steps. Enter "Y" and continue.

<figure><img src="../.gitbook/assets/image (27).png" alt=""><figcaption></figcaption></figure>

3. You will receive an OTP in your email. Enter the OTP into the terminal.

```
[Information] Requesting my.atsign.com to send a verification code
[Information] Successfully sent verification code to your registered e-mail or phone
[Action Required] Enter your verification code: DTBW
```

4. Your Atsign will then be activated and manager keys will be generated in `$HOME/.atsign/keys`

```
[Information] Fetching CRAM Key from my.atsign.com
[Information] CRAM Key fetched successfully

      Find : #[1/50] : Found atServer address for @blueshark82_jttest in atDirectory - 2b8169fc-de0b-5a90-90cb-ed600904f4f0.swarm0003.atsign.zo...
   Connect : #[2/50] : Connected to @blueshark82_jttest atServer
[Success] Your .atKeys file saved at /home/user/.atsign/keys/@blueshark82_jttest_key.atKeys
```

5. Backup a copy of this file in some place safe (like a personal drive or a secure cloud storage).

### Enrolling new devices

{% hint style="info" %}
This is also known as creating an "APKAM copy" of your keys.
{% endhint %}

Now that your Atsign is activated, we can administer a new copy of the .atKeys file with a namespace-restriction.&#x20;

A namespace-restriction means that this new .atKeys file copy will only have read and/or write access to certain namespaces. Read more on namespaces [here](atrecord.md#visibility-scope).

The following steps are typically done on two separate devices, but is still possible to do on one. We will refer to **Device 1** as the device with the manager set of keys and **Device 2** as the device who wishes to enroll under this Atsign.&#x20;

| Device   | Purpose                                                           |
| -------- | ----------------------------------------------------------------- |
| Device 1 | The device with the manager .atKeys file                          |
| Device 2 | The enrolling device that will generate the new .atKeys file copy |

Before beginning, you must establish a few strings and keep this in mind.

| Variable    | Purpose                                    | Example          |
| ----------- | ------------------------------------------ | ---------------- |
| App Name    | App namespace that this key will belong to | my\_app          |
| Device Name | The name of the enrolling device           | linux\_server\_1 |

**Device 1** will generate an OTP and create an auto approval process, then **Device 2** will send the enrollment request and this will automatically generate an APKAM .atKeys file copy.

1. On **Device 1**, generate an OTP. Take note of this OTP, as it will be needed in step 3.&#x20;

Replace `<@atsign>` with the Atsign you are making a copy of.

```
at_activate otp -a <@atsign>
```

**Tip:** you can make this OTP useable for longer by setting an expiry by appending `--expiry 2h`  to the command above.

2. On **Device 1**, create an auto approval service.&#x20;

| Parameter       | Description                                                                                 | Example           |
| --------------- | ------------------------------------------------------------------------------------------- | ----------------- |
| `<@atsign>`     | The Atsign you are making a copy of                                                         | `@alice`          |
| `<app_name>`    | Name of the application/use case. Think of this as the purpose you are making this key for. | `noports`         |
| `<device_name>` | Device name that uniquely identifies this enrollment from other enrollments.                | `linux_server_01` |

```
at_activate auto \
  -L 1 \
  -a <@atsign> \
  --arx <app_name> \
  --drx <device_name>
```

This will set up an auto service with a limit of 1. Leave this process running in the background.

3. On **Device 2,** send the enrollment request.

| Parameter            | Description                                                                                                                                                                            | Example                                                                                                     |
| -------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| `<@atsign>`          | Atsign you are making a copy of                                                                                                                                                        | `@alice`                                                                                                    |
| `<app_name>`         | The same app name from Step 2                                                                                                                                                          | `noports`                                                                                                   |
| `<device_name>`      | The same device name from Step 1                                                                                                                                                       | `linux_server_01`                                                                                           |
| `<[namespace:r?w?]>` | <p>A list of comma-separated namespace and their read/write permissions.<br><br>If you want this copy to simply be a revokable copy with all access, set this to <code>*:rw</code></p> | <p><code>"*:rw"</code></p><p></p><p></p><p></p><p><code>"sshnp:rw,sshrvd:rw,noports:r,at_talk:w"</code></p> |
| `<OTP>`              | The one-time passcode from Step 1                                                                                                                                                      | `ABC123`                                                                                                    |

Change all parameters. Note that you have to replace `<@atsign>` twice in this command (`-a` and `--keys` ).

```
at_activate enroll \
  -a <@atsign> \
  --keys ~/.atsign/keys/<@atsign>_key.atKeys \
  --app <app_name> \
  --device <device_name> \
  -n "<[namespace:r?w?]>" \
  -s <OTP>
```

If you have set up an auto approval service up correctly (which is running in the background from Step 2), then executing this enroll command should exit successfully after a couple of seconds.

Example command & output:

```
❯ at_activate enroll \
    -a @alice \
    --keys ~/.atsign/keys/@alice_key.atKeys \
    --app test_app \
    --device copy1 \
    -n "*:rw" \
    -s DWB1HY

    Enroll : submitting enrollment requestEnrollment ID: 7dec3458-a939-4296-82ea-a6e042e72f52
Waiting for approval; will check every 10 seconds
    Enroll : submitted OK
      PKAM : Enrollment has been approved (PKAM auth success)Creating atKeys file
[Success] Your .atKeys file saved at /home/user/.atsign/keys/@alice_key.atKeys
```

## Exceptions

### AT0032 - Exception

This is a common exception that many people run into.

Your error message will say that this enrollment is in an `approved`  state or a `pending` state.&#x20;

To fix the pending error, go to [#pending-state](atkeys.md#pending-state "mention").&#x20;

To fix the approved error, go to [#approved-state](atkeys.md#approved-state "mention").

#### Pending State

You most likely cancelled the enrollment process or missed a step during your [Enrolling new device](atkeys.md#enrolling-new-devices) handshake.

To fix this, go to **Device 1** (the device with the manager .atKeys file) and we will have to **deny** and **delete** this enrollment request. If your enrollment error says it's in a `pending` state, then you will need to run these two operations: `deny`  and `delete`.

1. Go to **Device 1** (the device with the manager .atKeys file).
2. Confirm your error message is similar to below (says "`in pending state` ")

```
    Enroll : submitting enrollment request

ERROR: enroll : ErrorCode: AT0032 - Exception: Exception: Another enrollment with id 460d24b9-9194-4718-bef2-96f7f467b04b exists with the app name: test_app and device name: copy2 in pending state

Please try again or contact support@atsign.com
```

3. Run the `list` command and copy the enrollment ID of the faulty enrollment request.

```
at_activate list -a <@atsign>
```

Example:

```
❯ at_activate list -a @alice
Connecting ... Connected
Found 3 matching enrollment records
Enrollment ID                         Status    AppName             DeviceName                            Namespaces
460d24b9-9194-4718-bef2-96f7f467b04b  pending   test_app            copy2                                 {*: rw}                        {*: rw}
a1de4e17-0eac-4860-a303-55a9c66358a8  approved  firstApp            firstDevice                           {__manage: rw, *: rw}
```

It is very important we leave the **firstDevice** enrollment untouched.

In this scenario, I will copy the enrollment ID `460d24b9-9194-4718-bef2-96f7f467b04b` .

4. Run the `deny` command.

```
at_activate deny -a <@atsign> -i <enrollment_id>
```

Example:

```
❯ at_activate deny -a @alice -i 460d24b9-9194-4718-bef2-96f7f467b04b
Connecting ... Connected
Denying enrollmentId 460d24b9-9194-4718-bef2-96f7f467b04b
Server response: data:{"status":"denied","enrollmentId":"460d24b9-9194-4718-bef2-96f7f467b04b"}
```

5. Run the `delete` command.

```
at_activate delete -a <@atsign> -i <enrollment_id>
```

Example:

```
❯ at_activate delete -a @alice -i 460d24b9-9194-4718-bef2-96f7f467b04b
Connecting ... Connected
Sending delete request
Server response: {"enrollmentId":"460d24b9-9194-4718-bef2-96f7f467b04b","status":"deleted"}
```

6. Follow steps 1-3 again in [Enrolling new devices](atkeys.md#enrolling-new-devices) and that should resolve the error from coming up again!

#### Approved State

You are most likely trying to re-enroll an Atsign that has been previously enrolled on another device, and you would like to now re-enroll on a new completely separate device.

To fix this, go to **Device 1** (the device with the manager .atKeys file) and we will need to `revoke` and `delete`.

1. Go to **Device 1** (the device with the manager .atKeys file)
2. Confirm that error message you got on **Device 2** is similar to below (it says "`in approved state`")

```
    Enroll : submitting enrollment request

ERROR: enroll : ErrorCode: AT0032 - Exception: Exception: Another enrollment with id 7dec3458-a939-4296-82ea-a6e042e72f52 exists with the app name: test_app and device name: copy1 in approved state

Please try again or contact support@atsign.com
```

3. Run the `list` command and copy the enrollment ID of the faulty enrollment request.

```
at_activate list -a <@atsign>
```

Example:

```
❯ at_activate list -a @alice
Connecting ... Connected
Found 3 matching enrollment records
Enrollment ID                         Status    AppName             DeviceName                            Namespaces                         {*: rw}
7dec3458-a939-4296-82ea-a6e042e72f52  approved  test_app            copy1                                 {*: rw}
a1de4e17-0eac-4860-a303-55a9c66358a8  approved  firstApp            firstDevice                           {__manage: rw, *: rw}
```

It is very important we leave the **firstDevice** enrollment untouched.

In this scenario, I will copy the enrollment ID `7dec3458-a939-4296-82ea-a6e042e72f52`.

4. Run the `revoke` command.

```
at_activate revoke -a <@atsign> -i <enrollment_id>
```

Example:

```
❯ at_activate revoke -a @alice -i 7dec3458-a939-4296-82ea-a6e042e72f52
Connecting ... Connected
Revoking enrollmentId 7dec3458-a939-4296-82ea-a6e042e72f52
Server response: data:{"status":"revoked","enrollmentId":"7dec3458-a939-4296-82ea-a6e042e72f52"
```

5. Run the `delete` command.

```
at_activate delete -a <@atsign> -i <enrollment_id>
```

Example:

```
❯ at_activate delete -a @alice -i 7dec3458-a939-4296-82ea-a6e042e72f52
Connecting ... Connected
Sending delete request
Server response: {"enrollmentId":"7dec3458-a939-4296-82ea-a6e042e72f52","status":"deleted"}
```

6. Follow steps 1-3 again in [Enrolling new devices](atkeys.md#enrolling-new-devices) and that should resolve the error from coming up again!

## Help! I've lost my .atKeys

Atsign cannot recover your lost cryptographic keys.

The only path forward is to reset your atServer which will result in all data being wiped. However, your Atsign (handle) will still be usable.

To reset your Atsign, go to [https://my.atsign.com](https://my.atsign.com/), login with the email address that is tied to that Atsign, then navigate to the manage page of your Atsign and open the Reset dropdown. Type the Atsign to proceed with resetting your atServer.

<figure><img src="../.gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>

You can also reach out to `support@atsign.com` and we can help you reset your Atsign. Please reach out with the corresponding email that is tied to the relevant Atsign.

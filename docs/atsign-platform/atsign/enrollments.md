---
description: Learn about what enrollments are in the platform and their capabilities
icon: dice-d12
---

# Enrollments

## What are enrollments?

An Atsign's `.atKeys` file holds every cryptographic key the Atsign owns. Copying that file to a second device gives the second device **unrestricted irrevocable access**. If the second device goes rogue, there is no way to revoke that access. Additionally, there's no permission scoping to control what namespaces that second `.atKeys` file copy gets; it gets full access to the Atsign, just like the first copy.

Enrollments solve this problem. Instead of making duplicate copies of the `.atKeys` , a new app **requests access** to a defined list of namespaces. The manager `.atKeys` then reviews the requests and either approves or denies it. If approved, the new app generates its a new `.atKeys` called an  enrollment. This `.atKeys` is scoped to the namespaces it requested and is revokable by the manager `.atKeys` at any time.

The underlying mechanism is called **APKAM** (Application Public Key Authentication Mechanism).

## The two roles

Every "enrollment dance" has two sides:

<table><thead><tr><th width="249">Side</th><th>What it does</th><th>What it needs</th></tr></thead><tbody><tr><td><strong>Requesting app</strong> - wants to enroll a new <code>.atKeys</code></td><td>Submits a request specifying which namespaces it wants (e.g. <code>sshnp:rw,sshrvd;rw</code> ), then waits for approval</td><td>A passcode (OTP or SPP) from the <strong>approving app</strong></td></tr><tr><td><strong>Approving app</strong> - approves/denies requesting apps</td><td>Generates passcodes (OTPs/SPPs) for <strong>requesting apps</strong>, lists pending requests, approves/denies <strong>requesting apps</strong>, can revoke later</td><td><code>__manage:rw</code> namespace access</td></tr></tbody></table>

## Enrollment flow

1. The **approving app** mints a passcode: either an OTP (one-time passcode) or a SPP (semi-permanent passcode).
2. A **human** carries the passcode to the new device out-of-band (reads it aloud, sends it via chat, shows a QR code).
3. The **requesting app** submits an enrollment request containing the passcode, the desired namespaces (e.g. `sshnp:rw,sshrvd:rw`), an app name, and a device name.
4. The atServer validates the passcode and holds the request as **pending**.
5. The **approving app** reviews the request and either approves or denies it.
6. On **approval**, the atServer issues APKAM keys scoped to the requested namespaces. The requesting app can now authenticate and do Atsign Protocol operations under only those approved namespaces.

```
  Requesting app               atServer               Approving app
       │                          │                         │
       │                          │◄── generate passcode ───┤
       │◄── user carries the passcode across ───────────────┤
       │                          │                         │
       ├── submit(request, otp)──►│                         │
       │                          ├── notification ────────►│
       │   polls, waiting...      │                         │
       │                          │◄── approve or deny ─────┤
       │◄── result ──────────────►│                         │
       │                          │                         │
```

## Enrollment statuses and actions

* Enrollment status - a state at which an enrollment can be in&#x20;
* Enrollment operation - move an enrollment from one state to another.

### Enrollment status and action diagram

```
pending ──approve──► approved ──revoke──► revoked
   │                                         │
   └──deny──► denied             unrevoke ───┘ 
                                  ▲
                                  └── restores to approved
```

### Enrollment statuses

An enrollment can only be in one state at a time.

| Status     | Meaning                                                                           |
| ---------- | --------------------------------------------------------------------------------- |
| `pending`  | Request submitted, waiting for a decision                                         |
| `approved` | Access granted. The app can authenticate                                          |
| `denied`   | Access refused. The app cannot authenticate                                       |
| `revoked`  | Access withdrawn. Active connections are closed, future authentication is blocked |
| `expired`  | The request timed out before anyone acted on it (server-managed)                  |

{% hint style="info" %}
An enrollment can only be deleted in either the revoked or denied state.
{% endhint %}

### Enrollment actions

An action takes an enrollment from one status to another.

| Action     | Meaning                                                                                                                |
| ---------- | ---------------------------------------------------------------------------------------------------------------------- |
| `approve`  | Approves an enrollment that submitted a request currently pending                                                      |
| `deny`     | Denies a pending enrollment; you do not give permission for that set of `.atKeys` to be created under those namespaces |
| `revoke`   | Revokes an `approved` enrollment; the `.atKeys` can no longer authenticate with the atServer.                          |
| `unrevoke` | Takes a `revoked` enrollment and restores it to `approved`                                                             |

## Namespace-scoped permissions

Each enrollment request specifies exactly which namespaces the app wants and at what access level:

* `rw` - read and write
* `r` - read only

The `__manage:rw`  namespace permission is special, as it allows that enrollment to manage other enrollments.

The table belows outlines some examples of namespace sets and their permission scopes.

| Namespace set                            | What it means                                                                                                                                                                                                                                                            |
| ---------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `my_app:r,sshnp:rw`                      | Read access to the `my_app` namespace, read and write access to the `sshnp` namespace. This `.atKeys` file has restricted permissions on those application namespaces.                                                                                                   |
| `*:rw`                                   | Read and write access to all namespaces.                                                                                                                                                                                                                                 |
| `*:rw,__manage:rw`                       | Read and write access to all namespaces and can manage all enrollments under all namespaces. This is the super administrator namespace set.                                                                                                                              |
| `wavi:rw,__manage:rw`                    | Read/write access to the `wavi` namespace. Can also manage APKAM enrollments that are a superset of its namespace set.                                                                                                                                                   |
| `wavi:rw,sshnp:rw,sshrvd:rw,__manage:rw` | This namespace set is able to manage enrollments such as `sshnp:rw` or `sshnp:rw,sshrvd:rw`, but wouldn't be able to manage the enrollment with a namespace set of `atmospherepro:rw,sshrvd:rw,sshnp:rw,wavi:rw` because that namespace set is not a subset of this one. |

## at\_activate

In this section, we will learn how to use `at_activate` to enroll new devices.

### Installing at\_activate

`at_activate` is Atsign's official activation CLI tool. It contains functions like initial activation (CRAM) and for generating additional namespace-scoped copies (APKAM).

{% tabs %}
{% tab title="Installation via dart pub" %}
#### Installation via dart pub

To install it through pub.dev must have Dart installed on your system.

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

If missing, check `$HOME/.pub-cache/bin` for the at\_activate binary. You may need to add this directory to your `$PATH`.&#x20;

#### Uninstallation

To uninstall, simply do&#x20;

```
dart pub global deactivate at_onboarding_cli
```
{% endtab %}

{% tab title="Installation via NoPorts archives" %}
#### Installation via NoPorts archives

You may also download at\_activate as an executable binary through our NoPorts release archives.

1. Download the correct archive according to your CPU architecture from [https://github.com/atsign-foundation/noports/releases/latest](https://github.com/atsign-foundation/noports/releases/latest).&#x20;

Refer to this table on which archive to download. Not sure which architecture you have?&#x20;

* On macOS or Linux run `uname -m` .&#x20;
* On Windows, check `Settings > System > About > System type`.

<table data-search="false"><thead><tr><th>Operating System</th><th>CPU architecture</th><th>Archive to download</th></tr></thead><tbody><tr><td>MacOS</td><td>x64</td><td>sshnp-macos-x64.zip</td></tr><tr><td>MacOS</td><td>ARM64</td><td>sshnp-macos-arm64.zip</td></tr><tr><td>Linux</td><td>x64</td><td>sshnp-linux-x64.tgz</td></tr><tr><td>Linux</td><td>ARM64</td><td>sshnp-linux-arm64.tgz</td></tr><tr><td>Linux</td><td>RiscV64</td><td>sshnp-linux-riscv64.tgz</td></tr><tr><td>Linux</td><td>ARM</td><td>sshnp-linux-arm.tgz</td></tr><tr><td>Windows</td><td>x64</td><td>sshnp-windows-x64.zip</td></tr></tbody></table>

2. Unarchive the zip, and move the binary to a folder in your path.

**macOS**

Substitute `sshnp-macos-x64.zip` if you are on an Intel Mac.

```shellscript
mkdir -p ~/.local/bin
unzip -q -o ~/Downloads/sshnp-macos-arm64.zip -d /tmp
install -m 755 /tmp/sshnp/at_activate ~/.local/bin/
rm -rf /tmp/sshnp ~/Downloads/sshnp-macos-arm64.zip
```

macOS does not include `~/.local/bin` on the default PATH, so you may need to add it:

```
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc
exec zsh
```

**Linux**

Substitute the archive name for your architecture.

```
mkdir -p ~/.local/bin
tar -xzf ~/Downloads/sshnp-linux-x64.tgz -C /tmp
install -m 755 /tmp/sshnp/at_activate ~/.local/bin/
rm -rf /tmp/sshnp ~/Downloads/sshnp-linux-x64.tgz
```

Most distributions already put `~/.local/bin` on your PATH. Check with `echo $PATH`, and if it is missing:

```
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
exec bash
```

**Windows**

The simplest route on Windows is the `NoPorts-x64.msi` installer from the same release page, which handles PATH for you. To use the archive instead, run the following in PowerShell:

```
$dest = "$env:LOCALAPPDATA\Programs\NoPorts"
New-Item -ItemType Directory -Force -Path $dest | Out-Null
Expand-Archive -Force -Path "$HOME\Downloads\sshnp-windows-x64.zip" -DestinationPath "$env:TEMP\noports"
Move-Item -Force "$env:TEMP\noports\sshnp\at_activate.exe" $dest
Remove-Item -Recurse -Force "$env:TEMP\noports", "$HOME\Downloads\sshnp-windows-x64.zip"
```

Add it to your user PATH:

```
$dest = "$env:LOCALAPPDATA\Programs\NoPorts"
$user = [Environment]::GetEnvironmentVariable("Path", "User")
[Environment]::SetEnvironmentVariable("Path", "$user;$dest", "User")
```

Close and reopen PowerShell for the change to take effect.
{% endtab %}
{% endtabs %}

### Enrolling new devices

{% hint style="info" %}
This is also known as creating an "APKAM copy" of your keys.
{% endhint %}

Now that your Atsign is activated, we can administer a new copy of the `.atKeys` file with a namespace-restriction.&#x20;

A namespace-restriction means that this new `.atKeys` file copy will only have read and/or write access to certain namespaces. Read more on namespaces [here](/broken/pages/ZYp1wr1O1nBJcGt5E9i2#visibility-scope).

The following steps are typically done on two separate devices, but is still possible to do on one. We will refer to **Device 1** as the device with the manager set of keys and **Device 2** as the device who wishes to enroll under this Atsign.&#x20;

| Device   | Purpose                                                           |
| -------- | ----------------------------------------------------------------- |
| Device 1 | The device with the manager `.atKeys` file                        |
| Device 2 | The enrolling device that will generate the new .atKeys file copy |

Before beginning, you must establish a few strings and keep this in mind.

| Variable    | Purpose                                    | Example          |
| ----------- | ------------------------------------------ | ---------------- |
| App Name    | App namespace that this key will belong to | `my_app`         |
| Device Name | The name of the enrolling device           | `linux_server_1` |

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

### AT0032 - Exception

This is a common exception that many people run into.

Your error message will say that this enrollment is in an `approved`  state or a `pending` state.&#x20;

To fix the pending error, go to [#pending-state](enrollments.md#pending-state "mention").&#x20;

To fix the approved error, go to [#approved-state](enrollments.md#approved-state "mention").

#### Pending State

You most likely cancelled the enrollment process or missed a step during your [Enrolling new device](enrollments.md#enrolling-new-devices) handshake.

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

6. Follow steps 1-3 again in [Enrolling new devices](enrollments.md#enrolling-new-devices) and that should resolve the error from coming up again!

#### Approved State

You are most likely trying to re-enroll an Atsign that has been previously enrolled on another device, and you would like to now re-enroll on a new completely separate device.

Please make note that the steps below will **delete** the enrollment and invalidate that set of `.atKeys`. This means that if the enrollment is being actively used somewhere, you will be revoking its access and deleting it, which cannot be reversed.

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

6. Follow steps 1-3 again in [Enrolling new devices](enrollments.md#enrolling-new-devices) and that should resolve the error from coming up again!

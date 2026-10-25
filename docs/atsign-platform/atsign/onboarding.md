---
description: Onboarding is also known as activation
icon: wave
---

# Onboarding

## What is onboarding?

{% hint style="info" %}
Onboarding is also known as "CRAM authentication" or "authentication"
{% endhint %}

**Onboarding** (or activating) is the one-time, irreversable bootstrapping of a freshly provisioned Atsign. During activation, the client generates a cryptographic identity (a keypair) and registers it with the server as the proof of ownership.&#x20;

The end result of activation is the **.atKeys** file; a portable credential file that represents your ownership of that Atsign. Ensure that you backup this file, as losing it means your Atsign's data becomes unrecoverable!

Activating can be done with the [at\_activate](https://pub.dev/packages/at_onboarding_cli) binary which can be downloaded several ways: from [the at\_onboarding\_cli pub.dev package](https://pub.dev/packages/at_onboarding_cli), [NoPorts release archives](https://github.com/atsign-foundation/noports/releases/latest), or built/ran from [GitHub source](https://github.com/atsign-foundation/at_client_sdk/tree/trunk/packages/at_onboarding_cli).

## at\_activate

In this section, we will learn how to use at\_activate to onboard an Atsign. at\_activate is a CLI tool that helps with activation and enrollments.

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

### Initial activation

Ensure you have [at\_activate](onboarding.md#installing-at_activate) installed before moving on.

1. Use the `onboard` command to onboard your Atsign. Replace `<@atsign>` with your Atsign. Example: `at_activate onboard -a @alice`

```
at_activate onboard -a <@atsign>
```

2. You will then be prompted with a yes/no question. It is very important you backup and keep the `.atKeys` file safe. Enter "Y" to confirm you acknowledge this message and continue to step 3.

<figure><img src="../../.gitbook/assets/image (27).png" alt=""><figcaption></figcaption></figure>

3. You will receive an OTP in your email. Enter the OTP into your terminal.

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

5. Backup a copy of this file in some place safe (like a personal drive or through secure cloud storage you trust).

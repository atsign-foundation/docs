---
description: It takes two to tango!
---

# Cutting your Atsign keys

To complete this tutorial, you  need at least two Atsigns to send and receive data. You can purchase Atsigns at the [Atsign Registrar](http://my.atsign.com/go).&#x20;

Once you have your Atsigns, you are ready to activate them, which means spinning up one atServer per Atsign and cutting your cryptographic keys for each Atsign. Sounds complicated, but it is easy! In fact, just a single command. In the terminal window type:

```
dart run .\bin\at_activate.dart 
```

You will get an error message telling you to add the `-a` flag and your atsign, so if your Atsign was @crunchfrog you would type:

```
dart run .\bin\at_activate.dart -a "@crunchyfrog"
```

\*Note on windows the Atsign needs to be in double quotes so the shell does not get confused with the @ symbol. Below you can see the activation process for two Atsigns @energetic22 and @7capricon. Once the command is run you will get and email with the one time password, enter that and the program will create an atKeys file for that Atsign. Any atKeys files you create will be located in your home directory then .atsign/keys.

These keys are important to be kept safe, as they are the only keys to your Atsign and your data. Speaking of data, let's send some between the two Atsigns next.

<figure><img src="../../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

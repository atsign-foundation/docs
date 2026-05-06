---
description: You have mail!
---

# Put and Get data asynchronously

## Sending and receiving data

The next step is to send and receive. For this to work, we need to get the atKeys file containing the cryptographic keys and log in or onboard to the atServer. From there we can `put` data and `get` data as if a database was local to us, but, in fact, data is remote on the data owner's atServer.

This is achieved by every data element having an owner (the sending Atsign) and a receiving Atsign, behind the scenes the atSDK encrypts the data and sends it on to the atServer for delivery. All we need to do is a simple:

```dart
await atClient.put(sharedRecordID, message, putRequestOptions: pro);
```

Of course, we also have to set up the atClient object and onboard, but that is all in the `at_key_put.dart` file for you. So let's send some data.&#x20;

&#x20;Running the program without arguments brings up this help, most is self explanatory.

```
dart run .\bin\at_key_put.dart                                                                
    --help                        Usage instructions
-a, --atsign (mandatory)          This client's atSign
-n, --namespace (mandatory)       Namespace
-k, --key-file                    Your atSign's atKeys file if not in ~/.atsign/keys/
-c, --cram-secret                 atSign's cram secret
-h, --home-dir                    home directory
-s, --storage-dir                 directory for this client's local storage files
-d, --root-domain                 Root Domain
                                  (defaults to "root.atsign.org")
-v, --verbose                     More logging
    --never-sync                  Do not run sync
-o, --other-atsign (mandatory)    The atSign we want to communicate with
-m, --message (mandatory)         The message we want to send
Invalid argument(s): Option other-atsign is mandatory.
```

\*Note here we are using the Atsigns activated in the previous section; just substitute your own Atsigns.

```
 dart run .\bin\at_key_put.dart -a "@energetic22" -n "atsign" -o "@7capricorn" -m "hello world" 
 
 dart run .\bin\at_key_get.dart -a "@7capricorn" -n "atsign" -o "@energetic22"
```

The `-n`namespace is lefthand side of the atRecord, so in this case the full atRecord would be `@7capricorn:message.atsign@energetic22` and you see this in the results, too. Experiment for yourself!

<figure><img src="../../../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

You may also notice that the first time you fail to get the message. This is because we specified in the metadata that the TTL is 60000 millisecond or 1 minute. Try again as you see in the above image and you should get your message too.

```dart
    ..metadata = (Metadata()
      ..ttl = 60000 // expire after 60 seconds
      ..ttr = -1); // allow recipient to keep a cached copy
```

The code is worth taking a look at, as it is less than 70 lines but allows data to be transferred from a device anywhere to a device anywhere, and the data is fully encrypted.&#x20;

The data in this example is "store and forward", which is ideal for many use cases, but sometimes we need near real time data and that's what we cover next.

---
description: spooky actions at a distance
---

# Remote Procedure Calls (RPC)



## Do something remotely and send back the answer.

RPC has been a design pattern for decades, and, no surprise, we have RPC in the atSDK also.  This demo code is in another directory named `at_rpc_demo`. You can get to it by using VS Code and open folder and select that folder. Next, once again, open two windowpanes.

In the right-hand pane enter (again using your Atsigns!)

```
dart pub get
```

This pulls in needed libraries, and then this starts the RPC listener.

```
dart pub get
dart run .\bin\arithmetic_server.dart -a "@energetic22" -n "atsign" --allow-list "@7capricorn"
```

Then in the left window, you can run up the client

```
dart run .\bin\arithmetic_client.dart -a "@7capricorn" -n "atsign" --server-atsign "@energetic22"
```

You will get a prompt after a second or two and you can put in a math expression and hit enter. The expression will be sent to the other Atsign, the answer calculated, and then returned.&#x20;

See our session in action:

<figure><img src="../../.gitbook/assets/RPC.png" alt=""><figcaption><p>RPC</p></figcaption></figure>

Something to notice here is that the RPC server will only respond to RPCs from the designated Atsign with the `--allow-list` argument.

The RPC can, of course, do anything you want it to, and the Atsigns can be running anywhere.

### The code

An AtRpc object is set up pretty simply with:

```dart
    var rpc = AtRpc(
      atClient: atClient,
      baseNameSpace: atClient.getPreferences()!.namespace!,
      domainNameSpace: 'at_rpc_arithmetic_demo',
      callbacks: DemoRpcServer(),
      allowList: allowList,
    );

    rpc.start();
  } catch (e) {
    print(e);
    print(CLIBase.argsParser.usage);
  }
```

The Callbacks are sent to the `DemoRpcServer`class and handled appropriately.

On the sending side the RPC client is initiated:

```dart
    var rpc = AtRpcClient(
        atClient: atClient,
        baseNameSpace: atClient.getPreferences()!.namespace!,
        domainNameSpace: 'at_rpc_arithmetic_demo',
        serverAtsign: serverAtsign);
```

Then the RPC simple sent and awaiting a reply

```dart
var response = await rpc.call({'expr': expr}).timeout(Duration(seconds: 5));
```

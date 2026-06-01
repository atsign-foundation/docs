---
description: Set up the atSDK for your preferred language
---

# Get Started

{% tabs %}
{% tab title="Dart" %}
## Starter Dart App

Guide coming soon!

For now, please check out our [atsdk-tutorial](../atsdk-walkthroughs/atsdk-tutorial/ "mention")
{% endtab %}

{% tab title="Flutter" %}
## Starter Flutter App

_Create a Flutter application initialized for use with the atSDK_

{% hint style="warning" %}
If you don't have a Flutter setup for development yet, please see the [Flutter docs](https://docs.flutter.dev/get-started).
{% endhint %}

**1. Create a new Flutter app**

First, generate a new Flutter app (replace `at_app` with the name of your application).

```sh
flutter create at_app
cd at_app
```

#### 2. Add dependencies

Then add the following dependencies to your project:

```sh
flutter pub add at_client_mobile at_onboarding_flutter flutter_dotenv path_provider
```

#### 3. Replace main.dart

Now copy the contents of `main.dart` from below into `lib/main.dart` into your project.

<details>

<summary>main.dart</summary>

```dart
import 'dart:async';

import 'package:flutter/material.dart';
import 'package:at_onboarding_flutter/at_onboarding_flutter.dart';
import 'package:flutter_dotenv/flutter_dotenv.dart';
import 'package:path_provider/path_provider.dart'
    show getApplicationSupportDirectory;

import 'home_screen.dart';

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await dotenv.load();
  runApp(const MyApp());
}

Future<AtClientPreference> loadAtClientPreference() async {
  var dir = await getApplicationSupportDirectory();

  return AtClientPreference()
    ..rootDomain = 'root.atsign.org'
    ..namespace = dotenv.env['NAMESPACE']
    ..hiveStoragePath = dir.path
    ..commitLogPath = dir.path
    ..isLocalStoreRequired = true;
  // * By default, this configuration is suitable for most applications
  // * In advanced cases you may need to modify [AtClientPreference]
  // * Read more here: https://pub.dev/documentation/at_client/latest/at_client/AtClientPreference-class.html
}

class MyApp extends StatefulWidget {
  const MyApp({Key? key}) : super(key: key);
  @override
  MyAppState createState() => MyAppState();
}

class MyAppState extends State<MyApp> {
  // * load the AtClientPreference in the background
  Future<AtClientPreference> futurePreference = loadAtClientPreference();

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      // * The onboarding screen (first screen)
      home: Scaffold(
        appBar: AppBar(
          title: const Text('MyApp'),
        ),
        body: Builder(
          builder: (context) => Center(
            child: ElevatedButton(
              onPressed: () async {
                AtOnboardingResult onboardingResult =
                    await AtOnboarding.onboard(
                  context: context,
                  config: AtOnboardingConfig(
                    atClientPreference: await futurePreference,
                    rootEnvironment: RootEnvironment.Production,
                  ),
                );
                if (mounted) {
                  switch (onboardingResult.status) {
                    case AtOnboardingResultStatus.success:
                      Navigator.push(
                        context,
                        MaterialPageRoute(builder: (_) => const HomeScreen()),
                      );
                      break;
                    case AtOnboardingResultStatus.error:
                      ScaffoldMessenger.of(context).showSnackBar(
                        const SnackBar(
                          backgroundColor: Colors.red,
                          content: Text('An error has occurred'),
                        ),
                      );
                      break;
                    case AtOnboardingResultStatus.cancel:
                      break;
                  }
                }
              },
              child: const Text('Onboard an @sign'),
            ),
          ),
        ),
      ),
    );
  }
}

```

</details>

#### 4. Add home\_screen.dart

Create a second file in `lib/` called `home_screen.dart`. Copy the contents of `home_screen.dart` from below into your project.

<details>

<summary>home_screen.dart</summary>

```dart
import 'package:at_client_mobile/at_client_mobile.dart';
import 'package:flutter/material.dart';

// * Once the onboarding process is completed you will be taken to this screen
class HomeScreen extends StatelessWidget {
  const HomeScreen({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    // * Getting the AtClientManager instance to use below
    AtClientManager atClientManager = AtClientManager.getInstance();
    // * Getting the AtClient instance to use below
    AtClient atClient = atClientManager.atClient;

    // * From this widget onwards, you are fully onboarded!
    // * You can use the AtClient instance to perform operations as the onboared atSign
    return Scaffold(
      appBar: AppBar(
        title: const Text('What\'s my current @sign?'),
      ),
      body: Center(
        child: Column(children: [
          const Text('Successfully onboarded and navigated to FirstAppScreen'),

          // * Use the AtClient to get the current @sign
          Text('Current @sign: ${atClient.getCurrentAtSign()}')
        ]),
      ),
    );
  }
}

```

</details>

#### 5**a. gitignore the .env file**

Add the following line to the top of `.gitignore`:

```
.env
```

This will prevent accidentally uploading the secret values stored in `.env` to any git repositories associated with your app.

**5b. Add and populate the .env file**

Create a new file in the root of the project (next to `pubspec.yaml`) called `.env`. Add the following lines to it:

```sh
NAMESPACE=
API_KEY=
```

These values should be populated with your unique app namespace, and registrar API key.

**5.c. Register the .env file in pubspec.yaml**

In `pubspec.yaml` there should be a `flutter:` key, under this key, we want to create and add `.env` the `assets:` list:

```yaml
...
flutter:
  ...
  assets:
    - .env
  ...
...
```

#### 6. Start developing!

Start your application with the following command:

```sh
flutter run
```

Now you are ready to begin developing!
{% endtab %}

{% tab title="C" %}
## Get Started C Application

Setup a CMake project which includes the atSDK package suite.

The full example can be found [here](https://github.com/atsign-foundation/at_demos/tree/trunk/demos/get_started_c/0-my-first-c-app).

#### 1. Ensure you have a C Compiler, CMake, and a build automation tool

The method at which you decide to use to install these prerequisites are up to you. These three tools can be installed using either `apt` or `pip`.

Firstly, ensure you have a C compiler such as [clang](https://clang.llvm.org/) or [gcc](https://gcc.gnu.org/). We recommend [gcc](https://gcc.gnu.org/) because that is currently what Atsign is focused on supporting.

```bash
gcc --version
```

Secondly, ensure you have [CMake](https://cmake.org/) installed using `cmake --version`. Ensure that your CMake version is >= 3.24. If you have issues obtaining a newer version of CMake, we have found lots of success using `pip` when installing CMake via `pip install cmake@3.24`

```bash
cmake --version
```

Lastly, ensure you have a automation tool installed, such as [GNU Make](https://www.gnu.org/software/make/). We recommend [GNU Make](https://www.gnu.org/software/make/) because that is currently what Atsign is focused on supporting. These examples are using GNU Make version 4.3, but any version should work.

```bash
make --version
```

#### 2. Create an empty directory

This command will create a directory and then go into it.

```sh
mkdir my_first_c_app && cd my_first_c_app
```

#### 3. Create CMakeLists.txt

Inside your project folder, create a new file called `CMakeLists.txt.`

Copy and paste the CMake code into the file. Feel free to change the project name by changing this line:`project(my_first_c_app)` .

```cmake
# Minimum cmake version required by atSDK is 3.24
cmake_minimum_required(VERSION 3.24)

project(my_first_c_app)

include(FetchContent)

FetchContent_Declare(
  atsdk
  URL https://github.com/atsign-foundation/at_c/archive/refs/tags/v0.1.0.tar.gz
  URL_HASH SHA256=7ca4215a473037ca07bef362b852291b0a1cf4e975d24d373d58ae9c1df832bc
)

FetchContent_MakeAvailable(atsdk)

add_executable(main ${CMAKE_CURRENT_LIST_DIR}/main.c)

target_link_libraries(main PRIVATE atclient)
```

The above `CMakeLists.txt` will use FetchContent to download the C atSDK for you. Then, we will create a target named `main` for you and link our `atclient` library statically.&#x20;

#### 4. Create main.c

Inside your project folder, create a new file called `main.c`

Change the line `#define ATSIGN "@jeremy_0"` to the Atsign that you have keys to. For example, if you own an Atsign `@alice` and have its keys in the correct directory `~/.atsign/keys/@alice_key.atKeys`, then I would change this line in my code to `#define ATSIGN "@alice"`

```c
#include <atclient/atclient.h>
#include <atclient/atclient_utils.h>
#include <atclient/constants.h>
#include <atlogger/atlogger.h>
#include <stdlib.h>

#define ATSIGN "@jeremy_0"

int main()
{
    int exit_code = -1;

    atlogger_set_logging_level(ATLOGGER_LOGGING_LEVEL_DEBUG);

    char *atserver_host = NULL;
    int atserver_port = 0;

    atclient_atkeys atkeys;
    atclient_atkeys_init(&atkeys);

    atclient atclient;
    atclient_init(&atclient);

    if ((exit_code = atclient_utils_find_atserver_address(ATCLIENT_ATDIRECTORY_PRODUCTION_HOST, ATCLIENT_ATDIRECTORY_PRODUCTION_PORT, ATSIGN, &atserver_host, &atserver_port)) != 0)
    {
        goto exit;
    }

    if ((exit_code = atclient_utils_populate_atkeys_from_homedir(&atkeys, ATSIGN)) != 0)
    {
        goto exit;
    }
    
    if ((exit_code = atclient_pkam_authenticate(&atclient, atserver_host, atserver_port, &atkeys, ATSIGN)) != 0)
    {
        goto exit;
    }

    atlogger_log("my_first_c_app", ATLOGGER_LOGGING_LEVEL_INFO, "Authenticated to atServer successfully!\n");

    exit_code = 0;
exit:
{
    free(atserver_host);
    atclient_atkeys_free(&atkeys);
    atclient_free(&atclient);
    return exit_code;
}
}
```

#### 5. CMake Configure

In the terminal, run the following command.

```bash
cmake -S . -B build
```

This is known as the "configure" step where CMake will build instructions on how to build the app specific to your machine.

Output should be similar to the following:

```bash
$ cmake -S . -B build
-- [ATSDK] ATSDK_AS_SUBPROJECT: ON
-- Building atlogger
-- [ATLOGGER] ATLOGGER_AS_SUBPROJECT: ON
-- Building atchops
-- [ATCHOPS] ATCHOPS_AS_SUBPROJECT: ON
-- [MbedTLS] fetching package...
-- [uuid4] fetching package...
-- Building atclient
-- [ATCLIENT] ATCLIENT_AS_SUBPROJECT: ON
-- [cjson] fetching package...
-- Configuring done
-- Generating done
-- Build files have been written to: /home/jeremy/GitHub/at_demos/demos/get_started_c/1-authentication/build
```

#### 6. Build

Use CMake to build your configure and build your programs:

```bash
cmake --build build
```

This command is the same as running `cd build && make all` (if you are using Makefiles).&#x20;

Your output will look similar to:

```
cmake --build build
[ 50%] Building C object CMakeFiles/main.dir/main.c.o
[100%] Linking C executable main
[100%] Built target main
```

#### 7. Run the binary

The previous command, if ran successfully, would have created a binary executable for you: `./build/main`.

Simply run the program:

```bash
./build/main
```

Your output will look similar to:

```bash
[DEBG] 2024-08-15 00:37:16.517243 | connection |        SENT: "jeremy_0"
[DEBG] 2024-08-15 00:37:16.550027 | connection |        RECV: "3b419d7a-2fee-5080-9289-f0e1853abb47.swarm0002.atsign.zone:5770"
[DEBG] 2024-08-15 00:37:16.892252 | connection |        SENT: "from:jeremy_0"
[DEBG] 2024-08-15 00:37:16.929632 | connection |        RECV: "data:_d6d1a569-b944-4fd1-8c1e-7fc26e508452@jeremy_0:75be9182-bfdc-437f-96bc-c45eb5a1c0ff"
[DEBG] 2024-08-15 00:37:16.968986 | connection |        SENT: "pkam:QLuCl8JITCbOF6J8A6WlqBX3FUKoRR8DHh0FMnbVhHYRiuXZdBHJks0aQtq2cUj47KdhJCvN3Uk728UJHzwLgIF00wE3893xUi+9luoD7OFdV4Pbtmrxvj4K/s81A4Z9XBEJwtKerWStUavX5hR39By6Y6NJ2HeCuqVvBw2WQ/gKM15cAndpXrYzEVNJk9eCCN4+VXWxJOm6FhoY41Qxn/QSYqnRqp6PfHiY1rFXjsikJX6ip3LrCFTabJAS78BHx4BmmUzGKVPn0dFtYwvEKBgumViQhKHwS1ae1xBkLTZCARdIUhdhudoyba/ogwIEMjTWanLzAbqtNRa6qPTDrw=="
[DEBG] 2024-08-15 00:37:17.018977 | connection |        RECV: "data:success"
[INFO] 2024-08-15 00:37:17.019891 | my_first_c_app | Authenticated to atServer successfully!
```

Congratulations! You have successfully run a barebones Atsign C application.
{% endtab %}
{% endtabs %}


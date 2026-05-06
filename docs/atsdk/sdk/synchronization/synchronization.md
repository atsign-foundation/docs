# Synchronization

{% tabs %}
{% tab title="Flutter / Dart" %}
## Flutter / Dart

### SyncService

SyncService syncs the client app and remote secondary server’s changes:

* If the client app’s changes are ahead, it pushes the changes to the remote secondary.
* If the remote secondary is ahead, it pulls the changes to the client app.

### Usage

To trigger a sync, call the `.sync` function:

```dart
AtClientManager atClientManager = AtClientManager.getInstance();
SyncService syncService = atClientManager.syncService;
syncService.sync();
```
{% endtab %}
{% endtabs %}

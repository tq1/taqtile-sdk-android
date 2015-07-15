The Geotrigger Service must be started from the onCreate method of the launcher activity:

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
  super.onCreate(savedInstanceState);
  [...]
  TQ.shared().setActivity(this);
  TQ.shared().init(SA_APP_HOST, SA_APP_KEY, GCM_SENDER_ID, GEOTRIGGER_CLIENT_ID, GEOTRIGGER_DEFAULT_TAGS, new UserDialog());
}
```

The arguments passed to the init method call are:

  - SA_APP_HOST: address of current application's TQ1 API (without '/' at the end);
  - SA_APP_KEY: the identifier of the application inside TQ1 API;
  - GCM_SENDER_ID: the twelve-digits project number as present in Google Developers Console;
  - GEOTRIGGER_CLIENT_ID: the application id to access the geotrigger service server;
  - GEOTRIGGER_DEFAULT_TAGS: an array with the initial device tags;
  - UserDialog: as explained [here](user-alerts).

At any moment you can retrieve or change the tracking profile.

```java
TQ.setTrackingProfile(true); // enables the "Adaptive" mode
TQ.setTrackingProfile(false); // disables the service ("off" mode)

TQ.checkTrackingProfile(); // returns true or false
```

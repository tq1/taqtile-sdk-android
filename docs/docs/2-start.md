# Start TQ1 without geonotification services

In order to start TQ1 you need to call 2 methods. One will give TQ1 all the information needed and the other will start the information gathering. They can be implemented as follows:

```java
TQ.shared().init(TQ1_KEY, GCM_ID, getApplicationContext());
TQ.shared().start();
```

Just remember to replace the `TQ1_KEY` with the app key provided by us and the `GCM_ID` with you Google project number.

# Start TQ1 with ESRI as geolocation service

- You can start TQ1 on the `onCreate` step of your launcher activity:

```java
public class MainActivity extends Activity {

    private final String TQ1_KEY = "";
    private final String GCM_ID = "";

    @Override
      protected void onCreate(Bundle savedInstanceState) {
          ...
          setupTQ1();
      }

    private void setupTQ1() {
        final Activity activity = this;
        ServiceConnection mServerConn = new ServiceConnection() {
            @Override
            public void onServiceConnected(ComponentName componentName, IBinder iBinder) {
                ESRIManager.LocalBinder mBinder = (ESRIManager.LocalBinder)iBinder;
                mBinder.getService();
                TQGeotrigger.shared().setManager(activity, mBinder.getService());

                TQ.shared().init(TQ1_KEY, GCM_ID, getApplicationContext());
                TQ.shared().start();
            }

            @Override
            public void onServiceDisconnected(ComponentName componentName) {}
        };

        Intent intent = new Intent(getApplicationContext(), ESRIManager.class);
        bindService(intent, mServerConn, Context.BIND_AUTO_CREATE);
    }
}
```

- Marshmallow support:

If you intend to support Android Marshmallow (Api level 23), there are noticeable changes in the [permission system](http://developer.android.com/training/permissions/requesting.html). You can ask for authorization using the following method:

```java
if (!TQ.shared().isLocationAuthorized(activity)) {
    TQ.shared().askForLocationPermission(activity);
}
```

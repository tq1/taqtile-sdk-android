##Custom content

The application can pass to the API any custom information needed about the user through the method *addCustomContent*. Below follows an example implementation:

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
  super.onCreate(savedInstanceState);
  [...]
  Map<String, String> meta = new HashMap<String, String>();
  meta.put("key1", "value1");
  meta.put("key2", "value2");
  TQAnalytics.shared().addCustomContent(meta);
  }
```

##Sessions count

There are two distinct approaches here. If your project already has a class extending *Application* the first solution fits best. Otherwise the second solution is advised.

  - First option:

```java
public class MyApplication extends Application {

  private ActivityLifecycleCallbacks mListener = null;
  private boolean mBound = false;

  @Override
  public void onCreate() {
    super.onCreate();
    [...]
    Intent intent = new Intent(getApplicationContext(), TQLifecycleListener.class);
    startService(intent);
    bindService(intent, mServerConn, Context.BIND_AUTO_CREATE);
  }
  protected ServiceConnection mServerConn = new ServiceConnection() {
    @Override
    public void onServiceConnected(ComponentName componentName, IBinder iBinder) {
      TQLifecycleListener.LocalBinder mLocalBinder = (TQLifecycleListener.LocalBinder)iBinder;
      mBound = true;
      mListener = mLocalBinder.getInstance();
      registerActivityLifecycleCallbacks(mListener);
    }
    @Override
    public void onServiceDisconnected(ComponentName componentName) {
      mBound = false;
      mListener = null;
    }
  };
}
```

  - Second option:

In the field *android:name* inside the application tag descriptor, put a reference to an Application provided by our SDK.

```xml
<application
    android:name="taqtile.android.sdk.common.TQApplication"
    android:allowBackup="true"
    android:icon="@drawable/ic_launcher"
    android:label="@string/app_name">
    [...]
    </application>
```

##Events

You can easily send to our API information of what screens the user has been navigating in your application (or any other kind of event you want). Just add the code below on every point of interest, replacing "Welcome" with any other identifier:

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
  super.onCreate(savedInstanceState);
  [...]
  Map<String, String> seg = new HashMap<String, String>();
  seg.put("Page name", "Welcome");
  TQAnalytics.shared().recordEvent("Page Views", seg, 1);
}
```

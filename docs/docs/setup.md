# SDK Project

This guide only supports Android Studio projects. If you use another platform, some steps might be different.

- Confirm that your project reads JCenter repositories. You should see this on your main `build.gradle` file
    
```java
allprojects {
  repositories {
    jcenter()
  }
}
```

- Then add the SDK dependency to your app specific `build.gradle` file

```java
dependencies {
  compile "com.taqtile:android-sdk:3.0.2"
}
```

# Remote Notifications

You must create a service that extends [GcmListenerService](https://developers.google.com/android/reference/com/google/android/gms/gcm/GcmListenerService) and override the method `onMessageReceived` to handle the incoming push notification.

```java
public class MyListenerService extends GcmListenerService {

    @Override
    public void onMessageReceived(String from, Bundle data) {
        TQ.shared().init(TQ1_KEY, GCM_ID, getApplicationContext());
        ...
    }

}
```

This service must also be referenced in your application `Manifest.xml` file (note that the main `android:name` field should match your package name) together with the `GcmReceiver` and the GCM permissions:

```xml
<manifest ...>
  <uses-permission
    android:name="android.permission.WAKE_LOCK" />
  <permission
    android:name="com.example.permission.C2D_MESSAGE"
    android:protectionLevel="signature" />
  <uses-permission 
    android:name="com.example.permission.C2D_MESSAGE" />
  
  <application ...>
    <receiver
      android:name="com.google.android.gms.gcm.GcmReceiver"
      android:exported="true"
      android:permission="com.google.android.c2dm.permission.SEND" >
      <intent-filter>
        <action android:name="com.google.android.c2dm.intent.RECEIVE" />
        <category android:name="com.example.gcm" />
      </intent-filter>
    </receiver>

    <service
      android:name="com.example.MyGcmListenerService"
      android:exported="false" >
      <intent-filter>
        <action android:name="com.google.android.c2dm.intent.RECEIVE" />
      </intent-filter>
    </service>
  </application>
</manifest>
```

# TQG Geotrigger Service

- Configure dependency

Add TQG's SDK information to the application `build.gradle` file as a dependency:

```java
dependencies {
  ...
  compile 'com.taqtile:tqg-android-sdk:0.3.2'
}
```

- TQG wrapper class

```java
public class TQGManager implements TQGeoTriggerManager {

    private final String TAG = TQGManager.class.getCanonicalName();

    private final String TQG_ID = "5566417da058220300a8d90a";

    private Activity mActivity;
    private TQGeoTriggerConfigureHandler mHandler;

    @Override
    public void configure(Activity activity, TQGeoTriggerConfigureHandler handler) {
        mActivity = activity;
        mHandler = handler;

        TQGeoTracker.sharedInstance().configure(activity, TQG_ID, TQGeoTracker.TriggerMode.LOCAL_ONLY,
                TQGeoTracker.TQEnvironment.DEVELOPMENT);

        mHandler.onFinish();
    }

    @Override
    public void start() {
        TQGeoTracker.sharedInstance().start(mActivity);
    }

    @Override
    public void stop() {
        TQGeoTracker.sharedInstance().stop(mActivity);
    }

    @Override
    public void pause() {
        stop();
    }

    @Override
    public void resume() {
        start();
    }

    @Override
    public String getDeviceId() {
        return TQGeoTracker.sharedInstance().getDeviceId(mActivity);
    }

}
```

The wrapper class must be passed on `TQGeotrigger` initialization. For more information on this matter, see the `start` menu on the left.

- Application Manifest

Add the required permission for location usage:

```xml
<manifest>
    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE"/>
</manifest>
```

# ESRI Geotrigger Service

- Configure dependency

You must add ESRI's repo to the main `build.gradle` file on the `allprojects` session. It should end up like this:

```java
allprojects {
  repositories {
    jcenter()
    maven {
      url 'http://dl.bintray.com/esri/android'
    }
  }
}
```

Also add ESRI's SDK information to the application `build.gradle` file as a dependency:

```java
dependencies {
  ...
  compile 'com.esri.android.geotrigger:geotrigger-sdk:1.1.0'
}
```

- Add initialization classes

```java
import android.annotation.SuppressLint;
import android.app.Activity;
import android.app.Dialog;
import android.content.Context;
import android.content.DialogInterface;
import android.location.LocationManager;
import android.net.wifi.WifiManager;
import android.os.Build;
import android.provider.Settings;
import android.util.Log;
import android.widget.Toast;
import com.esri.android.geotrigger.GeotriggerService;
import com.google.android.gms.common.ConnectionResult;
import com.google.android.gms.common.GooglePlayServicesUtil;

import taqtile.android.sdk.common.TQUserDialog;

public final class GeotriggerHelper {
    private static final String TAG = "GeotriggerHelper";
    private static boolean sSkipPlayServicesInstall;

    private GeotriggerHelper() {}

    private enum AvailableProviders {GPS, NETWORK, BOTH, NEITHER}

    public static void startGeotriggerService(final Activity activity, String clientId, String senderId, 
            String[] tags, String profile, TQUserDialog errorDialog) {
        startGeotriggerService(activity, Integer.MIN_VALUE, clientId, senderId, tags, profile, errorDialog);
    }

    @SuppressLint("NewApi") 
    public static void startGeotriggerService(final Activity activity, int requestCode, String clientId,
                              String senderId, String[] tags, String profile, TQUserDialog errorDialog) {
        if (activity == null) {
            throw new IllegalArgumentException("Activity cannot be null.");
        }

        int status = GooglePlayServicesUtil.isGooglePlayServicesAvailable(activity);

        if (status != ConnectionResult.SUCCESS && !sSkipPlayServicesInstall) {
            if (GooglePlayServicesUtil.isUserRecoverableError(status)) {
                Dialog playServicesDialog = GooglePlayServicesUtil.getErrorDialog(status, activity,
                        requestCode, new DialogInterface.OnCancelListener() {

                    @Override
                    public void onCancel(DialogInterface dialog) {
                        Toast.makeText(activity, "Google Play Services will help conserve your battery!",
                                Toast.LENGTH_LONG).show();

                        sSkipPlayServicesInstall = true;
                    }
                });

                if (playServicesDialog != null) {
                    playServicesDialog.show();
                }
            } else {
                Log.d(TAG, "Google Play Services not available, and cannot be installed on this device.");
            }

        } else {
            Log.d(TAG, "Google Play Services is available (or user has declined to install it). " +
                    "Checking on GPS and Network providers.");

            if (checkForRequiredProviders(activity, errorDialog) == AvailableProviders.BOTH) {
                WifiManager wifiManager = (WifiManager) activity.getSystemService(Context.WIFI_SERVICE);
                if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.JELLY_BEAN_MR2) {
                    if (!wifiManager.isScanAlwaysAvailable()) {
                        Log.d("Logger", "bg_scanning_enabled&{\"is_bg_scanning_enabled\":false}");
                        if(errorDialog != null) {
                            errorDialog.showSettingsDialog(activity, WifiManager.ACTION_REQUEST_SCAN_ALWAYS_AVAILABLE, 0);
                        }
                        return;
                    }
                    else{
                        Log.d("Logger", "bg_scanning_enabled&{\"is_bg_scanning_enabled\":true}");
                    }
                } else {
                    Log.d("Logger", "bg_scanning_enabled&{\"is_bg_scanning_enabled\":false}");
                    if (!wifiManager.isWifiEnabled()) {
                        if(errorDialog != null)
                            errorDialog.showSettingsDialog(activity, Settings.ACTION_WIFI_SETTINGS, 1);
                        return;
                    }
                }

                GeotriggerService.init(activity, clientId, senderId, tags, profile);
            } else {
                Log.d(TAG, "Delaying the start of Geotriggers, as we are awaiting the availability of " +
                        "at least one provider.");
            }
        }
    }

    private static AvailableProviders checkForRequiredProviders(final Context context, TQUserDialog errorDialog) {
        if (context == null) {
            throw new IllegalArgumentException("Context cannot be null.");
        }

        AvailableProviders returnVal;
        LocationManager locationManager = (LocationManager) context.getSystemService(Context.LOCATION_SERVICE);

        boolean gotGps = locationManager.isProviderEnabled(LocationManager.GPS_PROVIDER);
        boolean gotNetwork = locationManager.isProviderEnabled(LocationManager.NETWORK_PROVIDER);

        int msg;
        if (!gotGps && !gotNetwork) {
            msg = 2;
            returnVal = AvailableProviders.NEITHER;
            Log.d("Logger", "gps_mode&{\"gps_mode\":\"None\"}");
        } else if (!gotGps) {
            msg = 3;
            returnVal = AvailableProviders.NETWORK;
            Log.d("Logger", "gps_mode&{\"gps_mode\":\"Network\"}");
        } else if (!gotNetwork) {
            msg = 4;
            returnVal = AvailableProviders.GPS;
            Log.d("Logger", "gps_mode&{\"gps_mode\":\"GPS\"}");
        } else {
            Log.d(TAG, "Both GPS and Network providers are available.");
            Log.d("Logger", "gps_mode&{\"gps_mode\":\"GPS and Network\"}");
            return AvailableProviders.BOTH;
        }
        if(errorDialog != null)
            errorDialog.showSettingsDialog(context, android.provider.Settings.ACTION_LOCATION_SOURCE_SETTINGS, msg);

        return returnVal;
    }
}
```

```java
import android.app.AlertDialog;
import android.content.Context;
import android.content.DialogInterface;
import android.content.Intent;

import taqtile.android.sdk.common.TQUserDialog;

public class ErrorDialog implements TQUserDialog {

    public ErrorDialog() {}

    @Override
    public void showSettingsDialog(Context context, String action, int msg) {
        final Context fContext = context;
        final String fAction = action;
        String alert = null;
        switch(msg) {
            case 0:
                //"Please enable background scanning!"
                alert = "";
                break;
            case 1:
                //"Please enable wifi for accurate locations!"
                alert = "";
                break;
            case 2:
                //"GPS and Network Location Services are both unavailable!"
                alert = "";
                break;
            case 3:
                //"GPS Location Services are unavailable!"
                alert = "";
                break;
            case 4:
                //"Network Location Services are unavailable!"
                alert = "";
                break;
        }

        AlertDialog.Builder dialogBuilder = new AlertDialog.Builder(context);
        dialogBuilder.setMessage(alert);
        dialogBuilder.setPositiveButton("settings", new DialogInterface.OnClickListener() {
            @Override
            public void onClick(DialogInterface dialog, int which) {
                fContext.startActivity(new Intent(fAction));
            }
        });

        dialogBuilder.setNegativeButton("cancel", new DialogInterface.OnClickListener() {
            @Override
            public void onClick(DialogInterface dialog, int which) {}
        });

        dialogBuilder.show();
    }
}

```

- Add geolocation wrapper service

```java
import android.app.Activity;
import android.app.Service;
import android.content.Intent;
import android.location.Location;
import android.os.Binder;
import android.os.IBinder;
import android.support.annotation.Nullable;

import com.esri.android.geotrigger.GeotriggerBroadcastReceiver;
import com.esri.android.geotrigger.GeotriggerService;

import taqtile.android.sdk.TQGeoTriggerConfigureHandler;
import taqtile.android.sdk.TQGeoTriggerManager;

public class ESRIManager extends Service implements TQGeoTriggerManager, GeotriggerBroadcastReceiver.ReadyListener,
        GeotriggerBroadcastReceiver.LocationUpdateListener {

    private final String TAG = ESRIManager.class.getCanonicalName();
    private final String ESRI_ID = "";
    private final String GCM_ID = "";
    private final String[] ESRI_TAGS = [""];


    private Activity mActivity;
    private TQGeoTriggerConfigureHandler mHandler;
    private GeotriggerBroadcastReceiver mReceiver;
    private final LocalBinder mBinder = new LocalBinder();

    @Override
    public void configure(Activity activity, TQGeoTriggerConfigureHandler handler) {
        mActivity = activity;
        mHandler = handler;
        mReceiver = new GeotriggerBroadcastReceiver();
        registerReceiver(mReceiver, GeotriggerBroadcastReceiver.getDefaultIntentFilter());
    }

    @Override
    public void start() {
        GeotriggerHelper.startGeotriggerService(mActivity, ESRI_ID, GCM_ID, ESRI_TAGS, 
                GeotriggerService.TRACKING_PROFILE_ADAPTIVE, new ErrorDialog());

        GeotriggerService.setPushNotificationHandlingEnabled(mActivity, false);
    }

    @Override
    public void stop() {
        unregisterReceiver(mReceiver);
        GeotriggerService.setTrackingProfile(mActivity, GeotriggerService.TRACKING_PROFILE_OFF);
    }

    @Override
    public void pause() {
        stop();
    }

    @Override
    public void resume() {
        start();
    }

    @Override
    public String getDeviceId() {
        return GeotriggerService.getDeviceId(mActivity);
    }

    @Override
    public void onReady() {
        mHandler.onFinish();
    }

    @Override
    public void onLocationUpdate(Location location, boolean b) {
        // handle location updates
    }

    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        return mBinder;
    }

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        super.onStartCommand(intent, flags, startId);

        return START_NOT_STICKY;
    }

    public class LocalBinder extends Binder {
        public ESRIManager getService() {
            return ESRIManager.this;
        }
    }

}
```

- Edit application's Manifest

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest
  xmlns:android="http://schemas.android.com/apk/res/android"
  package="com.taqtile.tq1example" >

    ...
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />

    <application
        android:allowBackup="true"
        android:icon="@drawable/ic_launcher"
        android:label="@string/app_name"
        android:theme="@style/AppTheme" >
        
        ...
        <service
            android:name="com.esri.android.geotrigger.GeotriggerService"
            android:exported="false" />

        <service android:name=".geolocation.ESRIManager"
            android:exported="false" />
        ...

    </application>

</manifest>
```
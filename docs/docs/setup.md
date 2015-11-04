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

# Google Cloud Messaging

Google Cloud Messaging (GCM) is the name of the service that manages push notifications on Android. The official documentation can be found [here](https://developers.google.com/cloud-messaging/android/client). We will reproduce here the steps presented in the official guide:

- Add GCM dependency to your application

```java
dependencies {
  compile "com.google.android.gms:play-services-gcm:7.0.0"
}
```

- Add GCM aware classes

You must create two new classes in your application: one WakefulBroadcastReceiver and one IntentService. Below, we have one possible implementation for each of those classes:

```java
import android.app.Activity;
import android.content.ComponentName;
import android.content.Context;
import android.content.Intent;
import android.support.v4.content.WakefulBroadcastReceiver;

public class GcmBroadcastReceiver extends WakefulBroadcastReceiver {

  @Override
  public void onReceive(Context context, Intent intent) {
    ComponentName comp = new ComponentName(context.getPackageName(),
          GcmIntentService.class.getName());
    startWakefulService(context, (intent.setComponent(comp)));
    setResultCode(Activity.RESULT_OK);
  }

}
```

```java
import android.app.IntentService;
import android.content.Intent;
import android.os.Bundle;
import android.util.Log;

import com.google.android.gms.gcm.GoogleCloudMessaging;

public class GcmIntentService extends IntentService {

  private static final String TAG = GcmIntentService.class.getSimpleName();

  public GcmIntentService() {
    super("GcmIntentService");
  }

  @Override
  protected void onHandleIntent(Intent intent) {
    Bundle extras = intent.getExtras();

    GoogleCloudMessaging gcm = GoogleCloudMessaging.getInstance(this);
    String messageType = gcm.getMessageType(intent);

    if (!extras.isEmpty()) {
      if (GoogleCloudMessaging.MESSAGE_TYPE_MESSAGE.equals(messageType)) {
        handlePushNotification(extras);
      }
    }
    GcmBroadcastReceiver.completeWakefulIntent(intent);
  }

  private void handlePushNotification(Bundle extras) {
    // process the notification that just arrived
  }

}
```

- Edit application's Manifest

Add the required permissions and declare both the BroadcastReceiver and the IntentService:

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest 
  xmlns:android="http://schemas.android.com/apk/res/android"
  package="com.taqtile.tq1example" >

  <uses-permission android:name="android.permission.INTERNET" />
  <uses-permission android:name="android.permission.WAKE_LOCK" />
  <uses-permission android:name="com.google.android.c2dm.permission.RECEIVE" />

  <permission
    android:name=".gcm.permission.C2D_MESSAGE"
    android:protectionLevel="signature" />

  <uses-permission android:name=".gcm.permission.C2D_MESSAGE" />

  <application
    android:allowBackup="true"
    android:icon="@drawable/ic_launcher"
    android:label="@string/app_name"
    android:theme="@style/AppTheme" >

    ...
    <receiver
      android:name=".gcm.GcmBroadcastReceiver"
      android:permission="com.google.android.c2dm.permission.SEND" >
      <intent-filter>
        <action android:name="com.google.android.c2dm.intent.RECEIVE" />
        <category android:name="com.taqtile.tq1example" />
      </intent-filter>
    </receiver>

    <service
      android:name=".gcm.GcmIntentService"
      android:exported="false" />
    ...

  </application>

</manifest>

```

In the above manifest file, `com.taqtile.tq1example` is the main application package, whereas `com.taqtile.tq1example.gcm` is the package that contains both classes.

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
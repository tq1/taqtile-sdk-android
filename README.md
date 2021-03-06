taqtile-sdk-android
===================

The present tutorial aims to help implementing the TaqtileSDK in projects based on Android Studio.

Gradle
---------
To configure the build operations, both project's and applications's build.gradle files must be edited.

Project:

    buildscript {
      repositories {
        mavenCentral()
        flatDir {
          dir 'libs'
        }
      }
      dependencies {
        classpath 'com.android.tools.build:gradle:0.12.2'
        classpath 'de.undercouch:gradle-download-task:1.0'
      }
    }
    
    allprojects {
      repositories {
        mavenCentral()
        flatDir {
          dir 'libs'
        }
      }
    }
    
Application:

    apply plugin: 'com.android.application'
    apply plugin: 'download-task'
    
    import de.undercouch.gradle.tasks.download.Download
    
    android {
      compileSdkVersion 19
      buildToolsVersion "19.1.0"
    
      defaultConfig {
        applicationId "com.taqtile.tq1example"
        minSdkVersion 14
        targetSdkVersion 19
        versionCode 1
        versionName "0.0.1"
      }
      buildTypes {
        release {
          runProguard false
          proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
      }
    }
    
    def sdkVersion = '1.0.2'
    def sdkProjectPrefix = 'taqtile-sdk-android'
    def sdkLibPrefix = 'taqtile-sdk'
    def librariesDir = "${projectDir}//libs"
    
    dependencies {
      compile fileTree(dir: 'libs', include: ['*.jar'])
      compile "com.taqtile:${sdkLibPrefix}:${sdkVersion}@aar"
    }
    
    dependencies {
      compile 'com.google.android.gms:play-services:4.2.+'
    }
    
    task downloadZipFile(type: Download) {
      src "https://github.com/shingle/${sdkProjectPrefix}/archive/${sdkVersion}.zip"
      dest new File(librariesDir, "${sdkVersion}.zip")
    }
    
    task downloadAndUnzipFile(dependsOn: downloadZipFile, type: Copy) {
      from zipTree(downloadZipFile.dest)
      into librariesDir
    }
    
    task extractLibFile(dependsOn: downloadAndUnzipFile, type: Copy) {
      from "${librariesDir}//${sdkProjectPrefix}-${sdkVersion}//TaqtileSDK//${sdkLibPrefix}-${sdkVersion}.aar"
      into librariesDir
    }
    
    task downloadSDK(dependsOn: extractLibFile, type: Delete) {
      delete "${librariesDir}//${sdkProjectPrefix}-${sdkVersion}"
      delete "${librariesDir}//${sdkVersion}.zip"
    }

Code
---------

MainActivity.java:

    @Override
    protected void onCreate(Bundle savedInstanceState) {
      super.onCreate(savedInstanceState);
      
      [...]
      
      TQ.shared().init(this, Constants.SA_APP_HOST, Constants.SA_APP_KEY,
          Constants.GCM_SENDER_ID, Constants.ARCGIS_CLIENT_ID, Constants.ARCGIS_DEFAULT_TAGS);
    }

    @Override
    protected void onStart() {
      super.onStart();

      [...]
        
      TQ.shared().onStart(this);
    }

    @Override
    protected void onStop() {
      super.onStop();

      [...]
      
      TQ.shared().onStop(this);
    }

Any activity relevant for analytics:

    @Override
    protected void onCreate(Bundle savedInstanceState) {
      super.onCreate(savedInstanceState);

      [...]

      Map<String, String> seg = new HashMap<String, String>();
      seg.put("Page name", "");
      TQAnalytics.shared().recordEvent("Page Views", seg, 1);
    }
    
Any activity reachable through notification redirection:

    @Override
    protected void onCreate(Bundle savedInstanceState) {
      super.onCreate(savedInstanceState);

      [...]

      TQInbox.addPushToConnectionQueue(getIntent());
    }
    
AndroidManifest.xml:

    <service
      android:name="com.esri.android.geotrigger.GeotriggerService"
      android:exported="false" />

    <service android:name="taqtile.android.sdk.TQGeotrigger"
      android:exported="false" />

    <service
      android:name="taqtile.android.sdk.push.TQPush"
      android:exported="false" />

    <receiver
      android:name="taqtile.android.sdk.push.TQGCMBroadcastReceiver"
      android:permission="com.google.android.c2dm.permission.SEND" >
      <intent-filter>
        <action android:name="com.google.android.c2dm.intent.RECEIVE" />
        <action android:name="com.google.android.c2dm.intent.REGISTRATION" />
        <category android:name="com.taqtile.tq1example" />
      </intent-filter>
    </receiver>

Dialogs
---------

In order to achieve a reasonable performance with ESRI's Geotrigger℠ Service, the user must enable
some of the location/connection related device features. To show the user all the missing features 
create a class that implements the required method of the interface TQUserDialog _showSettingsDialog_.
How it appears to the user can be customized here based on the alert message.

    public class UserDialog implements TQUserDialog {

    public UserDialog() {}

    @Override
    public void showSettingsDialog(Context context, String action, int msg) {
        final Context fContext = context;
        final String fAction = action;
        String alert = null;
        switch(msg) {
            case 0:
                //"Please enable background scanning!"
                alert = context.getString(R.string.geotrigger_alert_0);
                break;
            case 1:
                //"Please enable wifi for accurate locations!"
                alert = context.getString(R.string.geotrigger_alert_1);
                break;
            case 2:
                //"GPS and Network Location Services are both unavailable!"
                alert = context.getString(R.string.geotrigger_alert_2);
                break;
            case 3:
                //"GPS Location Services are unavailable!"
                alert = context.getString(R.string.geotrigger_alert_3);
                break;
            case 4:
                //"Network Location Services are unavailable!"
                alert = context.getString(R.string.geotrigger_alert_4);
                break;
        }

        AlertDialog.Builder dialogBuilder = new AlertDialog.Builder(context);
        dialogBuilder.setMessage(alert);
        dialogBuilder.setPositiveButton(context.getString(R.string.geotrigger_alerts_change), 
                                                      new DialogInterface.OnClickListener() {
            @Override
            public void onClick(DialogInterface dialog, int which) {
                fContext.startActivity(new Intent(fAction));
            }
        });

        dialogBuilder.setNegativeButton(context.getString(R.string.geotrigger_alerts_cancel), 
                                                      new DialogInterface.OnClickListener() {
            @Override
            public void onClick(DialogInterface dialog, int which) {}
        });

        dialogBuilder.show();
      }
    }
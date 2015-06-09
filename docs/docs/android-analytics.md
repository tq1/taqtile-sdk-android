[block:api-header]
{
  "type": "basic",
  "title": "Custom Content"
}
[/block]
The application can pass to the API any custom information needed about the user through the method *addCustomContent*. Below follows an example implementation:
[block:code]
{
  "codes": [
    {
      "code": "@Override\nprotected void onCreate(Bundle savedInstanceState) {\n    super.onCreate(savedInstanceState);\n    [...]\n    Map<String, String> meta = new HashMap<String, String>();\n    meta.put(\"key1\", \"value1\");\n    meta.put(\"key2\", \"value2\");\n    TQAnalytics.shared().addCustomContent(meta);\n}",
      "language": "java",
      "name": "HomeActivity.java"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Sessions Count"
}
[/block]
There are two distinct approaches here. If your project already has a class extending *Application* the first solution fits best. Otherwise the second solution is advised.

+ First option:
[block:code]
{
  "codes": [
    {
      "code": "public class MyApplication extends Application {\n\n    private ActivityLifecycleCallbacks mListener = null;\n    private boolean mBound = false;\n\n    @Override\n    public void onCreate() {\n        super.onCreate();\n\t\t\t\t[...]\n        Intent intent = new Intent(getApplicationContext(), TQLifecycleListener.class);\n      \tstartService(intent);\n        bindService(intent, mServerConn, Context.BIND_AUTO_CREATE);\n    }\n\n    protected ServiceConnection mServerConn = new ServiceConnection() {\n        @Override\n        public void onServiceConnected(ComponentName componentName, IBinder iBinder) {\n            TQLifecycleListener.LocalBinder mLocalBinder = (TQLifecycleListener.LocalBinder)iBinder;\n            mBound = true;\n            mListener = mLocalBinder.getInstance();\n            registerActivityLifecycleCallbacks(mListener);\n        }\n        @Override\n        public void onServiceDisconnected(ComponentName componentName) {\n            mBound = false;\n            mListener = null;\n        }\n    };\n}",
      "language": "java",
      "name": "Project/app/MyApplication.java"
    }
  ]
}
[/block]
+ Second option:

In the field *android:name* inside the application tag descriptor, put a reference to an Application provided by our SDK.
[block:code]
{
  "codes": [
    {
      "code": "<application\n    android:name=\"taqtile.android.sdk.common.TQApplication\"\n    android:allowBackup=\"true\"\n    android:icon=\"@drawable/ic_launcher\"\n    android:label=\"@string/app_name\">\n    [...]\n</application>",
      "language": "xml",
      "name": "Project/app/AndroidManifest.xml"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Events"
}
[/block]
You can easily send to our API information of what screens the user has been navigating in your application (or any other kind of event you want). Just add the code below on every point of interest, replacing "Welcome" with any other identifier:
[block:code]
{
  "codes": [
    {
      "code": "@Override\nprotected void onCreate(Bundle savedInstanceState) {\n    super.onCreate(savedInstanceState);\n\t\t[...]\n    Map<String, String> seg = new HashMap<String, String>();\n    seg.put(\"Page name\", \"Welcome\");\n    TQAnalytics.shared().recordEvent(\"Page Views\", seg, 1);\n}",
      "language": "java",
      "name": "AnyActivity.java"
    }
  ]
}
[/block]
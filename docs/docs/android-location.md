The Geotrigger Service must be started from the onCreate method of the launcher activity:
[block:code]
{
  "codes": [
    {
      "code": "@Override\nprotected void onCreate(Bundle savedInstanceState) {\n    super.onCreate(savedInstanceState);\n    [...]\n  \tTQ.shared().setActivity(this);\n    TQ.shared().init(SA_APP_HOST, SA_APP_KEY, GCM_SENDER_ID, ARCGIS_CLIENT_ID, ARCGIS_DEFAULT_TAGS, new UserDialog());\n}",
      "language": "java",
      "name": "HomeActivity.java"
    }
  ]
}
[/block]
The arguments passed to the init method call are:
+ SA_APP_HOST: address of current application's TQ1 API (without '/' at the end);
+ SA_APP_KEY: the identifier of the application inside TQ1 API;
+ GCM_SENDER_ID: the twelve-digits project number as present in Google Developers Console;
+ ARCGIS_CLIENT_ID: the application id to access the ESRI server;
+ ARCGIS_DEFAULT_TAGS: an array with the initial device tags;
+ UserDialog: as explained [here](doc:android-user-alerts).

At any moment you can retrieve or change the [Tracking Profile](https://developers.arcgis.com/geotrigger-service/guide/tracking-profiles).
[block:code]
{
  "codes": [
    {
      "code": "TQ.setTrackingProfile(true); // enables the \"Adaptive\" mode\nTQ.setTrackingProfile(false); // disables the service (\"off\" mode)\n\nTQ.checkTrackingProfile(); // returns true or false",
      "language": "java"
    }
  ]
}
[/block]
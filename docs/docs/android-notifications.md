[block:api-header]
{
  "type": "basic",
  "title": "Start Service"
}
[/block]
Before handling an incoming push notification, you have to restart all the SDK constants, as can be seen below (if you don't know what these constants are, see [here](doc:android-location) ):
[block:code]
{
  "codes": [
    {
      "code": "private void handlePushNotification(Bundle extras) {\n    TQ.shared().init(SA_APP_HOST, SA_APP_KEY, GCM_SENDER_ID, ARCGIS_CLIENT_ID, ARCGIS_DEFAULT_TAGS, new UserDialog());\n\t\t[...]\n}",
      "language": "java",
      "name": "GcmIntentService.java"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Show Notification"
}
[/block]
The bundle of a push notification coming from the TQ1 API contains only two extras: the *text* that holds the alert message, and the *pid*, a unique identifier in whole API. The content should be requested as shown below:
[block:code]
{
  "codes": [
    {
      "code": "String pushId = extras.getString(\"pid\");\nString alert = extras.getString(\"text\");\nlong timestamp = System.currentTimeMillis();\n\nTQInbox inbox = new TQInbox(mContext);\ninbox.addAlert(pushId, alert, String.valueOf(timestamp));\ninbox.retrieveCustomContent(pushId, new TQInbox.TQInboxCustomContentCallback() {\n    @Override\n    public void onSuccess(TQInboxMessage message) {\n        Log.d(TAG, \"Content retrieved successfully\");\n    }\n    @Override\n    public void onError() {\n        Log.d(TAG, \"Error requesting content\");\n    }\n});",
      "language": "java"
    }
  ]
}
[/block]
A TQInboxMessage object has some attributes accessible through these public methods:
[block:code]
{
  "codes": [
    {
      "code": "public String getId()\n\n/* Alert is the text shown to the user by the Notification Center */\npublic String getAlert()\n\n/* Content of the message shown when the user clicks on some notification */\npublic String getContent()\n\n/* The type of the notification specifies how the application will show an incoming content to the user */\npublic TQInboxMessageType getType()\n\n/* Status tells if the application was read at the present moment */\npublic TQInboxMessageStatus getStatus()\n  \n/* A message is complete when its content was downloaded successfully from the API */\npublic Boolean getComplete()\n\n/* Moment that the message arrived */\npublic String getTimestamp()",
      "language": "java"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Mark as Open"
}
[/block]
It's highly recommended to put extras in the pendingIntent of all notifications warning that this intent came directly from a push notification and what notification was it:
[block:code]
{
  "codes": [
    {
      "code": "pendingIntent.putExtra(\"fromPush\", true);\npendingIntent.putExtra(\"pid\", \"5436fe51abda3d02007ebc96\");\n[...]\nmBuilder.setContentIntent(pendingIntent);\nmBuilder.setAutoCancel(true);\n\nnotificationManager.notify(id, mBuilder.build());",
      "language": "java"
    }
  ]
}
[/block]
Now each activity can check if it came from a push notification. If this is the case, the TQ1 API is notified that the notification was open by the user:
[block:code]
{
  "codes": [
    {
      "code": "TQInbox.addPushToConnectionQueue(getIntent());",
      "language": "java"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Listing Notifications"
}
[/block]
There are methods available in the class *TQInbox* to list all the locally stored notifications. Below you can see an example of use:
[block:code]
{
  "codes": [
    {
      "code": "TQInbox inbox = new TQInbox(getApplicationContext());\nList<TQInboxMessage> messages = inbox.getInboxMessageList();\n// Alternatively you can filter passing a TQInboxMessageStatus:\n// messages = inbox.getInboxMessageList(TQInboxMessageStatus.UNREAD);",
      "language": "java"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Mark as Read"
}
[/block]
When the user clicks on a notification in the list and opens its content, this notification should be marked as read. The following method could be used for this:
[block:code]
{
  "codes": [
    {
      "code": "TQInbox inbox = new TQInbox(getApplicationContext());\ninbox.markAsRead(\"5436fe51abda3d02007ebc96\");",
      "language": "java"
    }
  ]
}
[/block]

[block:api-header]
{
  "type": "basic",
  "title": "Removing Multiple Notifications"
}
[/block]
There is a method available for removing all notifications specified by its `TQInboxMessageStatus`: *READ*, *UNREAD* or *ALL*.
[block:code]
{
  "codes": [
    {
      "code": "TQInbox inbox = new TQInbox(getApplicationContext());\ninbox.removeMessages(TQInboxMessageStatus.ALL);",
      "language": "java"
    }
  ]
}
[/block]
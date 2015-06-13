##Start service
Before handling an incoming push notification, you have to restart all the SDK constants, as can be seen below (if you don't know what these constants are, see [here](doc:android-location) ):

```java
private void handlePushNotification(Bundle extras) {
  TQ.shared().init(SA_APP_HOST, SA_APP_KEY, GCM_SENDER_ID, ARCGIS_CLIENT_ID, ARCGIS_DEFAULT_TAGS, new UserDialog());
  [...]
}
```

##Show notification
The bundle of a push notification coming from the TQ1 API contains only two extras: the *text* that holds the alert message, and the *pid*, a unique identifier in whole API. The content should be requested as shown below:

```java
String pushId = extras.getString("pid");
String alert = extras.getString("text");
long timestamp = System.currentTimeMillis();

TQInbox inbox = new TQInbox(mContext);
inbox.addAlert(pushId, alert, String.valueOf(timestamp));
inbox.retrieveCustomContent(pushId, new TQInbox.TQInboxCustomContentCallback() {
  @Override
  public void onSuccess(TQInboxMessage message) {
    Log.d(TAG, "Content retrieved successfully");
  }
  @Override
  public void onError() {
    Log.d(TAG, "Error requesting content");
  }
});
```

A TQInboxMessage object has some attributes accessible through these public methods:

```java
public String getId()

/* Alert is the text shown to the user by the Notification Center */
public String getAlert()
/* Content of the message shown when the user clicks on some notification */
public String getContent()
/* The type of the notification specifies how the application will show an incoming content to the user */
public TQInboxMessageType getType()

/* Status tells if the application was read at the present moment */
public TQInboxMessageStatus getStatus()

/* A message is complete when its content was downloaded successfully from the API */
public Boolean getComplete()
/* Moment that the message arrived */
public String getTimestamp()
```

##Mark as open
It's highly recommended to put extras in the pendingIntent of all notifications warning that this intent came directly from a push notification and what notification was it:

```java
pendingIntent.putExtra("fromPush", true);
pendingIntent.putExtra("pid", "5436fe51abda3d02007ebc96");
[...]
mBuilder.setContentIntent(pendingIntent);
mBuilder.setAutoCancel(true);

notificationManager.notify(id, mBuilder.build());
```

Now each activity can check if it came from a push notification. If this is the case, the TQ1 API is notified that the notification was open by the user:

```java
TQInbox.addPushToConnectionQueue(getIntent());
```

##Listing notifications
There are methods available in the class *TQInbox* to list all the locally stored notifications. Below you can see an example of use:

```java
TQInbox inbox = new TQInbox(getApplicationContext());
List<TQInboxMessage> messages = inbox.getInboxMessageList();
// Alternatively you can filter passing a TQInboxMessageStatus:
// messages = inbox.getInboxMessageList(TQInboxMessageStatus.UNREAD);
```

##Mark as read

When the user clicks on a notification in the list and opens its content, this notification should be marked as read. The following method could be used for this:

```java
TQInbox inbox = new TQInbox(getApplicationContext());\ninbox.markAsRead("5436fe51abda3d02007ebc96");
```

##Removing multiple notifications

There is a method available for removing all notifications specified by its `TQInboxMessageStatus`: *READ*, *UNREAD* or *ALL*.

```java
TQInbox inbox = new TQInbox(getApplicationContext());\ninbox.removeMessages(TQInboxMessageStatus.ALL);
```

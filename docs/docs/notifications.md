# Show notification
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
        // content retrieved successfully
    }

    @Override
    public void onError() {
        // error requesting content
    }
});
```

There are four distinct notification types: PUSH, HTML, LINK and TAG.

- PUSH
    
    Notifications with `PUSH` type are plain messages. The intended content is only the message exhibited by te notification center.

- HTML

    The content of these notifications is an HTML text that should be presented in a web view.

- LINK
    
    The content is only the url to a web page that, again, should be presented in a web view.

- TAG
    
    This simply brings a set of key/value fields that can be used to describe a redirection inside the application.

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

# Actions

If there are actions related to the notification, the bundle will contain a field called `actionId`. Based on this identifier, the application should be able to present the designated actions to the user.

# Mark as open

It's highly recommended to put extras in the pendingIntent of all notifications warning that this intent came directly from a push notification and what notification was it:

```java
pendingIntent.putExtra("fromPush", true);
pendingIntent.putExtra("pid", "5436fe51abda3d02007ebc96");

...
mBuilder.setContentIntent(pendingIntent);
mBuilder.setAutoCancel(true);

notificationManager.notify(id, mBuilder.build());
```

Now each activity can check if it came from a push notification. If this is the case, the TQ1 API is notified that the notification was open by the user:

```java
String pushId = getIntent().getStringExtra("pid");
TQAnalytics.shared().updateStatus(pushId, "open", getApplicationContext());
```

# Custom status

`open` is just one possiblity, you can attach any custom status value to a push notification using the same method. This is specially useful when registering the value chosen for notifications with actions:

```java
TQAnalytics.shared().updateStatus(pushId, "anything", getApplicationContext());
```

# Listing notifications

There are methods available in the class *TQInbox* to list all the locally stored notifications. Below you can see an example of use:

```java
TQInbox inbox = new TQInbox(getApplicationContext());
List<TQInboxMessage> messages = inbox.getInboxMessageList();
// Alternatively you can filter passing a TQInboxMessageStatus:
// messages = inbox.getInboxMessageList(TQInboxMessageStatus.UNREAD);
```

# Mark as read

When the user clicks on a notification in the list and opens its content, this notification should be marked as read. The following method could be used for this:

```java
TQInbox inbox = new TQInbox(getApplicationContext());
inbox.markAsRead("5436fe51abda3d02007ebc96");
```

# Removing multiple notifications

There is a method available for removing all notifications specified by its `TQInboxMessageStatus`: *READ*, *UNREAD* or *ALL*.

```java
TQInbox inbox = new TQInbox(getApplicationContext());
inbox.removeMessages(TQInboxMessageStatus.ALL);
```
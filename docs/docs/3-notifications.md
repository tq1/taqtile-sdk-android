# Receiving notifications
The push notification receiving process is all handled by the application, that should use GCM in order to register and receive it. Only after receiving, TQ1 is called to get the notification content.
Here are some example classes that will do all this handling:

- If you have followed the setup instructions, you will already have the following method, and here you will find a full example of push notification handling


```java
public class MyListenerService extends GcmListenerService {
    //This class will be the push entry point
    @Override
    public void onMessageReceived(String from, Bundle data) {
        super.onMessageReceived(from, data);

        // Initiating TQ1 here as soon as the push is received
        TQ.shared().init(Constants.TQ1_KEY, Constants.GCM_ID, getApplicationContext());
        NotificationRouter.setContext(getApplicationContext());

        // data will have all the push information needed
        NotificationRouter.routePushNotification(data);
    }
}
```

And here is the push handling

```java
public class NotificationRouter {

    private static final String TAG = NotificationRouter.class.getSimpleName();

    private static Context mContext;

    public static void setContext(Context context) {
        mContext = context;
    }

    private static void markAsRead(String contentId) {
        TQInbox inbox = new TQInbox(mContext);
        inbox.markAsRead(contentId);
    }

    public static void routePushNotification(Bundle extras) {
        if(extras.getString("pid") != null && extras.getString("text") != null) {
            formNotification(extras);
        }
    }

    public static void routePushNotification(TQInboxMessage message) {
        Bundle extras = new Bundle();

        markAsRead(message.getId());
        extras.putString("pid", message.getId());
        extras.putString("text", message.getAlert());

        extras = addContentToExtras(message, extras);

        Intent intent =  mapNotification(extras);
        intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        mContext.startActivity(intent);
    }

    private static void callNotificationIntent(Intent intent) {
        String title = mContext.getString(R.string.app_name);
        String message = intent.getStringExtra("text");

        TaskStackBuilder stackBuilder = TaskStackBuilder.create(mContext);
        stackBuilder.addNextIntentWithParentStack(intent);
        PendingIntent contentIntent =
                stackBuilder.getPendingIntent(0, PendingIntent.FLAG_CANCEL_CURRENT);

        sendNotification(title, message, contentIntent);
    }

    private static void sendNotification(String title, String msg, PendingIntent contentIntent) {
        NotificationManager notificationManager = (NotificationManager)
                mContext.getSystemService(Context.NOTIFICATION_SERVICE);

        NotificationCompat.Builder mBuilder = new NotificationCompat.Builder(mContext)
                        .setSound(Settings.System.DEFAULT_NOTIFICATION_URI)
                        .setSmallIcon(R.mipmap.ic_launcher)
                        .setContentTitle(title)
                        .setStyle(new NotificationCompat.BigTextStyle().bigText(msg))
                        .setContentText(msg);

        mBuilder.setContentIntent(contentIntent);
        mBuilder.setAutoCancel(true);
        notificationManager.notify(0, mBuilder.build());
    }

    private static Intent mapNotification(Bundle extras) {
        Intent intent;

        if(extras.containsKey("html")) {
            intent = new Intent(mContext, ContentActivity.class);
        } else if(extras.containsKey("link")) {
            intent = new Intent(mContext, ContentActivity.class);
        } else {
            intent = new Intent(mContext, MainActivity.class);
        }

        intent.putExtras(extras);

        return intent;
    }

    private static Bundle addContentToExtras(TQInboxMessage message, Bundle extras) {
         switch(message.getType()) {
            case TAG:
                break;
            case LINK:
                extras.putString("html", "");
            case HTML:
                extras.putString("link", "");
            case PUSH:
                extras.putString("push", "");
        }

        return extras;
    }

    private static void formNotification(final Bundle extras) {
        // Here we will get the information that came with the push
        final String pushId = extras.getString("pid");
        final String timestamp = String.valueOf(System.currentTimeMillis());
        final String alert = extras.getString("text");

        TQInbox inbox = new TQInbox(mContext);
        inbox.addAlert(pushId, alert, timestamp, "");
        // And here we will download the custom content, if there is one (Like Tags, HTML or links)
        inbox.retrieveCustomContent(pushId, new TQInbox.TQInboxCustomContentCallback() {
            @Override
            public void onSuccess(TQInboxMessage message) {
                Log.i(TAG, "Message stored in database");
                Bundle bundle = addContentToExtras(message, extras);
                // Here we start to build the intent for a redirection
                callNotificationIntent(mapNotification(bundle));
            }

            @Override
            public void onError() {
                Log.i(TAG, "Error requesting custom content");
            }
        });
    }

}

```

Just remember that these are just examples, you can handle the push notifications the way you thing it's best. The TQ1 SDK methods called here are explained later is this doc.

# Showing notifications
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

For more information about the available methods and attributes for each class, check our [Android API reference](http://tq1.github.io/br-tq1-sdk-android/).

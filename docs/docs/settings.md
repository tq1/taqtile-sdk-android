# Location

- You can check if the user location is enabled using the following method:

```java
TQDeviceInfo.isLocationEnabled();
```

- At any moment you can enable/disable geonotifications and this decision will persist even if the application is closed:

```java
TQ.setTrackingProfile(true);  // enables "adaptive" mode
TQ.setTrackingProfile(false); // disables the service ("off" mode)
```

# Push Notification

- You can also enable/disable remote notifications and this decision will persist even if the application is closed:

```java
TQDeviceInfo.setPushEnabled(context, true);
TQDeviceInfo.setPushEnabled(context, false);
```
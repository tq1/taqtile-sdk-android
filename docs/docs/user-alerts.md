In order to be notified as soon as the Geotrigger Service starts, a class that implements the interface *TQUserDialog* must be created. In this tutorial, the class will be called *UserDialog*.

```java
import taqtile.android.sdk.common.TQUserDialog;

public class UserDialog implements TQUserDialog {

  public UserDialog() {}
  @Override
  public void showSettingsDialog(Context context, String action, int message) {
    [...]
    // start the android settings application
    context.startActivity(new Intent(action));
    [...]
  }
}
```

The only method declared by this interface uses three arguments:

  1. (android.content.Context) context: this is the context of the activity passed in start up [(See Location)](doc:android-location);
  2. (java.lang.String) action: the action that can be passed as an argument to a new intent;
  3. (int)  message: each possible error message is passed as a different integer.

According to the [Official Documentation](https://github.com/Esri/geotrigger-sdk-android/blob/master/sample/res/values/strings.xml#L5-L9) of ESRI's Geotrigger Service, there are five possible error messages (*message* = 0, 1, ..., 4). During the implementation, the developer should decide which ones will be exhibited to the user and how:

  - "Please enable background scanning!"
  - "Please enable wifi for accurate locations!"
  - "GPS and Network Location Services are both unavailable!"
  - "GPS Location Services are unavailable!"
  - "Network Location Services are unavailable!"

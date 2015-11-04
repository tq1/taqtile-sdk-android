# Start TQ1

- You can start TQ1 on the `onCreate` step of your launcher activiy

```java
import android.app.Activity;
import android.content.ComponentName;
import android.content.Context;
import android.content.Intent;
import android.content.ServiceConnection;
import android.os.Bundle;
import android.os.IBinder;

import com.taqtile.tq1example.R;
import com.taqtile.tq1example.geolocation.ESRIManager;
import com.taqtile.tq1example.geolocation.ErrorDialog;

import taqtile.android.sdk.TQ;
import taqtile.android.sdk.TQGeotrigger;

public class MainActivity extends Activity {

    private final String TQ1_HOST = "";
    private final String TQ1_KEY = "";
    private final String GCM_ID = "";

    @Override
      protected void onCreate(Bundle savedInstanceState) {
          ...
          setupTQ1();
      }

    private void setupTQ1() {
        final Activity activity = this;
        ServiceConnection mServerConn = new ServiceConnection() {
            @Override
            public void onServiceConnected(ComponentName componentName, IBinder iBinder) {
                ESRIManager.LocalBinder mBinder = (ESRIManager.LocalBinder)iBinder;
                mBinder.getService();
                TQGeotrigger.shared().setManager(activity, mBinder.getService());

                TQ.shared().init(TQ1_HOST, TQ1_KEY, GCM_ID);
                TQ.shared().setActivity(activity);
                TQ.shared().start();
            }

            @Override
            public void onServiceDisconnected(ComponentName componentName) {}
        };

        Intent intent = new Intent(getApplicationContext(), ESRIManager.class);
        bindService(intent, mServerConn, Context.BIND_AUTO_CREATE);
    }
}
```
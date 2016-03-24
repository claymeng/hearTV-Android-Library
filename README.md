# hearTV Android Library

## See the Android Studio project in the **Example** directory.

## Installation
*todo*


## Usage

#### Service Connection
The hearTV library consists of a [Bound Service](http://developer.android.com/guide/components/bound-services.html) called `hearTVservice`.  To use the service, implement a `ServiceConnection` to monitor the connection with the service and to save a reference to the service.  
```
    private hearTVservice mHearTVservice;

    private ServiceConnection hearTVserviceConnection = new ServiceConnection() {

        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            hearTVservice.hearTVserviceBinder binder = (hearTVservice.hearTVserviceBinder) service;
            mHearTVservice = binder.getService();
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {
        
        }
    };
```
#### Intents
Before initializing the hearTV service, register receivers for the various intents broadcast by the service.

**`HTVIntentWillStartSearchingForSources`**  
This intent is broadcast when the service is searching for sources.  This may take several seconds, so it's a good idea to use this intent to display feedback to the user, such as an indeterminate progress.  For example: `progressDialog = ProgressDialog.show(this, "", "Finding hearTV Sources", true, false);`.

**`HTVIntentDidStopSearchingForSources`**  
This intent is broadcast when the service is done searching for sources.  Use this intent to update the UI.  For example: `progressDialog.dismiss();`.

**`HTVIntentSourceListChanged`**  
Use `public String[] getSourceList(boolean includeHiddenSources)` to retrieve the currently available audio sources.  Normally, `includeHiddenSources` should be `false`.

**`HTVIntentNoSourcesFound`**  
When this intent is broadcast, display a message to the user.  The contents of the message are defined by two string intent extras: `HTVIntentExtraMessage1` and `HTVIntentExtraMessage2`.

**`HTVIntentPlaybackStatusChanged`**  
Use `public HTVPlaybackStatus getCurrentPlaybackStatus()` to get the current status in an `HTVPlaybackStatus` object, which contains a `message` and `type`.

**`HTVIntentUpdateVolumeMeter`**  
You may optionally register a receiver for this intent to display a dynamic volume meter to the user.  This notification occurs several times per second.  To retrieve the volume, get the `float` intent extra `HTVIntentExtraVolume`.  For example: `float volume = intent.getFloatExtra(hearTVservice.HTVIntentExtraVolume, 0);`.

##### Example
**Register broadcast receivers:**
```
    private void registerReceivers() {
        registerReceiver(willStartSearchingForSources, hearTVservice.HTVIntentWillStartSearchingForSources);
        registerReceiver(didStopSearchingForSources  , hearTVservice.HTVIntentDidStopSearchingForSources);
        registerReceiver(sourceListChanged           , hearTVservice.HTVIntentSourceListChanged);
        registerReceiver(noSourcesFound              , hearTVservice.HTVIntentNoSourcesFound);
        registerReceiver(playbackStatusChanged       , hearTVservice.HTVIntentPlaybackStatusChanged);
        registerReceiver(updateVolume                , hearTVservice.HTVIntentUpdateVolumeMeter);
    }

    private void registerReceiver(BroadcastReceiver receiver, String intentString) {
        LocalBroadcastManager.getInstance(this).registerReceiver(receiver, new IntentFilter(intentString));
    }
```

**Broadcast Receiver for `HTVIntentWillStartSearchingForSources`**
```
    private BroadcastReceiver willStartSearchingForSources = new BroadcastReceiver() {
        @Override
        public void onReceive(Context context, Intent intent) {
            // Update UI.  For example, show an indeterminate progress dialog.
        }
    };
```

**Broadcast Receiver for `HTVIntentDidStopSearchingForSources`**
```
    private BroadcastReceiver didStopSearchingForSources = new BroadcastReceiver() {
        @Override
        public void onReceive(Context context, Intent intent) {
            // Update UI.  For example, hide an indeterminate progress dialog.
        }
    };
```

**Broadcast Receiver for `HTVIntentSourceListChanged`**
```
    private BroadcastReceiver sourceListChanged = new BroadcastReceiver() {
        @Override
        public void onReceive(Context context, Intent intent) {
            String[] sources = mHearTVservice.getSourceList(false);
            // Update the list of sources being displayed
        }
    };
```

**Broadcast Receiver for `HTVIntentNoSourcesFound`**
```
    private BroadcastReceiver noSourcesFound = new BroadcastReceiver() {
        @Override
        public void onReceive(Context context, Intent intent) {
            String errorMessage1 = intent.getStringExtra(hearTVservice.HTVIntentExtraMessage1);
            String errorMessage2 = intent.getStringExtra(hearTVservice.HTVIntentExtraMessage2);
            // Display an error message
        }
    };
```

**Broadcast Receiver for `HTVIntentPlaybackStatusChanged`**
```
    private BroadcastReceiver playbackStatusChanged = new BroadcastReceiver() {
        @Override
        public void onReceive(Context context, Intent intent) {
            hearTVservice.HTVPlaybackStatus status = mHearTVservice.getCurrentPlaybackStatus();
            // Show the status message.

            switch (status.type) {
                // Update the UI depending on the various statuses.
                // For example, display or hide playback controls as needed.
                // Possible values are:
                //
                // htvPlaybackStatusIdle
                // htvPlaybackStatusConnecting
                // htvPlaybackStatusConnectionFailed
                // htvPlaybackStatusListening
                // htvPlaybackStatusStopping
            }
        }
    };
```

**Broadcast Receiver for `HTVIntentUpdateVolumeMeter`**
```
    private BroadcastReceiver updateVolume = new BroadcastReceiver() {
        @Override
        public void onReceive(Context context, Intent intent) {
            float volume = intent.getFloatExtra(hearTVservice.HTVIntentExtraVolume, 0);
            // Update UI
        }
    };
```

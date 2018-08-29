
# Change Log

## [2.8.1] - 2017-08-21
- [Changed] Reference latest `cordova-plugin-background-fetch` version `5.0.0`
- [Added] Javascript API to plugin's logging system.
- [Fixed] Minor issue with iOS flush where multiple threads might create multiple background-tasks, leaving some unfinished.

## [2.8.0] - 2017-08-15
- [Changed] Refactor iOS/Android core library event-subscription API.
- [Changed] Removed `taskId` supplied to all event-callbacks.  You no longer have to call `bgGeo.finish(taskId)` in your event-callbacks (though the method will still operate as a `noop` for backwards compatibility).  You will now be responsible for creating your own iOS background-tasks using the method `#startBackgroundTask` when performing long-running tasks in your event-callbacks.
- [Added] iOS and Android now support ability to remove single event-listeners with method `#un`

## [2.7.5] - 2017-07-27
- [Changed] Remove dependency `cordova-plugin-dialogs`.  It's not required.
- [Changed] Improve iOS/Android acquisition of `motionchange` location to ensure a recent location is fetched.
- [Changed] Implement `#getSensors` method for both iOS & Android.  Returns an object indicating the presense of *accelerometer*, *gyroscope* and *magnetometer*.  If any of these sensors are missing, the motion-detection system for that device will poor.
- [Changed] The `activitychange` success callback method signature has been changed from `{String} activityName` -> `{Object}` containing both `activityName` as well as `confidence`.  This event only used to fire after the `activityName` changed (eg: `on_foot` -> `in_vehicle`), regardless of `confidence`.  This event will now fire for *any* change in activity, including `confidence` changes.
- [Changed] iOS `emailLog` will gzip the attached log file.
- [Added] Implement new Android config `notificationPriority` for controlling the behaviour of the `foregroundService` notification and notification-bar icon.
- [Fixed] Android was creating a foreground notification even when `foregroundService: false`
- [Changed] Tweak iOS Location Authorization to not show locationAuthorizationAlert if user initially denies location permission.
- [Fixed] Android:  Remove isMoving condition from geofence proximity evaluator.
- [Fixed] iOS 11 fix:  Added new location-authorization string `NSLocationAlwaysAndWhenInUseUsageDescription`.  iOS 11 now requires location-authorization popup to allow user to select either `Always` or `WhenInUse`.

## [2.7.4] - 2017-07-10
- [Fixed] Android & iOS will ensure old location samples are ignored with `getCurrentPosition` 
- [Fixed] Android `providerchange` event would continue to persist a providerchange location even when plugin was disabled for the case where location-services is disabled by user.
- [Fixed] Don't mutate iOS `url` to lowercase.  Just lowercase the comparison when checking for `301` redirects. 
- [Changed] Android will attempt up to 5 motionchange samples instead of 3.
- [Changed] Android foregroundService notification priority set to `PRIORITY_MIN` so that notification doesn't always appear on top.
- [Fixed] Android plugin was not nullifying the odometer reference location when `#stop` method is executed, resulting in erroneous odometer calculations if plugin was stopped, moved some distance then started again.
- [Added] Android plugin will detect presense of Sensors `ACCELEROMETER`, `GYROSCOPE`, `MAGNETOMETER` and `SIGNIFICANT_MOTION`.  If any of these sensors are missing, the Android `ActivityRecognitionAPI` is considered non-optimal and plugin will add extra intelligence to assist determining when device is moving.
- [Fixed] Bug in broadcast event `GEOFENCE` not being fired when `MainActivity` is terminated (only applies to those using a `BroadcastReceiver`).
- [Fixed] Android scheduler issue when device is rebooted and plugin is currently within a scheduled ON period (fails to start)
- [Fixed] (Android) Fix error calling `stopWatchPosition` before `#configure` callback has executed.  Also add support for executing `#getCurrentPosition` before `#configure` callback has fired.
- [Added] (Android) Listen to LocationResult while stopTimeout is engaged and perform manual motion-detection by checking if location distance from stoppedAtLocation is > stationaryRadius
- [Fixed] Bug in literal schedule parsing for both iOS and Android
- [Fixed] Bug in Android scheduler after app terminated.  Configured schedules were not having their `onTime` and `offTime` zeroed, resulting in incorrect time comparison.

## [2.7.3] - 2017-06-15
- [Fixed] Bug in Android scheduler after app terminated.  Configured schedules were not having their `SECOND` and `MILLISECOND` zeroed resulting in incorrect time comparison.

## [2.7.2] - 2017-06-14
- [Added] New config `stopOnStationary` for both iOS and Android.  Allows you to automatically `#stop` tracking when the `stopTimeout` timer elapses.
- [Added] Support for configuring the "Large Icon" (`notificationLargeIcon`) on Android `foregroundService` notification.  `notificationIcon` has now been aliased -> `notificationSmallIcon`.
- [Fixed] iOS timing issue when fetching `motionchange` position after initial `#start` -- since the significant-location-changes API (SLC) is engaged in the `#stop` method and eagerly returns a location ASAP, that first SLC location could sometimes be several minutes old and come from cell-tower triangulation (ie: ~1000m accuracy).  The plugin could mistakenly capture this location as the `motionchange` location instead of waiting for the highest possible accuracy location that was requested.  SLC API will be engaged only after the `motionchange` location has been received. 
- [Fixed] On Android, when adding a *massive* number of geofences (ie: *thousands*), it can take several minutes to perform all `INSERT` queries.  There was a threading issue which could cause the main-thread to be blocked while waiting for the database lock from the geofence queries to be released, resulting in an ANR (app isn't responding) warning.
- [Changed] Changing the Android foreground-service notification is now supported (you no longer need to `#stop` / `#start` the plugin for changes to take effect).
- [Fixed] Improved Android handling of simultaneous `#getCurrentPosition`, `#start`, `#configure` requests when location-services are not yet authorized by the user (the plugin will buffer all these requests and execute them in order once location-services are authorized).
- [Added] New config option `httpTimeout` (milliseconds) for configuring the timeout where the plugin will give up on sending an HTTP request.
- [Fixed] When iOS engages the `stopTimeout` timer, the OS will pause background-execution if there's no work being performed, in spite of `startBackgroundTask`, preventing the `stopTimeout` timer from running.  iOS will now keep location updates running at minimum accuracy during `stopTimeout` to prevent this.
- [Fixed] Ensure iOS background "location" capability is enabled before asking `CLLocationManager` to `setBackgroundLocationEnabled`.
- [Added] Implement ability to provide literal dates to schedule (eg: `2017-06-01 09:00-17:00`) 
- [Added] When Android motion-activity handler detects `stopTimeout` has expired, it will initiate a `motionchange` without waiting for the `stopTimeout` timer to expire (there were cases where the `stopTimeout` timer could be delayed from firing due likely to vendor-based battery-saving software)
- [Fixed] Android `emailLog` method was using old `adb logcat` method of fetching logs rather than fetching from `#getLog`

## [2.7.1] - 2017-05-12
- [Fixed] iOS has a new hook to execute an HTTP flush when network reachability is detected.  However, it was not checking if `autoSync: true` or state of `autoSyncThreshold`.

## [2.7.0] - 2017-05-08
- [Added] When iOS detects a network connection with `autoSync: true`, an HTTP flush will be initiated.
- [Fixed] Improve switching between tracking-mode location and geofence.  It's not necessary to call `#stop` before executing `#start` / `#startGeofences`.
- [Fixed] iOS issue with `cordova-plugin-cocoalumberjack` dependency issue with Cordova 7.0:  plugin version (should be `~0.0.2`, not `^0.0.2`)
- [Fixed] iOS `maximumAge` with `getCurrentPosition` wasn't clearing the callbacks when current-location-age was `<= maximumAge`
- [Fixed] iOS when `#stop` is executed, nullify the odometer reference location.
- [Fixed] iOS issue with `preventSuspend: true`:  When a `motionchange` event with `is_moving: false` occurred, the event was incorrectly set to `heartbeat` instead of `motionchange`.
- [Fixed] Android null pointer exception when using `startOnBoot: true, forceReloadOnBoot: true`:  there was a case where last known location failed to return a location.  The lib will check for null location in this case.
- [Changed] iOS minimum version is now `8.4`.  Plugin will log an error when used on versions of iOS that don't implement the method `CLLocationManager#requestLocation`
- [Fixed] iOS bug executing `#setConfig` multiple times too quickly can crash the plugin when multiple threads attempt to modify an `NSMutableDictionary`

## [2.6.1] - 2017-04-18
- [Fixed] Android was rounding `battery_level` to 1 decimal place
- [Fixed] iOS geofences-only mode was not using significant-location-change events to evaluate geofences within proximity.
- [Changed] iOS now uses `CLLocationManager requestLocation` to request the `motionchange` position, rather than counting samples.  This is a more robust way to get a single location
- [Fixed] iOS crash when providing `null` values in `Object` config options (ie: `#extras`, `#params`, `#headers`, etc)
- [Added] New config param `locationsOrderDirection [ASC|DESC]` for controlling the order that locations are selected from the database (and syned to your server)
- [Added] iOS now supports geofence `DWELL` with `loiteringDelay` with my own custom implementation, just as Android does natively.
- [Fixed] iOS was creating `backgroundTask` in `location` listener even if no listeners were registered, resulting in growing list of background-tasks which would eventually be `FORCE KILLED`.

## [2.6.0] - 2017-03-09
- [Fixed] iOS bug when composing geofence data for peristence.  Sometimes it appended a `location.geofence.location` due to a shared `NSDictionary`
- [Fixed] Android issue with applying default settings the first time an app boots.  If you execute `#getState` before `#configure` is called, `#getState` would return an empty `{}`.
- [Changed] The licensing model of Android now enforces license only for **release** builds.  If an invalid license is configured while runningin **debug** mode, a Toast warning will appear **"BackgroundGeolocation is running in evaluation mode."**, but the plugin *will* work.
- [Fixed] iOS bug with HTTP `401` handling.
- [Added] The Android plugin now broadcasts all its events using the Android `BroadcastReceiver` mechanism.  You're free to implement your own native Android handler to receive and react to these events as you wish.

## [2.5.3] - 2017-03-01
- [Changed] Refactor Android settings-management.  Plugin will always load previously known state as soon as plugin comes alive.  `#configure` will reset all settings to default before applying supplied `{Config}`.
- [Fixed] Android database migration issue when upgrading from a very old version missed `geofences` table migration.

## [2.5.1] - 2017-02-26
- [Changed] Refactor iOS settings-management.  Plugin will always load previously known state as soon as plugin comes alive.  `#configure` will reset all settings to default before applying supplied `{Config}`.
- [Fixed] iOS Schedule evaluation edge-case when a current-schedule is referenced but expired: When evaulating, always check if current-schedule is expired; query for next if so.
- [Fixed] GeofenceManager edge-case:  GeofenceManager should not receive current location when plugin is disabled (eg: executing `#getCurrentPosition` when plugin is disabled).

- [Fixed] `geofence` event not passing configured geofence `#extras`.
- [Changed] Removed `taskId` from `geofence` event callback.  This change is backwards compatible.  If you want to do a long-running task, create your own `bgTask` with `#startBackgroundTask` (the plan is to remove `taskId` from **all** callbacks. Eg:

```javascript
bgGeo.on('geofence', function(geofence) {  // <-- taskId no longer provided!
  // Start your own bgTask:
  bgGeo.startBackgroundTask(function(taskId) {
    performLongRunningTask(function() {
      bgGeo.finish(taskId);
    });
  });
});
```

## [2.5.0] - 2017-02-21
- [Fixed] iOS geofence identifiers containing ":" character were split and only the last chunk returned.  The plugin itself prefixes all geofences it creates with the string `TSGeofenceManager:` and the string-splitter was too naive.  Uses a `RegExp` replace to clear the plugin's internal prefix.
- [Changed] Refactored API Documentation
- [Added] HTTP JSON template features.  See [HTTP Features](./docs/http.md).  You can now template your entire JSON request data sent to the server by the plugin's HTTP layer.

## [2.4.0] - 2017-02-08
- [Changed] **BREAKING** I've *finally* figured out how to configure a number of key variables required by the plugin within your `config.xml` file, namely the `NSLocationAlwaysUsageDescription`, `NSLocationWhenInUseUsageDescription`, `NSMotionUsageDescription`.  The plugin now requires a `<plugin />` config within your `config.xml`.  **BREAKING:** With the introduction of this new config mechanism, I decided to use this also for the Android `license` config.  You will no longer provide the `<parameter name="cordova-background-geolocation-license" />`.  See the [README](https://github.com/transistorsoft/cordova-background-geolocation/tree/config-xml-variables#configuring-the-plugin) for details.

```xml
<widget id="com.your.company.app.id">
  <plugin name="cordova-background-geolocation" spec="^2.4.0">
    <variable name="LICENSE" value="YOUR_LICENSE_KEY" />
    <variable name="LOCATION_ALWAYS_USAGE_DESCRIPTION" value="Background location-tracking is required" />
    <variable name="LOCATION_WHEN_IN_USE_USAGE_DESCRIPTION" value="Background location-tracking is required" />
    <variable name="MOTION_USAGE_DESCRIPTION" value="Using the accelerometer increases battery-efficiency by intelligently toggling location-tracking only when the device is detected to be moving" />
    <variable name="BACKGROUND_MODE_LOCATION" value="location" />
    <variable name="BACKGROUND_MODE_AUDIO" value="" />
  </plugin>
```
- [Fixed] Migrate Android `providerchange` mechanism out of the `Service` (which only runs when the plugin is `#start`ed) to a place where it will be monitored all the time, regardless if the plugin is enabled or not.
- [Fixed] Catch `IllegalStateException` reported when using `#getLog`
- [Changed] With new Android "Doze-mode", override "idle" on `stopTimeout` and `schedule` alarms
- [Changed] Tweak iOS accelerometer-only motion-detection system.
- [Fixed] Location-authorization alert being popped up after a `suspend` event because the plugin always attempts to ensure it has a stationary-region here.  Simply check current authorization-status is not == `Denied`.

## [2.3.1] - 2017-01-13
- [Fixed] iOS Location Authorization alert is shown multiple time.  Also discovered a bug where the `providerchange` `enabled` value was calculated based upon hard-coded `Always` where it should have compared to the configured `locationAuthorizationRequest`.
- [Added] If plugin's `#stop` method is called, the Location Authorization Alert will be hidden (if currently visible).

## [2.3.0] - 2017-01-09
- [Fixed] Locale issue when formatting Floats.  Some locale use "," as decimal separator.  Force Locale -> US when performing rounding.  Proper locale will be applied during the JSON encoding.
- [Added] Ability to provide optional arbitrary meta-data `extras` on geofences.
- [Changed] Location parameters `heading`, `accuracy`, `odometer`, `speed`, `altitude`, `altitudeAccuracy` are now fixed at 2 decimal places.
- [Fixed] Bug reported with `EventBus already registered` error.  Found a few cases where `EventBus.isRegistered` was not being used.
- [Added] Android will attempt to auto-sync on heartbeat events.
- [Changed] permission `android.hardware.location.gps" **android:required="false"**` 
- [Added] Implement `IntentFilter` to capture `MY_PACKAGE_REPLACED`, broadcast when user upgrades the app.  If you've configured `startOnBoot: true, stopOnTerminate: false` and optionally `foreceRelaodOnBoot: true`, the plugin will automatically restart when user upgrades the app.
- [Changed] When adding a geofence (either `#addGeofence` or `#addGeofences`), if a geofence already exists with the provided `identifier`, the plugin will first destroy the existing one before creating the new one.
- [Changed] When iOS Scheduler is engaged and a scheduled OFF event occurs, the plugin will continue to monitor significant-changes, since background-fetch events alone cannot be counted on.  This will guarantee the plugin evaluates the schedule each time the device moves ~ 1km.  This will have little impact on power consumption, since these sig.change events will not be persisted or `POST`ed, nor will they even be provided to Javascript.
- [Changed] Android Scheduler will `setExact` Alarm triggers (only works for API `>= KITKAT` or if OEM's OS doesn't override it (ie: Samsung)).
- [Fixed] iOS Scheduler was not listening to `BackgroundFetch` events while plugin was disabled, preventing schedule evaluation from fetch-events (user would have to open the app for scheduler to evaluate).
- [Fixed] `stopWatchPostion` callbacks not being called.
- [Fixed] Use more precise Alarm mechanism for `stopTimeout`
- [Fixed] Improve odometer accuracy.  Introduce `desiredOdometerAccuracy` for setting a threshold of location accuracy for calculating odometer.  Any location having `accuracy > desiredOdometerAccuracy` will not be used for odometer calculation.
- [Fixed] When configured with a schedule, the Schedule parser wasn't ordering the schedule entries by start-time.
- [Fixed] Had a report of null-pointer exception when processing an HTTP error response.  I could not reproduce the issue but find a case where accessing a `String` could produce a NPE.
- [Changed] Add ability to set odometer to any arbitrary value.  Before, odometer could only be reset to `0` via `resetOdometer`.  The plugin now uses `setOdometer(Float, successFn, failureFn`.  `resetOdometer` is now just an alias for `setOdometer(0)`.  `setOdometer` will now internally perform a `#getCurrentPosition`, so it can know the exact location where the odometer was set at.  As a result, using `#setOdometer` is exactly like performing a `#getCurrentPosition` and the `success` / `failure` callbacks use the same method-signature, where the `success` callback is provided the `location`.
- [Added] Added ability to create your own arbitrary **background tasks** with new `#startBackgroundTask` method.  Some of the plugin's methods receive a `taskId` which you've had to call `bgGeo.finish(taskId)` upon.  These automatically created `taskId` will soon be removed.  It will be **up to you** to create your own as desired, when you need to perform any long-running task in any of the plugin's callbacks.  `#finish` operates in the same manner as before.

## [2.2.0] - 2016-11-21
- [Fixed] Issue #1025 Bug with Android geofences not posting `event: geofence` and the actual `geofence` data was missing (The data sent to Javascript callback was ok, just the data sent to HTTP.
- [Fixed] Issue #1023 Logic bug in `TSGeofenceManager`; was not performing geospatial query when changing state from **MOVING -> STATIONARY**.
- [Added] Geofences-only mode for both iOS and Android **BETA**.  Start geofences-only mode with method `#startGeofences`.
- [Changed] Add some intelligence to iOS motion-detection system:  Use a Timer of `activityRecognitionInterval` seconds before engaging location-services after motion is detected.  This helps to reduce false-positives, particularly when using `preventSuspend` while walking around one's house or office.
- [Changed] Add more intelligence to iOS motion-detection system:  The plugin will be **eager** to engage the stop-detection, as soon as it detects `still`, regardless of confidence.  When the plugin is currently in the **moving** state and detects `still`, it will engage a timer of `activityRecognitionInterval` milliseconds -- when this timer expires and the motion-detector still reports `still`, the stop-detection system will be engaged.  If any *moving* type activity occurs during this time, the timer will be cancelled.  References #1002.
- [Fixed] Bug in Android Scheduler, failing to `startOnBoot`.  Issue #985
- [Added] `#removeListeners` method.  Removes all listeners registered with plugin via `#on` method.
- [Changed] With `preventSuspend: true`, the plugin will no longer immediately engage location-services as soon as it sees a "moving"-type motion-activity:  it will now calculate if the current position is beyond stationary geofence. This helps reduce false-positives engaging location-services while simply walking around one's home or office.
- [Fixed] iOS `batchSync`: When only 1 record in batch, iOS fails to pack the records in a JSON `location: []`, appending to a `location: {}` instead.  Fixes #1042

## [2.1.6] - 2016-11-08
- [Fixed] Android was only handling the first geofence event when multiple geofences fire simultaneously.  Issue #1004
- [Changed] The plugin will ignore `autoSyncThreshold` when a `motionchange` event occurs.
- [Fixed] Fixed ui-blocking issue when plugin boots with locations in its database with `autoSync: true`.  Found a case where the plugin was executing HTTP Service on the UI thread.  Fixes #995.
- [Fixed] Return current `state {Object}` in callback to `setConfig` (issue #985)
- [Fixed] iOS Scheduler puked when provided with a `null` or `[]` schedule.
- [Changed] iOS Scheduler behaviour changed to match Android, where `#stopSchedule` does **not** execute `#stop` on the plugin itself.

## [2.1.5] - 2016-11-04
- [Fixed] Issue #998.  FMDB [has issues](https://github.com/ccgus/fmdb/pull/180) binding array arguments (eg: DELETE FROM locations WHERE id IN(?)).  Solution is to simply compose the query string with concatenation.  Sanitization isn't required here anyway, since the ids come directly from my own query.
  
## [2.1.4] - 2016-11-02
- [Changed] Extract `CococaLumberjack` static-libary from compiled binary TSLocationManager.  It causes problems if other libs also use this dependency.  Extracted CocoaLumberjack to its own distinct plugin `cordova-plugin-cocoalumberjack`, which background-geolocation installs as a dependency.  This change should be completely transparent.

## [2.1.3] - 2016-10-19
- [Changed] Introduce database-logging for Android with [logback-android](https://github.com/tony19/logback-android).  Same API as iOS (@see `2.1.0`)
- [Fixed] iOS geofencing issue where multiple geofences trigger simultaneously, only the last geofence event would be transmitted to the client and persisted to database.
- [Fixed] Remove iOS motion-activity-based filtering of locations.  If a location was recorded while the motion-recognition system said the device was `still`, the location was ignored.  Fixes issue #954.
- [Changed] Implemented ability for iOS to trigger a geofence `ENTER` event immediately when device is already inside the geofence (Android has always done this).  This behaviour can be controlled with the new config `@param {Boolean} geofenceInitialTriggerEntry [true]`.  This behaviour defaults to `true`.

## [2.1.2] - 2016-10-17
- [Changed] Android will filter-out received locations detected to be same-as-last by comparing `latitude`, `longitude`, `speed` & `bearing`.
- [Fixed] Bug in `stopDetectionDelay` logic
- [Fixed] Geofencing transistion event logging wouldn't occur when configured for `debug: false`

## [2.1.1] - 2016-10-12
- [Fixed] Bug in Android geofencing

## [2.1.0] - 2016-10-10
- [Changed] Refactor iOS Logging system to use popular CocoaLumberjack library.  iOS logs are now stored in the database!  By default, logs are stored for 3 days, but is configurable with `logMaxDays`.  Logs can now be filtered by logLevel:

| logLevel | Label |
|---|---|
|`0`|`LOG_LEVEL_OFF`|
|`1`|`LOG_LEVEL_ERROR`|
|`2`|`LOG_LEVEL_WARNING`|
|`3`|`LOG_LEVEL_INFO`|
|`4`|`LOG_LEVEL_DEBUG`|
|`5`|`LOG_LEVEL_VERBOSE`|

`#getLog`, `#emailLog` operate in the same manner as before.

- [Fixed] If user declines "Motion Activity" permission, plugin failed to detect this authorization failure and fallback to the accelerometer-based motion-detection system.

- [Changed] Refactored Geolocation system.  The plugin is no longer bound by native platform limits on number of geofences which can be monitored (iOS: 20; Android: 100).  You may now monitor infinite geofences.  The plugin now stores geofences in its SQLite db and performs a geospatial query, activating only those geofences in proximity of the device (@config #geofenceProximityRadius, @event `geofenceschange`).  See the new [Geofencing Guide](./docs/geofencing.md)

## [2.0.13] - 2016-09-25
- [Fixed] Background-fetch event when causing app to boot in background, left plugin in preventSuspend mode when not configured to do so.

## [2.0.12] - 2016-09-25
- [Fixed] Bug in prevent-suspend where background-fetch operation where plugin was left in preventSuspend mode when not configured to do do

## [2.0.11] - 2016-09-22
- [Fixed] Bug in prevent-suspend where the plugin failed to re-start its prevent-suspend timer if no MotionActivity event occurred during that interval.  Prevent-suspend system should now operate completely independently of MotionDetector.
- [Fixed] `#stop` method wasn't calling `stopMonitoringSignificantChanges`, resulting in location-services icon failing to toggle OFF.  Fixes issue #908

## [2.0.10] - 2016-09-22
- [Fixed] Issue where iOS crashes when configured with null url.
- [Added] iOS `watchPosition` mechanism.
- [Changed] Refactored iOS motion-detection system.  Improved iOS motion-triggering when using `CMMotionActivityManager` (ie: when not using `disableMotionActivityUpdates: true`).  iOS can now trigger out of stationary-mode just like android, where it sees a 'moving-type' motion-activity (eg: 'on_foot', 'in_vehicle', etc).  Note: this will still occur only when your app isn't suspended (eg: app is in foreground, `preventSuspend: true`, or `#watchPosition` is engaged).
- [Changed] Refactored iOS "prevent suspend" system to be more robust.
- [Fixed] iOS locations sent to Javascript client had a different `uuid` than the one persisted to database (and synced to server).
-[Added] new iOS 10 .plist required key for accelerometer updates `NSMmotionUsageDescription` to `config.xml`.  Fixes #880
- [Added] New required android permission `<uses-feature android:name="android.hardware.location.gps" />`.  Fixes #896

## [2.0.9] - 2016-08-29
- [Fixed] `removeGeofences` was removing stationary-region.  This would prevent stationary-exit if called while device is in stationary-mode
- [Fixed] Issue #830 Android pukes when it receives an empty schedule `[]`.
- [Fixed] Issue #824.  Android when configured with `batchSync: true, autoSync: true` was failing because the plugin automatically tweaked `autoSync: false` but failed to reset it to the configured value.  This behaviour was obsolete and has been removed.
- [Added] Add new config `@param {Integer} autoSyncThreshold [0]`.  Allows you to specify a minimum number of persisted records to trigger an auto-sync action.
- [Fixed] Issue #837.  Android `SimpleDateFormat` used for rendering location timestamp was not being used in a thread-safe manner, resulting in corrupted timestamps for some

## [2.0.8] - 2016-08-17
- [Fixed] Issue #804, null pointer exeception on mGoogleApiClient
- [Fixed] Issue #806.  PlayServices connect error event was fired before listeners arrive; Dialog to fix problem was never shown.
- [Changed] Removed `app-compat` from Gradle dependencies.
- [Changed] Fire http error callback when HTTP request is not 200ish (ie: 200, 201, 204).  Fixes issue #819.  Contradicts #774.
- [Changed] Remove `play-services:app-compat-v7` from Gradle dependencies
- [Fixed] Android heartbeat location wasn't having its meta-data updated (ie: `event: 'heartbeat', battery:<current-data>, uuid: <new uuid>`)
- [Changed] Reduce Android `minimumActivityRecognitionConfidence` default from `80` to `75` (issue #825)
- [Changed] Android will ask for location-permission when `#configure` is executed, rather than waiting for `#start`.
- [Changeed] Android will catch `java.lang.SecurityException` when attempting to request location-updates without "Location Permission"

## [2.0.7] - 2016-08-08
- [Fixed] Scheduler parsing issue #785.

## [2.0.6] Skipped to sync version with Lite version
## [2.0.5] - 2016-08-07
- [Fixed] `addGeofences` issue #778
- [Fixed] iOS setting `method` not being respected (was always doing `POST`).  Issue #770
- [Changed] Implement latest version of `cordova-plugin-background-fetch` dependency (v4.0.0)
- [Changed] iOS Plugin will perform HTTP sync on background-fetch event.
- [Changed] Implemented latest version of `cordova-plugin-background-fetch@4.0.0`.

## [2.0.4] - 2016-07-28
- [Changed] Major Android refactor with significant architectural changes.  Introduce new `adapter.BackgroundGeolocation`, a proxy between the Cordova plugin and `BackgroundGeolocationService`.  Up until now, the functionality of the plugin has been contained within a large, monolithic Android Service class.  This monolithic functionality has mostly been moved into the proxy object now, in addition to spreading among a number of new Helper classes.  The functionality of the HTTP, SQLite, Location and ActivityRecognition layers are largely unchanged, just re-orgnanized.  This new structure will make it much easier going forward with adding new features.
- [Changed] SQLite & HTTP layers have been moved from the BackgroundGeolocationService -> the proxy.  This means that database & http operations can now be performed without enabling the plugin with start() method.
- [Changed] Upgrade EventBus to latest.
- [Changed] Implement new watchPosition method (Android-only), allowing you to get a continues stream of locations without using Javascript setInterval (Android-only currently)
- [Added] Implement new event "providerchange" allowing you to listen to Location-services change events (eg: user turns off GPS, user turns off location services).  Whenever a "providerchange" event occurs, the plugin will automatically fetch the current position and persist the location adding the event: "providerchange" as well as append the provider state-object to the location.
- [Changed] Significantly simplified Cordova plugin (CDVBackgroundGeolocation.java) by moving boiler-plate code into the Proxy object.  This significantly simplifies the Cordova plugin, making it much easier to support all the different frameworks the plugin has been ported to (ie: React Native, NativeScript).
- [Changed] Disable iOS start-detection system when no accelerometer detected (ie: when running in simulator, fixes issue #767)
- [Changed] Refactor iOS location-authorization request system.  The plugin will now constantly check the status of location-authorization and show an Alert popup directing the user to the **Settings** screen if the user changes the state to anything other than what you requested (eg: user changes authorization-request to `WhenInUse` or `Never` when you requestd `Always`)
- [Fixed] `stopOnTerminate` on iOS was broken when app is closed while in background.  

## [1.7.0] - 2016-07-18
- [Changed] `Scheduler` will use `Locale.US` in its Calendar operations, such that the days-of-week correspond to Sunday=1..Saturday=7.  Fixes issue #659
- [Changed] Refactor odometer calculation for both iOS and Android.  No longer filters out locations based upon average location accuracy of previous 10 locations; instead, it will only use the current location for odometer calculation if it has accuracy < 100.
- [Fixed] Missing iOS setting `locationAuthorizationRequest after Settings service refactor
- [Added] **Android-only currently, beta** New geofence-only tracking-mode, where the plugin will not actively track location, only geofences.  This mode is engaged with the new API method `#startGeofences` (instead of `#start`, which engages the usual location-tracking mode).  `#getState` returns a new param `#trackingMode [location|geofences]`
- [Changed] **Android** Refactor the Android "Location Request" system to be more robust and better handle "single-location" requests / asynchronous requests, such as `#getCurrentPosition` and stationary-position to fetch multiple samples (3), selecting the most accurate (ios has always done this, heard with the debug sound `tick`).  In debug mode, you'll hear these location-samples with new debug sound "US dialtone".  Just like iOS, these location-samples will be returned to your `#onLocation` event-listener with the property `{sample: true}`.  If you're doing your own HTTP in Javascript, you should **NOT** send these locations to your server.  Use them instead to simply update the current position on the Map, if you're using one.
- [Added] new `#getCurrentPosition` options `#samples` and `#desiredAccuracy`. `#samples` allows you to configure how many location samples to fetch before settling sending the most accurate to your `callbackFn`.  `#desiredAccuracy` will keep sampling until an location having accuracy `<= desiredAccuracy` is achieved (or `#timeout` elapses).
- [Added] new `#event` type `heartbeat` added to `location` params (`#is_heartbeat` is **@deprecated**).
- [Fixed] Issue #676.  Don't engage foreground-service when executing `#getCurrentPosition` while plugin is in disabled state.
- [Fixed] When enabling iOS battery-state monitoring, use setter method `setBatteryMonitoringEnabled` rather than setting property.  This seems to have changed with latest iOS
- [Fixed] error-callback incorrectly fired when `getCurrentPosition` is called while service is stopped (Fixes issue #691)
- [Added] Implement `disableStopDetection` for Android (Fixes issue #692)
- [Changed] `android.permission.GET_TASKS` changed to `android.permission.GET_REAL_TASKS`.  Hoping this removes deprecation warning.  This permission is required for Android `#forceReload` configs.
- [Added] New Anddroid config `#notificationIcon`, allowing you to customize the icon shown on notification when using `foregroundServcie: true`.
- [Changed] Take better care with applying `DEFAULT` settings for both iOS & Android.
- [Changed] Default settings: `startOnBoot: false`, `stopOnTerminate: true`, `distanceFilter: 10`.
- [Added] Allow setting `isMoving` as a config param to `#configure`.  Allows the plugin to automatically do a `#changePace` to your desired value when the plugin is first `#configure`d.
- [Added] New event `activitychange` for listening to changes from the Activit Recognition system.  See **Events** section in API docs for details.  Fixes issue #703.
- [Added] Allow Android `foregroundService` config to be changed dynamically with `#setConfig` (used to have to restart the start to apply this).
- [Added] Implement the new `#getCurrentPosition` options `#samples` and `#desiredAccuracy` for iOS.
- [Changed] Revert GET_REAL_TASKS back to GET_TASKS
## [1.6.3] - 2016-05-25
- [Fixed] Rebuild binary `tslocationmanager.aar` excluding dependencies `appcompat-v7` and `play-services`.  I was experiencing build-failures with react-native since other libs may include these dependencies:

## [1.6.2] - 2016-05-24
- [Fixed] Android `GeofenceService` namespace was changed but the `plugin.xml` file was not updated with the new namespace.

## [1.6.1] - 2016-05-22
- [Changed] Refactor iOS motion-detection system.  When not set to `disableMotionActivityUpdates` (default), the  plugin will not activate the accelerometer and will rely instead purely upon updates from the **M7** chip.  When `disableMotionActivityUpdates` **is** set to `false`, the pure acceleromoeter based activity-detection has been improved to give more accurate results of the detected activity (ie: `on_foot, walking, stationary`)
- [Fixed] Fixed configuration bug which could incorrectly set `pausesLocationUpdatesAutomatically` to false, preventing the plugin from automatically turning off location-updates.

## [1.6.0] - 2016-05-17
- [Added] Implement new Scheduling feature for iOS and Android.
- [Fixed] Bugs in iOS option `useSignificantChangesOnly`
- [Changed] Refactor HTTP Layer for both iOS and Android to stop spamming server when it returns an error (used to keep iterating through the entire queue).  It will now stop syncing as soon as server returns an error (good for throttling servers).
- [Added] Migrate iOS settings-management to new Settings service
- [Fixed] bugs in Scheduler
- [Fixed] Fix bug where Android was not JSON decoding the location object sent to heartbeat event #646
- [Added] Better BackgroundFetch plugin integration.  Background-fetch will retrieve the location-state when a fetch event fires.  This can help to trigger stationary-exit since bg-fetch typically fires about every 15min.
- [Added] Improved functionality with `stopOnTerminate: false`.  Ensure a stationary-geofence is created when plugin is closed while in **moving** state; this seems to improve the time it takes to trigger the iOS app awake after terminate.  When plugin *is* rebooted in background due to geofence-exit, the plugin will briefly sample the accelerometer to see if device is currently moving.

## [1.5.1] - 2016-04-12
- [Added] ios logic to handle being launched in the background (by a background-fetch event, for example).  When launched in the background, iOS will essentially do a `changePace(true)` upon itself and let the stop-detection system determine engage stationary-mode as detected.
- [Changed] ios halt stop-detection distance was using `distanceFilter`; changed to use `stationaryRadius`.  This effects users using the accelerometer-based stop-detection system:  after stop is detected, the device must move `stationaryRadius` meters away from location where stop was detected.
- [Fixed] Android `addGeofences` callbacks not being called.
- [Changed] When `maxRecordsToPersist == 0`, don't persist any record.
- [Added] Implement `startOnBoot` param for iOS.  iOS always ignored `startOnBoot`.  If you set `startOnBoot: false` now, iOS will not begin tracking when launched in background after device is rebooted (eg: from a background-fetch event, geofence exit or significant-change event)
- [Changed] Modified the method signature of `#configure`.  The config `{}` will now be the 1st param and the `callbackFn` will now signal successful configuration.  The `locationCallback` and `locationErrorCallback` must now be provided with a separate event-listener `#onLocation`.
- [Fixed] Bug in `getCurrentPosition` timeout when plugin is disabled.

## [1.5.0] - 2016-04-04
- [Fixed] Fixed issue #597, `NullPointerException` after `startOnBoot`
- [Fixed] Refactor `startOnBoot` system.  Moved `BootReceiver` to `com.transistorsoft.cordova.bggeo` namespace.
- [Added] New config option `#forceReloadOnBoot` for specifying the MainActivity should launch after a `startOnBoot` occurs.
## [1.5.0] - 2016-04-04
- [Added] Intelligence for `stopTimeout`.  When stop-timer is initiated, save a reference to the current-location.  If another location is recorded while during stop-timer, calculate the distance from location when stop-timer initiated:  if `distance > stationaryRadius`, cancel the stop-timer and stay in "moving" state.
- [Added] New debug sound **"booooop"**:  Signals the initiation of stop-timer of `stopTimeout` minutes.
- [Added] New debug sound **"boop-boop-boop"**: Signals the cancelation of stop-timer due to movement beyond `stationaryRadius`.
- [Fixed] When using setConfig to change `distanceFilter` while location-updates are already engaged, was mistakenly using value for `desiredAccuracy`!
- [Fixed] iOS Issue with timers not running on main-thread.
- [Fixed] iOS Issue with acquriring stationary-location on a stale location.  Ensure the selected stationary-location is no older than 1s.
- [Fixed] iOS Removed some log messages appearing when `{debug: false}`
- [Fixed] Android 5 geofence limit.  Refactored Geofence system.  Was using a `PendingIntent` per geofence; now uses a single `PendingIntent` for all geofences.
- [Fixed] Edge-case issue #540 when executing `#getCurrentPosition` followed immediately by `#start` in cases where location-timeout occurs.
- [Changed] Update Android Volley dependency to official version
- [Changed] When Android #stop is called, update `isMoving` state to `false`
- [Fixed] ios `stopOnTerminate` was defaulting to `false`.  Docs say default is `true`.
- [Fixed] ios `useSignificantChangesOnly` was broken.
- [Fixed] If location-request times-out while acquiring stationary-location, try to use last-known-location
- [Added] Add odometer to ios location JSON schema
- [Added] ios Log network reachability flags on connection-type changes.
- [Added] `maxRecordsToPersist` to limit the max number of records persisted in plugin's SQLite database.
- [Added] API methods `#addGeofences` (for adding a list-of-geofences), `#removeGeofences`
- [Changed] The plugin will no longer delete geofences when `#stop` is called; it will merely stop monitoring them.  When the plugin is `#start`ed again, it will start monitoringt any geofences it holds in memory.  To completely delete geofences, use new method `#removeGeofences`.
- [Fixed] iOS battery `is_charging` was rendering as `1/0` instead of boolean `true/false`
- [Fixed] Issue with `forceReloadOnX` params. These were not forcing the activity to reload on device reboot when configured with `startOnBoot: true`.  Implemented a better method for detecting when main Activity is active.
- [Fixed] `getOdometer` callabck wasn't running if BackgroundGeolocationService wasn't running.
- [Changed] When Activity is `forceReload`ed (eg: via `forceReloadOnLocationChange`), the Activity will automatically be minimized when launched now!

## [1.4.0] - 2016-03-08
- [Changed] Introduce new per-app licensing scheme.  I'm phasing out the unlimited 'god-mode' license in favour of generating a distinct license-key for each bundle ID.  This cooresponds with new Customer Dashboard for generating application keys and managing team-members.

## [1.3.0]

- [Changed] Upgrade `emailLog` method to attach log as email-attachment rather than rendering to email-body.  The result of `#getState` is now rendered to the 

##0.6.4

- pin google-play-services at 7.5.0 to help people suffering from Google Play Services bug
- Tweak stop-detection system.  add `#stopDetectionDelay`

![](https://camo.githubusercontent.com/76f20a03f04f455f8219dbbca2a40631cd009fb4/68747470733a2f2f646c2e64726f70626f7875736572636f6e74656e742e636f6d2f752f323331393735352f636f72646f76612d6261636b67726f756e642d67656f6c6f636169746f6e2f696f732d73746f702d646574656374696f6e2d74696d696e672e706e67)

## 0.6.3
- Introduce accelerometer-base stop-detection for ios.
- `@config {Boolean} useSignificantChangesOnly`
- When a geofence event occurs, the associated location will have geolocation meta-data attached to the associated location.  Find it in the POST data.  See Wiki Location Data Schema for details.

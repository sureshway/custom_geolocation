Background Geolocation
==============================

Cross-platform background geolocation for Cordova with battery-saving "circular region monitoring" and "stop detection".

Follows the [Cordova Plugin spec](http://cordova.apache.org/docs/en/3.0.0/plugin_ref_spec.md), so that it works with [Plugman](https://github.com/apache/cordova-plugman).

This plugin leverages Cordova/PhoneGap's [require/define functionality used for plugins](http://simonmacdonald.blogspot.ca/2012/08/so-you-wanna-write-phonegap-200-android.html).

## [:books: API Documentation](./docs/README.md)
- :wrench: [Configuration Options](./docs/README.md#wrench-configuration-options)
  + [Geolocation Options](./docs/README.md#wrench-geolocation-options)
  + [Activity Recognition Options](./docs/README.md#wrench-activity-recognition-options)
  + [HTTP & Persistence Options](./docs/README.md#wrench-http--persistence-options)
  + [Geofencing Options](./docs/README.md#wrench-geofencing-options)
  + [Application Options](./docs/README.md#wrench-application-options)
- :zap: [Events](./docs/README.md#zap-events)
- :small_blue_diamond: [Methods](./docs/README.md#large_blue_diamond-methods)
- :blue_book: Guides
  + [Philosophy of Operation](../../wiki/Philosophy-of-Operation)
  + [Geofencing](./docs/geofencing.md)
  + [HTTP Features](./docs/http.md)
  + [Location Data Schema](../../wiki/Location-Data-Schema)
  + [Debugging](../../wiki/Debugging)
  
## Using the plugin ##
The plugin creates the object `window.BackgroundGeolocation`.  See [API Documentation](docs) for details
  
## Installing the plugin ##

#### From master (latest, greatest.)

```
   cordova plugin add https://github.com/transistorsoft/cordova-background-geolocation.git
```

#### Installing a tagged version.

This plugin has tagged stable versions.  To install a particular version, append a version code to the github url prefixed by `#`.

```
    cordova plugin add <git.url>#1.4.0
```

![](https://dl.dropboxusercontent.com/u/2319755/cordova-background-geolocaiton/screenshot-github-tagged-branches.png)

## Configuring the plugin

The plugin requires configuration within your App's `config.xml`:

```xml
<widget id="com.your.company.app.id">
  <plugin name="cordova-background-geolocation">
    <variable name="LICENSE" value="YOUR_LICENSE_KEY" />
    <variable name="LOCATION_ALWAYS_AND_WHEN_IN_USE_USAGE_DESCRIPTION" value="Always use is required for background location tracking" />
    <variable name="LOCATION_ALWAYS_USAGE_DESCRIPTION" value="Background location-tracking is required" />
    <variable name="LOCATION_WHEN_IN_USE_USAGE_DESCRIPTION" value="Background location-tracking is required" />
    <variable name="MOTION_USAGE_DESCRIPTION" value="Using the accelerometer increases battery-efficiency by intelligently toggling location-tracking only when the device is detected to be moving" />
  </plugin>
  .
  .
  .
</widget>
```

:warning: To apply changes to these `<variable />`, you **must** remove/re-add the plugin (**append `--nosave` when removing the plugin**)

```
$ cordova plugin remove cordova-background-geolocation --nosave
$ cordova plugin add https://github.com/transistorsoft/cordova-background-geolocation.git
```

---


### Disabling Background "location"

For those using `useSignificantChangesOnly: true`, possibly because Apple *denied* your use of the background `location` capability, you can disable background `location` by providing the `BACKGROUND_MODE_LOCATION` `<variable />` with an empty-string:

```xml
<plugin name="cordova-background-geolocation">
  .
  .
  .
  <!-- Disable background "location" capability with empty-string -->
  <variable name="BACKGROUND_MODE_LOCATION" value="" />
</plugin>
```

##### `@variable LICENSE [""]` Android
**[Android]** Your Android license key generated from [Product Dashboard](http://www.transistorsoft.com/shop/customers)

##### `@variable LOCATION_ALWAYS_USAGE_DESCRIPTION ["Background location-tracking is required"]` iOS

**[iOS]** Customize the message displayed to the user when `Always` location authorization is requested.  This variable is added to your iOS `.plist` in the `NSLocationAlwaysUsageDescription` key.

##### `@variable LOCATION_WHEN_IN_USE_USAGE_DESCRIPTION ["Background location-tracking is required"` iOS

**[iOS]** Customize the message displayed to the user when `WhenInUse` location authorization is requested.  This variable is added to your iOS `.plist` in the `NSLocationWhenInUseUsageDescription` key.

##### `@variable MOTION_USAGE_DESCRIPTION ["Using the accelerometer increases battery-efficiency by..."` iOS

**[iOS]** Customize the message displayed to the user when "Motion & Fitness" permission is requested.  The plugin is **highly** optimized to use iOS `CMMotionActivityManager` API for intelligently toggling location-services only when the plugin is detected to be moving.

##### `@variable BACKGROUND_MODE_LOCATION ["&lt;string&gt;location&lt;/string&gt;"]` iOS
**[iOS]** Adds the iOS background-mode `location` to your iOS `.plist` file.  This is the default behaviour.  To disable this, (ie: for those using `useSignificantChangesOnly`), provide an empty-string:

```xml
  <variable name="BACKGROUND_MODE_LOCATION" value="" />
```

:warning: If you *do* want the default behaviour of background-location updates, simply **IGNORE** this variable -- Do **NOT** even provide it.  If you *do* provide it, you must provide the full escaped XML value of `&lt;string&gt;location&lt;/string&gt;` (the default value when not provided), not just `location`.

### Configure your license

1. Login to Customer Dashboard to generate an application key:
[www.transistorsoft.com/shop/customers](http://www.transistorsoft.com/shop/customers)
![](https://gallery.mailchimp.com/e932ea68a1cb31b9ce2608656/images/b2696718-a77e-4f50-96a8-0b61d8019bac.png)

2. Edit your application's `config.xml` and provide your `LICENSE` to the <plugin> element as follows:
```xml
<widget id="com.your.company.app.id">
  .
  .
  .
  <plugin name="cordova-background-geolocation" spec="^2.4.0">
    <variable name="LICENSE" value="YOUR_LICENSE_KEY" />
    .
    .
    .
  </plugin>
</widget>
```

## Android SDK

If building from your local machine (as you should be), ensure you have the following 2 items installed in **SDK Manager**:

- Google Repository
- Android Support Repository

![](https://camo.githubusercontent.com/e0394b09555896127b96e70e2a62dedc932e3236/68747470733a2f2f7777772e64726f70626f782e636f6d2f732f356c37316d766333377971726271682f53637265656e73686f74253230323031352d31312d313225323030362e33342e30362e706e673f646c3d31)


## Example

```javascript
////
// As with all Cordova plugins, you must configure within an #deviceready callback.
//
function onDeviceReady() {
    // Get a reference to the plugin.
    var bgGeo = window.BackgroundGeolocation;
    
    /**
    * This callback will be executed every time a geolocation is recorded in the background.
    */
    var callbackFn = function(location) {
        console.log('- Location', location);

        myUpdatePositionOnMapMethod(location);

        // The plugin records multiple samples when doing motionchange events.
        // It sends these to the location callback for you convenience.  These 
        // "samples" are NOT persisted to the plugin's database.
        // You might uses these to "progressively" update user's current
        // position on a map, for example.
        if (location.sample) {
            console.log('- Ignore samples');
            return;
        }
    };

    var failureFn = function(error) {
        console.log('BackgroundGeoLocation error', error);
    }
    
    // Listen to location event
    bgGeo.on('location', callbackFn, failureFn);
    
    // Configure the plugin
    bgGeo.configure({
        // Geolocation config
        desiredAccuracy: 0,
        distanceFilter: 50,

        // Activity recognition config
        activityRecognitionInterval: 10000,
        stopTimeout: 5,	 // Stop-detection timeout minutes (wait x minutes to turn off tracking)
        
        // Application config
        debug: true, // <-- enable this hear sounds for background-geolocation life-cycle.
        logLevel: bgGeo.LOG_LEVEL_VERBOSE,    // Verbose logging
        stopOnTerminate: false,               // Don't stop tracking when user closes app.
        startOnBoot: true,                    // Auto-start tracking on reboot        
        // HTTP / SQLite config
        url: 'http://your.server.url/locations',
        batchSync: false,       // <-- [Default: false] Set true to sync locations to server in a single HTTP request.
        autoSync: true,         // <-- [Default: true] Set true to sync each location to server as it arrives.
        maxDaysToPersist: 1,    // <-- Maximum days to persist a location in plugin's SQLite database when HTTP fails
    }, function(state) {
      // Plugin is configured and ready to use.
      if (!state.enabled) {
        bgGeo.start();  // <-- start-tracking
      }
    });
}

```

## Advanced Sample Application for Field-testing

A fully-featured [SampleApp](https://github.com/transistorsoft/cordova-background-geolocation-SampleApp) is available in its own public repo.  After first cloning that repo, follow the installation instructions in the **README** there.  This SampleApp includes a settings-screen allowing you to quickly experiment with all the different settings available for each platform.

![Home](https://dl.dropboxusercontent.com/u/2319755/cordova-background-geolocaiton/screenshot-iphone5-geofences-framed-README.png)
![Settings](https://dl.dropboxusercontent.com/u/2319755/cordova-background-geolocaiton/screenshot-iphone5-settings-framed-README.png)

If you're using XCode, boot the SampleApp in the iOS Simulator and enable ```Debug->Location->City Drive```.


## Licence ##
```
cordova-background-geolocation
Copyright (c) 2015, Transistor Software (9224-2932 Quebec Inc)
All rights reserved.
sales@transistorsoft.com
http://transistorsoft.com
```

1. Preamble:  This Agreement governs the relationship between YOU OR THE ORGANIZATION ON WHOSE BEHALF YOU ARE ENTERING INTO THIS AGREEMENT (hereinafter: Licensee) and Transistor Software, a LICENSOR AFFILIATION whose principal place of business is Montreal, Quebec, Canada (Hereinafter: Licensor). This Agreement sets the terms, rights, restrictions and obligations on using [{software}] (hereinafter: The Software) created and owned by Licensor, as detailed herein

2. License Grant: Licensor hereby grants Licensee a Personal, Non-assignable &amp; non-transferable, Commercial, Royalty free, Including the rights to create but not distribute derivative works, Non-exclusive license, all with accordance with the terms set forth and other legal restrictions set forth in 3rd party software used while running Software.

	2.1 Limited: Licensee may use Software for the purpose of:
		- Running Software on Licensee's Website[s] and Server[s];
		- Allowing 3rd Parties to run Software on Licensee's Website[s] and Server[s];
		- Publishing Software&rsquo;s output to Licensee and 3rd Parties;
		- Distribute verbatim copies of Software's output (including compiled binaries);
		- Modify Software to suit Licensee&rsquo;s needs and specifications.

	2.2 Binary Restricted: Licensee may sublicense Software as a part of a larger work containing more than Software, distributed solely in Object or Binary form under a personal, non-sublicensable, limited license. Such redistribution shall be limited to unlimited codebases.</li><li>

	2.3 Non Assignable &amp; Non-Transferable: Licensee may not assign or transfer his rights and duties under this license.

	2.4 Commercial, Royalty Free: Licensee may use Software for any purpose, including paid-services, without any royalties

	2.5 Including the Right to Create Derivative Works: </strong>Licensee may create derivative works based on Software, including amending Software&rsquo;s source code, modifying it, integrating it into a larger work or removing portions of Software, as long as no distribution of the derivative works is made.

3. Term & Termination:  The Term of this license shall be until terminated. Licensor may terminate this Agreement, including Licensee's license in the case where Licensee : 

	3.1 became insolvent or otherwise entered into any liquidation process; or

	3.2 exported The Software to any jurisdiction where licensor may not enforce his rights under this agreements in; or

	3.3 Licensee was in breach of any of this license's terms and conditions and such breach was not cured, immediately upon notification; or

	3.4 Licensee in breach of any of the terms of clause 2 to this license; or

	3.5 Licensee otherwise entered into any arrangement which caused Licensor to be unable to enforce his rights under this License.

4. Payment: In consideration of the License granted under clause 2, Licensee shall pay Licensor a FEE, via Credit-Card, PayPal or any other mean which Licensor may deem adequate. Failure to perform payment shall construe as material breach of this Agreement.

5. Upgrades, Updates and Fixes: Licensor may provide Licensee, from time to time, with Upgrades,  Updates or Fixes, as detailed herein and according to his sole discretion. Licensee hereby warrants to keep The Software up-to-date and install all relevant updates and fixes, and may, at his sole discretion, purchase upgrades, according to the rates set by Licensor. Licensor shall provide any update or Fix free of charge; however, nothing in this Agreement shall require Licensor to provide Updates or Fixes.

	5.1 Upgrades: for the purpose of this license, an Upgrade  shall be a material amendment in The Software, which contains new features   and or major performance improvements and shall be marked as a new version number. For example, should Licensee purchase The Software under   version 1.X.X, an upgrade shall commence under number 2.0.0.

	5.2 Updates: for the purpose of this license, an update shall be a minor amendment   in The Software, which may contain new features or minor improvements and   shall be marked as a new sub-version number. For example, should   Licensee purchase The Software under version 1.1.X, an upgrade shall   commence under number 1.2.0.

	5.3 Fix: for the purpose of this license, a fix shall be a minor amendment in   The Software, intended to remove bugs or alter minor features which impair   the The Software's functionality. A fix shall be marked as a new   sub-sub-version number. For example, should Licensee purchase Software   under version 1.1.1, an upgrade shall commence under number 1.1.2.

6. Support: Software is provided under an AS-IS basis and without any support, updates or maintenance. Nothing in this Agreement shall require Licensor to provide Licensee with support or fixes to any bug, failure, mis-performance or other defect in The Software.

	6.1 Bug Notification: Licensee may provide Licensor of details regarding any bug, defect or   failure in The Software promptly and with no delay from such event;  Licensee  shall comply with Licensor's request for information regarding  bugs,  defects or failures and furnish him with information,  screenshots and  try to reproduce such bugs, defects or failures.

	6.2 Feature Request: Licensee may request additional features in Software, provided, however, that (i) Licensee shall waive any claim or right in such feature should feature be developed by Licensor; (ii) Licensee shall be prohibited from developing the feature, or disclose such feature   request, or feature, to any 3rd party directly competing with Licensor or any 3rd party which may be, following the development of such feature, in direct competition with Licensor; (iii) Licensee warrants that feature does not infringe any 3rd party patent, trademark, trade-secret or any other intellectual property right; and (iv) Licensee developed, envisioned or created the feature solely by himself.

7. Liability: To the extent permitted under Law, The Software is provided under an   AS-IS basis. Licensor shall never, and without any limit, be liable for   any damage, cost, expense or any other payment incurred by Licensee as a   result of Software&rsquo;s actions, failure, bugs and/or any other  interaction  between The Software &nbsp;and Licensee&rsquo;s end-equipment, computers,  other  software or any 3rd party, end-equipment, computer or  services. Moreover, Licensor shall never be liable for any defect in  source code  written by Licensee when relying on The Software or using The Software&rsquo;s source  code.

8. Warranty: 

	8.1 Intellectual Property:  Licensor   hereby warrants that The Software does not violate or infringe any 3rd   party claims in regards to intellectual property, patents and/or   trademarks and that to the best of its knowledge no legal action has   been taken against it for any infringement or violation of any 3rd party   intellectual property rights.

	8.2 No-Warranty: The Software is provided without any warranty; Licensor hereby disclaims   any warranty that The Software shall be error free, without defects or code   which may cause damage to Licensee&rsquo;s computers or to Licensee, and  that  Software shall be functional. Licensee shall be solely liable to  any  damage, defect or loss incurred as a result of operating software  and  undertake the risks contained in running The Software on License&rsquo;s  Server[s]  and Website[s].

	8.3 Prior Inspection:  Licensee hereby states that he inspected The Software thoroughly and found   it satisfactory and adequate to his needs, that it does not interfere   with his regular operation and that it does meet the standards and  scope  of his computer systems and architecture. Licensee found that  The Software  interacts with his development, website and server environment  and that  it does not infringe any of End User License Agreement of any  software  Licensee may use in performing his services. Licensee hereby  waives any  claims regarding The Software's incompatibility, performance,  results and  features, and warrants that he inspected the The Software.</p>

9. No Refunds:  Licensee warrants that he inspected The Software according to clause 7(c)   and that it is adequate to his needs. Accordingly, as The Software is   intangible goods, Licensee shall not be, ever, entitled to any refund,   rebate, compensation or restitution for any reason whatsoever, even if   The Software contains material flaws.

10. Indemnification:  Licensee hereby warrants to hold Licensor harmless and indemnify Licensor for any lawsuit brought against it in regards to Licensee&rsquo;s use   of The Software in means that violate, breach or otherwise circumvent this   license, Licensor's intellectual property rights or Licensor's title  in  The Software. Licensor shall promptly notify Licensee in case of such  legal  action and request Licensee's consent prior to any settlement in relation to such lawsuit or claim.

11. Governing Law, Jurisdiction:  Licensee hereby agrees not to initiate class-action lawsuits against Licensor in relation to this license and to compensate Licensor for any legal fees, cost or attorney fees should any claim brought by Licensee against Licensor be denied, in part or in full.


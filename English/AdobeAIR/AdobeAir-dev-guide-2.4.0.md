# Adobe Air Extension Instructions

### Before you begin:

* **The Vungle Extension Requires Adobe AIR SDK 4.0 or higher.** For instructions on updating the AIR SDK in Flash Builder or Flash Professional, refer to “How do I update the AIR SDK?” at the end of this guide.

* If working with Android, the Vungle AIR extension requires JDK 7 to be installed on the development system. Android 3.0 (Honeycomb - API version 11) or later required for the application to run.

* You can view the ActionScript 3 Class Documentation [here](http://help.adobe.com/en_US/FlashPlatform/reference/actionscript/3/index.html).

* Review 'example/VungleExample.as' for a sample application class. (If you're a Flash Professional user and are not sure how to use a Document Class, see “How do I use the VungleExample Document Class in Flash CS6?” at the end of this guide.)

## 1. Include the Extension Library

Start by creating a new AIR for mobile project and adding the native extension.

If targeting *Android*: you may also need to add Google Play Services library to your project. Many other extensions already include this library so you might not need to add it. To add the extension, repeat the steps below but use com.vungle.extensions.android.GooglePlayServices.ane in place of com.vungle.extensions.Vungle.ane.

### In Flash Professional CS6 or Higher:

1. Create a new AIR for Android or AIR for iOS project
2. Choose *File > Publish Settings...*
3. Select the wrench icon next to 'Script' for 'ActionScript Settings'
4. Select the Library Path tab
5. Click 'Browse for Native Extension (ANE) File' and select the com.vungle.extensions.Vungle.ane file. Press OK
6. Select the wrench icon next to 'Target' for Player Settings
7. If targeting *Android*: Select the 'Permissions' tab and enable 'INTERNET', 'WRITE_EXTERNAL_STORAGE', and 'ACCESS_NETWORK_STATE'
8. Check the 'Manually manage permissions and manifest additions for this app' box
9. Press OK

### In Flash Builder 4.6 or Higher:

1. Go to *Project Properties*
2. Select *Native Extensions* under *Actionscript Build Path*
3. Choose *Add ANE...* and navigate to the com.vungle.extensions.Vungle.ane file
4. Select *Actionscript Build Packaging > Google Android*
5. Select the *Native Extensions* tab and click the *'Package'* checkbox next to the extension 
6. If targeting iOS, repeat steps 4 and 5 for the 'Apple iOS' target

## 2. Update Your Application Descriptor

For Vungle to work, changes are required to the application XML file for your app. modify the XML file created by your IDE with the following changes (if you're a Flash Professional user, make sure you've followed the steps above for 'Include the Library in Flash Professional CS6 or Higher', otherwise Flash might undo your changes as you make them):

1. Set your AIR SDK to 4.0 (or later) in the app descriptor file:

  ```as3
  <application xmlns="http://ns.adobe.com/air/application/4.0">
  ```

2. Include a link to the extension in the descriptor:

  ```as3
  <extensions>
  <extensionID>com.vungle.extensions.Vungle</extensionID>
  </extensions>
  ```

3. If targeting *Android*: you may need to include the Google Play Services extension. Add its extension ID here as well.

  ```as3
  <extensions>
  <extensionID>com.vungle.extensions.Vungle</extensionID>
  <extensionID>com.vungle.extensions.android.GooglePlayServices</extensionID>
  </extensions>
  ```

### For AIR Applications Targeting Android

*If targeting Android*, update your Android Manifest Additions in the android XML element, to include the INTERNET, WRITE_EXTERNAL_STORAGE, and ACCESS_NETWORK_STATE permissions; add the VideoFullScreenAdActivity and MraidFullScreenAdActivity activity definitions; and add the google-play-services version meta-data tag:  

```as3
<android> 
<manifestAdditions><![CDATA[ 
<manifest android:installLocation="auto"> 

  <uses-permission android:name="android.permission.INTERNET"/> 
  <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" /> 
  <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" /> 

  <application>
    <meta-data android:name="com.google.android.gms.version" android:value="@integer/google_play_services_version"/>

    <activity android:name="com.vungle.publisher.VideoFullScreenAdActivity"
      android:configChanges="keyboardHidden|orientation|screenSize"
      android:theme="@android:style/Theme.NoTitleBar.Fullscreen"/>
    <activity android:name="com.vungle.publisher.MraidFullScreenAdActivity"
      android:configChanges="keyboardHidden|orientation|screenSize"
      android:theme="@android:style/Theme.Translucent.NoTitleBar.Fullscreen"/>
  </application> 
</manifest> 
]]></manifestAdditions>
</android>
```

## 3. Integrate the Vungle API

The Vungle API can be added your application in just a few lines of ActionScript.

###  Initialize the Vungle Extension

Initialize the API when your application starts. (If using pure ActionScript, do this in the constructor of your Document Class. If using Flex, call this in initialize() event of the main class. If using timeline code in Flash, do this on Frame 1).

1. Import the API Classes:

  ```as3
  import com.vungle.extensions.*; 
  import com.vungle.extensions.events.*;
  ```

2. Initialize the API by calling `Vungle.create()`, and passing in an array containing your application ID from the Vungle Dashboard. If you are targeting both iOS and Android from the same project, include both IDs in the array - with the iOS id first and the Android id second.
  
  You should wrap your call to `Vungle.create()` in a try/catch since Vungle may throw an error during the creation process (for instance, the extension throws an error if running on the Desktop):

  ```as3
  try
  { 
    // initialize with your app id 
    Vungle.create(["your_vungle_id"]); 

    // -OR- initialize including both ios and android ids for multiplatform apps 
    // Vungle.create(["your_ios_vungle_id","your_android_vungle_id"]); 
  } catch (error:Error) {
    // could not create extension. Are you running on something besides iOS/Android?
  }
  ```

### Display an Interstitial Ad

To display an interstitial ad, call `playAd()`. You'll want to first check than ad is available using the `isAdAvailable()` method:

```as3
if (Vungle.vungle.isAdAvailable())
{
	Vungle.vungle.playAd();
}
```

### Display an Incentivized Ad

To display an incentivized ad, call `playAd()` with a configuration object and set `incentivized` option to `true`. You'll want to first check than ad is available using the `isAdAvailable()` method.

```as3
if (Vungle.vungle.isAdAvailable())
{
	var config:VungleAdConfig = new VungleAdConfig();
	config.incentivized = true;
	Vungle.vungle.playAd(config);
}
```

To reward the player for completing an incentivized view, you'll want to implement the `AD_FINISHED` event listener as described below.

### Add Event Listeners

The Vungle Extension dispatches four events: `VungleEvent.AD_PLAYABLE`, `VungleEvent.AD_STARTED`, `VungleEvent.AD_FINISHED`, and `VungleEvent.AD_LOG`.

1. The `AD_PLAYABLE` is dispatched when an ad is ready to play:

```as3
Vungle.vungle.addEventListener(VungleEvent.AD_PLAYABLE, onAdPlayable);

function onAdPlayable(e:VungleEvent):void
{
	trace("ad playable");
}
```

2. The `AD_STARTED` and `AD_FINISHED` events are dispatched when an ad is displayed and dismissed, respectively:

  ```as3
  Vungle.vungle.addEventListener(VungleEvent.AD_STARTED, onAdStarted); 
  Vungle.vungle.addEventListener(VungleEvent.AD_FINISHED, onAdFinished); 
   
  function onAdStarted(e:VungleEvent):void 
  { 
    trace("ad displayed"); 
  } 

  function onAdFinished(e:VungleEvent):void
  { 
    trace("ad dismissed, CTA = " + e.wasCallToActionClicked);
    if (e.wasSuccessfulView)
    {
      trace("counts a completed view - present reward.");
    }
  }
  ```

3. The `AD_LOG` is dispatched when a log message is sent by the Vungle SDK. You can use it for debugging. Logging is implemented only in Vungle SDK for iOS, so this event is platform-specific.

```as3
Vungle.vungle.setLoggingEnabled(true);
Vungle.vungle.addEventListener(VungleEvent.AD_LOG, onAdLog);

private function onAdLog(e:VungleEvent):void
{
	log("ad log: " + e.message);
}

```

### More options

As you have already seen before, you can pass an object with configuration options when calling `playAd()` method. These are the available properties in `VungleAdConfig`:

#### orientation

With this property you can specify the orientation of the ad. There are different set of flags for Android and iOS, see `VungleOrientation` class for details. The flags can be combined with a bitwise `OR` operator:

```as3
config.orientation = VungleOrientation.ANDROID_AUTOROTATE | VungleOrientation.IOS_PORTRAIT;
```

#### soundEnabled

You can use this property to toggle whether ads will play sound, or be muted.

#### backButtonImmediatelyEnabled

This option is Android-specific. If `true`, allows the user to immediately exit an ad using the back button. If `false`, the user cannot use the back button to exit the ad until the on-screen close button is shown.

#### immersiveMode

This option is Android-specific. It enables or disables [immersive mode](https://developer.android.com/training/system-ui/immersive.html) on KitKat+ devices.

#### incentivized

Sets the incentivized mode. If `true`, user will be prompted with a confirmation dialog when attempting to skip the ad.

#### incentivizedUserId

The unique user id to be passed to your application to verify that this user should rewarded for watching an incentivized ad.

#### incentivizedCancelDialogTitle, incentivizedCancelDialogBodyText, incentivizedCancelDialogCloseButtonText, incentivizedCancelDialogKeepWatchingButtonText

These options allow you to customize the confirmation dialog that appears when skipping an incentivized ad.

#### extra1 … extra8

You can use this to keep track of metrics such as age group, gender, etc.

#### placement

An optional ad placement name for enhanced reporting on the dashboard.

#### largeButtons

This option is iOS-specific. If set to `true`, the on-screen buttons displayed over the video will be larger.

### Global defaults

You can use global configuration object to set default values for the options:

```as3
// set any configuration options you like
VungleAdConfig.globalConfig.orientation = VungleOrientation.ANDROID_MATCH_VIDEO;
VungleAdConfig.globalConfig.soundEnabled = false;
```

Then enery new `VungleAdConfig` object is created with these values set by default. `playAd()` without options also uses the global configuration.

### Deprecated methods

Since v2.4.0 the following methods are deprecated:

```as3
Vungle.vungle.displayAd(showCloseButton:Boolean, orientationHint:int):void;
Vungle.vungle.displayIncentivizedAd(name:String, showCloseButton:Boolean, orientationHint:int):void;
Vungle.vungle.setSoundEnabled(enabled:Boolean):void;
Vungle.vungle.setBackButtonEnabled(backEnabled:Boolean):void;
Vungle.vungle.setIncentivizedBackButtonEnabled(backEnabled:Boolean):void;
```

These methods have been kept for backward compatibility. It's not recommended to use them in new applications or mix with the `playAd()` method call.

## Troubleshooting and FAQ

**“How do I use the VungleExample.as Document Class in Flash Professional CS6?”**

1. First, create the application and add the extension by following this guide, Sections 1-3. 

2. Copy and paste VungleExample.as into the same folder as your .fla. Do not copy and paste its contents on to the timeline. That will not work.

3. Change the app ids on line 51 to be your own Vungle App Ids. 

4. In Flash properties, under 'Document Class', type 'VungleExample' (no quotes) and press OK. 

5. Build and install the application.

**“How do I install a newer version of the AIR SDK (4.0 or higher) in Flash Professional CS6?”**

You can download the latest AIR SDK [here](http://www.adobe.com/devnet/air/air-sdk-download.html). If you have already installed AIR 4.0 or higher, you may skip this step. Otherwise, follow the instructions below:

1. Unzip the AIR 4.0 or later SDK package to a location on your hard drive. 

2. Launch Flash Professional CS6. 

3. Select Help > Manage AIR SDK... 

4. Press the Plus (+) Button and navigate to the location of the unzipped AIR SDK 

5. Press OK 

6. Select File > Publish Settings 

7. Select the latest AIR SDK for iOS from the 'Target' Dropdown menu

**“How do I install a newer version of the AIR SDK (4.0 or higher) in Flash Builder?”**

You can download the latest AIR SDK [here](http://www.adobe.com/devnet/air/air-sdk-download.html). If you have already installed AIR 4.0 or higher, you may skip this step. You can find Adobe's latest instructions for updating Flash Builder AIR SDKs [here](http://helpx.adobe.com/flash-builder/kb/overlay-air-sdk-flash-builder.html).


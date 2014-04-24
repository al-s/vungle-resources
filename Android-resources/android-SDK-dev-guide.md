# VungleSDK 

## Android Developer Guide

### Overview

The Publisher SDK allows you to integrate Vungle full screen video ads into your Android application. Please note the following requirements:
* Android 2.2 (Froyo - API version 8) or later
* If your application is written in C/C++, you'll need to use JNI to interface with the Publisher SDK written in Java
* If you are interested in advertising your application on the Vungle network, but not in showing Vungle ads in your application, please see the [Advertiser SDK](http://bd.vungle.com/dev/downloads).

### Integration

You can integrate the Publisher SDK by following these 4 easy steps:

#### 1. Add Vungle SDK To Your Project

[Download](http://bd.vungle.com/dev/android#download) the latest Android Publisher SDK zip file. Unzip it and copy `vungle-publisher-[version].jar` to the `/libs` directory of your project. (Create the directory if it doesn't already exist.)

This should automatically add the Publisher SDK to your build path.

#### 2. Update `AndroidManifest.xml`

Add the following lines:

```xml
<manifest>

  ...
  
  <!-- permissions to download and cache video ads for playback -->
  <uses-permission android:name="android.permission.INTERNET" />
  <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
  <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
  <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
  
  <application>
  
    ...
    
    <!--
      Required Activity for playback of Vungle video ads
      
      NOTE:  The 'configChanges' value 'screenSize' was introduced in Android 3.2 (API level 13).
      If your 'targetSdKVersion' is less than 13, you can either:
      * increase your 'targetSdkVersion' to 13 or greater (recommended)
      * omit the 'screenSize' value
    -->
    <activity
      android:name="com.vungle.publisher.FullScreenAdActivity"
      android:configChanges="keyboardHidden|orientation|screenSize"
      android:theme="@android:style/Theme.NoTitleBar.Fullscreen"
    />
    
    
    <service android:name="com.vungle.publisher.VungleService" android:exported="false"/>
    
  </application>
  
</manifest>
```

#### 3. Integrate the SDK

##### Application Startup

Initialize the Publisher SDK in your application's first `Activity`. This starts video caching and prepares the SDK to display ads.
```java
import com.vungle.publisher.VunglePub;

public class FirstActivity extends android.app.Activity {

  // get the VunglePub instance
  final VunglePub vunglePub = VunglePub.getInstance();

  @Override
  public void onCreate(Bundle savedInstanceState) {
      super.onCreate(savedInstanceState);
      
      // get your App ID from the app's main page on the Vungle Dashboard after setting up your app
      final String app_id = "your Vungle App ID";
      
      // initialize the Publisher SDK
      vunglePub.init(this, app_id);
  }
}
```

##### Each Activity

In addition, override the `onPause` and `onResume` methods in each `Activity` (including the first) to notify the Publisher SDK when your application gains or loses focus:
```java
import com.vungle.publisher.VunglePub;

public class EachActivity extends android.app.Activity {

  // get the VunglePub instance
  final VunglePub vunglePub = VunglePub.getInstance();

  ...
  
  @Override
  protected void onPause() {
      super.onPause();
      vunglePub.onPause();
  }
  
  @Override
  protected void onResume() {
      super.onResume();
      vunglePub.onResume();
  }
}
```

#### 4. Play Video Ad

##### Default Configuration

When you want to actually play the ad in your application, simply call `playAd`
```java
import com.vungle.publisher.VunglePub;

public class GameActivity extends android.app.Activity {

  // get the VunglePub instance
  final VunglePub vunglePub = VunglePub.getInstance();

  ...
  
  private void onLevelComplete() {
      vunglePub.playAd();
  }
}
```

<a name="advancedStartupConfig"></a>
##### Advanced Startup Configuration

After calling `init` you can optionally get access to the global `AdConfig` object. This object allows you to set options that will be automatically applied to every ad you play.
```java
import com.vungle.publisher.VunglePub;
import com.vungle.publisher.AdConfig;
import com.vungle.publisher.Orientation;

public class FirstActivity extends android.app.Activity {

  ...

  @Override
  public void onCreate(Bundle savedInstanceState) {
      super.onCreate(savedInstanceState);
      
      ...

      vunglePub.init(this, app_id);

      // get a handle on the global AdConfig object
      final AdConfig globalAdConfig = vunglePub.getGlobalAdConfig();

      // set any configuration options you like. 
      // For a full description of available options, see the 'Configuration Options' section.
      globalAdConfig.setSoundEnabled(true);
      globalAdConfig.setOrientation(Orientation.portrait);

  }
}
```

##### Advanced Play Configuration

You can optionally customize each individual ad you play by providing an `AdConfig` object to `playAd`. If you [set any options in the global ad configuration](#advancedStartupConfig), those options will be overriden by the provided options.
```java
import com.vungle.publisher.VunglePub;
import com.vungle.publisher.AdConfig;

public class GameActivity extends android.app.Activity {
  ...
  
  private void onLevelComplete() {
  	  // create a new AdConfig object
  	  final AdConfig overrideConfig = new AdConfig();

  	  // set any configuration options you like. 
  	  // For a full description of available options, see the 'Configuration Options' section.
  	  overrideConfig.setIncentivized(true);
  	  overrideConfig.setSoundEnabled(false);

  	  // the overrideConfig object will only affect this ad play. 
  	  // See the Application Startup section for how to set persistent global configurations.
      vunglePub.playAd(overrideConfig);
  }
}
```

### Configuration Options

#### The `AdConfig` Object

One global `AdConfig` object controls settings for all ad plays, and you can optionally pass override instances to each individual ad play. These are the available setters in `AdConfig`:
<table>
	<thead>
		<tr>
			<th>Method</th>
			<th>Default</th>
			<th>Description</th>
		</tr>
	</thead>
	<tbody>
		<tr>
			<td>setOrientation</td>
			<td>Orientation.autoRotate</td>
			<td>Sets the orientation of the ad. Orientation options include portrait, landscape, or auto</td>
		</tr>
		<tr>
			<td>setSoundEnabled</td>
			<td>true</td>
			<td>Sets the starting sound state for the ad. If true, audio respects device volume and sound settings. If false, video begins muted but user may modify</td>
		</tr>
    <tr>
      <td>setBackButtonImmediatelyEnabled</td>
      <td>false</td>
      <td>Enables or disables the back button. If true the user can back out of the ad, otherwise they cannot</td>
    </tr>
    <tr>
      <td>setShowClose</td>
      <td>true</td>
      <td>Enables or disables the close button on the video ad. If false, the close button will never appear</td>
    </tr>
		<tr>
			<td>setIncentivized</td>
			<td>false</td>
			<td>Sets the incentivized mode. If true, user will be prompted with a confirmation dialog when attempting to skip the ad. If false, no confirmation is shown</td>
		</tr>
    <tr>
      <td>setIncentivizedUserId</td>
      <td></td>
      <td>Set the unique user id to be passed to your application to verify that this user should rewarded for watching an incentivized ad. N/A if ad is not incentivized.</td>
    </tr>
		<tr>
			<td>setIncentivizedCancelDialogTitle</td>
			<td>"Close video?"</td>
			<td>Sets the title of the confirmation dialog when skipping an incentivized ad. N/A if ad is not incentivized.</td>
		</tr>
		<tr>
			<td>setIncentivizedCancelDialogBodyText</td>
			<td>"Closing this video early will prevent you from earning your reward. Are you sure?"</td>
			<td>Sets the body of the confirmation dialog when skipping an incentivized ad. N/A if ad is not incentivized.</td>
		</tr>
		<tr>
			<td>setIncentivizedCancelDialogCloseButtonText</td>
			<td>"Close video"</td>
			<td>Sets the 'cancel button' text of the confirmation dialog when skipping an incentivized ad. N/A if ad is not incentivized.</td>
		</tr>
		<tr>
			<td>setIncentivizedCancelDialogKeepWatchingButtonText</td>
			<td>"Keep watching"</td>
			<td>Sets the 'keep watching button' text of the confirmation dialog when skipping an incentivized ad. N/A if ad is not incentivized.</td>
		</tr>
	</tbody>
</table>

#### The `EventListener` Interface
The Publisher SDK raises several events that you can handle programmatically by implementing `com.vungle.publisher.EventListener` and setting it in your `VunglePub` instance using `setEventListener`
```java
import com.vungle.publisher.EventListener;
...

public class FirstActivity extends android.app.Activity {
  ...

  private final EventListener vungleListener = new EventListener(){

    @Override
    public void onVideoView(boolean isCompletedView, int watchedMillis, int videoDurationMillis) {
        // Called each time a video completes. isCompletedView is true if the video was not skipped.     
    }

    @Override
    public void onAdStart() {
        // Called before playing an ad
    }

    @Override
    public void onAdEnd() {
        // Called when the user leaves the ad and control is returned to your application
    }

    @Override
    public void onCachedAdAvailable() {
        // Called when ad is downloaded and ready to be played
    }
    
  };

  @Override
  public void onCreate(Bundle savedInstanceState) {
      ...

      vunglePub.init(this, app_id);
      vunglePub.setEventListener(vungleListener);

  }
}
```

### Upgrading to the Latest Version

We try to make upgrading to the latest version as easy as possible. Simply replace your ```VunglePub.jar``` or ```vungle-publisher-[version].jar``` file with the latest version and recompile!

And if you're upgrading from a version prior to 1.3.x, don't forget to add the new [service](http://bd.vungle.com/dev/android#service) element to your manifest.

### Support

Hopefully, you found integrating the Publisher SDK to be easy. If you're having problems, or if you have feedback, please contact us at [tech-support@vungle.com](mailto:tech-support@vungle.com).
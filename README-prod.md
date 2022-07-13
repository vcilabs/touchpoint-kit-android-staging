# Touchpoint Mobile SDK
The purpose of the Touchpoint mobile SDK is to create an integration between a mobile app and Alida Touchpoint. This integration enables the ability to publish and manage the Touchpoint activities that are displayed in a mobile app without the need to release a new version of the mobile app or make any code changes.

Important concepts regarding Touchpoint can be found [here](https://touchpoint.help.alida.com/redirect.html#9C2DA302D89B4DCD8DE1377F84A07BD1). Some of these concepts are also described briefly below but the link above has a more thorough description.

## Minimum Requirements
- minSdkVersion supported: 23

## Sample App
https://github.com/vcilabs/touchpointkit-sample-android

## Installation
Add the jitpack.io repository to the `build.gradle` or `settings.gradle` file (next to the other repository definitions in the app). For example:

```gradle
dependencyResolutionManagement {
   repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
   repositories {
       google()
       mavenCentral()
       maven { url 'https://jitpack.io' }
   }
}
```

Then add the following dependencies to the build.gradle file. To determine the latest tag please see https://github.com/vcilabs/touchpoint-kit-android/tags.

```gradle
implementation 'com.github.vcilabs:touchpoint-kit-android:0.1.5'
implementation 'com.android.volley:volley:1.2.1'
implementation 'com.google.android.material:material:1.2.1'
implementation 'androidx.startup:startup-runtime:1.0.0'
```

## Concepts
A central design goal of the SDK is to require only an upfront effort from the mobile app developer to "Touchpoint enable" various screens in the mobile app, and eliminate any ongoing effort related to switching which particular activity is being shown. The Touchpoint admin is able to switch the Touchpoint activity with no development effort required.
* Triggers: a trigger is how a Touchpoint activity gets activated and displayed to the user. There are three different types of triggers that are supported in the SDK.
    * Banner: a Banner is displayed after a user navigates to a particular screen. A UI element is shown at the bottom of the screen with a call to action. Tapping the banner will display the Activity.
    * Pop-up: a Pop-up is displayed after a user navigates to a particular screen (after an optional delay). There is no UI component to a Pop-up, the Touchpoint activity simply displays.
    * Custom Component: a Custom Component is a way to trigger a Touchpoint activity during any arbitrary lifecycle event and is fully under the developer's control. An activity can be triggered after a button tap, after the user visits a screen for the 2nd time, or any other custom logic.
* Screens: a Screen is a page or view in your mobile app, such as the home screen, product list screen, product details screen, settings screen, etc. You can designate any screen in your mobile app as being available to display a Touchpoint activity.
* Components: a component is some UI element on a screen in your mobile app. This could be something like a button, or something more abstract like the number of times a user has visited a particular screen.
* Targeting: it is possible for the Touchpoint admin to target particular Touchpoint activities based on certain User Attributes of the current user of the app. The User Attributes of the current user can be defined in the mobile app and are then used by Touchpoint to perform targeting.

### Putting It All Together
As a developer of a mobile app you can designate which Screens and which Screen Components are capable of triggering Touchtpoint activities. You don't need to be concerned about which specific Touchpoint activity is assigned to a particular screen as that is controlled by the Touchpoint admin.

If the desire is to have a Banner or Pop-up activity on a particular screen you will only need to define and provide a screen name. If using a Custom Component you will need to provide both a screen name and a component name. For example:

```kotlin
val screenComponents:List<HashMap<String, String>> = listOf(
    hashMapOf("screenName" to "Home"),
    hashMapOf("screenName" to "Settings", "componentName" to "Lightbulb"),
    hashMapOf("screenName" to "ProductList")
)
```

Here we are defining the `Home` screen and `ProductList` screen as being able to display Banner and Pop-up types of triggers. Then on our `Settings` screen we have a button we've named `Lightbulb` that is now able to display a Custom Component trigger. A Screen can support multiple Components.

## Implementation

### Initial Setup
Add the following to the strings.xml file. `touchpoint_api_key`, `touchpoint_api_secret`, and `touchpoint_pod_name` are required, the rest are optional and for developer convenience.

```xml
<!-- API key and secret are provided by the Touchpoint UI -->
<string name="touchpoint_api_key">API_KEY</string>
<string name="touchpoint_api_secret">API_SECRET</string>

<!-- The pod is the geographical region hosting your instance of Touchpoint. -->
<!-- Easiest determined from your URL while logged in, e.g. eu2.alida.com -->
<!-- Valid values are: NA1, NA2, EU1, EU2, AP2, AP3 -->
<string name="touchpoint_pod_name">EU2</string>

<!-- Should be "false" in production and is "false" by default. -->
<!-- Touchpoint generally won't show an activity to the same user twice which -->
<!-- can make it tricky to test. Setting this to "true" makes it -->
<!-- possible for a user to be served the same activity more than once. -->
<bool name="touchpoint_disable_api_filter">false</bool>

<!-- To disable web view caching, set to "true". "false" by default. -->
<bool name="touchpoint_disable_caching">false</bool>

<!-- Logs at debug level are silenced by default. To enable debug logs, -->
<!-- set this to "true". "false" by default. -->
<bool name="touchpoint_enable_debug_logs">true</bool>

<!-- Prevents any logs being generated. "false" by default. -->
<bool name="touchpoint_disable_all_logs">false</bool>
```

Import the Touchpoint SDK using `import com.visioncritical.touchpointkit.utils.TouchPointActivity` and add the following initialization code. For example in the `onCreate` function of `MainActivity` or similar entry point.

```kotlin
// These are the Screens and Screen Components in your mobile app that you 
// designate as being able to render Touchpoint activities.
val screenComponents:List<HashMap<String, String>> = listOf(
    hashMapOf("screenName" to "Home"),
    hashMapOf("screenName" to "Settings", "componentName" to "Lightbulb"),
    hashMapOf("screenName" to "ProductList")
)

// The visitor payload describes the current user of the app. The "id"
// is used to help determine if this particular user has already
// seen certain activities and should be a unique identifier.
// "userAttributes" are the targeting parameters. "type" is the data type
// found in the "value". The data type is required as Touchpoint has
// various operators that make sense for certain data types and not 
// others, such as "greater than" or "less than" for numbers.
// Valid values are number, boolean, string and date.
val visitor: HashMap<String, Any> = HashMap<String, Any>()
visitor["id"] = "12345"
val userAttributes: Array<HashMap<String, Any>> = Array(4) { i -> HashMap<String, Any>() }
userAttributes[0] = hashMapOf("key" to "age", "type" to "number", "value" to 53)
userAttributes[1] = hashMapOf("key" to "city", "type" to "string", "value" to "Springfield")
userAttributes[2] = hashMapOf("key" to "isLoyaltyMember", "type" to "boolean", "value" to true)
userAttributes[3] = hashMapOf("key" to "previousVisitDate", "type" to "date", "value" to "2022-04-11T21:51:34+0000")
visitor["userAttributes"] = userAttributes

TouchPointActivity.shared.configure(screenComponents, visitor)
```

#### NOTE
Do not leverage visitor attributes to pass personally identifying information (PII) into Touchpoint. To protect visitor privacy, do not pass data into Touchpoint that could be considered as personally identifiable information (PII). PII includes, but is not limited to, information such as social security numbers, personal home addresses, credit card numbers, financial account numbers, street address, etc.

### Triggering Banners and Pop-ups
For Banner and Pop-up triggers you will just need to tell the SDK which screen is currently visible. To do this import the Touchpoint SDK using `import com.visioncritical.touchpointkit.utils.TouchPointActivity` and set the screen name, preferably in the `onCreate` function of an Activity.

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    TouchPointActivity.shared.setCurrentScreen(this, SCREEN_NAME)
}
```

Any Banner or Pop-up assigned to the specified screen will be triggered and display automatically.

Since Pop-ups can be configured by the Touchpoint admin to trigger only after a specified amount of time has passed, it is possible for a user to navigate to a screen and quickly navigate away before the Pop-up is shown. The Pop-up will then be displayed on the second screen, which may be undesired. To prevent this it is possible to cancel the Pop-up when the first screen is being dismissed:

```kotlin
TouchPointActivity.shared.cancelPopupForScreen(SCREEN_NAME)
```

### Triggering Custom Components
For the Custom Component trigger you can hook into any lifecycle event and invoke the Touchpoint activity directly:

```kotlin
TouchPointActivity.shared.openActivityForScreenComponent(this, SCREEN_NAME, COMPONENT_NAME, listener = this)
```

Before calling the `openActivityForScreenComponent` function, it is possible to check if a Touchpoint activity needs to be shown using the following method:

```kotlin
if (TouchPointActivity.shared.shouldShowActivity(SCREEN_NAME, COMPONENT_NAME)) {
    // Call openActivityForScreenComponent
}
```

This will allow you to manage how you render out your screen, for example by hiding or showing a button based on whether or not a Touchpoint activity is availble.

## React Native Integration for Android

In your React Native project will be a folder named `android`. In this folder will be the Android native part of your app. To install this SDK into your app use the method described [above](#installation). Then add the necessary config elements to your `strings.xml` file as discussed in the [Initial Setup](#initial-setup) section.

In the `onCreate` function of your `MainApplication`, import the SDK by adding an import line to the top of the file: `import com.visioncritical.touchpointkit.utils.TouchPointActivity` and add following code snippet. Change the `screenComponents` and `visitor` object according to your requirements.

Please see the [Initial Setup](#initial-setup) section above for a description of how to use the various parameters in the snippet below and what their usage is.

```java
@Override
public void onCreate() {
  super.onCreate();
  
  // ...

  List<HashMap<String, String>> screenComponents = Arrays.asList(
    new HashMap<String, String>() {{
      put("screenName", "Banner Screen");
    }},
    new HashMap<String, String>() {{
      put("screenName", "Popup Screen");
    }},
    new HashMap<String, String>() {{
      put("screenName", "Custom Component Screen");
      put("componentName", "Button 1");
    }},
    new HashMap<String, String>() {{
      put("screenName", "Custom Component Screen");
      put("componentName", "Button 2");
    }},
    new HashMap<String, String>() {{
      put("screenName", "Custom Component Screen");
      put("componentName", "Button 3");
    }}
  );

  HashMap<String, Object> visitor = new HashMap<String, Object>() {{
    put("id", "12345");
  }};
  HashMap[] userAttributes = new HashMap[]{
    new HashMap<String, String>() {{
      put("key", "age");
      put("type", "number");
      put("value", "53");
    }},
    new HashMap<String, String>() {{
      put("key", "isLoyaltyMember");
      put("type", "boolean");
      put("value", "true");
    }},
    new HashMap<String, String>() {{
      put("key", "city");
      put("type", "string");
      put("value", "Springfield");
    }},
    new HashMap<String, String>() {{
      put("key", "previousVisitDate");
      put("type", "date");
      put("value", "2022-04-11T21:51:34+0000");
    }}
  };
  visitor.put("userAttributes", userAttributes);

  TouchPointActivity.Companion.getShared().configure(screenComponents, visitor);

  // ...
}
```

Now create a file named `TouchPointKitBridge.java`. Add the following code to this file:

```java
import android.content.Context;
import android.content.SharedPreferences;
import android.util.Log;

import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.bridge.ReactContextBaseJavaModule;
import com.facebook.react.bridge.ReactMethod;
import com.facebook.react.bridge.ReadableArray;
import com.facebook.react.bridge.ReadableMap;
import com.facebook.react.bridge.ReadableMapKeySetIterator;
import com.facebook.react.bridge.ReadableType;
import com.facebook.react.modules.core.DeviceEventManagerModule;
import com.visioncritical.touchpointkit.utils.TouchPointActivity;
import com.visioncritical.touchpointkit.utils.TouchPointActivityInterface;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;

public class TouchPointKitBridge extends ReactContextBaseJavaModule implements TouchPointActivityInterface {
    private static ReactApplicationContext reactContext;

    TouchPointKitBridge(ReactApplicationContext context) {
        super(context);
        reactContext = context;
    }

    @Override
    public String getName() {
        return "TouchPointKitBridge";
    }

    @ReactMethod
    public void configure(List<HashMap<String, String>> screenComponents, HashMap<String, Object> visitor) {
        TouchPointActivity.Companion.getShared().configure(screenComponents, visitor);
    }

    @ReactMethod
    public void setScreen(String screenName) {
        Context context = getCurrentActivity();

        if (context == null) {
            context = reactContext;
        }

        TouchPointActivity.Companion.getShared().setCurrentScreen(context, screenName);
    }

    @ReactMethod
    public void openActivity(String screenName, String componentName) {
        if(TouchPointActivity.Companion.getShared().shouldShowActivity(screenName, componentName)) {
            Context context = getCurrentActivity();

            if (context == null) {
                context = reactContext;
            }
            TouchPointActivity.Companion.getShared().openActivityForScreenComponent(context, screenName, componentName, this);
        }
    }

    @ReactMethod
    public void enableDebugLogs(Boolean enable) {
        TouchPointActivity.Companion.getShared().setEnableDebugLogs(enable);
    }

    @ReactMethod
    public void disableAllLogs(Boolean disable) {
        TouchPointActivity.Companion.getShared().setDisableAllLogs(disable);
    }

    @ReactMethod
    public void disableAPIFilter(Boolean apiFilter) {
        TouchPointActivity.Companion.getShared().setDisableApiFilter(apiFilter);
    }

    @ReactMethod
    public void disableCaching(Boolean caching) {
        TouchPointActivity.Companion.getShared().setDisableCaching(caching);
    }

    @ReactMethod
    public void setVisitor(HashMap<String, Object> visitor) {
        TouchPointActivity.Companion.getShared().setVisitor(visitor);
    }

    @Override
    public void onTouchPointActivityFinished() {
        Log.d("TouchPointKitBridge","onTouchPointActivityFinished...");
        this.reactContext.getJSModule(DeviceEventManagerModule.RCTDeviceEventEmitter.class)
                .emit("onActivityComplete", "TouchPointActivityFinished");
    }
}
```

And then create another file named `TouchPointKitPackage.java` and add the following code into it:

```java
import com.facebook.react.ReactPackage;
import com.facebook.react.bridge.NativeModule;
import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.uimanager.ViewManager;

import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class TouchPointKitPackage implements ReactPackage {

    @Override
    public List<ViewManager> createViewManagers(ReactApplicationContext reactContext) {
        return Collections.emptyList();
    }

    @Override
    public List<NativeModule> createNativeModules(
            ReactApplicationContext reactContext) {
        List<NativeModule> modules = new ArrayList<>();

        modules.add(new TouchPointKitBridge(reactContext));

        return modules;
    }
}
```

In `MainApplication.java`, in the method `protected List<ReactPackage> getPackages()` add the following line:
```java
packages.add(new TouchPointKitPackage());
```

From your App.js call `TouchPointKitBridge` methods using `NativeModules`.

```javascript
import {
  NativeModules,
  NativeEventEmitter,
} from 'react-native';

// Register for event listening from SDK (activity complete event)
const { TouchPointKitBridge } = NativeModules;
const eventEmitter = new NativeEventEmitter(TouchPointKitBridge);
const subscription = eventEmitter.addListener(
  'onActivityComplete',
  onActivityComplete,
);

const onActivityComplete = event => {
  console.log('onActivityComplete called');
  console.log(event);
};

// To trigger a Pop-up or Banner
NativeModules.TouchPointKitBridge.setScreen('SCREEN_NAME');

// To trigger a custom component
NativeModules.TouchPointKitBridge.openActivity('SCREEN_NAME', 'COMPONENT_NAME');
```

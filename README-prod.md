# Touchpoint Mobile SDK
The purpose of the Touchpoint mobile SDK is to create an integration between a mobile app and Alida Touchpoint. This integration enables the ability to publish and manage the Touchpoint activities that are displayed in a mobile app without the need to release a new version of the mobile app or make any code changes.

Important concepts regarding Touchpoint can be found here: ### insert webhelp link here ###. Some of these concepts are also described briefly below but the link above has a more thorough description.

## Minimum Requirements
- minSdkVersion supported: 23

## Sample App
https://github.com/vcilabs/touchpointkit-sample-android

## Installation using Pods
Add the jitpack.io repository to the build.gradle or settings.gradle file (next to the other repository definitions in the app). For example:

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

Add the following to the strings.xml file. `api_key`, `api_secret`, and `pod_name` are required, the rest are optional and for developer convenience.

```xml
<!-- API key and secret are provided by the Touchpoint UI -->
<string name="api_key">API_KEY</string>
<string name="api_secret">API_SECRET</string>

<!-- The pod is the geographical region hosting your instance of Touchpoint. -->
<!-- Easiest determined from your URL while logged in, e.g. eu2.alida.com -->
<!-- Valid values are: na1, na2, eu1, eu2, ap2, ap3 -->
<string name="pod_name">EU2</string>

<!-- Should be "false" in production and is "false" by default. -->
<!-- Touchpoint generally won't show an activity to the same user twice which -->
<!-- can make it tricky to test. Setting this to "true" makes it -->
<!-- possible for a user to be served the same activity more than once. -->
<bool name="disable_api_filter">false</bool>

<!-- To disable web view caching, set to "true". "false" by default. -->
<bool name="disable_caching">false</bool>

<!-- Logs at debug level are silenced by default. To enable debug logs, -->
<!-- set this to "true". "false" by default. -->
<bool name="enable_debug_logs">true</bool>

<!-- Prevents any logs being generated, default is "false". -->
<bool name="disable_all_logs">false</bool>
```

Import the Touchpoint SDK using `import com.visioncritical.touchpointkit.utils.TouchPointActivity` and add the following initialization code. For example in the `onCreate` function of `MainActivity` or similar entry point.

```kotlin
val screenComponents:List<HashMap<String, String>> = listOf(
    hashMapOf("screenName" to "Home"),
    hashMapOf("screenName" to "Settings", "componentName" to "Lightbulb"),
    hashMapOf("screenName" to "ProductList")
)

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

### Triggering Banners and Pop-ups

For Banner and Pop-up triggers you will just need to tell the SDK which screen is currently visible. To do this import the Touchpoint SDK using `import com.visioncritical.touchpointkit.utils.TouchPointActivity` and set the screen name, preferably in the `onCreate` function of an Activity.

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    TouchPointActivity.shared.setCurrentScreen(this, SCREEN_NAME)
}
```

Any Banner or Pop-up assigned to the specified screen will be triggered and display automatically.

Since Pop-ups can be configured by the Touchpoint admin to trigger only after a specified amount of time has passed, it is possible for a user to navigate to a screen and quickly navigate away before the Pop-up is shown. The Pop-up will then be displayed on the second screen, which may be undesired. To prevent this it is possible to cancel the Pop-up when the first screen is being destroyed:

```kotlin
override fun onDestroy() {
    super.onDestroy()
    TouchPointActivity.shared.cancelPopupForScreen(SCREEN_NAME)
}
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
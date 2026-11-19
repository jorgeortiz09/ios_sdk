## Summary

This is the iOS SDK of Adjust™. You can read more about Adjust™ at [adjust.com].

If your app is an app which uses web views you would like to use adjust tracking from Javascript code, please consult our [iOS web views SDK guide][ios-web-views-guide].

Read this in other languages: [English][en-readme], [中文][zh-readme], [日本語][ja-readme], [한국어][ko-readme].

## Table of contents

* [Example apps](#example-apps)
* [Basic integration](#basic-integration)
   * [Add the SDK to your project](#sdk-add)
   * [Add iOS frameworks](#sdk-frameworks)
   * [Integrate the SDK into your app](#sdk-integrate)
   * [Basic setup](#basic-setup)
      * [iMessage specific setup](#basic-setup-imessage)
   * [Adjust logging](#adjust-logging)
   * [Build your app](#build-the-app)
* [Additional features](#additional-features)
   * [Event tracking](#event-tracking)
      * [Revenue tracking](#revenue-tracking)
      * [Revenue deduplication](#revenue-deduplication)
      * [In-App Purchase verification](#iap-verification)
      * [Callback parameters](#callback-parameters)
      * [Partner parameters](#partner-parameters)
      * [Callback identifier](#callback-id)
   * [Session parameters](#session-parameters)
      * [Session callback parameters](#session-callback-parameters)
      * [Session partner parameters](#session-partner-parameters)
      * [Delay start](#delay-start)
   * [Attribution callback](#attribution-callback)
   * [Event and session callbacks](#event-session-callbacks)
   * [Disable tracking](#disable-tracking)
   * [Offline mode](#offline-mode)
   * [Event buffering](#event-b

There are example apps inside the [`examples` directory][examples] for [`iOS (Objective-C)`][example-ios-objc], [`iOS (Swift)`][example-ios-swift], [`tvOS`][example-tvos], [`iMessage`][example-imessage] and [`Apple Watch`][example-iwatch]. You can open any of these Xcode projects to see an example of how the Adjust SDK can be integrated.

## <a id="basic-integration">Basic integration

We will describe the steps to integrate the Adjust SDK into your iOS project. We are going to assume that you are using Xcode for your iOS development.

### <a id="sdk-add"></a>Add the SDK to your project

If you're using [CocoaPods][cocoapods], you can add the following line to your `Podfile` and continue from [this step](#sdk-integrate):

```ruby
pod 'Adjust', '~> 4.17.1'
```

or:

```ruby
pod 'Adjust', :git => 'https://github.com/adjust/ios_sdk.git', :tag => 'v4.17.1'
```

---

If you're using [Carthage][carthage], you can add following line to your `Cartfile` and continue from [this step](#sdk-frameworks):

```ruby
github "adjust/ios_sdk"
```

---

You can also choose to integrate the Adjust SDK by adding it to your project as a framework. On the [releases page][releases] you can find the following archives:

* `AdjustSdkStatic.framework.zip`
* `AdjustSdkDynamic.framework.zip`
* `AdjustSdkTv.framework.zip`
* `AdjustSdkIm.framework.zip`

Since the release of iOS 8, Apple has introduced dynamic frameworks (also known as embedded frameworks). If your app is targeting iOS 8 or higher, you can use the Adjust SDK dynamic framework. Choose which framework you want to use – static or dynamic – and add it to your project.

If you are having `tvOS` app, you can use the Adjust SDK with it as well with usage of our tvOS framework which you can extract from `AdjustSdkTv.framework.zip` archive.

If you are having `iMessage` app, you can use the Adjust SDK with it as well with usage of our IM framework which you can extract from `AdjustSdkIm.framework.zip` archive.

### <a id="sdk-frameworks"></a>Add iOS frameworks

1. Select your project in the Project Navigator
2. In the left-hand side of the main view, select your target
3. In the `Build Phases` tab, expand the `Link Binary with Libraries` group
4. At the bottom of that section, select the `+` button
5. Select the `AdSupport.framework`, then the `Add` button 
6. Unless you are using tvOS, repeat the same steps to add the `iAd.framework` and `CoreTelephony.framework`
7. Change the `Status` of the frameworks to `Optional`.

### <a id="sdk-integrate"></a>Integrate the SDK into your app

If you added the Adjust SDK via a Pod repository, you should use one of the following import statements:

```objc
#import "Adjust.h"
```

or

```objc
#import <Adjust/Adjust.h>
```

---

If you added the Adjust SDK as a static/dynamic framework or via Carthage, you should use the following import statement:

```objc
#import <AdjustSdk/Adjust.h>
```

---

If you are are using the Adjust SDK with your tvOS app, you should use the following import statement:

```objc
#import <AdjustSdkTv/Adjust.h>
```

---

If you are are using the Adjust SDK with your iMessage app, you should use the following import statement:

```objc
#import <AdjustSdkIm/Adjust.h>
```

Next, we'll set up basic session tracking.

### <a id="basic-setup"></a>Basic setup

In the Project Navigator, open the source file of your application delegate. Add the `import` statement at the top of the file, then add the following call to `Adjust` in the `didFinishLaunching` or `didFinishLaunchingWithOptions` method of your app delegate:

```objc
#import "Adjust.h"
// or #import <Adjust/Adjust.h>
// or #import <AdjustSdk/Adjust.h>
// or #import <AdjustSdkTv/Adjust.h>
// or #import <AdjustSdkIm/Adjust.h>

// ...

NSString *yourAppToken = @"{YourAppToken}";
NSString *environment = ADJEnvironmentSandbox;
ADJConfig *adjustConfig = [ADJConfig configWithAppToken:yourAppToken
                                            environment:environment];

[Adjust appDidLaunch:adjustConfig];
```

![][delegate]

**Note**: Initialising the Adjust SDK like this is `very important`. Otherwise, you may encounter different kinds of issues as described in our [troubleshooting section](#ts-delayed-init).

Replace `{YourAppToken}` with your app token. You can find this in your [dashboard].

Depending on whether you build your app for testing or for production, you must set `environment` with one of these values:

```objc
NSString *environment = ADJEnvironmentSandbox;
NSString *environment = ADJEnvironmentProduction;
```

**Important:** This value should be set to `ADJEnvironmentSandbox` if and only if you or someone else is testing your app. Make sure to set the environment to `ADJEnvironmentProduction` just before you publish the app. Set it back to `ADJEnvironmentSandbox` when you start developing and testing it again.

We use this environment to distinguish between real traffic and test traffic from test devices. It is very important that you keep this value meaningful at all times! This is especially important if you are tracking revenue.

### <a id="basic-setup-imessage"></a>iMessage specific se``

### <a id="build-the-app"></a>Build your app

Build and run your app. Ife build succeeds, you should carefully read the SDK logs in the console. After the app launches for the first time, you should see the info log `Install tracked`.

![][run]

## <a id="additional-feature">Additional features

Once you integrate the Adjust SDK into your project, you can take advantage of the following features.

### <a id="event-tracking"></a>Event tracking

You can use adjust to track events. Lets say you want to track every tap on a particular button. You would create a new event token in your [dashboard], which has an associated event token - looking something like `abc123`. In your button's `buttonDown` method you would then add the following lines to track the tap:

```objc
ADJEvent *event = [ADJEvent eventWithEventToken:@"abc123"];
[Adjust trackEvent:event];
```

When tapping the button you should now see `Event tracked` in the logs.

The event instance can be used to configure the event further before tracking itOnce you have successfully enabled the universal links feature in the dashboard, you need to do this in your app as well:

After enabling `Associated Domains` for your app in Apple Developer Portal, you need to do the same thing in your app's Xcode project. After enabling `Assciated Domains`, add the universal link which was generated for you in the adjust dashboard in the `Domains` section by prefixing it with `applinks:` and make sure that you also remove the `http(s)` part of the universal link.

![][associated-domains-applinks]

After this has been set up, your app will be opened after you click the adjust tracker universal link. After app is opened, `continueUserActivity` method of your `AppDelegate` class will be triggered and the place where the content of the universal link URL will be delivered. If you want to access the content of the deep link, override this method.

``` objc
- (BOOL)application:(UIApplication *)application continueUserActivity:(NSUserActivity *)userActivity
 restorationHandler:(void (^)(NSArray *restorableObjects))restorationHandler {
    if ([[userActivity activityType] isEqualToString:NSUserActivityTypeBrowsingWeb]) {
        NSURL *url = [userActivity webpageURL];

        // url object contains your universal link content
    }

    // Apply your logic to determine the return value of this method
    return YES;
    // or
    // return NO;
}
```

With this setup, you have successfully set up deep linking handling for iOS devices with iOS 9 and later versions.

We provide a helper function that allows you to convert a universal link to an old style deep link URL, in case you had some custom logic in your code which was always expecting deep link info to arrive in old style custom URL scheme format. You can call this method with universal link and the custom URL scheme name which you would like to see your deep link prefixed with and we will generate the custom URL scheme deep link for you:

``` objc
- (BOOL)application:(UIApplication *)application continueUserActivity:(NSUserActivity *)userActivity
 restorationHandler:(void (^)(NSArray *restorableObjects))restorationHandler {
    if ([[userActivity activityType] isEqualToString:NSUserActivityTypeBrowsingWeb]) {
        NSURL *url = [userActivity webpageURL];

        NSURL *oldStyleDeeplink = [Adjust convertUniversalLink:url scheme:@"adjustExample"];
    }

    // Apply your logic to determine the return value of this method
    return YES;
    // or
    // return NO;
}
```

### <a id="deeplinking-deferred"></a>Deferred deep linking scenario

You can register a delegate callback to be notified before a deferred deep link is opened and decide if the Adjust SDK will try to open it. The same optional protocol `AdjustDelegate` used for the [attribution callback](#attribution-callback) and for [event and session callbacks](#event-session-callbacks) is used.

Follow the same steps and implement the following delegate callback function for deferred deep links:

```objc
- (BOOL)adjustDeeplinkResponse:(NSURL *)deeplink {
    // deeplink object contains information about deferred deep link content

    // Apply your logic to determine whether the Adjust SDK should try to open the deep link
    return YES;
    // or
    // return NO;
}
```

The callback function will be called after the SDK receives a deffered deep link from our server and before opening it. Within the callback function you have access to the deep link. The returned boolean value determines if the SDK will launch the deep link. You could, for example, not allow the SDK to open the deep link at the current moment, save it, and open it yourself later.

If this callback is not implemented, **the Adjust SDK will always try to open the deep link by default**.

### <a id="deeplinking-reattribution"></a>Reattribution via deep links

Adjust enables you to run re-engagement campaigns with usage of deep links. For more information on how to do that, please check our [official docs][reattribution-with-deeplinks].

If you are using this feature, in order for your user to be properly reattributed, you need to make one additional call to the Adjust SDK in your app.

Once you have received deep link content information in your app, add a call to the `appWillOpenUrl` method. By making this call, the Adjust SDK will try to find if there is any new attribution info inside of the deep link and if any, it will be sent to the adjust backend. If your user should be reattributed due to a click on the adjust tracker URL with deep link content in it, you will see the [attribution callback](#attribution-callback) in your app being triggered with new attribution info for this user.

The call to `appWillOpenUrl` should be done like this to support deep linking reattributions in all iOS versions:

```objc
- (BOOL)application:(UIApplication *)application openURL:(NSURL *)url
  sourceApplication:(NSString *)sourceApplication annotation:(id)annotation {
    // url object contains your deep link content
    
    [Adjust appWillOpenUrl:url];

    // Apply your logic to determine the return value of this method
    return YES;
    // or
    // return NO;
}
```

``` objc
- (BOOL)application:(UIApplication *)application continueUserActivity:(NSUserActivity *)userActivity
 restorationHandler:(void (^)(NSArray *restorableObjects))restorationHandler {
    if ([[userActivity activityType] isEqualToString:NSUserActivityTypeBrowsingWeb]) {
        NSURL url = [userActivity webpageURL];

        [Adjust appWillOpenUrl:url];
    }

    // Apply your logic to determine the return value of this method
    return YES;
    // or
    // return NO;
}
```

## <a id="troubleshooting"></a>Troubleshooting

### <a id="ts-delayed-init"></a>Issues with delayed SDK initialisation

As described in the [basic setup step](#basic-setup), we strongly advise you to initialise the Adjust SDK in the `didFinishLaunching` or `didFinishLaunchingWithOptions` method of your app delegate. It is imperative to initialise the Adjust SDK in as soon as possible so that you can use all the features of the SDK.

Deciding not to initialise the Adjust SDK immediately can have all kinds of impacts on the tracking in your app: **In order to perform any kind of tracking in your app, the Adjust SDK *must* be initialised.**

If you decide to perform any of these actions:

* [Event tracking](#event-tracking)
* [Reattribution via deep links](#deeplinking-reattribution)
* [Disable tracking](#disable-tracking)
* [Offline mode](#offline-mode)

before initialising the SDK, `they won't be performed`.

If you want any of these actions to be tracked with the Adjust SDK before its actual initialisation, you must build a `custom actions queueing mechanism` inside your app. You need to queue all the actions you want our SDK to perform and perform them once the SDK is initialised.

Offline mode state won't be changed, tracking enabled/disabled state won't be changed, deep link reattributions will not be possible to happen, any of tracked events will be `dropped`.

Another thing which might be affected by delayed SDK initialisation is session tracking. The Adjust SDK can't start to collect any session length info before it is actually initialised. This can affect your DAU numbers in the dashboard which might not be tracked properly.

As an example, let's assume this scenario: You are initialising the Adjust SDK when some specific view or view controller is loaded and let's say that this is not the splash nor the first screen in your app, but user has to navigate to it from the home screen. If user downloads your app and opens it, the home screen will be displayed. At this moment, this user has made an install which should be tracked. However, the Adjust SDK doesn't know anything about this, because the user needs to navigate to the screen mentioned previously where you decided to initialise the Adjust SDK. Further, if the user decides that he/she doesn't like the app and uninstalls it right after seeing home screen, all the information mentioned above will never be tracked by our SDK, nor displayed in the dashboard.

#### Event tracking

For the events you want to track, queue them with some internal queueing mechanism and track them after SDK is initialised. Tracking events before initialising SDK will cause the events to be `dropped` and `permanently lost`, so make sure you are tracking them once SDK is `initialised` and [`enabled`](#is-enabled).

#### Offline mode and enable/disable tracking

Offline mode is not the feature which is persisted between SDK initialisations, so it is set to `false` by default. If you try to enable offline mode before initialising SDK, it will still be set to `false` when you eventually initialise the SDK.

Enabling/disabling tracking is the setting which is persisted between the SDK initialisations. If you try to toggle this value before initialising the SDK, toggle attempt will be ignored. Once initialised, SDK will be in the state (enabled or disabled) like before this toggle attempt.

#### Reattribution via deep links

As described [above](#deeplinking-reattribution), when handling deep link reattributions, depending on deep linking mechanism you are using (old style vs. universal links), you will obtain `NSURL` object after which you need to make following call:

```objc
[Adjust appWillOpenUrl:url]
```

If you make this call before the SDK has been initialised, information about the attribution information from the deep link URL will be permanetly lost. If you want the Adjust SDK to successfully reattribute your user, you would need to queue this `NSURL` object information and trigger `appWillOpenUrl` method once the SDK has been initialised.

#### Session tracking

Session tracking is something what the Adjust SDK performs automatically and is beyond reach of an app developer. For proper session tracking it is crucial to have the Adjust SDK initialised as advised in this README. Not doing so can have unpredicted influences on proper session tracking and DAU numbers in the dashboard.

For example:
* A user opens but then deletes your app before the SDK was even inialised, causing the install and session to have never been tracked, thus never reported in the dashboard.
* If a user downloads and opens your app before midnight, and the Adjust SDK gets initialised after midnight, all queued install and session data will be reported on wrong day.
* If a user didn't use your app on some day but opens it shortly after midnight and the SDK gets initialised after midnight, causing DAU to be reported on another day from the day of the app opening.

For all these reasons, please follow the instructions in this document and initialise the Adjust SDK in the `didFinishLaunching` or `didFinishLaunchingWithOptions` method of your app delegate.

### <a id="ts-arc"></a>I'm seeing "Adjust requires ARC" error

If your build failed with the error `Adjust requires ARC`, it looks like your project is not using [ARC][arc]. In that case we recommend [transitioning your project][transition] so that it does use ARC. If you don't want to use ARC, you have to enable ARC for all source files of adjust in the target's Build Phases:

Expand the `Compile Sources` group, select all adjust files and change the `Compiler Flags` to `-fobjc-arc` (Select all and press the `Return` key to change all at once).

### <a id="ts-categories"></a>I'm seeing "[UIDevice adjTrackingEnabled]: unrecognized selector sent to instance" error

This error can occur when you are adding the Adjust SDK framework to your app. The Adjust SDK contains `categories` among it's source files and for this reason, if you have chosen this SDK integration approach, you need to add `-ObjC` flags to `Other Linker Flags` in your Xcode project settings. Adding this flag will fix this error.

### <a id="ts-session-failed"></a>I'm seeing the "Session failed (Ignoring too frequent session.)" error

This error typically occurs when testing installs. Uninstalling and reinstalling the app is not enough to trigger a new install. The servers will determine that the SDK has lost its locally aggregated session data and ignore the erroneous message, given the information available on the servers about the device.

This behaviour can be cumbersome during tests, but is necessary in order to have the sandbox behaviour match production as much as possible.

You can reset the session data of the device in our servers. Check the error message in the logs:

```
Session failed (Ignoring too frequent session. Last session: YYYY-MM-DDTHH:mm:ss, this session: YYYY-MM-DDTHH:mm:ss, interval: XXs, min interval: 20m) (app_token: {yourAppToken}, adid: {adidValue})
```

<a id="forget-device">With the `{y

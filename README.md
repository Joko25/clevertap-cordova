CleverTap Cordova Plugin
========

## Supported Versions

Tested on Cordova 5.3.3

- [CleverTap Android SDK version 2.0.9](https://github.com/CleverTap/clevertap-android-sdk/releases/tag/2.0.9)
- [CleverTap iOS SDK version 2.0.10, Xcode 7 only](https://github.com/CleverTap/clevertap-ios-sdk/releases/tag/2.0.10)

## Install

Using Cordova  
```
cordova plugin add https://github.com/CleverTap/clevertap-cordova.git
```

Using Ionic  
```
ionic plugin add https://github.com/CleverTap/clevertap-cordova.git
```

## Integration

To integrate CleverTap for Cordova:

1. Sign up 

2. Set up and register for Push notifications, if required.

3. Integrate your Javascript with the CleverTap Plugin.

### 1. Sign up

[Sign up](http://www.clevertap.com/sign-up/) for a free account.  

When you create your CleverTap account, you will also automatically get a -Test account.  Use the -Test account for development and the main account for production.

#### iOS

Update your .plist file:

* Create a key called CleverTapAccountID with a string value
* Create a key called CleverTapToken with a string value
* Insert the values from your CleverTap [Dashboard](https://dashboard.clevertap.com) -> Settings -> Integration Details.


![plist account values](http://staging.support.wizrocket.com.s3-website-eu-west-1.amazonaws.com/images/integration/plist-account.png)

#### Android

Add the following inside the `<application></application>` tags of your AndroidManifest.xml:  

    <meta-data  
        android:name="CLEVERTAP_ACCOUNT_ID"  
        android:value="Your CleverTap Account ID"/>  
    <meta-data  
        android:name="CLEVERTAP_TOKEN"  
        android:value="Your CleverTap Account Token"/>

Replace "Your CleverTap Account ID" and "Your CleverTap Account Token" with actual values from your CleverTap [Dashboard](https://dashboard.clevertap.com) -> Settings -> Integration -> Account ID, SDK's.

**Set the Lifecycle Callback** 

IMPORTANT!

Add the `android:name` property to the `<application>` tag of our AndroidManifest.xml:

    <application
        android:name="com.clevertap.android.sdk.Application">

**Note:** The above step is **extremely important** and enables CleverTap to track notification opens, display in-app notifications, track deep links, and other important **user behaviour**.

**Add Permissions**

Please ensure that you're requesting the following permissions for your app:

    <!-- Required to retrieve a unique identifier for the device, see note below -->
    <uses-permission android:name="android.permission.READ_PHONE_STATE"/>
    <!-- Required to allow the app to send events -->
    <uses-permission android:name="android.permission.INTERNET"/>
    <!-- Recommended so that we can be smart about when to send the data -->
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
    <!-- Recommended so that we can get the user's primary e-mail address -->
    <uses-permission android:name="android.permission.GET_ACCOUNTS"/>
    <!-- Recommended so that we can get the user's location -->
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>
    <!-- Recommended -->
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>

[Please see the example AndroidManifest.xml here](https://github.com/CleverTap/clevertap-cordova/blob/master/Starter/platforms/android/AndroidManifest.xml).

**Add Dependencies**

Make sure Google Play Services and Android v4 support library, minimum revision 23.1.1 are added as dependencies in your project.   

Add an extra line to YOUR_PROJECT/platforms/android/project.properties: `cordova.system.library.1=com.google.android.gms:play-services:8.3.0`  
 
Make sure your build.gradle file includes the play-services and support library dependencies:

    dependencies {
        compile fileTree(dir: 'libs', include: '*.jar'  )
        // SUB-PROJECT DEPENDENCIES START  
        debugCompile project(path: "CordovaLib", configuration: "debug")  
        releaseCompile project(path: "CordovaLib", configuration: "release")  
        compile 'com.google.android.gms:play-services:8.3.0'
        compile 'com.android.support:support-v4:23.1.1'
        // SUB-PROJECT DEPENDENCIES END   


### 2. Set up and register for push notifications

#### iOS

[Follow Steps 1 and 2 in these instructions to set up push notifications for your app.](https://support.clevertap.com/messaging/push-notifications/#ios)

If you plan on including deep links in your push notifications, [please register your custom url scheme as described here](https://support.clevertap.com/messaging/deep-linking/#step-1-register-your-custom-scheme).  

Afterwards, call the following from your Javascript.

    CleverTap.registerPush();


To handle deep links contained in push notifications, you can pass the deep link url to your javascript by firing a custom document event from your AppDelegate application:openURL:sourceApplication: method.  See the example below:   
     
    - (BOOL)application:(UIApplication*)application openURL:(NSURL*)url sourceApplication:(NSString*)sourceApplication annotation:(id)annotation
    {
        if (!url) {
            return NO;
        }

        // all plugins will get the notification, and their handlers will be called
        [[NSNotificationCenter defaultCenter] postNotification:[NSNotification notificationWithName:CDVPluginHandleOpenURLNotification object:url]];
        
        // example Deep Link Handling
        NSString *scheme = [url scheme];
        if([scheme isEqualToString:@"<your custom url scheme>"]) {
            NSString *js = [NSString stringWithFormat:@"cordova.fireDocumentEvent('onDeepLink', {'deeplink':'%@'});", url.description];
            [self.viewController.commandDelegate evalJs:js];
        }

        return YES;
    }

[See the included example Starter Cordova project](https://github.com/CleverTap/clevertap-cordova/blob/master/Starter/platforms/ios/CleverTapStarter/Classes/AppDelegate.m).


#### Android

[Follow these instructions to set up push notifications for your app.](https://support.clevertap.com/messaging/push-notifications/#android)

To handle deep links contained in push notifications, you can pass the deep link url to your javascript by firing a custom document event from your MainActivity.  See the example below:   

Add the required intent filter to your MainActivity tag in AndroidManifest.xml:

    <!-- Example Deep Link Handling, substitute your deeplink scheme for clevertapstarter -->
            <intent-filter android:label="@string/app_name">
                <action android:name="android.intent.action.VIEW" />

                <category android:name="android.intent.category.DEFAULT" />
                <category android:name="android.intent.category.BROWSABLE" />

                <data android:scheme="clevertapstarter" />
            </intent-filter>

Handle the intent in your MainActivity.java:  

    // example deeplink handling
    @Override
    public void onNewIntent(Intent intent) {

        super.onNewIntent(intent);

        if (intent.getAction().equals(Intent.ACTION_VIEW)) {
            Uri data = intent.getData();
            if (data != null) {
                final String json = "{'deeplink':'"+data.toString()+"'}";
                loadUrl("javascript:cordova.fireDocumentEvent('onDeepLink'," + json + ");");
            }
        }
    }
     
### 3. Integrate Javascript with the Plugin

After integrating, all calls to the CleverTap SDK should be made from your Javascript.

Start by adding the following listeners to your Javascript:

    document.addEventListener('deviceready', this.onDeviceReady, false);
    document.addEventListener('onCleverTapProfileSync', this.onCleverTapProfileSync, false); // optional: to be notified of CleverTap user profile synchronization updates 
    document.addEventListener('onDeepLink', this.onDeepLink, false); // example: optional, register your own custom listener to handle deep links passed from your native code.

Then:  

- [see the included iOS Starter Cordova project for example usage](https://github.com/CleverTap/clevertap-cordova/blob/master/Starter/platforms/ios/www/js/index.js).   

- [see the included Android Starter Cordova project for example usage](https://github.com/CleverTap/clevertap-cordova/blob/master/Starter/platforms/android/assets/www/js/index.js).  



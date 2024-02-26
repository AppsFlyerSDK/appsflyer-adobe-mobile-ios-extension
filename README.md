<img src="gitresources/AF_Logo_primary_logo.png"  width="400" > 

# appsflyer-adobe-mobile-ios-extension

[![Version](https://img.shields.io/cocoapods/v/AppsFlyerAdobeExtension.svg?style=flat)](http://cocoapods.org/pods/AppsFlyerAdobeExtension)


🛠 In order for us to provide optimal support, we would kindly ask you to submit any issues to support@appsflyer.com

> *When submitting an issue please specify your AppsFlyer sign-up (account) email , your app ID , production steps, logs, code snippets and any additional relevant information.*

## Table of content

- [Adding the SDK to your project](#add-sdk-to-project)
- [Initializing the SDK](#init-sdk)
- [Manual Mode](#manual-mode)
- [Guides](#guides)
- [API](#api) 
- [Data Elements](#data-elements)
- [Send consent for DMA compliance](#dma_support)
- [Swift Example](#swift-example)


### <a id="plugin-build-for"> This plugin is built for
    
- iOS AppsFlyer SDK **v6.13.0**

## <a id="add-sdk-to-project"> 📲 Adding the SDK to your project

Add the following to your app's `Podfile`:

```javascript
pod 'AppsFlyerAdobeExtension', '6.13.0'
```

## <a id="init-sdk"> 🚀 Initializing the SDK
    
Register the AppsFlyer extension from your `Application` class, alongside the Adobe SDK initialisation code: 
```objc
...
#import "AppsFlyerAdobeExtension/AppsFlyerAdobeExtension.h"

@implementation AppDelegate


- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    [ACPCore configureWithAppId:@"Key"];
    ...
    [AppsFlyerAdobeExtension registerExtension];
    ...
    
    [AppsFlyerAdobeExtension registerCallbacks:^(NSDictionary *dictionary) {
        NSLog(@"[AppsFlyerAdobeExtension] Received callback: %@", dictionary);
        if([[dictionary objectForKey:@"callback_type"] isEqualToString:@"onConversionDataReceived"]){
            if([[dictionary objectForKey:@"is_first_launch"] boolValue] == YES){
                NSString* af_status = [dictionary objectForKey:@"af_status"];
                if([af_status isEqualToString:@"Non-organic"]){
                    NSLog(@"this is first launch and a non organic install!");
                }
            }
        } else if([[dictionary objectForKey:@"callback_type"] isEqualToString:@"onAppOpenAttribution"]) {
            NSLog(@"onAppOpenAttribution Received");
        }
     }];

    [AppsFlyerAdobeExtension callbacksErrorHandler:^(NSError *error) {
          NSLog(@"[AppsFlyerAdobeExtension] Error receivng callback: %@" , error);
      }];
    
    return YES;
}

```

In Addition to adding the init code, the settings inside the launch dashboard must be set.

<img src="./gitresources/LaunchAFInitNew.png" width="550" >

| Setting  | Description   |
| -------- | ------------- |
| AppsFlyer iOS App ID      | Your iTunes [application ID](https://support.appsflyer.com/hc/en-us/articles/207377436-Adding-a-new-app#available-in-the-app-store-google-play-store-windows-phone-store)  (required for iOS only)  |
| AppsFlyer Dev Key   | Your application [devKey](https://support.appsflyer.com/hc/en-us/articles/211719806-Global-app-settings-#sdk-dev-key) provided by AppsFlyer (required)  |
| Bind in-app events for    | Bind adobe event to appsflyer in-app events. For more info see the doc [here](/docs/Guides.md#events). |
| Send attribution data    | Send conversion data from the AppsFlyer SDK to adobe. This is required for data elements. |
| Debug Mode    | Debug mode - set to `true` for testing only.  |
| Wait for ECID   | Once enabled, the SDK Initialization will be delayed until the Experience Cloud ID is set.  |

> Note: For Send attribution data, use this feature if you are only working with ad networks that allow sharing user level data with 3rd party tools.

## <a id="manual-mode"> Manual mode
Starting version `6.13.0`, we support a manual mode to separate the initialization of the AppsFlyer SDK and the start of the SDK.</br>
In this case, the AppsFlyer SDK won't start automatically, giving the developer more freedom when to start the AppsFlyer SDK.</br>
Please note that in manual mode, the developer is required to implement the API ``[[AppsFlyerLib shared] start]`` in order to start the SDK.</br> 
You should set this mode before starting the `MobileCore` SDK and after registering the extensions in `AppDelegate`.</br>
If you are using CMP to collect consent data this feature is needed. See explanation [here](/SendConsentForDMACompliance.md).
### Example:  
```objective-c
[ACPCore configureWithAppId:@"<ADOBE_DEV_KEY>"];
[AppsFlyerAdobeExtension registerExtension];
AppsFlyerAdobeExtension.shared.manual = true;
...
[ACPCore start:^{
    [ACPCore lifecycleStart:nil];
}];
``` 
Please look at the example below to see how to start SDK once you want.</br>
Keep in mind you shouldn't put the `start()` on a lifecycle method.</br>
To start the AppsFlyer SDK, use the `start()` API, like the following :  
```objective-c
[[AppsFlyerLib shared] start];
AppsFlyerAdobeExtension.shared.manual = false;
```   
You need to end the manual mode. This will  tell the Extension to call <code>start()</code> from this point.


## <a id="guides"> 📖 Guides

- [Deep Linking](/docs/Guides.md#deeplinking)
- [In-App Events](/docs/Guides.md#events)
- [Data Elements](/docs/Guides.md#data-elements)
- [Attribution Data tracking with Adobe Analytics](/docs/Guides.md#attr-data)
- [Deeplink Data tracking with Adobe Analytics](/docs/Guides.md#deeplink-data)

## <a id="api"> 📑 API
  
See the full [API](/docs/API.md) available for this plugin.


## <a id="data-elements"> 📂 Data Elements
  
Check out the available data elements [here](/docs/DataElements.md).

## <a id="dma_support"> Send consent for DMA compliance
For a general introduction to DMA consent data, see [here](https://dev.appsflyer.com/hc/docs/send-consent-for-dma-compliance).<be>
The SDK offers two alternative methods for gathering consent data:<br>
- **Through a Consent Management Platform (CMP)**: If the app uses a CMP that complies with the [Transparency and Consent Framework (TCF) v2.2 protocol](https://iabeurope.eu/tcf-supporting-resources/), the SDK can automatically retrieve the consent details.<br>
<br>OR<br><br>
- **Through a dedicated SDK API**: Developers can pass Google's required consent data directly to the SDK using a specific API designed for this purpose.
### Use CMP to collect consent data
A CMP compatible with TCF v2.2 collects DMA consent data and stores it in <code>NSUserDefaults</code>. To enable the SDK to access this data and include it with every event, follow these steps:<br>
<ol>
  <li> Call <code>[[AppsFlyerLib shared] enableTCFDataCollection:true]</code> to instruct the SDK to collect the TCF data from the device.
  <li> Add the extension to MobileCore extensions : <code>[AppsFlyerAdobeExtension registerExtension]</code>.
  <li> Set the extension to be on manual mode to <code>true</code>- meaning the developer will have the responsability to start the SDK.</br>
       This will allow us to delay the Conversion call in order to provide the SDK with the user consent.<be>
       <code>AppsFlyerAdobeExtension.shared.manual = true</code>
  <li> Initialize <code>MobileCore</code>. 
  <li> In the <code>applicationDidBecomeActive</code> lifecycle method, use the CMP to decide if you need the consent dialog in the current session to acquire the consent data. If you need the consent dialog move to step 4; otherwise move to step 5.
  <li> Get confirmation from the CMP that the user has made their consent decision and the data is available in <code>NSUserDefaults</code>.
  <li> Call <code>start()</code> to the SDK.
  <li> Set manual mode to <code>false</code>.
</ol>


```objective-c
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
    [[AppsFlyerLib shared] enableTCFDataCollection:true];

    // Override point for customization after application launch.
    [ACPCore setLogLevel:ACPMobileLogLevelVerbose];
    [ACPCore configureWithAppId:@"DEV_KEY"];

    
    [AppsFlyerAdobeExtension registerExtension];
    [ACPAnalytics registerExtension];
    [ACPIdentity registerExtension];
    [ACPLifecycle registerExtension];
    [ACPSignal registerExtension];
    AppsFlyerAdobeExtension.shared.manual = true;
    
    [ACPCore start:^{
        [ACPCore lifecycleStart:nil];
    }];

    [AppsFlyerAdobeExtension registerCallbacks:^(NSDictionary *dictionary) {
        NSLog(@"[AppsFlyerAdobeExtension] Received callback: %@", dictionary);
    }];
    
    [AppsFlyerAdobeExtension callbacksErrorHandler:^(NSError *error) {
        NSLog(@"[AppsFlyerAdobeExtension] Error receivng callback: %@" , error);
    }];

    return YES;
}

// Deep Link reporting using Univeral Links.
 - (BOOL) application:(UIApplication *)application continueUserActivity:(NSUserActivity *)userActivity restorationHandler:(void (^)(NSArray<id<UIUserActivityRestoring>> *restorableObjects))restorationHandler {
     [AppsFlyerAdobeExtension continueUserActivity:userActivity restorationHandler:restorationHandler];
    return YES;
}

// Deep Link reporting for URL Schemes.
- (BOOL)application:(UIApplication *)application openURL:(NSURL *)url options:(NSDictionary *) options {
    [AppsFlyerAdobeExtension openURL:url options:options];
    return YES;
}
```

### Manually collect consent data
If your app does not use a CMP compatible with TCF v2.2, use the SDK API detailed below to provide the consent data directly to the SDK.
<ol>
  <li> Add the extension to MobileCore extensions : <code>[AppsFlyerAdobeExtension registerExtension]</code>.
  <li> Set the extension to be on manual mode to <code>true</code>- meaning the developer will have the responsability to start the SDK.</br>
       This will allow us to delay the Conversion call in order to provide the SDK with the user consent.<be>
       <code>AppsFlyerAdobeExtension.shared.manual = true</code>
  <li> Initialize <code>MobileCore</code>. 
  <li> In the <code>applicationDidBecomeActive</code> lifecycle method determine whether the GDPR applies or not to the user.<br>
  - If GDPR applies to the user, perform the following: 
      <ol>
        <li> Given that GDPR is applicable to the user, determine whether the consent data is already stored for this session.
            <ol>
              <li> If there is no consent data stored, show the consent dialog to capture the user consent decision.
              <li> If there is consent data stored continue to the next step.
            </ol>
        <li> To transfer the consent data to the SDK create an AppsFlyerConsent object with the following parameters:<br>
          - <code>forGDPRUserWithHasConsentForDataUsage</code>- Indicates whether the user has consented to use their data for advertising purposes.
          - <code>hasConsentForAdsPersonalization</code>- Indicates whether the user has consented to use their data for personalized advertising.
        <li> Call <code>[[AppsFlyerLib shared] setConsentData:[[AppsFlyerConsent alloc] initForGDPRUserWithHasConsentForDataUsage:BOOL hasConsentForAdsPersonalization:BOOL]]</code>. 
        <li> Call <code>start()</code> to the SDK.
        <li> Set manual mode to <code>false</code>.
      </ol><br>
    - If GDPR doesn’t apply to the user perform the following:
      <ol>
        <li> Call <code>[[AppsFlyerLib shared] setConsentData:[[AppsFlyerConsent alloc] initNonGDPRUser]]</code>.
        <li> Call <code>start()</code> to the SDK.
        <li> Set manual mode to <code>false</code>.
      </ol>
</ol>

## <a id="swift-example"> Swift Example
  
See the Swift Example [here](/docs/SwiftExample.md).

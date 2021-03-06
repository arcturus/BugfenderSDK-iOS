Bugfender SDK for iOS [![Build Status](https://travis-ci.org/bugfender/BugfenderSDK-iOS.svg)](https://travis-ci.org/bugfender/BugfenderSDK-iOS) [![Available in CocoaPods](https://img.shields.io/cocoapods/v/BugfenderSDK.svg)](https://cocoapods.org/pods/BugfenderSDK) [![CocoaDocs](https://img.shields.io/badge/docs-%E2%9C%93-blue.svg)](http://cocoadocs.org/docsets/BugfenderSDK/) 
===================

Bugfender is a cloud service to collect mobile application logs. Developers can control log sending programmatically and manually for each device. Logs are available at the [Bugfender console](https://app.bugfender.com/). You'll need an account.

BugfenderSDK works for iOS 6.0 and better.

# Getting started

## 1. Install the SDK

If using CocoaPods:

```ruby
pod 'BugfenderSDK', '~> 0.3'
```

If you prefer to install the SDK manually, go to your Project > Your Target > General > Linked Frameworks and Libraries and drag `BugfenderSDK.framework` there. Make sure you have `SystemConfiguration.framework` and `MobileCoreServices.framework` there as well. Also [add -ObjC to your linker flags](https://developer.apple.com/library/mac/qa/qa1490/_index.html).

Make Bugfender available project-wide by adding the following line to the `.pch` file:

```objective-c
#import <BugfenderSDK/BugfenderSDK.h>
```

Get an API key from the [Bugfender console](https://app.bugfender.com/). In your `AppDelegate` call [activateLogger](http://cocoadocs.org/docsets/BugfenderSDK/0.3.9/Classes/Bugfender.html#//api/name/activateLogger:) when the application starts, like this:

```objective-c
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
    ...
    // Activate the remote logger with an App Key.
    [Bugfender enableAllWithToken:@"YOUR_API_KEY"];
    ...
}
```

## 2. Send logs

That's it! You don't have to do anything. Anything you write to `NSLog` will be received by Bugfender, plus user interactions will be logged automatically.

You may also want to specify a logging level by using the following macros:

- `BFLog(...)`: Default log.
- `BFLogWarn(...)`: Warning log.
- `BFLogErr(...)`: Error log.

## 3. Send an Issue

Bugfender allows you to send issues to the server. An issue is similar to a session but they are showed in the `issues` section and you can send issues any time from the app, even if the device is not enabled in the system. Issues are useful to keep track of important errors that you can detect in your code.

For sending an issue you can use the following function:

	+(void)sendIssueWithTitle:(NSString*)title text:(NSString*)text;

*The `text` parameter has Markdown notation support on the server, so you can add some style to the text being sent.*

Here you have an example on how to send an issue using Markdown for the text:

	[Bugfender sendIssueWithTitle:@"App Error" text:@"We have found an **Error**, we need to check it"];

# Advanced features

## Having your app decide when to send logs

In some special circumstances you may want to send logs, regardless of the enabled state of the device in the Bugfender console, for example in a custom exception handler. Use `forceSendOnce` to force sending the logs once, and use `setForceEnabled:` to force it for some period of time.

## Device associated data
You can associate information to a device as if it were a dictionary:

```objective-c
[Bugfender setDeviceString:@"john.smith@example.com" forKey:@"user email"];
```

You can find more information in our blog post [Associated device information](https://bugfender.com/blog/associated-device-information/).

## Device  identifier

Bugfender automatically generates an identifier for the application install in a device. You can retrieve it to show in your UI or send to your server:

```objective-c
NSString *bugfenderDeviceIdentifier = [Bugfender deviceIdentifier];
```

To help your users find the device identifier, one easy way to do it is adding it to the app's user defaults, so it shows up in the app's section inside the device Settings.

```objective-c
// Displaying the device identifier in the app's settings.
[[NSUserDefaults standardUserDefaults] setObject:[Bugfender deviceIdentifier] forKey:@"bugfenderDeviceIDKey"];
[[NSUserDefaults standardUserDefaults] synchronize];
```

## Log buffer size

Bugfender keeps up to 5 MB worth of log data in the device. This way Bugfender can work offline, and you can get some log data from the past when enabling a device. You can change that limit with `setMaximumLocalStorageSize`.

```objective-c
// Setting maximum cache size to 1 Mb
[Bugfender setMaximumLocalStorageSize:1024*1024];
```

## Advanced logging

When compiling in DEBUG, Bugfender will also print messages to the console. Bugfender won't output anything on the console in release mode.

Besides the logging level you can manually specify a tag o set of tags (string separated by comas) and a log level by using the following method:

- `BFLog2(level, tag, ...)`: Where log level is one of the enums shown above, tag is an string containing tags separated by coma, and then the log itself.

Use like this:

```objective-c
- (void)foo
{
    // Default log
    BFLog(@"Foo method started at time: %@", [[NSDate date] description]);
    
    // Warning log
    BFLogWarn(@"This is a warning with error code: %ld", 23);
    
    // Error log
    BFLogErr(@"This is an error with error code: %ld", 42);
    
    // Custom log, specifiying level, tags, and the text
    BFLog2(BFLogLevelWarning, @"networking, error", @"This is a warning with some tags. Error code: %ld", (long)23);
}
```

# Swift

Bugfender works well with Swift. Just follow the previous instructions, plus add the following line to your Bridging Header:

```objective-c
#import <BugfenderSDK/BugfenderSDK.h>
```

Use `Bugfender.LogLineNumber` method to log, or write a helper function similar to `println`:

```swift
func BFLog(message: String, filename: String = __FILE__, line: Int = __LINE__, funcname: String = __FUNCTION__) {
    Bugfender.logLineNumber(line, method: funcname, file: filename.lastPathComponent, level: BFLogLevel.Default, tag: nil, message: message)
    #if DEBUG
        NSLog("[\(filename.lastPathComponent):\(line)] \(funcname) - %@", message)
    #endif
}
```

Use like this:
```swift
func sliderChanged(slider: UISlider) {
    BFLog("Slider Value: \(slider.value)");
}
```

Check out the full documentation at [CocoaDocs](http://cocoadocs.org/docsets/BugfenderSDK/).

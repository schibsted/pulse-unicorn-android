# Pulse Unicorn Android


[![Build Status](https://travis.schibsted.io/dtpt-datafoundations/pulse-unicorn-android.svg?token=zzxUpTfpymUwiUGpwDgr&branch=master)](https://travis.schibsted.io/github/dtpt-datafoundations/pulse-unicorn-android)

Pulse Unicorn is an extension
over [Android event SDK]([https://github.schibsted.io/spt-dataanalytics/event-sdk-android](https://github.com/schibsted/pulse-sdk-android))
It makes your life easier!

- [Pulse Unicorn Android](#pulse-unicorn-android)
  * [Current version](#current-version)
  * [What is the purpose?](#what-is-the-purpose-)
  * [Current Features](#current-features)
- [Pulse Unicorn Integration](#pulse-unicorn-integration)
  * [Easy mode](#easy-mode)
  * [Advanced mode](#advanced-mode)
  * [Starting Pulse Unicorn Activity](#starting-pulse-unicorn-activity)
  * [Testing the validity of events](#testing-the-validity-of-events)
    + [Validator response](#validator-response)
    + [Example](#example)
- [Contact](#contact)

## Current version
```
1.1.3
```

## What is the purpose?

The purpose of this project is to help with Pulse Tracking development and maintenance using
[Android event SDK](https://github.schibsted.io/spt-dataanalytics/event-sdk-android) library.<br />
This extension allows quick examination of traffic that is sent by `event-sdk` library.<br />
Pulse Unicorn provides basic UI that displays events in list, so that you can verify, if events send
by your app are implemented and sent exactly as you wanted them to be implemented.


## Current Features

- Watch for ongoing events in the real time in a easy to read format
- Easy opening by tapping the notification
- Quick overview for number of collected events via notification information
- Receive notification about invalid / failed events
- Search across events by any matching phrase
- Search for any value in event Json body in event details view
- Add any custom Json property display in event cell for easy quick value debugging
- Filter events by default pulse `EVENT_TYPE`, `OBJECT_TYPE`, `TRACKER_TYPE` and `STATUS`
- Sharing Json of event


# Pulse Unicorn Integration

1. Be sure that you have access to Schibsted Artifactory configured in your project or `mavenCentral`
2. Add `Pulse Unicorn` maven to `gradle.build` file.

```kotlin
implementation("com.schibsted.pulse:unicorn-android:{$current-version}")
```

3. Initialize Pulse Unicorn in declared `Application` class in your application.
## Easy mode
```kotlin
PulseUnicorn.init(this)
# Place initialization at the top of your Application class, before any DI and other parts of code.
```

## Advanced mode
To manipulate Pulse Unicorn notification settings from code you can pass `PulseUnicornConfig` class to do so.
It will set up your desired settings on each Pulse Unicorn run.
Any settings change made through UI will persistent during PulseUnicorn lifecycle. They will be lost after relaunch / reinstall.

```kotlin
PulseUnicorn.init(
            this, PulseUnicornConfig(
                enableStickyNotification = true,
                stickyNotificationType = StickyNotificationEnum.SIMPLE_NOTIFICATION,
                enablePopupNotification = true,
                enableInvalidEventNotification = true,
                enableFailedEventNotification = false,
                notificationChannelIdConfig = PulseUnicornNotificationChannelConfig(
                    stickyNotificationChannelId = "123123123",
                    failedPopUpNotificationChannelId = "234234234",
                    invalidPopUpNotificationChannelId = "345345345"
                )
            )
        )
```

Default settings for Notification settings
```kotlin
data class PulseUnicornConfig(
    var enableStickyNotification: Boolean = true,
    var stickyNotificationType: StickyNotificationEnum = StickyNotificationEnum.DETAILED_NOTIFICATION,
    var enablePopupNotification: Boolean = true,
    var enableInvalidEventNotification: Boolean = true,
    var enableFailedEventNotification: Boolean = true,
    var notificationChannelIdConfig: PulseUnicornNotificationChannelConfig?
)
```

4. Add `PulseUnicorn.getUnicornInterceptor()` in `PulseEnvironment.Builder` via `addInterceptor` method.

```kotlin
PulseEnvironment.Builder(context, clientId).apply {
    addInterceptor(PulseUnicorn.getUnicornInterceptor())
}.build()
```

## Starting Pulse Unicorn Activity
You can open Pulse Unicorn from notification card or use dedicated method
```kotlin
PulseUnicorn.openPulseUnicorn(context = this)
```

## Testing the validity of events

To simply check the validation of event Json before sending or during your tests you can
use `PulseUnicorn().getUnicornValidator()`. It creates a quick HTTP request to Pulse Console and
returns `PulseValidatorResponse` class which holds information about validation results.

### Validator response

`success` - bool information about validation
<br />
`errorsJsonString` - Json String with errors

### Example

```kotlin
lifecycleScope.launch(Dispatchers.IO) {
  // async
  PulseUnicorn.getUnicornValidator().validate(eventJsonString)
    ?.let { pulseValidator ->
      if (pulseValidator.success) {
        // event is valid -> send
        PulseTrackerProvider().get().track(event)
      } else {
        // event is not valid -> check errors
        Log.d(
          "PulseValidator",
          pulseValidator.errorsJsonString
            ?: "Received empty error msg from validator"
        )
      }
    }
}
```
## Contact
When, for technical reasons Slack cannot work for you, please make sure to use [GitHub issues link](https://github.com/schibsted/pulse-unicorn-android/issues) instead.
We will provide all the necessary answers within the relevant Slack threads (or in Github issue when your case is inserted there).
Thank you for your cooperation!

---
title: "Push notifications Voice API Android"
excerpt: ""
---
When an application is not running, or the `Active Connection` feature is not enabled, the user must be notified of an incoming call or instant message by a push notification.

By invoking `setSupportManagedPush(true)` the Sinch SDK will automatically register to *Firebase Cloud Messaging* and the Sinch backend will initiate push messages to your application when needed. This feature requires Google Play Services on the device. If you distribute your application through other channels than Google Play, push notifications will not be available on devices that do not have Google Play Services.

If using the Sinch backend and Google Cloud Messaging is not viable in the application, please see \[Push Notifications sent via your application server\]\[\] and \[Active connection\]\[\].

As a developer, you will be responsible for implementing the code that receives the FCM push message. For an example implementation, please see the sample app “Sinch Push” which is bundled with the SDK.

Sinch SDK moved from deprecated *Google Cloud Messaging* (GCM) to it’s most up-to-date and Google-recommended version *Firebase Cloud Messaging* (FCM), which requires client app to be modified in accordance with the Google’s official [GCM to FCM migration guide](https://developers.google.com/cloud-messaging/android/android-migrate-fcm)

The following sections cover how to support receiving calls and messages via push notifications.

## FCM configuration file required (google-services.json)

You can add Firebase to your app either semi-automatically using Android Studio, or manually [following this step-by-step official guide](https://firebase.google.com/docs/android/setup). In brief, to perform manual setup you first need to register your application in [firebase console](https://console.firebase.google.com/). If you already have GCM project, the console will prompt you to import it as new Firebase Cloud Messaging project. Register your application using the console, and download relevant google-services.json into your project’s main folder. More information about adding Firebase to your Android app can be found [here](https://firebase.google.com/docs/android/setup)

Sample SDK projects *sinch-rtc-sample-push* and *sinch-rtc-sample-video-push* will require you to supply your own *google-services.json* in order to be built. In the absence of this file gradle will show relevant error with explanation and relevant links and stop the build. That *google-services.json* file is the main mean of automatization of support of Firebase services to your app. Android Studio’s *‘com.google.gms.google-services’* plugin parses and adds relevant resources and permissions to your applications manifest automatically.

## Permissions required

Unlike GCM setup, FCM application developer does not need to manually add any permission to application manifest. For relevant changes in you application’s manifest when migrating from GCM to FCM please consult official [GCM to FCM migration guide](https://developers.google.com/cloud-messaging/android/android-migrate-fcm)

## Enable push notifications

To enable push notifications, set the following capability before starting the Sinch client:
```java
sinchClient.setSupportManagedPush(true);
sinchClient.start();
```




> **Note**    
>
> - You must catch the `MissingGCMException` if you distribute your app to devices without *Google Play Services*.
> - Using `setSupportManagedPush(true)` will register a token with Firebase Cloud Messaging using a Sender ID connected to Sinch, which will *NOT* unregister your own token, so you *CAN* use Firebase Cloud Messages for your own purpose filtering them in *onMessageReceived(RemoteMessage remoteMessage)* method of your FCM Listening Service using Sinch helper API *SinchHelpers.isSinchPushPayload*.


```java
public class FcmListenerService extends FirebaseMessagingService {

@Override
public void onMessageReceived(RemoteMessage remoteMessage){
    if (SinchHelpers.isSinchPushPayload(remoteMessage.getData())) {
        // it's Sinch message - relay it to SinchClient
    } else {
        // it's NOT Sinch message - process yourself
    }
  }}
```

### Explicit push token registration

There are certain situations where it is either desirable to explicitly register push token and/or get assurance that the push token is indeed registered, e.g.:

- The application is designed to receive calls only, and thus must register push token with the Sinch backend on the very first start, while it's desireable to terminate SinchClient as soon as the registration concludes (e.g. to free resources). In this situation, the application should be notified by a specific callback on the registration result.
- The application detects that FCM push token is invalidated abd should be refreshed and re-registered with Sinch backend. Here, if SinchClient is in the running state, it would take care of re-registering of the push token iteself, otherwise, the application is responsible for re-registering.

Both situation should be handled with using new **ManagedPush API** available via *Beta* interface:

```java
public ManagedPush getManagedPush(String username) {
     // create client, but you don't need to start it
     initClient(username);
     // retrieve ManagedPush
     return Beta.createManagedPush(mSinchClient);
}
```

The former situation is showcased in *LoginActivity.java* in *sinch-rtc-sample-push* and *sinch-rtc-sample-video-push* sample applications. The activity implements *PushTokenRegistrationCallback* interface,

```java
public class LoginActivity extends BaseActivity implements SinchService.StartFailedListener, PushTokenRegistrationCallback {

private void loginClicked() {
       ...
       if (!mPushTokenIsRegistered) {
              getSinchServiceInterface().getManagedPush(userName).registerPushToken(this);
       }
       ...
}

@Override
public void tokenRegistered() {
       mPushTokenIsRegistered = true;
       nextActivityIfReady();
}

@Override
public void tokenRegistrationFailed(SinchError sinchError) {
       mPushTokenIsRegistered = false;
       Toast.makeText(this, "Push token registration failed - incoming calls can't be received!", Toast.LENGTH_LONG).show();
}
}
```

And the UI andvances to the next activity only when both conditions are met:
- the SinchClient is started;
- the push token is registered

```java
public class FcmListenerService extends FirebaseMessagingService {

@Override
public void onNewToken(String newToken) {
// newToken supplied here is the token for `default` FCM project.
// but the mere fact of receiving this callback informs the application
// that ALL tokens should be re-acquired
       instanceOfMyPushTokenRegistrationClass.registerPushToken();
}
}
```

Where `instanceOfMyPushTokenRegistrationClass.registerPushToken()` behavior is defined by the same pattern as in the previous situation - call the `ManagedPush.registerPushToken(final PushTokenRegistrationCallback callback)` and wait for callback to decide how to proceed depending on result.


## Receive and forward push notifications to a Sinch client

For more details regarding how to implement receiving a FCM downstream message, please see the [Android developer site for FCM](https://firebase.google.com/docs/cloud-messaging/android/receive).

Once you have received the `RemoteMessage` in your `FirebaseMessagingService`, forward it to the Sinch client using the method `relayRemotePushNotificationPayload`.
```java
// make sure you have created a SinchClient
if (SinchHelpers.isSinchPushPayload(remoteMessage.getData())) {
    NotificationResult result = sinchClient.relayRemotePushNotificationPayload(remoteMessage.getData());
}
```


The returned `result` can be inspected to see whether the push was for an IM or a call using `result.isMessage()` and `result.isCall()`.

### Incoming message

If the payload that was forwarded to the Sinch client was for an instant message, call `SinchClient.startListeningOnActiveConnection()` which will make sure the `onIncomingMessage` callback is called. See \[Active Connection\]\[\] for more information.

### Incoming call

If the payload that was forwarded to the Sinch client was for a call, the `onIncomingCall` callback will automatically be triggered as for any other call. The `CallNotificationResult` object provides details about participants, whether the call timed out and whether the call offers video.

### Send and receive custom headers via Sinch managed push

The Sinch SDK supports adding custom headers in push notification messages when initiating a call, so developers do not need to implement their own push mechanism if they only need to deliver small pieces of information along the Sinch managed push between their app instances. The Sinch SDK allows up to *1024* bytes of custom headers.

Setting custom headers on the sender side when initiating a call:
```java
// setting up custom headers
Map<String,String> headers = new HashMap<>();
headers.put("The first value is ", "@123");
headers.put("Custom value ", "two!");
Call call = callClient.callUser(userId, headers);   
```


If custom headers were supplied by call initiator, they can be retrieved from notification result using `getHeaders()` API:
```java
// make sure you have created a SinchClient
if (SinchHelpers.isSinchPushPayload(remoteMessage.getData())) {
NotificationResult result = sinchClient.relayRemotePushNotificationPayload(remoteMessage.getData());
  if (result.isCall()) {
    CallNotificationResult callResult = result.getCallResult();
    Map<String, String> customHeaders = callResult.getHeaders());
  }
}
```




> **Note**    
>
> It is possible to retrieve custom headers from the push message using *SinchHelpers.queryPushNotificationPayload* without starting the client.


```java
// SinchClient is not needed to be created at all!
if (SinchHelpers.isSinchPushPayload(remoteMessage.getData())) {
  NotificationResult result = SinchHelpers.queryPushNotificationPayload(applicationContext, remoteMessage.getData());
  if (result.isCall()) {
    CallNotificationResult callResult = result.getCallResult();
    Map<String, String> customHeaders = callResult.getHeaders());
    // analyse headers, decide whether to process message/call and start Sinch client or ignore
    ...
  }
}
```


### Show local notifications for missed calls

When the push notification is enabled on a Sinch client, besides the incoming call notification, the Sinch SDK will also send a push notification for a canceled call when the caller cancels the call before it is answered. This gives developers a good opportunity to present local notifications for missed calls in their apps:
```java
NotificationResult result = sinchService.relayRemotePushNotificationPayload(payload);
// handle result, e.g. show a notification for a missed call:    
if (result.isValid() && result.isCall()) {
    CallNotificationResult callResult = result.getCallResult();
    if (callResult.isCallCanceled()) {
        // user-defined method to show notification
        createNotification(callResult.getRemoteUserId());
    }
}
```




> **Note**    
>
> If the message forwarded to the Sinch client happened to be *call cancel* message, the client arranges the termination of the call automatically resulting in `CallListener.onCallEnded()` event being triggered, allowing UI to handle canceling the call.

## Unregister a device

If the user of the application logs out or performs a similar action, the push notification device token can be unregistered via `SinchClient.unregisterManagedPush()` to prevent further notifications to be sent to the device. Starting a client with `setSupportManagedPush(true)` will register the device again.


<a class="gitbutton pill" target="_blank" href="https://github.com/sinch/docs/blob/master/docs/voice/voice-for-android/voice-android-push-notifications.md"><span class="fab fa-github"></span>Edit on GitHub</a>
---
title: "Sinch Client Voice API iOS"
excerpt: ""
---
The *SINClient* is the Sinch SDK entry point. It is used to configure the user’s and device’s capabilities, as well as providing access to feature classes such as the *SINCallClient*, *SINMessageClient* and *SINAudioController*.

## Creating the *SINClient*

Set up the client and its delegate (*SINClientDelegate*, see [Reference](reference/html/Protocols/SINClientDelegate.html) documentation).
```objectivec
#import <Sinch/Sinch.h>

// Instantiate a Sinch client object
id<SINClient> sinchClient = [Sinch clientWithApplicationKey:@"<application key>"
                                          applicationSecret:@"<application secret>"
                                            environmentHost:@"sandbox.sinch.com"
                                                     userId:@"<user id>"];
```


The *Application Key* and *Application Secret* are obtained from the Sinch Developer Dashboard. See \[Production and Sandbox Environments\]\[\] for valid values for *environmentHost*. The User ID should uniquely identify the user on the particular device.

## Specifying capabilities

The SINClient can be configured to enable / disable certain functionality. Please see the [Reference](reference/html/Protocols/SINClient.html) for details.
The following example shows how to setup the client with both voice calling and instant messaging enabled, and using [push notifications](doc:voice-ios-local-and-remote-push-notifications).

```objectivec
// Specify the client capabilities.
// (At least one of the messaging or calling capabilities should be enabled.)
[sinchClient setSupportCalling:YES];
[sinchClient setSupportMessaging:YES];

[sinchClient enableManagedPushNotifications];
```


## Starting the Sinch client
Before starting the client, make sure you assign a *SINClientDelegate*.
```objectivec
// Assign as SINClientDelegate                             
sinchClient.delegate = ... ;

// Start the Sinch Client
[sinchClient start];

// Start listening for incoming calls and messages
[sinchClient startListeningOnActiveConnection];
```




> **Note**    
>
> If the application is meant to only make outgoing calls but not receive incoming calls, don’t call the `startListeningOnActiveConnection`. Outgoing calls can be made after calling the start method, and after the delegate has received the callback `clientDidStart:`.

For applications that want to receive incoming calls while not running in the foreground, [push notifications](doc:voice-ios-local-and-remote-push-notifications) are required.

### Life cycle management of a *SINClient*-instance

We recommend that you initiate the Sinch client, start it, but not terminate it, during the lifetime of the running application. That also implies that the *SINClient*-instance should be *retained* by the application code.
 If incoming events are not needed, stop listening for incoming events by invoking `-[SINClient stopListeningOnActiveConnection]`), but **do not** invoke `-[SINClient terminateGracefully]` or `-[SINClient terminate]`. The reason is initializing and *starting* the client is relatively resource-intensive in terms of CPU.

It is best to keep the client instance alive and started unless there are reasons specific to your application. It should *not* be necessary to dispose of the client instance if memory warnings are received from iOS, because once the client is started it does not use much memory in comparison to view layers, view controllers etc. For the same reasons, if support for push notifications is enabled, the preferred method of temporarily stopping incoming events is to \[Unregister a push device token\]\[\].

The Sinch client can of course be completely stopped and also disposed. To do so, call one of the terminate methods on the client before the application code releases its last reference to the client object.

The following example shows how to dispose the Sinch client:
```objectivec
[sinchClient stopListeningOnActiveConnection];
[sinchClient terminateGracefully]; // or invoke -[SINClient terminate]
sinchClient = nil;
```


<a class="gitbutton pill" target="_blank" href="https://github.com/sinch/docs/blob/master/docs/voice/voice-for-ios/voice-ios-sinch-client.md"><span class="fab fa-github"></span>Edit on GitHub</a>
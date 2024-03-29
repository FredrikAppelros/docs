---
title: "Release Notes for the Sinch JavaScript SDK"
excerpt: "See how the Sinch JavaScript SDK is evolving and find out about new features and bug fixes."
---

See how our platform is evolving. Keep track of new features, API versions and bug fixes.

<h3>2017-10-06 | Sinch SDK JavaScript 1.4.5 </h3>
-   Updated dependencies
-   Fix for App to Web compatibility
-   Bug fixes for recent Chrome and Firefox versions

<h3>2016-08-26 | Sinch SDK JavaScript 1.4.4 </h3>
- Bug fix: Resolved issue with newer browsers using ECDSA for key generation

<h3>2016-01-27 | Sinch SDK JavaScript 1.4.3</h3>
-   Added support for SIP Calling with Error header support

<h3>2015-12-17 | Sinch SDK JavaScript 1.4.2</h3>
-   Added support for verification using Callout
-   Bug fix: Custom header support for native client compatibility

<h3>2015-10-05 | Sinch SDK JavaScript 1.4.1</h3>
-   Bug fix: userAgent parsing when Safari running embedded in apps, solving “50 character” limit bug
-   Bug fix: verification of numbers containing spaces would fail
-   Bug fix: prevent sending Javascript objects in IM’s
-   Bug fix: Firefox from making voice over data calls to recent Android SDK
-   Bug fix: Firefox video calls to work with Chrome/Android
-   Bug fix: Early media 20s call issue resolved
-   Stability: Robust proxy fallback, using relay without srflx candidate

<h3>2015-09-25 | Sinch SDK JavaScript 1.4.0</h3>
-   Introduced Sinch conference calling.

<h3>2015-06-24 | Sinch SDK JavaScript 1.3.1</h3>
-   SMS Verification GA
-   Calling verify method on a previously verified verification object will return that it is a verified number.

<h3>2015-05-21 | Sinch SDK JavaScript 1.3.0</h3>
-   Introducing SMS Verification Beta
    -   New method in SinchClient, “createSmsVerification()”
    -   New verification object (retrieved by createSmsVerification), with support for sending and resending a verification SMS as well as methods to validate a verification code
    -   Sample application for SMS Verification
-   Increased call-setup timeout to 10s, in line with iOS & Android SDK
-   Enabled SinchClient to start without any capabilities
-   Bug fix: audio group calling (note: still beta feature)

<h3>2015-02-18 | Sinch SDK JavaScript 1.2.0</h3>
-   New capability, “video” for video calling (beta)
-   New capability, “multiCall”, make sinchClient capable of handling multiple simultaneous calls (beta)
-   New method, callClient.callGroup(‘groupName’), for making video group calls (beta)
-   Support for calling native clients (compatible with Sinch iOS SDK v3.4.0 and Android v3.5.0 and above)
-   Support for receiving calls from native clients (compatible with Sinch iOS SDK v3.4.0 and Android SDK v3.5.0 and above)

<h3>2015-02-05 | Sinch SDK JavaScript 1.1.0</h3>
-   Support for calling and receiving native clients (compatible with Sinch iOS SDK v3.4.0 and above)
-   Mute and unmute mic on a call using call.mute() and call.unmunte()
-   API Change: “startActiveConnection” configuration option now enables online capability and starts the active connection.
-   Automatic mute during call setup for privacy
-   Support for self signed requests for authentication (backend in Node)
-   Improved robustness on signalling issues and Firefox compatibility

<h3>2014-12-18 | Sinch SDK JavaScript 1.0</h3>
-   API Change: Call timestamps are now javascript Date-objects
-   API Change: “supportActiveConnection” configuration option now only enable online capability, remember to manage your active connection using startActiveConnection() and stopActiveConnection()
- API Change: “startActiveConnection” configuration option now enables online capability and starts the active connection.
- Deprecated: SinchClient.stop(), replaced with terminate()
- Performance: reduced instance count by reuse of instance across sessions
- IM statistics are gathered and available in partner dashboard
- Instant messaging capability, enabling developers to specifically allow or disallow certain sessions from receiving instance messages.
- Bug fix: Call headers now available in onIncomingCall()
- Bug fix: Early media will now playback properly if PSTN call fails.
- Sinch added to npm repository for Node JS applications
- Sinch added to Bower repository

<h3>2014-11-05 | Sinch SDK JavaScript 0.9.9 Beta</h3>
-   Sample web application and backend demonstrating how to integrate authentication with your backend
-   Manage active connections with sinchClient.startActiveConnection() and sinchClient.stopActiveConnection(). **Important: Active connections MUST now be either started manually, or with option “supportActiveConnection: true” on SinchClient instantiation, if application will receive calls / IM.**
-   Send & Receive custom headers on call, new method: call.getHeaders()
-   Consolidate customer headers in Call Detail Reports (CDR)
-   Bug fixes & development convenience improvements
    -   On demand signal connection; on incoming call or when making call, reduce open connection count
    -   Handle quick/rapid hang-ups for web-to-web and web-to-phone calls
    -   Timeouts on calling when recipient is not online
    -   Proxy fallback on data calling

<h3>2014-10-17 | Sinch SDK JavaScript 0.9.8 Beta</h3>
-   Application to application calling (web-to-web only)
-   Updated sample apps, web-to-web example added
-   Several bugfixes in Sinch SDK
-   Call reporting for statistics, PSTN & Data
-   Refactor build infrastructure for Node JS loader compatibility
-   Changed call.callEndCause to integers. For human readable end-cause use call.getEndCause()
-   Fixed bug: Client timestamps synchronized with backend

<h3>2014-09-04 | Sinch SDK JavaScript 0.9.7 Beta</h3>
This is the first public beta version, which includes:

-   Instant messaging
-   App to Phone Calling
-   Partner user management
-   Sinch user management



<a class="gitbutton pill" target="_blank" href="https://github.com/sinch/docs/blob/master/docs/release-notes/release-notes-javascript-sdk.md"><span class="fab fa-github"></span>Edit on GitHub</a>
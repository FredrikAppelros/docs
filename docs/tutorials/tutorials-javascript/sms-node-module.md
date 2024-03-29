---
title: "SMS Node Module"
excerpt: "This is an open source Node module to send SMS with the Sinch SMS API."
---
One of the teams at TheNextWeb hackathon this past weekend was kind enough to open source a node module to send SMS with the [Sinch SMS API](https://www.sinch.com/sms-api/). (Shoutout to Mihir and Mikhail\!) Here’s an example of how to use it:

```javascript
var sinchAuth = require('sinch-auth');

var sinchSms = require('sinch-messaging');

var auth = sinchAuth("your-app-key", "your-app-secret");

sinchSms.sendMessage("+16507141052", "Hello world!");
```

The above script will print an ID for the SMS. It will look like this:

```json
{"MessageId":123456789}
```

You can use this ID to check the status of the SMS at any point like so…

```javascript
console.log(sinchSms.getStatus("123456789");
```

…which will give you a response like so:

```json
{"Status":"Successful"}
```

If you’re interested in digging into the node modules for authentication and sending SMS, you can find the source code on GitHub:

 1.  [sinch-auth](https://github.com/ChewTeaYeah/sinch-auth)
 1.  [sinch-messaging](https://github.com/ChewTeaYeah/sinch-messaging)

To get your own app key and secret, [sign up](https://portal.sinch.com/#/signup) for a Sinch account and create a new app. To start, you will have $2 on your account to send some test messages. See [SMS pricing by country here](https://www.sinch.com/products/messaging/sms/).

<a class="gitbutton pill" target="_blank" href="https://github.com/sinch/docs/blob/master/docs/tutorials/javascript/sms-node-module.md"><span class="fab fa-github"></span>Edit on GitHub</a>
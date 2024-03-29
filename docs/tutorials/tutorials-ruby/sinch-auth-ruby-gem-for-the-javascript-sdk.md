---
title: "Sinch Auth Ruby Gem for the JavaScript SDK"
excerpt: "This tutorial shows how you can use the gem sinch_auth to generate a user ticket that authenticates users through the Sinch JavaScript SDK."
---
You can use the gem `sinch_auth` to generate a user ticket that authenticates users through the Sinch JavaScript SDK. First, install the gem:

```ruby
$ gem install sinch_auth
```

To use:

```ruby
sinchAuth = SinchAuth.new
ticket = sinchAuth.get_auth_ticket(username, expires_in, key, secret)
```

Where username is a string that uniquely identifies the current user, expires\_in is the number of seconds the ticket expires in, and key and secret are your app key and secret from the Sinch dashboard.

If you do not yet have a key and secret, sign up for a [free Sinch developer account](https://portal.sinch.com/#/signup). Once logged in, you will see a button to create a new app. This generates a key and secret for you.

## Use the ticket

In the view where you want to use the Sinch client, you will create a new Sinch client and start it with the ticket you just generated. Make sure to use the same app key that you used to generate the ticket.

```ruby
<script>
  sinchClient = new SinchClient({
    applicationKey: "your_app_key",
    capabilities: {messaging: true, calling: true},
    startActiveConnection: true,
    onLogMessage: function(message) {
      console.log(message.message);
    },
  });

  sinchClient.start({"userTicket":ticket});
</script>
```

Once the client is started, you can use it to make browser-to-browser phone calls, browser-to-phone calls, and send web-to-web instant messages. Follow the tutorials below to build these features:

>   - [Browser-to-Browser Calling](doc:using-sinch-js-sdk-to-call-a-phone-number)
>   - [Browser-to-Phone Calling](doc:turn-your-browser-into-a-phone-with-the-sinch-js-sdk)

<a class="gitbutton pill" target="_blank" href="https://github.com/sinch/docs/blob/master/docs/tutorials/ruby/sinch-auth-ruby-gem-for-the-javascript-sdk.md"><span class="fab fa-github"></span>Edit on GitHub</a>
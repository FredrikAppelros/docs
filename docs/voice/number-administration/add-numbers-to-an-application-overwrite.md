---
title: "Add numbers to an application (Overwrite)"
excerpt: ""
---
This API overwrites all numbers assigned to your app with the new set of numbers that you specify.

> **Attention**
> 
> When using this API method, any numbers that may already be configured for the particular application will be overwritten by the new list of numbers that you specify in this API call.

A new endpoint to only add a specific number to an application without overwriting the existing numbers will be available very soon.

## Headers

This is a protected resource and requires an [instance signed request](doc:using-rest#section-instance-signed-request).

    Authorization: Instance {instance ID}:{instance signature}
    X-Timestamp: {now}
    
    eg:
    Authorization: Instance 00a3ffb1-0808-4dd4-9c7d-e4383d82e445:6mLYthlkatoYKkq15oI/RuwtC8sIwfJsPrSHkOLDmUM=
    X-Timestamp: 2015-06-20T11:43:10.944Z

## Request

    [PUT]
    
    URL:
    https://portalapi.sinch.com/v1/applications/key/{applicationKey}/numbers/{numberCapability}
    
    eg:
    https://portalapi.sinch.com/v1/applications/key/bb7b4e39-4227-4913-8c81-2db4abb54fb3/numbers/voice

    [body]
        long[] - numbers
    
    eg:
    {
        "numbers":[14150005550]
    }

> **Note**
> 
> Accepted values for ‘numberCapability’ are **sms** or **voice**.

## Response

    Number[]
    
    [Number]
        long - number
        integer - groupId
        string[] - capabilities
        string? - applicationKey
        string - countryId
        string - city
        string - country
        string - areaCode
        string? - expires
        Money    - cost
    
    eg:
```json
[
    {
        "number" : 14150005550,
        "groupId" : 13,
        "capabilities" : ["voice"],
        "applicationKey" : "bb7b4e39-4227-4913-8c81-2db4abb54fb3",
        "countryId" : "US",
        "city" : "SAN FRANCISCO",
        "country" : "US",
        "areaCode" : "415",
        "expires" : "2015-12-31T23:59:59+00:00",
        "cost":{
            "amount":0.00,
            "currencyId":"USD"
        }
    }, {
        "number" : 14150005551,
        "groupId" : 13,
        "capabilities" : ["voice"],
        "applicationKey" : "bb7b4e39-4227-4913-8c81-2db4abb54fb3",
        "countryId" : "US",
        "city" : "SAN FRANCISCO",
        "country" : "US",
        "areaCode" : "415",
        "expires" : "2015-12-31T23:59:59+00:00",
        "cost":{
            "amount":0.00,
            "currencyId":"USD"
        }
    }
]
```
## Error Codes

    [Error Codes]
        ParameterValidation (40001) - Invalid number type: {NUMBER_TYPE}.  Type must be one of: sms, voice.
        InvalidScheme (40301)       - The authorization scheme 'instance' is required.
        ResourceNotFound (40400)    - Application with the key {KEY_HERE} was not found.
        TemporaryDown (50300)       - Service is temporarily unavailable, please try again later.

<div id="removeanumbersfromanapplication">

Remove a number from an application

</div>

With this API you can remove (unassign) a number from your application. They will still be assigned to your account.

## Headers

This is a protected resource and requires an [instance signed request](doc:using-rest#section-instance-signed-request).

    Authorization: Instance {instance ID}:{instance signature}
    X-Timestamp: {now}
    
    eg:
    Authorization: Instance 00a3ffb1-0808-4dd4-9c7d-e4383d82e445:EA85F4qGFB0+tYZUh68g22R8DgtzxpsCXttLq1NKPgM=
    X-Timestamp: 2015-06-20T11:43:10.944Z

## Request

    [DELETE]
    
    URL:
    https://portalapi.sinch.com/v1/applications/key/{applicationKey}/numbers/{number}
    
    eg:
    https://portalapi.sinch.com/v1/applications/key/bb7b4e39-4227-4913-8c81-2db4abb54fb3/numbers/14150005551

    [body]
        empty
    
    eg:
    {}

## Response

    204 - No Content

## Error Codes

    [Error Codes]
        InvalidRequest (40003)      - The 'number' specified is not a valid number.
        InvalidScheme (40301)       - The authorization scheme 'instance' is required.
        ResourceNotFound (40400)    - Application with the key {KEY_HERE} was not found.
        TemporaryDown (50300)       - Service is temporarily unavailable, please try again later.



<a class="gitbutton pill" target="_blank" href="https://github.com/sinch/docs/blob/master/docs/voice/number-administration/add-numbers-to-an-application-overwrite.md"><span class="fab fa-github"></span>Edit on GitHub</a>
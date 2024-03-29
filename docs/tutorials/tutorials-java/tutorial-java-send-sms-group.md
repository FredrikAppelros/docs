---
title: "Send an SMS to a pre-defined group of recipients"
excerpt: "This tutorial will show you how to use Groups, a more advanced feature of the Sinch REST API, to send a single message to a pre-defined list of recipients."
---
Now that you've successfully [sent a message to ad-hoc recipients using the Sinch REST API](doc:tutorial-java-send-sms), you're ready to take on the next step: creating a group of recipients that will be stored on Sinch servers, and sending a message to this group.

## Prerequisites

Before starting, please make sure that:

 - You have created your Sinch account
 - The right version of the JDK is installed on your computer
 - You have added the Sinch Java SDK JAR to your project

All details of those steps are available on the [Java Getting Started page](doc:tutorials-java).

## What Are Groups?

The Sinch REST API lets you manage groups of contacts that you can use as recipients for your outbound messages. This makes it easy to maintain distribution lists without the need to store them in your own system. Sending to large groups is also faster since there is no need to transmit the entire list of recipients for each message you send, you only specify the group ID as the recipient.

> **Limitations**
>
> Groups may contain a maximum of 10,000 members.

Although we won't be covering them, Groups also have some more advanced features:

 - Groups can be nested; a group can contain members of up to 10 other groups.
 - Groups can be automatically updated upon certain user-initiated interactions (ex: sending JOIN or STOP to your dedicated phone number or short code).
 
Check out the [API reference documentation on Groups](doc:sms-rest-groups-endpoint) for more details. 

## Let's Get Started

In this tutorial, we'll first make sure a group named `subscribers` exists on our account, then we'll add a pre-determined list of members to this group. Once our group contains our list of subscribers, we'll send them a message.

Since we already covered how to start and close a connection to the REST API in the [first tutorial](doc:tutorial-java-send-sms), we won't be covering that part again here. 

### Ensure Your Group Exists

To make sure our group exists, we'll need to check if a group with our name has previously been created. 

To do this, we'll first need to **list all groups that exist** on our Sinch account:

```java
PagedFetcher<GroupResult> groups = connection.fetchGroups(GroupFilter.builder().build());
```

The Groups endpoint being paginated, we must **iterate through groups page by page**:

```java
GroupResult group;
for (Page<GroupResult> page : groups.pages()) {
    for (GroupResult currentGroup : page) {
        if ("subscribers".equals(currentGroup.name())) {
            group = currentGroup;
            break;
        }
    }
}
```

You can also accomplish the same things as the above in a single statement using the Streams API if you feel like it:

```java
StreamSupport.stream(groups.pages().spliterator(), false)
        .flatMap(page -> page.content().stream())
        .filter(group -> "subscribers".equals(group.name()))
        .findFirst()
```

If no group was found with our name, we'll need to **create a group**:

```java
if (group == null) {
    group = connection.createGroup(ClxApi.groupCreate().name("subscribers").build());
}
```

### Update the Group With Our List of Members

Now that we are certain our `subscribers` group exists, we can add our subscribers to it:

```java
connection.updateGroup(group.id(), ClxApi.groupUpdate()
        .addMemberInsertion("{PHONE1}", "{PHONE2}", "{PHONE3}")
        .build());
```

### Send a Message to Your Group

To send a message to one of your groups, simply use the group ID as one of the recipients:

```java
connection.createBatch(ClxApi.batchTextSms()
        .sender("ignored")
        .addRecipient(group.id().toString())
        .body("Hello from Sinch!")
        .build());
```

## Wrap Up

The code sample below puts together all the pieces you've learned above to send a message to a group of subscribers. Before running the example, replace contents of the different String constants at the beginning of the class (ex: `{YOUR_SERVICE_PLAN_ID}`) with the correct values for your account and replace the phone numbers of each group member you want to test with (the `GROUP_MEMBERS` constant).

```java
import com.clxcommunications.xms.*;
import com.clxcommunications.xms.api.GroupResult;
import com.clxcommunications.xms.api.MtBatchTextSmsResult;

import java.util.Optional;
import java.util.logging.Level;
import java.util.logging.Logger;
import java.util.stream.StreamSupport;

public class SendMessageToGroup {
    private static final Logger log = Logger.getLogger(SendMessageToGroup.class.getName());

    private static final String SERVICE_PLAN_ID = "{YOUR_SERVICE_PLAN_ID}";
    private static final String API_TOKEN = "{YOUR_API_TOKEN}";
    private static final String GROUP_NAME = "subscribers";
    private static final String[] GROUP_MEMBERS = new String[] {
            "{PHONE_NUMBER_1}",
            "{PHONE_NUMBER_2}",
            "{PHONE_NUMBER_3}"
    };

    private final ApiConnection connection;

    private SendMessageToGroup(ApiConnection connection) {
        this.connection = connection;
    }

    public static void main(String[] args) {
        ApiConnection connection = ApiConnection.builder()
                .servicePlanId(SERVICE_PLAN_ID)
                .token(API_TOKEN)
                .start();

        SendMessageToGroup sendMessageToGroup = new SendMessageToGroup(connection);

        try (connection) {
            GroupResult group = sendMessageToGroup.ensureGroupExists(GROUP_NAME);

            sendMessageToGroup.ensureGroupHasMembers(group, GROUP_MEMBERS);

            MtBatchTextSmsResult batch = sendMessageToGroup.sendToGroup(group, "Hello from Sinch!");

            log.log(Level.INFO, "SMS sent to group {0} with batch ID {1}", new Object[] {group.id(), batch.id()});

        } catch (Exception e) {
            log.log(Level.SEVERE, "Exception while communicating with Sinch REST API.", e);
        }
    }

    private GroupResult ensureGroupExists(String groupName) {
        return findGroupByName(groupName)
                .orElseGet(() -> createGroup(groupName));
    }

    private void ensureGroupHasMembers(GroupResult group, String... phoneNumbers) throws ApiException, InterruptedException {
        connection.updateGroup(group.id(), ClxApi.groupUpdate()
                .addMemberInsertion(phoneNumbers)
                .build());
    }

    private MtBatchTextSmsResult sendToGroup(GroupResult group, String message) throws ApiException, InterruptedException {
        return connection.createBatch(ClxApi.batchTextSms()
                .sender("ignored")
                .addRecipient(group.id().toString())
                .body(message)
                .build());
    }

    private Optional<GroupResult> findGroupByName(String name) {
        PagedFetcher<GroupResult> groups = connection.fetchGroups(GroupFilter.builder().build());

        return StreamSupport.stream(groups.pages().spliterator(), false)
                .flatMap(page -> page.content().stream())
                .filter(group -> name.equals(group.name()))
                .findFirst();
    }

    private GroupResult createGroup(String name) {
        try {
            GroupResult createdGroup = connection.createGroup(ClxApi.groupCreate().name(name).build());
            log.info("Created group with ID " + createdGroup.id());
            return createdGroup;
        } catch (InterruptedException | ApiException e) {
            throw new RuntimeException(e);
        }
    }
}
```

<a class="gitbutton pill" target="_blank" href="https://github.com/sinch/docs/blob/master/docs/tutorials/java/tutorial-java-send-sms-group.md"><span class="fab fa-github"></span>Edit on GitHub</a>
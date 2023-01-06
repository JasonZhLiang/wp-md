---
title: "Bundling API Calls"
date: "2019-08-13"
---

brainCloud track's most API calls with brainCloud as +1 API call.

Some API calls that involve communicating with external systems are an additional +1's - like sending a mail is +1 API, and +1 Emails Sent. Sending a request to an external server is +1 API and +1 Outgoing Web Service.

This following document will explain how brainCloud client bundles API calls, and how you can use cloud code to reduce API counts.

* * *

### Performing separate API calls

After your end-user authenticates, let's say your app wishes to set the user's attributes, name, and get their current user state.

var attributes = BrainCloud.JsonFx.Json.JsonWriter.Serialize(
	new Dictionary<string, object>
	{
		{"favoriteCoffee", "darkRoast"}
	});
var wipeExisting = false;
var displayName = "John Doe";

Task.Delay(100).ContinueWith(\_ =>
	\_bc.PlayerStateService.ReadUserState(successCallback, failureCallback));
Task.Delay(300).ContinueWith(\_ =>
	\_bc.PlayerStateService.UpdateAttributes(attributes, wipeExisting, successCallback,
		failureCallback));
Task.Delay(500).ContinueWith(\_ =>
	\_bc.PlayerStateService.UpdateUserName(displayName, successCallback, failureCallback));

These actions count as +3 API calls.

![](images/SeparateApiCalls.png)

You will be able to review each an every API call made on the brainCloud portal.

* * *

## Merging API calls into one packet

In cases where multiple API calls occur at the same time, the brainCloud client will automatically bundle them into one packet.

\_bc.PlayerStateService.ReadUserState(successCallback, failureCallback);
\_bc.PlayerStateService.UpdateAttributes(attributes, wipeExisting, successCallback, failureCallback);
\_bc.PlayerStateService.UpdateUserName(displayName, successCallback, failureCallback);

![](images/BundledApiCalls.png)

On the brainCloud Monitoring dashboard, a bundle packet will look like the following.

The brainCloud server will also respond in one packet. The client library will automatically split the packet responses for the appropriate success and failure callbacks.

Note that this bundling does not affect the tracking of API counts. Each API call is still executed separately on the server, and the above example will always count as +3 API calls.

* * *

### Using cloud code to merge API calls together

Let's say in your example, you are always going to be calling ReadUserState, UpdateAttributes, and UpdateUserName at the same time. There is a method of bundling these three API calls, that will allow us to _reduce the API count_, and that method is cloud code.

If you are new to cloud code, getting started tutorials can be found [here](https://getbraincloud.com/apidocs/cloud-code-central/cloud-code-tutorials/).

So on the brainCloud portal, you are going to create the following cloud code script call `SetUserStateData`.

var response = {};

var playerStateProxy = bridge.getPlayerStateServiceProxy();

var attributes = data.attributes;
var wipeExisting = data.wipeExisting;
var playerStateProxy = bridge.getPlayerStateServiceProxy();
response.updateAttributesResponse = playerStateProxy.updateAttributes(attributes, wipeExisting);

var displayName = data.displayName;
var playerStateProxy = bridge.getPlayerStateServiceProxy();
response.updateUserNameResponse = playerStateProxy.updateUserName(displayName);

response.userState = playerStateProxy.readUserState();

response;

This script will handle the `updateAttributes` and `updateUserName` functions, while also returning the user read `userState`.  
In your app, you can now merge those three processes into one single method that calls the cloud code script.

string scriptName = "SetUserStateData";
var userStateData = BrainCloud.JsonFx.Json.JsonWriter.Serialize(
    new Dictionary<string, object>
    {
        {
            "attributes", new Dictionary<string, object>
            {
                {"favoriteCoffee", "darkRoast"}
            }
        },
        {"wipeExisting", false},
        {"displayName", "John Doe"}
    });

\_bc.ScriptService.RunScript(scriptName, userStateData, successCallback, failureCallback);

The first three API calls made in a cloud code script are free, and future API calls can occur at 1/2+ API count.

Find additional information on pricing rules related rules [here](http://help.getbraincloud.com/en/articles/82325-how-does-braincloud-pricing-work).

So besides neatly packaging related logic together into one cloud code script, you have successfully reduced your ongoing API counts by +2!

* * *

### Concluding thoughts

Before releasing your app, you should check which API calls brainCloud is merging together into one packet, and use that as a starting point of where you should start merging API calls into one cloud code script.

Merge similar logic when applicable, to bundle logic, but also to reduce overall API counts when the bulk of your end-users start using your app.

Happy coding!

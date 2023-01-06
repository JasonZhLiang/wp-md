---
title: "Cloud Code Exception Handling (Deprecated)"
date: "2015-05-06"
---

Note: This mechanism has been deprecated. Cloud code method return values now are formatted to better mirror the other client APIs. They will now return status codes, reason codes, etc in the method return object as well as any data.

This mechanism will continue to be supported for legacy scripts. Legacy behaviour is controlled by a flag in the "Core App Info - Advanced Settings" portal page. It's labeled "Use Legacy Script Result Format".

Braincloud provides a mechanism for querying data regarding exceptions that occur within the service proxies. The methods are:

- isProcessingException - this tells you whether it's a braincloud processing exception or a vanilla java exception
- getExceptionData - this returns and object describing some details about the exception

Both methods take the "err" object passed in the catch clause as a parameter.

The exception data object has the following properties

- class - a simple class name of the exception
- errorCode - processing exception status code
- reasonCode - processing exception reason code
- message - exception message

The "class" and "message" properties apply to all exceptions. The "errorCode" and "reasonCode" properties are only for processing exceptions. Processing exceptions are essentially the same as what is returned to a regular client (like Unity) as errors.

Here's a simple example of the use of this feature:

\[codesyntax lang="javascript"\]

try {
	do something....
}
catch (err) {
	if (bridge.isProcessingException(err))
	{
		exceptionData = bridge.getExceptionData(err);
		if (exceptionData.reasonCode == 40344)
		{
			do something...
		}
	}
}

\[/codesyntax\]

Here is a more complicated example that demonstrates a potential use of the exception handling. The following methods implement a simple clan membership feature that could be implemented in cloud code. The methods allow you to create a clan, join it and leave it.

CreateClan:

\[codesyntax lang="javascript"\]

function createClan(clanName, clanCrest) {
   
// Create empty result object
    var result = {};

// retrieve the global entity service proxy
    entityService = bridge.getGlobalEntityServiceProxy();

// Create a query data object
    var readQuery = {};
    readQuery\['data.name'\] = clanName;

// Execute query
    entityList = entityService.getList(readQuery, null, 1);
    
// If anything found, already exists
    if (entityList\['entityListCount'\] > 0)
    {
        result\['success'\] = false;
        result\['message'\] = "Clan already exists";
        return result;
    }

// Create data and acl objects
    var newEntityData = {};
    var newEntityAcl = {};

// Set data properties
    newEntityData\['name'\] = clanName;
    newEntityData\['crest'\] = clanCrest;

// 2 means all access
    newEntityAcl\['other'\] = 2;

// Create the clan
    var newEntity = entityService.createEntity("Clan", 0, newEntityAcl, newEntityData);
    result\['newClan'\] = newEntity;
    result\['success'\] = true;

// return the newly created object and success/fail set
    return result;
}

// Retrieve script parameters
var name = data\['name'\];
var crest = data\['crest'\];

// Call create function
createClan(name, crest);

\[/codesyntax\]

JoinClan:

\[codesyntax lang="javascript"\]

function joinClan(clanName) {

// Create empty result object
    var result = {};

// Retrieve the player state service proxy
    playerStateService = bridge.getPlayerStateServiceProxy();
    
// Read the player state to get the profile id
    playerState = playerStateService.readPlayerState();
    profileId = playerState.profileId;

// retrieve the global entoty service proxy
    entityService = bridge.getGlobalEntityServiceProxy();

// Limit the number of retries
    var retryCount = 0;
    
    for (retryCount = 0; retryCount < 10; retryCount++)
    {
        try
        {
            // Create a query data object
            var readQuery = {};
            readQuery\['data.name'\] = clanName;
        
            // Execute the query
            entityListResult = entityService.getList(readQuery, null, 1);
            
            // Missing clan error
            if (entityListResult.entityListCount == 0)
            {
                result\['success'\] = false;
                result\['message'\] = "Clan does not exist";
                return result;
            }

            // grab the first entry (there should only be one)
            var entityList = entityListResult.entityList;
            var entity = entityList\[0\];
            
            // Initialize member map if empty
            if (entity.data.members === undefined)
            {
                entity.data.members = {};
            }
            
            // Add him to the member list
            entity.data.members\[profileId\] = true;
            
            // Grab the version from the entity read to pass into the update call
            // This is the concurrent access support mechanism
            version = entity\['version'\];

            // Update the entry getting the updated entry as a result
            var updateEntity = {};
            updatedEntity = entityService.updateEntity(entity.entityId, version, entity.data);
            result\['success'\] = true;
            result\['updatedEntity'\] = updatedEntity;
        
            // return the updated entry and the success/fail set
            return result;
        }
        catch (err)
        {
            // Exception handling
            // If its a processing error and the code is 40344 that means a version mismatch
            // Somebody else has updated the record
            // Therefore loop back to read to retrieve the new version and try again
            if (bridge.isProcessingException(err))
            {
                var exceptionData = bridge.getExceptionData(err);
                if (exceptionData.reasonCode == 40344)
                {
                    continue;
                }
            }

            // Else just rethrow the exception
            throw err;
        }
    }
    
    // If you get here the retry count has been exhausted
    result\['success'\] = false;
    result\['message'\] = "Update retry count exhausted";
    return result;
}

// Retrieve the clan name from the input parameters
var name = data.clanName;

// Call join function
joinClan(name);

\[/codesyntax\]

LeaveClan:

\[codesyntax lang="javascript"\]

function leaveClan(clanName) {
    
// Create empty result object
    var result = {};

// Retrieve the player state service proxy
    playerStateService = bridge.getPlayerStateServiceProxy();
    
// Read the player state to get the profile id
    playerState = playerStateService.readPlayerState();
    profileId = playerState.profileId;

// retrieve the global entity service proxy
    entityService = bridge.getGlobalEntityServiceProxy();

// Limit the number of retries
    var retryCount = 0;
    
    for (retryCount = 0; retryCount < 10; retryCount++)
    {
        try
        {
            // Create a query data object
            var readQuery = {};
            readQuery\['data.name'\] = clanName;
        
            // Execute the query
            entityListResult = entityService.getList(readQuery, null, 1);
            
            // Missing clan error
            if (entityListResult.entityListCount == 0)
            {
                result\['success'\] = false;
                result\['message'\] = "Clan does not exist";
                return result;
            }
            
            // grab the first entry (there should only be one)
            var entityList = entityListResult.entityList;
            var entity = entityList\[0\];

            // No members means he must have already left
            if (entity.data.members === undefined)
            {
                result\['success'\] = true;
                result\['message'\] = "Already left clan";
                result\['updatedEntity'\] = entity;
                return result;
            }
            
            // Remove him from the members list
            delete entity.data.members\[profileId\];

            // Grab the version from the entity read to pass into the update call
            // This is the concurrent access support mechanism
            version = entity\['version'\];

            // Update the entry getting the updated entry as a result
            var updateEntity = {};
    
            updatedEntity = entityService.updateEntity(entity.entityId, version, entity.data);
            result\['success'\] = true;
            result\['updatedEntity'\] = updatedEntity;
        
            // return the updated entry and the success/fail set
            return result;
        }
        catch (err)
        {
            // Exception handling
            // If its a processing error and the code is 40344 that means a version mismatch
            // Somebody else has updated the record
            // Therefore loop back to read to retrieve the new version and try again
            if (bridge.isProcessingException(err))
            {
                var exceptionData = bridge.getExceptionData(err);
                if (exceptionData.reasonCode == 40344)
                {
                    continue;
                }
            }
            
            // Else just rethrow the exception
            throw err;
        }
    }
    
    // If you get here the retry count has been exhausted
    result\['success'\] = false;
    result\['message'\] = "Update retry count exhausted";
    return result;
}

// Retrieve the clan name from the input parameters
var name = data.clanName;

// Call leave function
leaveClan(name);

\[/codesyntax\]

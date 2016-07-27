# Salesforce Integration

## Table of contents
#### Endpoints
- [Get recently viewed](../documentation/Salesforce.md#get-recently-viewed)
- [Searching Salesforce records](../documentation/Salesforce.md#searching-salesforce-records)
- [Getting contacts related to a Salesforce account object](../documentation/Salesforce.md#getting-contacts-related-to-a-Salesforce-account-object)
- [Getting Salesforce invitee details](../documentation/Salesforce.md#getting-salesforce-invitee-details)
- [Getting opportunity suggestions from contacts](../documentation/Salesforce.md#getting-opportunity-suggestions-from-contacts)
- [Getting Salesforce SObjects](../documentation/Salesforce.md#getting-salesforce-sobjects)
- [Getting all fields of a Salesforce SObject](../documentation/Salesforce.md#getting-all-fields-of-a-salesforce-sobject)
- [Updating a user's selected objects/fields](../documentation/Salesforce.md#updating-a-users-selected-objectsfields)
- [Updating a user's primary/secondary fields](../documentation/Salesforce.md#updating-a-users-primarysecondary-fields)
- [Get all event/task record types](../documentation/Salesforce.md#get-all-eventtask-record-types)
- [Sync Salesforce layout data](../documentation/Salesforce.md#sync-salesforce-layout-data)

#### Discussion on invitees
- [View here](../documentation/Salesforce.md#discussion-on-invitees)

***
## Endpoints
### Get recently viewed
**Discussion**

To access a user's recently viewed Salesforce records, call this endpoint and pass in their account ID. This endpoint will automatically pull in the user's ActivityHub preferences, determine which Salesforce objects they have selected to view (i.e., `Account`, `Contact`, `Project__c`), as well as any fields the user has selected to use as their primary/secondary display fields. (Primary/secondary - or "Main" - fields are used to essentially replace the default `Name` field that Salesforce shows and add sub-information that the User Chooses. This is especially useful for Salesforce objects with auto-numbers as their `Name`.)

**File**: `/src/salesforce_objects.js`

**Function**: `get_sf_recents_for_acct`

**Parameters**
- `token` (**required**) - a valid ActivityHub authentication token for this user
- `account_id` (**required**) - ID of a user's Salesforce `account_linked` object
- `ignore_retry` (**optional**) - boolean used internally to force the function not to try refreshing an OAuth token

**Supports internal override?** 
Yes

**Example request body**
```
{
  "token": "XXXXX",
  "account_id": "WMajQtTiYr"
}
```

**Example response**
```
{
  "Recents": [
    {
      "Prefix": "003",
      "API": "Contact",
      "Display": "Contacts",
      "Items": [
        {
          "ID": "003o000000u9qwgAAA",
          "Email": "john@test.com",
          "PrimaryText": "John Doe",
          "SecondaryText": "5/25/16 4:57am"
        }
      ]
    },
    {
      "Prefix": "005",
      "API": "User",
      "Display": "Users",
      "Items": [
        {
          "ID": "005o0000001dWRTAA2",
          "Email": "michael@nexmachine.com",
          "PrimaryText": "Cynthia Capobianco",
          "SecondaryText": ""
        }
      ]
    }
  ]
}
```
***
### Searching Salesforce records
**Discussion**

Since recently viewed records may not contain all the items that a user needs, they can easily search Salesforce using this endpoint. Like the endpoint for getting recently viewed, this also grabs and utilizes the user's primary and secondary field selections, if any.

**File**: `/src/salesforce_objects.js`

**Function**: `get_sf_search_results`

**Parameters**
- `token` (**required**) - a valid ActivityHub authentication token for this user
- `account_id` (**required**) - ID of a user's Salesforce `account_linked` object
- `sf_object` (**required**) - API name of the Salesforce object being queried
- `search_term` (**required**) - the string being searched for (at least 1 character)
- `ignore_retry` (**optional**) - boolean used internally to force the function not to try refreshing an OAuth token

**Supports internal override?** 
Yes

**Example request body**
```
{
  "sf_object": "Contact",
  "search_term": "ash",
  "token": "XXXXX",
  "account_id": "WMajQtTiYr"
}
```

**Example response**
```
{
  "Results": [
    {
      "ID": "003o000000u8W9kAAE",
      "Email": "ashley@test.io",
      "PrimaryText": "Ashley James",
      "SecondaryText": "5/20/16 11:32pm"
    }
  ]
}
```
***
### Getting contacts related to a Salesforce account object
**Discussion**

Within Salesforce, a contact can be related to an account object in a given organization. This function can be used to retrieve contacts associated with a given account ID.

**File**: `/src/salesforce_objects.js`

**Function**: `get_sf_contact_relations`

**Parameters**
- `token` (**required**) - a valid ActivityHub authentication token for this user
- `account_id` (**required**) - ID of a user's Salesforce `account_linked` object
- `sf_account_id` (**required**) - ID of the account object within the user's Salesforce organization for which related contacts need to be returned
- `ignore_retry` (**optional**) - boolean used internally to force the function not to try refreshing an OAuth token

**Supports internal override?** 
Yes

**Example request body**
```
{
  "token": "XXXXX",
  "sf_account_id": "001o000000jS8PS",
  "account_id": "WMajQtTiYF"
}
```

**Example response**
```
{
  "Results": [
    {
      "ID": "003o000000u8W9kAAE",
      "Email": "james@johnson.com",
      "PrimaryText": "James Johnson",
      "SecondaryText": "5/20/16 11:32pm"
    }
  ]
}
```
***
### Getting Salesforce invitee details
**Discussion**

*For internal use only.* Because of how Salesforce returns data in queries for events and tasks (only IDs, no name or email), we have to manually pull this information back. This function goes out to Salesforce and requests a name (or primary field, if the user has one selected) and email for a given set of `event_invitee` or `task_invitee` IDs, then updates them within our database. This works seamlessly with Contacts, Leads, and Users. Normally, this is called by functions that are pulling events or tasks in from Salesforce. **For internal use only, external requests will be rejected.**

**File**: `/src/salesforce_objects.js`

**Function**: `get_sf_invitee_details`

**Parameters**
- `key` (**required**) - internal key for authorizing this endpoint
- `account_id` (**required**) - ID of a user's Salesforce `account_linked` object
- `invitee_ids` (**required**) - array of `event_invitee` IDs for Salesforce invitees
- `is_task` (**required**) - flag indicating whether these are `event_invitee` or `task_invitee` object IDs
- `ignore_retry` (**optional**) - boolean used internally to force the function not to try refreshing an OAuth token

**Supports internal override?** 
Yes

**Example request body**
```
{
  "key": "XXXXX",
  "is_task": false,
  "invitee_ids": [
    "rMazztTiYF"
  ],
  "account_id": "WMajQtTiYF"
}
```

**Example response**
```
{
	"Updated": 1
}
```
### Getting opportunity suggestions from contacts
**Discussion**

It can be very useful to be able to suggest opportunities based on contacts associated with them. ActivityHub can do this with a list of Salesforce contact IDs by using opportunity contact roles within Salesforce. Calling this function will return a list of suggestions, prioritized by **highest relevancy first**.

**File**: `/src/salesforce_objects.js`

**Function**: `get_sf_opp_suggestions`

**Parameters**
- `token` (**required**) - a valid ActivityHub authentication token for this user
- `account_id` (**required**) - ID of a user's Salesforce `account_linked` object
- `sf_contact_ids` (**required**) - array of IDs of contacts within a Salesforce account
- `ignore_retry` (**optional**) - boolean used internally to force the function not to try refreshing an OAuth token

**Supports internal override?** 
Yes

**Example request body**
```
{
  "token": "XXXXX",
  "account_id": "WMajQtriYF",
  "sf_contact_ids": [
    "003o000000BTxwF",
    "003o000000BTxwB",
    "003o000000BTxwA"
  ]
}
```

**Example response**
```
{
  "Suggestions": [
    {
      "ID": "006o000000IsIZFAA3",
      "Name": "Smash Hit Productions"
    },
    {
      "ID": "006o000000IsIZKAA3",
      "Name": "Synergetic Tech, Inc."
    }
  ]
}
```
### Getting Salesforce SObjects
**Discussion**

This endpoint retrieves all SObjects from the specified Salesforce account.

**File**: `/src/salesforce_objects.js`

**Function**: `get_sf_sobjects`

**Parameters**
- `token` (**required**) - a valid ActivityHub authentication token for this user
- `account_id` (**required**) - ID of a user's Salesforce
- `ignore_retry` (**optional**) - boolean used internally to force the function not to try refreshing an OAuth token

**Supports internal override?** 
Yes

**Example request body**
```
{
  "token": "XXXXX",
  "account_id": "WMajQtriYF",
  "ignore_retry": false
}
```

**Example response**
```
{
  "sObjects": [
    {
      "Name": "Account",
      "Prefix": "001",
      "DisplayName": "Accounts"
    },
    {
      "Name": "User",
      "Prefix": "005",
      "DisplayName": "Users"
    }
  ]
}
```
### Getting all fields of a Salesforce SObject
**Discussion**

This endpoint retrieves all fields to a SObjects from the specified Salesforce account.

**File**: `/src/salesforce_objects.js`

**Function**: `get_sf_sobject_fields`

**Parameters**
- `token` (**required**) - a valid ActivityHub authentication token for this user
- `account_id` (**required**) - ID of a user's Salesforce
- `sf_object_name` (**required**) - API name of the Salesforce SObject
- `ignore_retry` (**optional**) - boolean used internally to force the function not to try refreshing an OAuth token

**Supports internal override?** 
Yes

**Example request body**
```
{
  "token": "XXXXX",
  "account_id": "WMajQtriYF",
  "sf_object_name": "Account",
  "ignore_retry": false
}
```

**Example response**
```
{
  "Fields": [
    {
      "Name": "Account Name",
      "APIName": "Name",
      "Type": "string"
    },
    {
      "Name": "Account Type",
      "APIName": "Type",
      "Type": "picklist"
    }
  ]
}
```
### Updating a user's selected objects/fields
**Discussion**

This endpoint takes raw Salesforce object names and their selected fields. It then verifies them against the users Salesforce and makes sure there's at least one non-"who" SObject selected.

**File**: `/src/salesforce_objects.js`

**Function**: `update_sf_objects_fields`

**Parameters**
- `token` (**required**) - a valid ActivityHub authentication token for this user
- `account_id` (**required**) - ID of a user's Salesforce
- `objects_array` (**required**) - Map of the following values
- `api_name` (**required**) - SF SObject API name
- `fields` (**optional**) - Map of field values

**Supports internal override?** 
No

**Example request body**
```
{
  "token": "XXXXX",
  "account_id": "JjCe6C2UHK",
  "objects_array": [
    {
      "api_name": "Account",
      "fields": [
        {
          "SFAPIName": "Phone",
          "Name": "Phone",
          "SFType": "phone"
        },
        {
          "SFAPIName": "Type",
          "Name": "Type",
          "SFType": "picklist"
        }
      ]
    }
  ]
}
```

**Example response**
```
{
  "ID": "GefhQnsBrP"
}
```
### Updating a user's primary/secondary fields
**Discussion**
This endpoint updates the user's selected primary and secondary fields.

**File**: `/src/salesforce_objects.js`

**Function**: `update_sf_primary_secondary`

**Parameters**
- `token` (**required**) - a valid ActivityHub authentication token for this user
- `objects_map` (**required**) - Map of the following values
	- `api_name` (**required**) - SF SObject API name
	- `primary_api_name` (**required**) - API name of primary
	- `primary_type` (**required**) - type of primary field
	- `secondary_api_name` (**optional**) - API name of secondary field
	- `secondary_type` (**optional**) - type of secondary field

**Supports internal override?** 
No

**Example request body**
```
{
  "token": "XXXXX",
  "account_id": "JjCe8C2UHK",
  "objects_map": {
    "Account": {
      "primary_api_name": "Name",
      "secondary_api_name": "Account_Owner",
      "primary_type": "string",
      "secondary_type": "string"
    },
    "Case": {
      "primary_api_name": "Subject",
      "secondary_api_name": "CaseNumber",
      "primary_type": "string",
      "secondary_type": "string"
    }
  }
}
```

**Example response**
```
{
  "ID": "GethQnSBrP"
}
```
### Get all event/task record types
**Discussion**

This endpoint returns all event and task record types available within Salesforce.

**File**: `/src/salesforce_objects.js`

**Function**: `get_sf_record_types`

**Parameters**
- `token` (**required**) - a valid ActivityHub authentication token for this user
- `account_id` (**required**) - ID of the Salesforce account_linked object)
- `ignore_retry` (**optional**) - boolean used internally to force the function not to try refreshing an OAuth token

**Supports internal override?** 
Yes

**Example request body**
```
{
  "token": "XXXXX",
  "account_id": "JjCe8c1UhK",
  "ignore_retry": "false"
}
```

**Example response**
```
{
  "EventRecordTypes": [
    {
      "Name": "Master",
      "TypeID": "012000000000000ABA",
      "URL": "sobjects/Event/describe/layouts/012000000000000ABA"
    }
  ],
  "TaskRecordTypes": [
    {
      "Name": "Master",
      "TypeID": "012000000000000ABA",
      "URL": "sobjects/Task/describe/layouts/012000000000000ABA"
    }
  ]
}
```
### Sync Salesforce layout data
**Discussion**

This endpoint fetches the following information from Salesforce and updates the `account_linked`'s `salesforceManyWho` setting, as well as the `sfLayoutsJSON` value stored in `user_prefs` for this user:
- Syncs event **and** task layouts for either the user's selected record type's layout, or for the `Master` record type's layout if none is specified
	- Custom fields
	- Viable task picklist values
	- Which fields are hidden from the layout (i.e., `IsAllDay`)
- Syncs available phone fields for querying on contacts
- Syncs available roles to assign as opportunity contact roles
- Syncs whether shared activities (ManyWho) are enabled

**File**: `/src/salesforce_objects.js`

**Function**: `sync_sf_layout_data`

**Parameters**
- `token` (**required**) - a valid ActivityHub authentication token for this user
- `account_id` (**required**) - ID of the Salesforce account_linked object)

**Supports internal override?** 
Yes

**Example request body**
```
{
  "token": "XXXXX",
  "account_id": "JjCe8c1UhK"
}
```

**Example response**
```
{
  "Profile": {
    //Will be exactly the same as what's returned by get_user_info
  },
  "Response": {
    "Updated": true
  }
}
```
***
## Discussion on invitees
**General functionality for who IDs**
*Before you read further, take a few shots of vodka - then maybe you'll be able to understand why Salesforce designed it like this... Or, more likely, you'll still be just as confused.*
- When ManyWho is on, everything is uploaded as an `EventRelation`, which marks them as related (except for users, who are always invitees)
- When ManyWho is off for **events**...
	- Everything is uploaded as an `EventRelation`, which marks them **all** as an invitee automatically. However, the `WhoId` field is set to the first contact/lead selected by the user so that it's related to something (this is the Salesforce UI's behavior as well). Why a contact or lead, you ask? Well, simply put, Salesforce doesn't allow users to be used in this field. *Why? Take a few more shots of vodka and maybe you can figure it out*
	- Setting a lead as a `WhoId` in this case renders the activity unable to have a `WhatId`. The reverse is true as well, because symmetry
- Invitees are not technically related to events inside of Salesforce, so you can't run reports on them
- We handle `WhoId`s by essentially converting them to invitees on download and back to `WhoId`s on upload (if necessary). This makes our code much cleaner and has resulted in fewer bugs

**Other notes on who IDs**
- Salesforce does not support adding multiple relationships to recurring activities
- Salesforce does not allow you to relate converted leads to activities
- If `User A` invites `User B` and `User C`, `User B` and `User C` each have `EventRelation` records on `User A`'s event
- However, all 3 users have **separate** `Event` objects, `User A`'s being the master, and `User B` and `User C` being the children (their events have `IsChild` = `true`)
- `User B` and `User C` cannot update any of these events, aside from their status, response notes, and reminder time within Salesforce
- In addition, `User B` and `User C` cannot see any other invitees (or each other) on the event
- Since `User B` and `User C` each technically have their own `Event` record, ActivityHub will show them as the owner of their respective copy, even though they're technically children of the same event (and from the user's perspective they're all the *same* event)

**Fun facts**
- Fact: If you sign in as `User A` in Salesforce and open the event ID of `User B` or `User C`, you can respond to the invite **on their behalf *as that person***
- Fact: `User B` and `User C` can't respond to invitations via the API because they don't have access to the `EventRelation` records on the master `Event` record as invitees
- Fact: The reason `User B` and `User C` can't see the `EventRelation` records is because those records' `EventId` fields correspond to the master event record - which, keep in mind, they have a *copy* of. Hence, there's no way to query it (to my knowledge) since they don't know the master event's ID
- Fact: When you relate a lead as a non-invitee `EventRelation`, you cannot relate *any* other record as either a `WhoId`, invitee, or `WhatId`
- Fact: The `TaskRelation` object in Salesforce **does not exist** when ManyWho is off, but `EventRelation` is still used
- Fact: The fact that there is no `TaskRelation` when ManyWho is off means that you can only associate one `WhoId` with a task when ManyWho is off
- Fact: Since invitees aren't a thing on tasks, users cannot be associated with tasks (aside from being assigned to them as `OwnerId`s)
- Fact: If you had taken a shot for every flaw in Salesforce's activity architecture while reading the above, you wouldn't be reading this. Because you'd be dead.

***

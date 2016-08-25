# Salesforce Integration

## Table of contents
#### Endpoints
- [Get recently viewed](#get-recently-viewed)
- [Getting selected field values for a record](#get-selected-field-values-for-a-salesforce-record)
- [Searching Salesforce records](#searching-salesforce-records)
- [Getting info on Salesforce records](#getting-info-on-salesforce-records)
- [Getting contacts related to a Salesforce account object](#getting-contacts-related-to-a-Salesforce-account-object)
- [Getting Salesforce invitee details](#getting-salesforce-invitee-details)
- [Getting opportunity suggestions from contacts](#getting-opportunity-suggestions-from-contacts)
- [Getting Salesforce SObjects](#getting-salesforce-sobjects)
- [Getting all fields of a Salesforce SObject](#getting-all-fields-of-a-salesforce-sobject)
- [Updating a user's selected objects/fields](#updating-a-users-selected-objectsfields)
- [Updating a user's primary/secondary fields](#updating-a-users-primarysecondary-fields)
- [Get all event/task record types](#get-all-eventtask-record-types)
- [Update selected event/task record types](#update-selected-eventtask-record-types)
- [Sync Salesforce layout data](#sync-salesforce-layout-data)

#### Internal functions
- [`stringFromReturnedVal()`](#stringfromreturnedval)
- [`isWho()`](#iswho)
- [`touchRecord()`](#touchrecord)
- [`apiNameForID()`](#apinameforid)
- [`selectedSFObjects()`](#selectedsfobjects)
- [`selectedSFMainFields()`](#selectedsfmainfields)
- [`selectedSFRecordTypes()`](#selectedsfrecordtypes)
- [`salesforceLayoutData()`](#salesforcelayoutdata)
- [`fieldsToQueryForLayout()`](#fieldstoqueryforlayout)
- [`updateSalesforceInvitees()`](#updatesalesforceinvitees)

#### Discussion on invitees
- [View here](#discussion-on-invitees)

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
### Get selected field values for a Salesforce record
**Discussion**

Users are able to select additional fields to view for any given Salesforce object using the `update_sf_objects_fields` endpoint. In the ActivityHub mobile application, for example, these fields are displayed on the quick view. Calling this endpoint for a given record returns displayable **string** values for all of the selected fields for the given Salesforce object type.

**File**: `/src/salesforce_objects.js`

**Function**: `get_sf_record_values`

**Parameters**
- `token` (**required**) - a valid ActivityHub authentication token for this user
- `account_id` (**required**) - ID of a user's Salesforce `account_linked` object
- `record_id` (**required**) - ID of the record to query details for
- `ignore_retry` (**optional**) - boolean used internally to force the function not to try refreshing an OAuth token

**Supports internal override?** 
Yes

**Example request body**
```
{
  "token": "XXXXX",
  "account_id": "czar4zFSPe",
  "record_id": "001o000000jS8PS"
}
```

**Example response**
```
{
  "DisplayValues": {
    "BillingCity": "Somecity",
    "Name": "Test Account 1"
  }
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
### Getting info on Salesforce records
**Discussion**

Sometimes you may need to get additional information on a given Salesforce record (or set of records). This endpoint will return the primary and secondary field for a given set of records IDs - you **can** even mix different object types, like Account and Contact, in one call! An example use case of this function would be getting the names to display of `RelatedTo` fields for events or tasks.

Note: Records whose Salesforce type can't be found as a selected SObject by the user in their ActivityHub profile will be returned in the `Failed` list.

**File**: `/src/salesforce_objects.js`

**Function**: `get_sf_record_info`

**Parameters**
- `token` (**required**) - a valid ActivityHub authentication token for this user
- `account_id` (**required**) - ID of a user's Salesforce `account_linked` object
- `record_ids` (**required**) - array of IDs of the records to fetch information on
- `ignore_retry` (**optional**) - boolean used internally to force the function not to try refreshing an OAuth token

**Supports internal override?** 
Yes

**Example request body**
```
{
  "token": "XXXXX",
  "account_id": "YxjcW6K0l6",
  "record_ids": [
    "001o000000jS8PS",
    "003o000000uk3St",
    "some-invalid-id"
  ]
}
```

**Example response**
```
{
  "Failed": [
    "some-invalid-id"
  ],
  "Found": [
    {
      "ID": "001o000000jS8PSAA0",
      "PrimaryText": "Test Account 1",
      "SFAPIName": "Account",
      "SecondaryText": ""
    },
    {
      "ID": "003o000000uk3StAAI",
      "PrimaryText": "Daniel Kauppi",
      "SFAPIName": "Contact",
      "SecondaryText": "Example value"
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
***
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
***
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
  "SObjects": [
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
***
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
***
### Updating a user's selected objects/fields
**Discussion**

This endpoint takes raw Salesforce object names and their selected fields. It then verifies them against the users Salesforce and makes sure there's at least one non-"who" SObject selected.

**File**: `/src/salesforce_objects.js`

**Function**: `update_sf_objects_fields`

**Parameters**
- `token` (**required**) - a valid ActivityHub authentication token for this user
- `account_id` (**required**) - ID of a user's Salesforce linked account
- `objects` (**required**) - array of maps with the following values:
	- `api_name` (**required**) - Salesforce object's API name
	- `fields` (**optional**) - array of maps with the following values:
		- `SFAPIName` (**required**) - API name of the field
		- `Name` (**required**) - display name of the field
		- `SFType` (**required**) - Salesforce type of the field

**Supports internal override?** 
No

**Example request body**
```
{
  "token": "XXXXX",
  "account_id": "JjCe6C2UHK",
  "objects": [
    {
      "api_name": "Account",
      "fields": [
        {
          "SFAPIName": "Phone",
          "Name": "Phone",
          "SFType": "phone"
        },
        {
          "SFAPIName": "My_Options__c",
          "Name": "My Options",
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
***
### Updating a user's primary/secondary fields
**Discussion**
This endpoint updates the user's selected primary and secondary fields. It acts similarly to a `PATCH` request, updating only what you specify. Here are some details:
- Excluding a selected object in the `objects` parameter will have no effect on that object
- Setting `PrimaryAPIName` to `null` will revert it back to the standard `Name` field
- Setting `SecondaryAPIName` to `null` will remove the secondary field
- To remove an object's primary/secondary customizations, pass `null` *instead of a map* for the object

**File**: `/src/salesforce_objects.js`

**Function**: `update_sf_primary_secondary`

**Parameters**
- `token` (**required**) - a valid ActivityHub authentication token for this user
- `objects` (**required**) - map of maps (each with the following values - or `null`), at the key of the object's API name
	- `PrimaryAPIName` (**required**) - API name of primary
	- `PrimaryType` (**required**) - type of primary field
	- `SecondaryAPIName` (**optional**) - API name of secondary field
	- `SecondaryType` (**optional**) - type of secondary field - **required if a secondary API name is provided**)

**Supports internal override?** 
No

**Example request body**
```
{
  "token": "XXXXX",
  "objects": {
    "Account": {
      "PrimaryAPIName": "Name",
      "SecondaryAPIName": "Account_Owner",
      "PrimaryType": "string",
      "SecondaryType": "string"
    },
    "Project__c": null,
    "Filing__c": {
      "PrimaryAPIName": null,
      "SecondaryAPIName": "Value__c",
      "SecondaryType": "number"
    },    
    "Example__c": {
      "PrimaryAPIName": "Value__c",
      "PrimaryType": "currency"
    }
  }
}
```

**Example response**
```
{
  "MainFields": {
    "Account": {
      "PrimaryAPIName": "Name",
      "SecondaryAPIName": "Account_Owner",
      "PrimaryType": "string",
      "SecondaryType": "string"
    },
    "Filing__c": {
      "PrimaryAPIName": "Name",
      "PrimaryType": "string",
      "SecondaryAPIName": "Value__c",
      "SecondaryType": "number"
    },
    "Example__c": {
      "PrimaryAPIName": "Value__c",
      "PrimaryType": "currency"
    }
  }
}
```
***
### Get all event/task record types
**Discussion**

This endpoint returns all event and task record types available within Salesforce.

**File**: `/src/salesforce_objects.js`

**Function**: `get_sf_record_types`

**Parameters**
- `token` (**required**) - a valid ActivityHub authentication token for this user
- `account_id` (**required**) - ID of the Salesforce `account_linked` object
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
***
### Update selected event/task record types
**Discussion**

This endpoint should be used to update a given user's selected record types for use within ActivityHub. This will result in a consequent update of their Salesforce layouts within ActivityHub, and will re-sync their data with Salesforce to pull in new events/tasks with the specified record type(s).

**File**: `/src/salesforce_objects.js`

**Function**: `update_sf_record_types`

**Parameters**
- `token` (**required**) - a valid ActivityHub authentication token for this user
- `account_id` (**required**) - ID of the Salesforce `account_linked` object
- `selections` (**required**) - map of the following values:
	- `Event` (**optional**) - string ID of the event's record type. Pass `null` to revert to master. If the master record type ID is specified, it will be changed to `null` automatically
	- `Task` (**optional**) - string ID of the task's record type. Pass `null` to revert to master. If the master record type ID is specified, it will be changed to `null` automatically

**Supports internal override?** 
No

**Example request body**
```
{
  "token": "XXXXX",
  "account_id": "YxjcW6K0l6",
  "selections": {
    "Event": "012000000000000AAA",
    "Task": null
  }
}
```

**Example response**
```
{
  "RecordTypes": {
    "Event": {
      "Name": "Master",
      "TypeID": "012000000000000AAA"
    },
    "Task": null
  }
}
```
***
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
## Internal functions
### stringFromReturnedVal()
**Discussion**

This function returns a formatted string value for a given Salesforce record's field that can then be displayed to a user.

**File**: `/src/salesforce_objects.js`

**Function**: `stringFromReturnedVal(value, fieldType)`

**Arguments**
- `value` - whatever the value of the field is (text, number, `null`, etc.)
- `fieldType` - the type of field this is within Salesforce. Custom parsers for the following types are implemented: "currency", "boolean", "number", "address", "date", "datetime", "string"

**Returns**: A formatted string
***
### isWho()
**Discussion**

It can be useful to determine if a record (or API name of a record) is a Salesforce Contact, Lead, or User - a "who". This function does just that by comparing API names or ID prefixes.

**File**: `/src/salesforce_objects.js`

**Function**: `isWho(apiNameOrID)`

**Arguments**
- `apiNameOrID` - either an API name of a Salesforce object (like "Contact") or an ID of a Salesforce record (like "003xxxxxxxxxx"). This is the string that will be evaluated to determine if it is a "who" record

**Returns**: A boolean value
***
### touchRecord()
**Discussion**

When making requests to Salesforce involving other records (i.e., creating a `EventRelation` associated with another object), Salesforce doesn't automatically add this item to `RecentlyViewed` if done via the API. Thus, we have to do a special `FOR VIEW` SOQL query to enforce this. Calling this function returns a promise for an HTTP request that will do just that. Note that this function will NOT attempt to refresh the account token in the result of a `401` or `403` response code.

**File**: `/src/salesforce_objects.js`

**Function**: `touchRecord(account, prefs, ID)`

**Arguments**
- `account` - an `account_linked` object
- `prefs` - the user's `user_prefs` object (to look up the type of the Salesforce object)
- `ID` - the ID of the Salesforce object

**Returns**: A promise that resolves upon completion of the HTTP request
***
### apiNameForID()
**Discussion**

This function is used to get a Salesforce object API name for a given record's ID. Information is pulled from the user's user preferences within our database.

**File**: `/src/salesforce_objects.js`

**Function**: `apiNameForID(ID, prefs)`

**Arguments**
- `ID` - the ID of the Salesforce object
- `prefs` - the user's `user_prefs` object (to look up the type of the Salesforce object)

**Returns**: The Salesforce API name of the object, or `null` if not found
***
### selectedSFObjects()
**Discussion**

A convenience function for getting the values (**or defaults, if any**) for the `sfSObjectsJSON` field of a given `user_prefs` object.

**File**: `/src/salesforce_objects.js`

**Function**: `selectedSFObjects(prefs)`

**Arguments**
- `prefs` - the user's `user_prefs` object

**Returns**: Something like this:
```
[
  {
    "FieldsShowing": [
      {
        "Name": "Phone",
        "SFAPIName": "Phone",
        "SFType": "phone"
      }
    ],
    "Name": "Accounts",
    "Prefix": "001",
    "SFAPIName": "Account"
  }
]
```
***
### selectedSFMainFields()
**Discussion**

A convenience function for getting the values (**or defaults, if any**) for the `sfMainFieldsJSON` field of a given `user_prefs` object. If none are specified, the default `Name` value should be used as `PrimaryAPIName`, and no `SecondaryAPIName` can be specified without a primary.

**File**: `/src/salesforce_objects.js`

**Function**: `selectedSFMainFields(prefs)`

**Arguments**
- `prefs` - the user's `user_prefs` object

**Returns**: Something like this:
```
{
  "Account": {
    "PrimaryAPIName": "Name",
    "SecondaryAPIName": "Example_Field__c",
    "PrimaryType": "string",
    "SecondaryType": "string"
  }
}
```
***
### selectedSFRecordTypes()
**Discussion**

A convenience function for getting the values (**or defaults, if any**) for the `sfTypeSelectionsJSON` field of a given `user_prefs` object. If none are specified a map with `Event` and `Task` (both having `null` values) is returned.

**File**: `/src/salesforce_objects.js`

**Function**: `selectedSFRecordTypes(prefs)`

**Arguments**
- `prefs` - the user's `user_prefs` object

**Returns**: Something like this:
```
{
  "Event": {
    "Name": "Master",
    "TypeID": "012000000000000AAA"
  },
  "Task": null
}
```
***
### salesforceLayoutData()
**Discussion**

A convenience function for getting the values (**or defaults, if any**) for the `sfLayoutsJSON` field of a given `user_prefs` object. This will contain data on `Contact`, `Event`, `Task`, and `OpportunityContactRole` (whose `RoleValues` will be empty if the user doesn't have access to it in Salesforce).

**File**: `/src/salesforce_objects.js`

**Function**: `salesforceLayoutData(prefs)`

**Arguments**
- `prefs` - the user's `user_prefs` object

**Returns**: Something like this:
```
{
  "Contact": {
    "PhoneFields": [
      {
        "Display": "Primary",
        "SFAPIName": "Phone"
      }
    ]
  },
  "Event": {
    "LocationExists": true,
    "AllDayExists": true,
    "PrivateExists": true,
    "LayoutCustomFields": []
  },
  "Task": {
    "StatusValues": [
      "Not Started",
      "In Progress",
      "Completed",
      "Deferred",
      "Waiting on someone else"
    ],
    "PriorityValues": [
      "Low",
      "Normal",
      "High"
    ],
    "LayoutCustomFields": []
  },
  "OpportunityContactRole": {
    "RoleValues": []
  }
}
```
***
### fieldsToQueryForLayout()
**Discussion**

When querying Salesforce for events and tasks, custom fields (and fields that could potentially be hidden from the layout) must be queried. This function provides an easy way to get a list of these fields.

**File**: `/src/salesforce_objects.js`

**Function**: `fieldsToQueryForLayout(object, prefs)`

**Arguments**
- `object` - string, either "Event" or "Task"
- `prefs` - the user's `user_prefs` object

**Returns**: Something like this:
```
[
  {
    "Name": "Location",
    "Type": "string"
  },
  {
    "Name": "My_Field__c",
    "Type": "boolean"
  }
]
```
***
### updateSalesforceInvitees()
**Discussion**

Because of the way Salesforce designed activities (events and tasks) and their respective invitees, it is an absolute *nightmare* to deal with them. Therefore, we created an internal function that can handle updating them (for the most part).

This function was designed to be given a list of invitees for an event or task, and appropriately sync Salesforce with this list. That includes deleting old (removed) invitees, as well as adding new ones. There are a few notes to keep in mind (well, more than a few - see the comments after this function's documentation...), but here's what you need to know to use this:
- This function does not update the `WhoId` of the Salesforce event or task, only the `EventRelation` or `TaskRelation` records
- This function **will fail** if called for a Salesforce task (events work) when ManyWho is disabled in the user's organization
- This function does *not* try to refresh access tokens upon an unauthorized response

**File**: `/src/salesforce_objects.js`

**Function**: `updateSalesforceInvitees(activity, sfActivityID, invitees, prefs, account, isTask)`

**Arguments**
- `activity` - event or task object
- `sfActivityID` - ID of the event or task in Salesforce
- `invitees` - array of event_invitee or task_invitee objects (or null)
- `prefs` - user_prefs object
- `account` - account_linked object
- `isTask` - boolean indicating if these are event or task invitees

**Returns**: A promise resolved upon completion of the update
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

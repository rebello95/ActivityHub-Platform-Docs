# Task Management

## Table of contents
#### Definitions
- [Date formats](../documentation/Task%20Management.md#date-formats)

#### Endpoints
- [Getting a user's tasks](../documentation/Task%20Management.md#getting-a-users-tasks)
- [Creating/updating a task](../documentation/Task%20Management.md#creatingupdating-a-task)
- [Deleting a task](../documentation/Task%20Management.md#deleting-a-task)

***
## Definitions
**Date formats**

The ActivityHub platform uses the [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) standard for date formatting:
- Dates: `YYYY-MM-DD`
	- Example: "2016-07-31"

***
## Endpoints
### Getting a user's tasks
**Discussion**

Fetching a list of a given user's Salesforce tasks is very simple using ActivityHub's API. Simply call this function to retrieve all tasks within a given timeframe. If no timeframe is specified, ActivityHub will default to the last synced timeframe.
- The `AccountID` field on a given task indicates the ID of the `account_linked` that the task belongs to
- The `MatchedUserID` field of an invitee is non-empty if ActivityHub detected another ActivityHub user with this Salesforce ID inside the same Salesforce organization at the time this task was last synced. If one was found, this will be the ID of the user within our database.

**File**: `/src/manage_tasks.js`

**Function**: `get_user_tasks`

**Parameters**
- `token` (**required**) - a valid ActivityHub authentication token for this user
- `start` (**optional**) - start date to return tasks for in YYYY-MM-DD format. If excluded, defaults to the earliest synced date
- `end` (**optional**) - end date to return tasks for in YYYY-MM-DD format. If excluded, defaults to the earliest latest date

**Supports internal override?** 
Yes

**Example request body**
```
{
  "token": "XXXXX",
  "end": "2016-07-31",
  "start": "2016-05-01"
}
```

**Example response**
```
{
  "Tasks": [
    {
      "Status": "Not Started",
      "Date": "2016-06-16",
      "Priority": "Normal",
      "AccountID": "WMajQtTiYr",
      "Color": "#2D69B4",
      "SalesforceData": {
        "CustomFields": {
          "Picklist_Test__c": null,
          "Test_1__c": false,
          "Test_2__c": "abc123"
        }
      },
      "Invitees": [
        {
          "ID": "51hRHFiyp4",
          "Email": "john@test.io",
          "Name": "John Doe",
          "ServiceID": "003o000000u8W9kAAE",
          "MatchedUserID": ""
        }
      ],
      "Notes": "",
      "ID": "1b3zWz1qEz",
      "Subject": "Example Task"
    }
  ],
  "Timeframe": {
    "End": "2016-07-31",
    "Start": "2016-05-01"
  }
}
```
***
### Creating/updating a task
**Discussion**

ActivityHub allows for easy creation and updating of tasks within Salesforce and our database through this endpoint. Here are some additional notes:
- Excluding the `ID` parameter within `task` in your request will result in the creation of a new task
- The `Invitees` value within `task` is optional. If it is not included, no invitees will be changed on the task. If it is included, however, all invitees must be specified
- Failing to include an invitee's `ServiceID` parameter will result in failed requests to Salesforce when updating invitees 
- This endpoint returns a follow up suggestion if keywords are used. If not, the FollowUp value will be null. 

**File**: `/src/manage_tasks.js`

**Function**: `upsert_task`

**Parameters**
- `token` (**required**) - a valid ActivityHub authentication token for this user
- `task` (**required**) - map with the following task metadata:
	- `Subject` (**optional**) - string to use as the task's subject. Defaults to "(Unknown subject)"
	- `Notes` (**optional**) - string to use as the task's notes/description
	- `Priority` (**optional**) - string to use as the task's priority in Salesforce (must be valid picklist values for the specific organization)
	- `Status` (**optional**) - string to use as the task's status in Salesforce (must be valid picklist values for the specific organization)
	- `Date` (**required**) - task date in UTC using YYYY-MM-DD format
	- `ID` (**optional**) - ID of the ActivityHub task. If this is not populated, ActivityHub will simply create a new task
	- `InAccount` (**required**) - ID of the linked account that the task should be in
	- `SalesforceData` (**optional**) - to update additional Salesforce data, use this map of values {`string` => `value`}. This may include the following, none of which are required:
		- `CustomFields` (**optional**) - map of any custom fields you'd like to update on this event. Any specified fields that are recognized by the system will be updated. They should be formatted in a map of {`Salesforce API name` => `value`}
		- `RelatedTo` (**optional**) - string ID of a non-who ID within the event's Salesforce organization
		- `AssignedTo` (**optional**) - string ID of a Salesforce user (cannot be listed as an invitee), to assign the task to. ***If filled in as an ID that is not the requester's, `null` will be returned as the task ID***
	- `Invitees` (**optional**) - array, each with the following information:
		- `Email` (**required**) - email address of the invitee
		- `Name` (**optional**) - name of the invitee
		- `ServiceID` (**required**) - used to identify the invitee in Salesforce. For example, this would be a Salesforce contact's ID (`003xxxxxxxxxxxx`)

**Supports internal override?** 
Yes

**Example request body**
```
{
  "token": "XXXXX",
  "task": {
    "Status": "Not Started",
    "Date": "2016-06-17",
    "Priority": "Normal",
    "Notes": "My test notes",
    "ID": "Cvt8yABrVL",
    "Invitees": [
      {
        "Email": "test@abc.com",
        "Name": "John Doe",
        "ServiceID": "003o000000u9qwg"
      }
    ],
    "InAccount": "Yxjcr6K0r6",
    "Subject": "Test Task",
    "SalesforceData": {
      "CustomFields": {
        "Test_2__c": "This is my custom text"
      },
      "RelatedTo": "006o000000IsIZFAA3",
      "AssignedTo": "005o000000IrAZFAA3"
    }
  }
}
```

**Example response**
```
{
  "TaskID": "ak3jdx9HB3",
  "FollowUp": {
    "Date": "2016-09-06T00:00:00Z",
    "Notes": "This is a follow-up task regarding the conversation from 9/2/16.",
    "Subject": "Follow-Up: Meeting with Bob",
    "Question": "Would you like to create a follow-up event for Friday (9/6)?"
  }
}
```
***
### Deleting a task
**Discussion**

Calling this function for a given task will remove it from both the user's Salesforce account, as well as within our database.

**File**: `/src/manage_tasks.js`

**Function**: `delete_task`

**Parameters**
- `token` (**required**) - a valid ActivityHub authentication token for this user
- `task_id` (**required**) - the ID of the task to be deleted

**Supports internal override?** 
Yes

**Example request body**
```
{
  "token": "XXXXX",
  "task_id": "2cjCu1RfsM"
}
```

**Example response**
```
{
    "Deleted": 1
}
```
***
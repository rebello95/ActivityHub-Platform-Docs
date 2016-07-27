# Event Management

## Table of contents
#### Definitions
- [Invitee response statuses](../documentation/Event%20Management.md#invitee-response-statuses)
- [Event access options](../documentation/Event%20Management.md#event-access-options)
- [Date formats](../documentation/Event%20Management.md#date-formats)
- [Event ID prefixes](../documentation/Event%20Management.md#event-id-prefixes)

#### Endpoints
- [Getting a user's events](../documentation/Event%20Management.md#getting-a-users-events)
- [Creating/updating an event](../documentation/Event%20Management.md#creatingupdating-an-event)
- [Deleting an event](../documentation/Event%20Management.md#deleting-an-event)
- [Responding to an invitation](../documentation/Event%20Management.md#responding-to-an-invitation)

***
## Definitions
**Invitee response statuses**

Each invitee (and owner of an event) is assigned a status, categorized into one of these values:
- `Organizer`: The person has been determined as the owner/organizer of the event based on their email address
- `Going`: The person has confirmed they are attending the event
- `Tentative`: The person has tentatively accepted the invitation
- `Not going`: The person has declined the event invitation
- `Awaiting response`: The person has either not responded, or no data was found

***
**Event access options**

For every event in every account that a given user has, there is a permission access set assigned to it based on the account type, the owner of the event, and the settings in that given account externally:
- `SharedLocked`: This event belongs to another person's calendar and this user is seeing it as a shared calendar. They have no write access to this event
- `ReadOnly`: The user is an invitee on this event and does not have access to write to it
- `InviteesOnly`: The user may only add invitees to this event. Any other changes will only be made in *their copy* of the event (normally implemented by Google)
- `SharedFull`: This event is on a shared calendar, and the user has full write access to it
- `FullAccess`: The user has full access to this event (normally when they own it)

***
**Date formats**

The ActivityHub platform uses the [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) standard for date and timestamp formatting:
- Dates: `YYYY-MM-DD`
	- Example: "2016-07-31"
- Timestamps: `YYYY-MM-DDTHH:MM:SSZ`
	- Example: "2016-05-17T23:30:00Z" 

***
**Event ID prefixes**

ActivityHub internally prefixes event IDs for use by clients in order to identify the ID as one of a `event` or one of an `event_match` within the database. Although **these are irrelevant to the client**, for documentation's sake, here they are:
- `e-` is used to prefix an ID of an `event` object before being returned
	- Example: "e-zEuLsGqEii"
- `m-` is used to prefix an ID of an `event_match` object before being returned
	- Example: "m-zEzzsGqEra"

***
## Endpoints
### Getting a user's events
**Discussion**

To get all of a user's events within ActivityHub's synced timeframe, simply call the function below. It will return detailed information on each event, but there are some important things to understand:
- Each event returned contains an array of strings, `InAccounts`, which indicates the IDs of the user's accounts that this event was found and matched between by ActivityHub. (If this array is `> 1` element, an event with a similar subject at the same time was found in each of those accounts)
- Each event's `Invitees` map is organized as `{account ID => array of maps of details on invitees in this account}`. See the below example for details
- The `ServiceID` field of an event invitee is currently only used to indicate a record's ID for a Salesforce invitee, and may be a blank string
- The `Permissions` map returns the [tier of access and response status](../documentation/Event%20Management.md#definitions) that the user has for each given account that the event is in (see example below). This should be used to implement client-side restrictions as necessary so the user has a basic understanding of what they can do
- The `MatchedUserID` field of an invitee is non-empty if ActivityHub detected another ActivityHub user with this email address at the time the event was last synced. If one was found, this will be the ID of the user within our database
- If the event is on a shared calendar that the user has selected for viewing, `SharedCalendar` will contain the ID of this calendar within the given account. Otherwise, it will be `null`
- If one of the events is a Salesforce event, `SalesforceData` will be non-`null`, and will contain values for custom fields and other information
- If ActivityHub detected a GoToMeeting within an event (in any of the accounts it's in), the `GoToMeeting` value will be non-`null`, and will contain information on how to dial into the meeting or join it via web
- Some fields may not be updatable in Salesforce (such as `Location`) if they aren't on the user's layout. Information on which fields are enabled (as well as details on custom fields) are specified in the `get_user_info` endpoint

**File**: `/src/manage_events.js`

**Function**: `get_user_events`

**Parameters**
- `token` (**required**) - a valid ActivityHub authentication token for this user
- `start` (**optional**) - start date to return events for in YYYY-MM-DD format. If excluded, defaults to the earliest synced date
- `end` (**optional**) - end date to return events for in YYYY-MM-DD format. If excluded, defaults to the earliest latest date

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
  "Events": [
    {
      "AllDay": false,
      "Color": "#FDA808",
      "InAccounts": [
        "9XkukaVW4r"
      ],
      "Permissions": {
        "9XkukaVW4r": {
          "Access": "InviteesOnly",
          "ResponseStatus": "Organizer"
        }
      },
      "Private": false,
      "SalesforceData": null,
      "SharedCalendar": null,
      "Start": "2016-06-02T21:30:00Z",
      "Location": "",
      "End": "2016-06-02T22:30:00Z",
      "Invitees": {
        "9XkukaVW4r": [
          {
            "Status": "Awaiting response",
            "ID": "ZyXsTvMwXJ",
            "Phone": "",
            "MatchedUserID": "moX4zH2gJp",
            "Name": "John Doe",
            "Email": "test@123.com",
            "ServiceID": ""
          }
        ]
      },
      "GoToMeeting": {
        "MeetingID": "291413522",
        "MeetingURL": "https://www.gotomeeting.com/join/291413522",
        "PhoneNumbers": {
          "United States": "1(646)749-3117,,291413573#"
        }
      }
    },
    {
      "AllDay": false,
      "Color": "#1A8800",
      "End": "2016-07-15T17:00:00Z",
      "ID": "e-2YYlNOgQIQ",
      "InAccounts": [
        "YxjcW6K0l6"
      ],
      "Invitees": {
        "YxjcW6K0l6": []
      },
      "Location": "",
      "Notes": "",
      "Permissions": {
        "YxjcW6K0l6": {
          "Access": "SharedLocked",
          "ResponseStatus": "Organizer"
        }
      },
      "Private": false,
      "SalesforceData": {
        "CustomFields": {
          "Number_XYZ__c": null,
          "Test_1__c": false,
          "Test_2__c": "abc123"
        },
        "IsInvitation": false
      },
      "SharedCalendar": "005o0000001dWRTAA2",
      "Start": "2016-07-15T16:00:00Z",
      "Subject": "Assigned Event",
      "GoToMeeting": null
    }
  ],
  "Timeframe": {
    "End": "2016-07-31",
    "Start": "2016-05-01"
  }
}
```
***
### Creating/updating an event
**Discussion**

ActivityHub's platform provides a very unique way to create and update events in any number of linked accounts (like Google or Salesforce) simultaneously - in one function call. We do this by passing an array of account IDs that the event should be in, as well as an `ID` to indicate if the event is brand new or being updated from an existing one.

There are some important notes on the functionality of this function that are worth mentioning:
- Excluding the `ID` parameter within `event` in your request will result in the creation of new events in all specified accounts
- If creating a new event, at least one item must be included in the `event`'s `InAccounts` array
- When updating an event that was previously matched by ActivityHub, you can **remove** events from a given external account by *excluding* it from the `InAccounts` array in the `event` parameter. This is very powerful functionality that lets you simultaneously create, update, and delete events from distinct accounts simultaneously
- The `Invitees` value within `event` is optional. If it is not included, no invitees will be changed on the event. If it is included, however, all invitees must be specified for each account
- Failing to include an invitee's `ServiceID` parameter will result in failed requests to Salesforce when updating invitees
- Multi-day events are not explicitly supported and may cause errors
- If you are attempting to add a shared calendar's event to another account, the request will be rejected since this is not allowed
- Requests to modify existing events where access is set to `SharedLocked` or `ReadOnly` will be ignored silently (which, by nature, automatically includes events with `SalesforceData.IsInvitation = true`)
- Attempting to write to Salesforce fields that are disabled or do not exist on the user's layout (as specified in the `get_user_info` endpoint) will result in those fields being ignored
- When writing to Salesforce custom fields, any combination of valid fields may be specified (those left out in the request will not be updated) 

**File**: `/src/manage_events.js`

**Function**: `upsert_event`

**Parameters**
- `token` (**required**) - a valid ActivityHub authentication token for this user
- `send_updates` (**optional**) - boolean indicating whether updates should be sent to invitees, defaulting to `false`. Note that some services (Salesforce and Microsoft Live) do not support this (and will be silently ignored) and that Google will only send updates for *future* events. Lastly, Office 365 will always send updates automatically
- `event` (**required**) - map with the following event metadata:
	- `Subject` (**optional**) - string to use as the event's subject. Defaults to "(Unknown subject)"
	- `Notes` (**optional**) - string to use as the event's notes/description
	- `Location` (**optional**) - string to use as the event's notes/description
	- `Start` (**required**) - start date in UTC using YYYY-MM-DDTHH:MM:SSZ format
	- `End` (**required**) - end date in UTC using YYYY-MM-DDTHH:MM:SSZ format
	- `AllDay` (**required**) - boolean for whether this is an all-day event
	- `Private` (**required**) - boolean indicating if this is private. Defaults to `false`, and is *always unsupported* by Office 365, and *can be unsupported* in some Salesforce instances
	- `ID` (**optional**) - ID of the ActivityHub event. If this is not populated, ActivityHub will simply create new events in the account(s) specified
	- `InAccounts` (**required**) - array of IDs for linked accounts that the event should be in. See the above notes for an explanation of how this parameter is used
	- `SalesforceData` (**optional**) - if one of the accounts this event is in belongs to Salesforce, you may update Salesforce-specific data within this map of values {`string` => `value`}. This may include the following, none of which are required:
		- `CustomFields` (**optional**) - map of any custom fields you'd like to update on this event. Any specified fields that are recognized by the system will be updated. They should be formatted in a map of {`Salesforce API name` => `value`}
		- `RelatedTo` (**optional**) - string ID of a non-who ID within the event's Salesforce organization
	- `Invitees` (**optional**) - map of invitees {`account ID` => `array`}. If included, this should have 1 array of invitees at the key of every account ID specified in the `InAccounts` parameter, each with the following information:
		- `Email` (**required**) - email address of the invitee
		- `Name` (**optional**) - name of the invitee
		- `Phone` (**optional**) - phone number of the invitee
		- `ServiceID` (**required for Salesforce**) - used to identify the invitee in an external account. For example, this would be a Salesforce contact's ID (`003xxxxxxxxxxxx`)

**Supports internal override?** 
Yes

**Example request body**
```
{
  "token": "XXXXX",
  "send_updates": true,
  "event": {
    "AllDay": false,
    "InAccounts": [
      "YxjcWrK0l6"
    ],
    "Private": false,
    "Start": "2016-06-14T18:15:00Z",
    "Location": "Test location",
    "Notes": "Example notes",
    "End": "2016-06-14T19:15:00Z",
    "Invitees": {
      "YxjcW6K0l6": [
        {
          "Email": "test@abc.com",
          "Name": "John Doe",
          "ServiceID": "003o000000u9qwg"
        }
      ]
    },
    "ID": "m-KdUH0AjIk3",
    "Subject": "Example title",
    "SalesforceData": {
      "CustomFields": {
        "Test_2__c": "My text here"
      },
      "RelatedTo": "006o000000IsIZFAA3"
    }
  }
}
```

**Example response**
```
{
    "Events": 2,
    "Invitees": 2
}
```
***
### Deleting an event
**Discussion**

Continuing with ActivityHub's design of linking "matched" events together, calling this endpoint to delete an event will delete that event from **all** accounts in which it is stored. For example, if an event with ID `m-zEuLsGqErv` is referenced in Google and Salesforce, calling this endpoint will delete the event from both places. If you want to remove the event from only *one* account, use `upsert_event` (above) instead, and specify the accounts there.

**File**: `/src/manage_events.js`

**Function**: `delete_event`

**Parameters**
- `token` (**required**) - a valid ActivityHub authentication token for this user
- `event_id` (**required**) - the ID of the event to be deleted
- `send_updates` (**optional**) - boolean indicating whether or not to send updates to invitees. Some services (Salesforce and Microsoft Live) do not support this, and other services (Office 365) automatically send updates

**Supports internal override?** 
Yes

**Example request body**
```
{
  "token": "XXXXX",
  "send_updates": true,
  "event_id": "m-rtNTNAysjD"
}
```

**Example response**
```
{
    "Deleted": 2
}
```
***
### Responding to an invitation
**Discussion**

If a user is invited to an event, they can respond to that invitation using this endpoint. Note that calling this endpoint on a matched event between multiple accounts will set the response status *for each one where the user is not the organizer*.

**File**: `/src/manage_events.js`

**Function**: `respond_to_invite`

**Parameters**
- `token` (**required**) - a valid ActivityHub authentication token for this user
- `event_id` (**required**) - the ID of the event
- `new_status` (**required**) - string representing one of the allowed [response statuses](../documentation/Event%20Management.md#definitions), *excluding* `Organizer` and `Awaiting response`

**Supports internal override?** 
Yes

**Example request body**
```
{
  "token": "XXXXX",
  "new_status": "Going",
  "event_id": "e-376gwQBNrS"
}
```

**Example response**
```
{
    "Updated": 1
}
```
***
# GoToMeeting Integration

## Table of contents
#### Endpoints
- [Creating a meeting](../documentation/GoToMeeting.md#creating-a-meeting)

#### Internal functions
- [`goToMeetingInfoFromNotes()`](../documentation/GoToMeeting.md#gotomeetinginfofromnotes)

***
## Endpoints
### Creating a meeting
**Discussion**

Creating a meeting with GoToMeeting will allow others to join a meeting with the creator via web or phone call. This function creates a meeting using a user's linked GoToMeeting account.

**File**: `/src/gotomeeting.js`

**Function**: `create_gotomeeting`

**Parameters**
- `token` (**required**) - a valid ActivityHub authentication token for this user
- `account_id` (**required**) - ID of a user's GoToMeeting `account_linked` object
- `start` (**required**) - datetime of the meeting's start in YYYY-MM-DDTHH:MM:SS format
- `end` (**required**) - datetime of the meeting's end in YYYY-MM-DDTHH:MM:SS format
- `subject` (**optional**) - name of the meeting

**Supports internal override?** 
Yes

**Example request body**
```
{
  "token": "XXXXX",
  "subject": "My meeting",
  "account_id": "9XkukaVW4r",
  "end": "2016-06-14T20:30:00Z",
  "start": "2016-06-14T19:30:00Z"
}
```

**Example response**
```
{
  "URL": "https://www.gotomeeting.com/join/329975109",
  "MeetingID": 329975109,
  "Description": "Please join my meeting.\nhttps://www.gotomeeting.com/join/329975109\n\nUS: +1 (872) 240-3412\nAccess Code: 329-975-109"
}
```
***
## Internal functions
### goToMeetingInfoFromNotes()
**Discussion**

This function can be used to determine if an event's notes (or just a string in general) contains information on a GoToMeeting. This uses smart detection based on known GoToMeeting formatting. If information is found, it formats it before returning (including a handy feature for making the phone numbers include the access code).

**File**: `/src/salesforce_objects.js`

**Function**: `goToMeetingInfoFromNotes(eventNotes)`

**Arguments**
- `eventNotes` - a string (generally notes from a calendar event) that could potentially contain GoToMeeting conference call information

**Returns**: The following data structure.
```
{
  "MeetingURL": "https://www.gotomeeting.com/join/293386789",
  "MeetingID": "293386789",
  "PhoneNumbers": {
    "France (toll-free)": "0805541052,,293386789#",
    "United States (toll-free)": "18773092070,,293386789#",
    "Switzerland (toll-free)": "0800000452,,293386789#",
    "Japan (toll-free)": "0120242200,,293386789#",
    "India (toll-free)": "0008001008227,,293386789#",
    "Germany (toll-free)": "08007235274,,293386789#",
    "United Kingdom (toll-free)": "08000314727,,293386789#",
    "Singapore (toll-free)": "8001013000,,293386789#",
    "United States": "1(571)317-3116,,293386789#",
    "Australia (toll-free)": "1800191358,,293386789#",
    "Canada (toll-free)": "18777773281,,293386789#",
    "Ireland (toll-free)": "1800818263,,293386789#"
  }
}
```
***


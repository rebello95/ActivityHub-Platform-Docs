# GoToMeeting Integration

## Table of contents
#### Endpoints
- [Creating a meeting](../documentation/GoToMeeting.md#creating-a-meeting)

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

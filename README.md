## Get started
Welcome to the ActivityHub Platform! Below is a quick introduction of how to connect to and use the API.

### 1. Obtain credentials
In order to use ActivityHub as a developer, you'll need to [obtain a keyset](../master/documentation/Authentication.md#obtain-client-keys) for your application.

### 2. Make your first API call
As an example API call, try hitting the [`get_available_acct_types`](../master/documentation/User%20Management.md#get-available-account-types) endpoint. You'll need to use the following information for **all** requests made to our servers:

**URL:** `https://api.activityhub.io/parse/functions/<FUNCTION>`

**HTTP verb:** `POST` 

*Unless otherwise specified, all ActivityHub API calls should be `POST` requests.*

**Headers:**

`X-Parse-Application-Id`: "ACTIVITYHUB-PLATFORM-API"

`Content-Type`: "application/json"

### 3. Make authenticated API calls
Now that you know how to make an API call to ActivityHub, the next step is to authenticate a user and start making real requests! You've already completed the first few steps, and the [authentication documentation](../master/documentation/Authentication.md#register-and-login) is a great place to go from here.

***
## Table of contents
Endpoints are grouped into the following categories:

1. [Authentication](../master/documentation/Authentication.md)
	- Provides an explanation for how to authenticate your app and your users with the ActivityHub service
2. [User management](../master/documentation/User%20Management.md)
	- Explains things like getting a user's profile, updating their account information, and handling licenses
3. [Event management](../master/documentation/Event%20Management.md)
	- Provides details on getting, creating, updating, responding to, and deleting events via ActivityHub
4. [Task management](../master/documentation/Task%20Management.md)
	- Provides details on getting, creating, updating, and deleting tasks in Salesforce via ActivityHub
5. [Contact management](../master/documentation/Contacts.md)
	- ActivityHub supports fetching a user's contacts from any of their accounts they've added to our system, as well as creating new ones
6. [Salesforce integration](../master/documentation/Salesforce.md)
	- Documentation on using Salesforce-specific ActivityHub functionality
7. [GoToMeeting integration](../master/documentation/GoToMeeting.md)
	- Creating GoToMeeting data, as well as detecting it in an event
8. [Phones and SMS](../master/documentation/Phones%20and%20SMS.md)
	- ActivityHub has some pretty neat capabilities via SMS, documented here
9. [Bots!](../master/documentation/Bots.md)
	- Information on how to utilize our growing bot functionality
10. [Surveys](../master/documentation/Surveys.md)
	- Getting and responding to surveys
11. [Utilities](../master/documentation/Utilities.md)
	- Various helper functions pertaining to general data and location services.
12. [Developer console](../master/documentation/Developer.md)
	- Endpoints used to run our developer console
13. [Event state machine & task syncing](../master/documentation/State%20Machine.md)
	- Details on how ActivityHub syncs events and tasks between all users' calendars, our database, and Salesforce
14. [Data tables](../master/documentation/Tables.md)
	- Brief summaries of each table in our database

***
## Response error handling
### Overview
When ActivityHub returns an error, the format will always include the following information:
- `StatusCode` - number indicating the type of error (possible values listed below)
- `InternalError` - boolean, `true` if this was an error within ActivityHub. An example an a non-internal error would be Google returning a failure message that ActivityHub can't auto-resolve
- `ErrorDesc` - string providing basic information on the problem
- `ErrorDetails` - string providing more technical information or debug details on the error. This value may be `null`

### Error codes
- `400` - Malformed request to ActivityHub
- `401` - Invalid ActivityHub access token
- `403` - Unauthorized to access this endpoint within ActivityHub
- `404` - Resource not found (this can include failing to find data due to insufficient permissions)
- `500` - General internal error
- `502` - General external error (i.e., an error response from Google)

### Example error response
```
{
  "StatusCode": 500,
  "InternalError": true,
  "ErrorDesc": "An internal error has occurred.",
  "ErrorDetails": "If non-null, this will contain technical details."
}
```

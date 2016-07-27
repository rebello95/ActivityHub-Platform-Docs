# Documentation
### Notes before you get started
- Most functions documented are accessible as an authenticated ActivityHub user by passing in a `token` value in the body of your REST request
- Many of the functions allow internal overrides (so the system can easily call itself). These functions will have this feature denoted in their documentation. To internally override a function, the following values must be passed in the body. *Do not pass in both a `token` and `key` - `key` will take precedence, and `token` will be ignored*:
	- `key`: internal key to authorize the request as internal
	- `user_id`: the ID of the user that this function is being called on behalf of
- **Unless otherwise specified, all API calls are `POST` requests**
- You may want to look at the [error handling](#universal-error-handling) documentation near the bottom of this ReadMe file. *Every* API endpoint supports one universalized error format, and it will be very helpful for you to know what it looks like

***
### Table of contents
Endpoints are grouped into categories. See this table for links to each ReadMe.

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
9. Bots!
	- Not publicly available
10. [Surveys](../master/documentation/Surveys.md)
	- Getting and responding to surveys
11. [Utilities](../master/documentation/Utilities.md)
	- Various helper functions pertaining to general data and location services.
12. Event state machine & task syncing
	- Not publicly available
13. Data tables
	- Not publicly available

***
### Universal error handling
##### Overview
When ActivityHub returns an error, the format will always include the following information:
- `StatusCode` - number indicating the type of error (possible values listed below)
- `InternalError` - boolean, `true` if this was an error within ActivityHub Cloud. An example an a non-internal error would be Google returning a failure message that ActivityHub can't auto-resolve
- `ErrorDesc` - string providing basic information on the problem
- `ErrorDetails` - string providing more technical information or debug details on the error. This value may be `null`

##### Error codes
- `400` - Malformed request to ActivityHub Cloud
- `403` - Unauthorized to access this endpoint within ActivityHub Cloud
- `404` - Resource not found (this can include failing to find data due to insufficient permissions)
- `500` - General internal error
- `502` - General external error (i.e., an error response from Google)

##### Example response
```
{
  "StatusCode": 500,
  "InternalError": true,
  "ErrorDesc": "An internal error has occurred.",
  "ErrorDetails": "If non-null, this will contain technical details."
}
```
***
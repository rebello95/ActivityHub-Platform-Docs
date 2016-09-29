# Developer console

## Table of contents
#### Endpoints
- [Join the developer program](#join-the-developer-program)
- [Get developer apps](#get-developer-apps)
- [Create an app](#create-an-app)
- [Update an app](#update-an-app)
- [Get app usage](#get-app-usage)

***
##Endpoints
### Join the developer program
**Discussion**

In order to access any other developer endpoints, a given user must be registered as a developer. Calling this endpoint as an authenticated user will do just that.

**File**: `/src/external_developers`

**Function**: `dev_join_program`

**Parameters**
- `token` (**required**) - a valid ActivityHub authentication token for this user

**Supports internal override?** 
No

**Example request body**
```
{
  "token": "XXXXX"
}
```

**Example response**
```
{
  "Success": true
}
```
***
### Get developer apps
**Discussion**

This endpoint will return information on a given developer's applications. A developer only has access to their own apps.

**File**: `/src/external_developers`

**Function**: `dev_get_apps`

**Parameters**
- `token` (**required**) - a valid ActivityHub authentication token for this user

**Supports internal override?** 
No

**Example request body**
```
{
  "token": "XXXXX"
}
```

**Example response**
```
{
  "Applications": [
    {
      "ApplicationID": "dkj4Xk9dh2",
      "RedirectURIs": [
        "myurlscheme://authenticated",
        "https://api.mysite.com/authenticated"
      ],
      "ClientID": "XXXXX",
      "ClientSecret": "YYYYY",
      "Name": "My awesome app",
      "VersionRequired": null,
      "Active": true
    }
  ]
}
```
***
### Create an app
**Discussion**

To create an app for use with the platform, this endpoint should be called by an authenticated developer.
- The `redirect_uris` parameter is a set of one or more valid URLs that can be used for authenticating users
- `version_required` is generally not used, but it can be useful for forcing your users to update to a specific version of your app

**File**: `/src/external_developers`

**Function**: `dev_create_app`

**Parameters**
- `token` (**required**) - a valid ActivityHub authentication token for this user
- `name` (**required**) - the name of the application
- `redirect_uris` (**required**) - must contain at least one redirect URL for authentication
- `version_required` (**required**) - number to indicate to the client what version of their app is required to use it

**Supports internal override?** 
No

**Example request body**
```
{
  "token": "XXXXX",
  "redirect_uris": [
    "myurlscheme://authenticated",
    "https://api.mysite.com/authenticated"
  ],
  "version_required": 8,
  "name": "My awesome app"
}
```

**Example response**
```
{
  "Application": {
    "ApplicationID": "dkj4Xk9dh2",
    "RedirectURIs": [
      "myurlscheme://authenticated",
      "https://api.mysite.com/authenticated"
    ],
    "ClientID": "XXXXX",
    "ClientSecret": "YYYYY",
    "Name": "My awesome app",
    "VersionRequired": null,
    "Active": true
  }
}
```
***
### Update an app
**Discussion**

This endpoint allows a developer to modify the information on their application (normally used for changing/adding redirect URLs).

**File**: `/src/external_developers`

**Function**: `dev_update_app`

**Parameters**
- `token` (**required**) - a valid ActivityHub authentication token for this user
- `app_id` (**required**) - ID of the application being modified
- `name` (**optional**) - name of the application, unchanged if not specified
- `redirect_uris` (**optional**) - array of redirect URLs for authentication (specifying this value will replace all existing URIs)
- `version_required` (**optional**) - number to indicate to the client what version of their app is required to use it, unchanged if not specified

**Supports internal override?** 
No

**Example request body**
```
{
  "token": "XXXXX",
  "app_id": "YYYYY",
  "redirect_uris": [
    "myurlscheme://authenticated",
    "https://api.mysite.com/authenticated"
  ],
  "version_required": 8,
  "name": "My awesome app"
}
```

**Example response**
```
{
  "Application": {
    "ApplicationID": "dkj4Xk9dh2",
    "RedirectURIs": [
      "myurlscheme://authenticated",
      "https://api.mysite.com/authenticated"
    ],
    "ClientID": "XXXXX",
    "ClientSecret": "YYYYY",
    "Name": "My awesome app",
    "VersionRequired": null,
    "Active": true
  }
}
```
***
### Get app usage
**Discussion**

This endpoint will return API consumption information for a given app owned by the developer. *This endpoint will return data for the past 90 days.*

**File**: `/src/external_developers`

**Function**: `dev_get_usage`

**Parameters**
- `token` (**required**) - a valid ActivityHub authentication token for this user
- `app_id` (**required**) - ID of the application to get usage for

**Supports internal override?** 
No

**Example request body**
```
{
  "token": "XXXXX",
  "app_id": "YYYYY"
}
```

**Example response**
```
{
  "Usage": [
    {
      "Date": "2016-09-26",
      "APICount": 1502
    },
    {
      "Date": "2016-09-27",
      "APICount": 1452
    },
    {
      "Date": "2016-09-28",
      "APICount": 975
    }
  ]
}
```
***
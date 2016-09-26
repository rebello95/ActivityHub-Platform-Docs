# User management

## Table of contents
#### Endpoints
- [Get available account types](#get-available-account-types)
- [Get a user's list of accounts](#get-a-users-list-of-accounts)
- [Get a user's full profile details](#get-a-users-full-profile-details)
- [Remove a linked account](#remove-a-linked-account)
- [Update a linked account](#update-a-linked-account)
- [Get all lock-able features](#get-all-lock-able-features)
- [Get available licenses](#get-available-licenses)
- [Get a license suggestion for a user](#get-a-license-suggestion-for-a-user)
- [Upgrade a user after completing a purchase](#upgrade-a-user-after-completing-a-purchase)
- [Refresh the access token for a linked account](#refresh-the-access-token-for-a-linked-account)
- [Update a user's list view filters](#update-a-users-list-view-filters)
- [Get event/task templates for a user](#get-eventtask-templates-for-a-user)
- [Create/update an event or task template](#createupdate-an-event-or-task-template)
- [Delete event/task templates for a user](#delete-eventtask-templates-for-a-user)
- [Update a user's preferences](#update-a-users-preferences)
- [Get available shared calendars for an account](#get-available-shared-calendars-for-an-account)

#### beforeSave triggers
- [_User](#_user)
- [link_account_session](#link_account_session)
- [access_grant](#access_grant)
- [user_prefs](#user_prefs)
- [account_linked](#account_linked)
- [enterprise_prefs](#enterprise_prefs)
- [event_template](#event_template)
- [task_template](#task_template)

#### afterSave triggers
- [account_linked](#account_linked-1)
- [license_purchase](#license_purchase)

#### afterDelete triggers
- [_User](#_user)
- [account_linked](#account_linked-2)

***
## Endpoints
### Get available account types
**Discussion**

The ActivityHub platform is designed to allow for *dynamically adding support for new accounts* on a given client without pushing any updates to the client's code. In order to take advantage of this, you'll need to grab details on available accounts from this endpoint.
- `EventsSupported` should only be `false` for odd accounts like GoToMeeting. We recommend **not** supporting dynamic adding of accounts where this value is set to `false`, as there is little likelihood that their uses will be universal
- `SandboxSupported` will be `true` for accounts like Salesforce that support production and sandbox instances (and different login pages for both)
- `InviteesSupported` will be `false` for accounts that don't support fetching/posting invitees and contacts. Microsoft Live would be an example of this
- `SharedCalendarsSupported` will be `false` for accounts that don't support viewing other peoples' shared calendars (Office 365, for example)
- `AllowsSignup` is `true` for accounts that allow for sign-ups. For example, a user is allowed to sign up with a Google account, but not with a GoToMeeting account
- `OnePerUser` indicates if the user is limited to adding 1 of these accounts to their profile. As an example, users can only have 1 Salesforce account associated with their profile (they can, however, sign out and sign in with the account in order to create a new profile)

**File**: `/src/manage_users.js`

**Function**: `get_available_acct_types`

**Parameters**
- `client_id` (**required**) - the client ID of the app this request is being made on behalf of

**Supports internal override?**
Yes

**Example request body**
```
{
	"client_id": "XXXXX"
}
```

**Example response**
```
{
  "AccountTypes": [
    {
      "ID": "8krrGradP1",
      "InviteesSupported": true,
      "AllowsSignup": true,
      "OnePerUser": false,
      "LogoMedium": {
        "URL": "http://example.domain.com/Office365-dbe2.png"
      },
      "LogoSquare": {
        "URL": "http://example.domain.com/Office365-sdfe.png"
      },
      "EventsSupported": true,
      "SandboxSupported": false,
      "SharedCalendarsSupported": false
      "Name": "Office 365"
    }
  ]
}
```
***
###Get a user's list of accounts
**Discussion**

A given user will have at least 1 account registered to their user record (i.e., a Google or Salesforce account). Users can (and normally do) have several accounts associated with their user profile. To fetch the list of accounts that a user has linked, call this endpoint. Some important notes:
- `Sync.NeedsAuth` will be `true` if the user needs to re-authenticate with this service provider. Syncing with this account will remain inactive until this is done
- `Sync.SyncEnabled` will be `true` if events created in this account should be automatically pushed to the user's other linked account(s)
- `Sync.SyncPrivateEvents` will be `true` if private events created in this account should be automatically pushed to the user's other linked account(s)
- If the account *is* a Salesforce account, the `SalesforceSettings` value will be **non-null**, and the `NonSalesforceSettings` value will be `null`. The inverse is also true for non-Salesforce accounts
- The `IsEventDefault` value is `true` if the user has marked this account as one that should have events created in it by default when adding them
- The Salesforce `ManyWhoOn` value will be `true` if the user's Salesforce organization has "Shared activities" enabled
- The non-Salesforce `AddSalesforceInvitees` value is `true` if the user expects a given client to automatically copy Salesforce invitees to this account's event (if it exists) while they are selecting them
- `Type` refers to the ID of the ActivityHub account type (available in the `get_available_acct_types` endpoint)
- Colors returned are standard hex values

**File**: `/src/manage_users.js`

**Function**: `get_user_accounts`

**Parameters**
- `token` (**required**) - a valid ActivityHub authentication token for this user

**Supports internal override?**
Yes

**Example request body**
```
{
	"token": "XXXXX"
}
```

**Example response**
```
{
  "Accounts": [
    {
      "SalesforceSettings": {
        "UserID": "005o000000182qeAAA",
        "Domain": "https://na17.salesforce.com/",
        "ManyWhoOn": true,
        "Username": "test@test.com"
      },
      "SharedCalendars": [
        {
          "ID": "005d00000018r3xeAAA",
          "Color": "#00DEAF"
        }
      ],
      "Details": {
        "ID": "T3amaMi7UO",
        "Color": "#2D69B4",
        "Email": "johndoe@email.com",
        "IsEventDefault": true,
        "Nickname": "My Salesforce",
        "Type": "ilqtUQT36y"
      },
      "Sync": {
        "NeedsAuth": false,
        "SyncEnabled": true,
        "SyncPrivateEvents": false
      },
      "NonSalesforceSettings": {
        "AddSalesforceInvitees": true
      }
    }
  ]
}
```
***
###Get a user's full profile details
**Discussion**

Want to know all the details on a user's profile, including their license information and linked accounts? Call this function.
- `License.Expires` will be `null` if no expiration date is set

**File**: `/src/manage_users.js`

**Function**: `get_user_info`

**Parameters**
- `token` (**required**) - a valid ActivityHub authentication token for this user

**Example request body**
```
{
	"token": "XXXXX"
}
```

**Supports internal override?**
Yes

**Example response**
```
{
  "LinkedAccounts": "<IDENTICAL TO `Accounts` IN THE `get_user_accounts` call>",
  "License": {
    "Features": [
      {
        "ID": "XVzOrzmcQ3",
        "Name": "Salesforce Accounts"
      },
      {
        "ID": "Nx0t1e37nd",
        "Name": "GoToMeeting Accounts"
      }
    ],
    "Description": "Wow this is such a good license",
    "Name": "Example license",
    "LicenseID": "moIPrTfrWG",
    "Tier": 2,
    "Expires": "2017-01-01"
  },
  "Details": {
    "JobTitle": "",
    "HasBetaAccess": false,
    "Username": "john.doe-287",
    "ID": "moIPr2s21s",
    "Phone": "",
    "VerifiedPhones": [
      "+12223334444"
    ],
    "FirstName": "John",
    "LastName": "Doe",
    "Company": "American Corp"
  },
  "Preferences": {
    "DisplayFilters": {
      "Show": [
        "Events",
        "Tasks"
      ],
      "TaskPriorities": [
        "High"
      ],
      "SearchType": "Subject",
      "TaskStatuses": [
        "Not Started"
      ]
    },
    "EventReminderMinutes": 15,
    "FollowUpSuggestions": false,
    "SalesforceOnly": {
      "AutoFindInvitees": false,
      "GlyphsOn": false,
      "UseSubjectPicklist": false,
      "LayoutData": {
        "Contact": {
          "PhoneFields": [
            {
              "Display": "Business Phone",
              "SFAPIName": "Phone"
            },
            {
              "Display": "Mobile Phone",
              "SFAPIName": "MobilePhone"
            }
          ]
        },
        "Event": {
          "AllDayExists": true,
          "LayoutCustomFields": [
            {
              "DecimalPlaces": null,
              "DisplayName": "Test Field #1",
              "Length": 0,
              "PicklistValues": null,
              "Required": false,
              "SFAPIName": "Test_1__c",
              "Type": "boolean"
            },
            {
              "DecimalPlaces": 4,
              "DisplayName": "Number XYZ",
              "Length": 18,
              "PicklistValues": null,
              "Required": false,
              "SFAPIName": "Number_XYZ__c",
              "Type": "double"
            }
          ],
          "LocationExists": true,
          "PrivateExists": false,
          "SubjectPicklistValues": [
            "Call",
            "Meeting"
          ]
        },
        "OpportunityContactRole": {
          "RoleValues": [
            "Business User",
            "Decision Maker",
            "Other"
          ]
        },
        "Task": {
          "LayoutCustomFields": [
            {
              "DecimalPlaces": null,
              "DisplayName": "Picklist Test",
              "Length": 255,
              "PicklistValues": [
                "One",
                "Two",
                "Three"
              ],
              "Required": false,
              "SFAPIName": "Picklist_Test__c",
              "Type": "picklist"
            }
          ],
          "SubjectPicklistValues": [
            "Call",
            "Meeting"
          ],
          "PriorityValues": [
            "High",
            "Normal",
            "Low"
          ],
          "StatusValues": [
            "Not Started",
            "In Progress",
            "Completed",
            "Waiting on someone else",
            "Deferred"
          ]
        }
      },
      "MainFields": {
        "Account": {
          "PrimaryAPIName": "Name",
          "SecondaryAPIName": "Example_Field__c",
          "PrimaryType": "string",
          "SecondaryType": "boolean"
        }
      },
      "ObjectSelections": [
        {
          "FieldsShowing": [
            {
              "Name": "Type",
              "SFAPIName": "Type",
              "SFType": "picklist"
            }
          ],
          "Name": "Accounts",
          "Prefix": "001",
          "SFAPIName": "Account"
        },
        {
          "FieldsShowing": [
            {
              "Name": "Title",
              "SFAPIName": "Title",
              "SFType": "string"
            }
          ],
          "Name": "Contacts",
          "Prefix": "003",
          "SFAPIName": "Contact"
        }
      ],
      "RecordTypes": {
        "Event": {
          "Name": "Master",
          "TypeID": "012000000000000AAA"
        },
        "Task": null
      },
      "ShowInvitations": false
    }
  }
}
```
***
### Remove a linked account
**Discussion**

Users are allowed to remove external accounts they've linked to their ActivityHub account *as long as they always have at least 1 remaining*. Removing the only account a user has associated will result in an error. To remove an account, call this function.

**File**: `/src/manage_users.js`

**Function**: `remove_user_account`

**Parameters**
- `token` (**required**) - a valid ActivityHub authentication token for this user
- `account_id` (**required**) - ID of the `account_linked` that the user would like to remove

**Supports internal override?**
No

**Example request body**
```
{
	"token": "XXXXX",
	"account_id": "BkIinpGMYz"
}
```

**Example response**
```
{
	"Deleted": 1
}
```
***
### Update a linked account
**Discussion**

Users can update certain values on a given linked account they have with ActivityHub. To push a modification to ActivityHub, call this function.

**File**: `/src/manage_users.js`

**Function**: `update_user_account`

**Parameters**
- `token` (**required**) - a valid ActivityHub authentication token for this user
- `account_id` (**required**) - ID of the `account_linked` that the user would like to update
- `details` (**optional**) - a map of the following values:
	- `Color` (**optional**) - hex string representation of this user's account color
	- `IsEventDefault` (**optional**) - boolean indicating if this user's account should have events created in it by default
	- `Nickname` (**optional**) - a nickname that a user provides for this account
- `settings` (**optional**) - a map of the following values:
	- `AddSalesforceInvitees` (**optional**) - boolean indicating if this account (non-Salesforce only) should have Salesforce invitees automatically added here. If called on a Salesforce account, this value will be **ignored**
- `sync` (**optional**) - a map of the following values:
	- `SyncEnabled` (**optional**) - whether events created in this account should be copied to other linked accounts. If `false`, `SyncPrivateEvents` will automatically be turned off
	- `SyncPrivateEvents` (**optional**) - whether private events created in this account should be copied to other linked accounts. If `true`, `SyncEnabled` will be set to `true` automatically
- `shared_calendars` (**optional**) - an array of maps with `Color` and `ID` properties. If this value is included, any calendars that are not included will be removed

**Supports internal override?**
No

**Example request body**
```
{
  "token": "XXXXX",
  "account_id": "DmChvR4VZa",
  "details": {
    "Color": "#2D69B4",
    "IsEventDefault": true,
    "Nickname": "My Gmail"
  },
  "settings": {
    "AddSalesforceInvitees": true
  },
  "sync": {
    "SyncEnabled": true,
    "SyncPrivateEvents": false
  },
  "shared_calendars": [
    {
      "ID": "jane.doe@email.com",
      "Color": "#3D87B4"
    }
  ]
}
```

**Example response**
```
{
	"Updated": 1
}
```
***
### Get all lock-able features
**Discussion**

Being the enterprise platform that it is, ActivityHub has the ability to restrict certain features based on a given user's license type. This function returns a list of all features that *can be locked*. This should be compared against a user's list of features that they have access to in order to determine which ones to disable on the client side.

**File**: `/src/manage_users.js`

**Function**: `get_lockable_features`

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
  "Features": [
    {
      "ID": "PrIudWePIe",
      "Name": "Opportunity Suggestions"
    },
    {
      "ID": "SFhO9000fe",
      "Name": "Real-Time Support"
    }
  ]
}
```
***
### Get available licenses
**Discussion**

As ActivityHub supports different tiers of licenses, this function will return a list of details on each.

**File**: `/src/manage_users.js`

**Function**: `get_available_licenses`

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
  "Licenses": [
    {
      "Purchases_Android": {
        "12Month": "com.android.xyz.license2",
        "1Month": "com.android.xyz.license1"
      },
      "Description": "This is one of our enterprise licenses.",
      "Name": "Enterprise License XYZ",
      "Purchases_iOS": {
        "12Month": "com.ios.xyz.license2",
        "1Month": "com.ios.xyz.license1"
      },
      "Features": [
        {
          "ID": "XVzOR72cQ4",
          "Name": "Salesforce Accounts"
        },
        {
          "ID": "RI48iTZ3mR",
          "Name": "Task Templates"
        }
      ],
      "Tier": 2
    }
  ]
}
```
***
### Get a license suggestion for a user
**Discussion**

The data returned by the `get_available_licenses` endpoint may be a bit much to show to a user. Instead of manually parsing and displaying it all, you can call this function afterwards and just get the license suggested by ActivityHub that the user should purchase. You can also include a `feature_id`, which will ensure that the license returned includes access to that feature.

**File**: `/src/manage_users.js`

**Function**: `get_license_suggestion`

**Parameters**
- `token` (**required**) - a valid ActivityHub authentication token for this user
- `feature_id` (**optional**) - the ID of a `license_feature` that the suggested license must include

**Supports internal override?**
No

**Example request body**
```
{
	"token": "XXXXX",
	"feature_id": "GLAZYCWKn4"
}
```

**Example response**
```
{
  "License": {
    "ID": "ipihZr7NSN",
    "Description": "This license is very good.",
    "Name": "Example License",
    "Purchases_Android": {
      "12Month": "com.android.xyz.test2",
      "1Month": "com.android.xyz.test1"
    },
    "Purchases_iOS": {
      "12Month": "com.ios.xyz.test2",
      "1Month": "com.ios.xyz.test2"
    },
    "Features": [
      {
        "ID": "XVzOR72cQ4",
        "Name": "Salesforce Accounts"
      },
      {
        "ID": "RI48iTZ3mR",
        "Name": "Task Templates"
      }
    ]
  }
}
```
***
### Upgrade a user after completing a purchase
**Discussion**

*For internal use only.* This will perform the process of upgrading/extending a user's license after they purchase an upgrade.

**File**: `/src/manage_users.js`

**Function**: `upgrade_after_purchase`

**Parameters**
- `token` (**required**) - a valid ActivityHub authentication token for this user
- `client_id` (**required**) - the client ID of the app this request is being made on behalf of, must have assigned permissions for purchases
- `iap_id` (**optional**) - ID of the in-app purchase in the relevant store
- `trans_id` (**required**) - ID of the transaction/receipt from the relevant store
- `license_id` (**required**) - ID of the license that the user purchased

**Supports internal override?**
No

**Example request body**
```
{
  "token": "XXXXX",
  "client_id": "YYYYY",
  "iap_id": "com.ios.xyz.license1",
  "trans_id": "81726y3ijskfiuy897123",
  "license_id": "kEdB9J3Hmb"
}
```

**Example response**
```
{
  "Successful": true,
  "Expires": "2016-09-21",
  "LicenseID": "kEdB9J3Hmb"
}
```
***
### Refresh the access token for a linked account
**Discussion**

*For internal use only.* This will handle communicating with an external service to refresh the access token of a given `account_linked`. This endpoint can be called for any account (excluding GoToMeeting, which [does not support](https://developer.citrixonline.com/content/getting-new-access-token-using-refresh-token) refreshing tokens), to update the `account_linked`'s token in the ActivityHub database, and return the new token for immediate use.

**File**: `/src/manage_users.js`

**Function**: `refresh_access_token`

**Parameters**
- `key` (**required**) - internal key for authorizing this endpoint
- `account_id` (**required**) - ID of a user's `account_linked` object to refresh the token for

**Supports internal override?**
Yes

**Example request body**
```
{
	"key": "XXXXX",
	"account_id": "lkd8xS39S7"
}
```

**Example response**
```
{
	"Token": "XXXXX"
}
```
#### Internal function

In addition to the function above, there is an internally available function which can be used to easily refresh a token and retry another Cloud Code function. This function will automatically set the `ignore_retry` parameter of the function being retried to `true` for use avoiding infinite loops upon failure.

`refreshTokenAndRetry(endpoint, params, accountID, config)`

**Arguments**
- `endpoint` - name of the function to retry after refreshing the token
- `params` - parameters to pass in to the endpoint when retrying
- `accountID` - ID of the `account_linked` object to refresh the token for
- `config` - fetched `Parse.Config` object

**Returns**:  A promise that is resolved upon retrying of the function or failure of the token refresh
***
### Update a user's list view filters
**Discussion**

This endpoint takes filter parameters and updates them in the `user_prefs` object. If there are no settings, it creates them. Validated filters are returned upon a successful call.

**File**: `/src/manage_users.js`

**Function**: `update_list_filters`

**Parameters**
- `token` (**required**) - a valid ActivityHub authentication token for this user
- `filters` (**required**) - map of the following values:
	- `Show` (**optional**) - an array with the potential values "Events" and "Tasks"
	- `SearchType` (**optional**) - either "Subject" or "RelatedTo". **Specifying "RelatedTo" will force "Show" to be `["Tasks"]`**
	- `TaskPriorities` (**optional**) - array of Salesforce task priority strings
	- `TaskStatuses` (**optional**) - array of Salesforce task status strings

**Supports internal override?**
No

**Example request body**
```
{
  "token": "XXXXX",
  "filters": {
      "Show": ["Events", "Tasks"],
      "SearchType": "Subject",
      "TaskPriorities": ["Low", "High"],
      "TaskStatuses": ["Not Started"]
  }
}
```

**Example response**
```
{
  "DisplayFilters": {
    "SearchType": "Subject",
    "Show": [
      "Events",
      "Tasks"
    ],
    "TaskPriorities": [
      "Low",
      "High"
    ],
    "TaskStatuses": [
      "Not Started"
    ]
  }
}
```
***
### Get event/task templates for a user
**Discussion**

ActivityHub has templates that a user can create, modify, and use to act as a set of defaults when creating a new event or task. This endpoint fetches information on all of a given user's templates so that they can later be shown to the user.

**File**: `/src/manage_users.js`

**Function**: `get_user_templates`

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
  "EventTemplates": [
    {
      "Details": {
        "ID": "nor2MaXVOu",
        "Nickname": "Standard Meeting"
      },
      "Presets": {
        "AllDay": false,
        "Location": "Starbucks",
        "Notes": "Bring charger",
        "Private": false,
        "SalesforceData": {
          "CustomFields": {
            "My_Field__c": 14.99
          },
          "RelatedTo": null
        },
        "Subject": "Coffee Discussion"
      }
    }
  ],
  "TaskTemplates": [
    {
      "Details": {
        "ID": "wmIzp8fGTO",
        "Nickname": "Template #1"
      },
      "Presets": {
        "Notes": "Example notes",
        "Priority": "Normal",
        "SalesforceData": {
          "CustomFields": {
            "My_Field2__c": "#15"
          },
          "RelatedTo": "a0X3200000B03EW"
        },
        "Status": "In Progress",
        "Subject": "Successful Meeting"
      }
    }
  ]
}
```
***
### Create/update an event or task template
**Discussion**

To create or update a template for a user's events or tasks, call this endpoint.
- To **create** a new template, exclude the `details.ID` value. If you want to **update** an existing template, be sure to include this value
- Event and task templates differ a little bit, see the required parameters below for details

**File**: `/src/manage_users.js`

**Function**: `upsert_template`

**Parameters**
- `token` (**required**) - a valid ActivityHub authentication token for this user
- `type` (**required**) - either "event" or "task"
- `nickname` (**required if creating**) - a nickname for the template
- `details` (**required**) - a map with the following information:
	- `ID` (**optional**) - ID of the template you're updating (a new template will be created if not specified)
	- `Subject` (**optional**) - string
	- `Notes` (**optional**) - string
	- `SalesforceData` (**optional**) -  you may update Salesforce-specific data within this map of values {`string` => `value`}. This may include the following, none of which are required:
		- `CustomFields` (**optional**) - map of any custom fields that should be filled in which are on the user's Salesforce layout. Any specified fields that are recognized by the system will be updated. They should be formatted in a map of {`Salesforce API name` => `value`}
		- `RelatedTo` (**optional**) - string ID of a non-who ID within the user's Salesforce organization
	- `Status` (**optional**) - TASKS ONLY, string
	- `Priority` (**optional**) - TASKS ONLY, string
	- `Location` (**optional**) - EVENTS ONLY, string
	- `AllDay` (**optional**) - EVENTS ONLY, boolean
	- `Private` (**optional**) - EVENTS ONLY, boolean

**Supports internal override?**
Yes

**Example request body**
```
{
  "token": "XXXXX",
  "type": "event",
  "nickname": "My first template",
  "details": {
    "ID": "9KCUBWqykr",
    "Subject": "My Event",
    "Notes": "Example notes",
    "SalesforceData": {
      "CustomFields": {
        "Test_1__c": "My value"
      },
      "RelatedTo": "ax023hdh3847588888"
    },
    "Location": "Chili's",
    "AllDay": false,
    "Private": false
  }
}
```

**Example response**
```
{
  "TemplateID": "lkj3jds09e"
}
```
***
### Delete event/task templates for a user
**Discussion**

A number of templates for a given user can be deleted using this endpoint.

**File**: `/src/manage_users.js`

**Function**: `delete_user_templates`

**Parameters**
- `token` (**required**) - a valid ActivityHub authentication token for this user
- `template_ids` (**required**) - array of strings representing template IDs to delete

**Supports internal override?**
No

**Example request body**
```
{
    "token": "XXXXX",
    "template_ids": ["Lr1r0NsFSy", "OXaC37ezfz"]
}
```

**Example response**
```
{
  "Deleted": 2
}
```
***
### Update a user's preferences
**Discussion**

This endpoint takes preference items and updates them in the user object. If there are no settings, it creates them. Returns the ID of the preferences object.

**File**: `/src/manage_users.js`

**Function**: `update_user_preferences`

**Parameters**
- `token` (**required**) - a valid ActivityHub authentication token for this user
- `preferences` (**required**) - map of the following keys/values
	- `EventReminderMinutes` (**optional**) - int value: `-1` (no reminders), `0`, `10`, `15`, `30`, `60`
	- `FollowUpSuggestions` (**optional**) - boolean indicating if follow up suggestions should trigger
	- `SalesforceOnly` (**optional**) - map of the following keys/values. These are **ignored** if the user doesn't have a Salesforce account linked
		- `AutoFindInvitees` (**optional**) - boolean value indicating if auto find invitees should trigger
		- `ShowInvitations` (**optional**) - boolean value indicating if salesforce invitations should be shown
		- `GlyphsOn` (**optional**) - boolean, shows 'related to' object glyph on events/tasks
		- `UseSubjectPicklist` (**optional**) - boolean, whether the user prefers to use the Salesforce subject picklist instead of plaintext for subject fields

**Supports internal override?**
No

**Example request body**
```
{
  "token": "XXXXX",
  "preferences": {
    "EventReminderMinutes": 15,
    "FollowUpSuggestions": true,
    "SalesforceOnly": {
      "AutoFindInvitees": false,
      "ShowInvitations": false,
      "GlyphsOn": false,
      "UseSubjectPicklist": false
    }
  }
}
```

**Example response**
```
{
  "ID": "GehsQnsBrP"
}
```
***
### Get available shared calendars for an account
**Discussion**

Returns information on shared calendars that a user can link to a given account. Note that not all accounts support this, use the `SharedCalendarsSupported` flag on accounts to determine if it's available.

**File**: `/src/manage_users.js`

**Function**: `get_available_shared_cals`

**Parameters**
- `token` (**required**) - a valid ActivityHub authentication token for this user
- `account_id` (**required**) - ID of the account_linked to get shared calendars for

**Supports internal override?**
No

**Example request body**
```
{
  "token": "XXXXX",
  "account_id": "sk3bdk9snE"
}
```

**Example response**
```
{
  "ID": "GehsQnsBrP",
  "SharedCalendars": {
    "Color": "#cd74e6",
    "Display": "John Doe",
    "ID": "john@doe.com"
  }
}
```
## beforeSave triggers
### `_User`
**Discussion**

For **new** objects, this trigger ensures that the user has a defined value for their list of devices.
***
### `link_account_session`
**Discussion**

For **new** objects, this trigger sets the ACL on the record to `Master key only`.
***
### `access_grant`
**Discussion**

For **new** objects, this trigger generates all values on the object that are not set (except for `name`) and sets the ACL on the record to `Master key only`. This allows for easily creating a new `access_grant` by simply specifying a `name`, and allowing the trigger to do the rest.
***
### `user_prefs`
**Discussion**

For **new** objects, this sets some default values that should be populated in a user's preferences, and sets the ACL on the record to `Master key only`.
***
### `account_linked`
**Discussion**

For **new** objects, this trigger sets the ACL on the record to `Master key only`.
***
### `enterprise_prefs`
**Discussion**

For **new** objects, this trigger sets the ACL on the record to `Master key only`.
***
### `event_template`
**Discussion**

For **new** objects, this trigger sets the ACL on the record to `Master key only`.
***
### `task_template`
**Discussion**

For **new** objects, this trigger sets the ACL on the record to `Master key only`.
***

## afterSave triggers
### `account_linked`
**Discussion**

For **new** objects, this trigger sends a request to ActivityHub's internal Salesforce organization to notify CRM of the newly linked account.
***
### `license_purchase`
**Discussion**

For **new** objects, this trigger sends a request to ActivityHub's internal Salesforce organization to notify CRM of the newly purchased license for a given user.
***

## afterDelete triggers
### `_User`
**Discussion**

This trigger essentially does some cleanup after a user is erased from our database. Since ActivityHub doesn't allow customers to delete their profiles, this will normally be fired when someone internally removes a user. The trigger will automatically remove relevant `user_prefs` and `account_linked` objects (therefore resulting in a cascade of triggers, wiping out the user's metadata).
***
### `account_linked`
**Discussion**

This trigger will automatically delete all `event` and `task` objects (and their associates) related to this account. It will then send a request to ActivityHub's internal Salesforce organization to notify CRM of the removal. In addition, if this was a Salesforce account, the user's `preferences` will be stripped of all Salesforce-specific data.
***

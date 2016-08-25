# Contact management

## Table of contents
#### Endpoints
- [Get contacts](../documentation/Contacts.md#get-contacts)
- [Create contacts](../documentation/Contacts.md#create-contacts)
- [Search account contacts](../documentation/Contacts.md#search-account-contacts)

***
## Endpoints
### Get contacts
**Discussion**

This endpoint returns an array of contacts for a given account linked to a user. This endpoint does not work with Microsoft Live (since their API doesn't support contact management). To access Salesforce contacts, use the [`get_sf_recents_for_acct`](../documentation/Salesforce.md) endpoint instead.

**File**: `/src/contact_handling.js`

**Function**: `get_contacts_for_acct`

**Parameters**
- `token` (**required**) - a valid ActivityHub authentication token for this user
- `account_id` (**required**) - ID of the user's `account_linked` object
- `ignore_retry` (**optional**) - boolean used internally to force the function not to try refreshing an OAuth token

**Supports internal override?** 
Yes

**Example request body**
```
{
  "token": "XXXXX",
  "account_id": "42lgNp1Mwt"
}
```

**Example response**
```
{
  "Contacts": [
    {
      "ID": "AAMkADVmMTlhOTg1LTQx",
      "Email": "john@doe.com",
      "Name": "John Doe",
      "Phone": ""
    }
  ]
}
```
***
### Create contacts
**Discussion**

This endpoint allows for the creation of a contact in any number of a given user's accounts. It does not work with Microsoft Live (since their API doesn't support contact management).

**File**: `/src/contact_handling.js`

**Function**: `create_contact`

**Parameters**
- `token` (**required**) - a valid ActivityHub authentication token for this user
- `account_ids` (**required**) - array of IDs of `account_linked` objects in which the contact should be created
- `first_name` (**required**) - first name of the contact being created
- `last_name` (**required**) - last name of the contact being created
- `email` (**required**) - email address to assign the contact
- `phone` (**optional**) - phone number to assign to the contact (no particular format)
- `phone_type` (**optional**) - type of phone number the `phone` value represents. Possible values are "primary", "mobile", "home", "work", or "other". Defaults to "primary"
- `sf_account_id` (**optional**) - the ID of an Account object within Salesforce to which the newly created contact should be related to. Irrelevant for non-Salesforce accounts

**Supports internal override?** 
Yes

**Example request body**
```
{
  "token": "XXXXX",
  "phone": "1234567890",
  "last_name": "Doe",
  "email": "john@test.com",
  "sf_account_id": "001o000000jS8PS",
  "phone_type": "home",
  "account_ids": [
    "9XkukaVW4x"
  ],
  "first_name": "John"
}
```

**Example response**
```
{
  "CreatedInAccounts": {
    "WMajQtTiYF": {
      "Email": "john@test.com",
      "Name": "John Doe",
      "Phone": "1234567890",
      "ServiceID": "lakjsdfb3lkja23klj123bsdlf"
    }
  }
}
```
***
### Search account contacts
**Discussion**

Returns a list of contacts for the given account.

**File**: `/src/contact_handling.js`

**Function**: `search_contacts_for_acct`

**Parameters**
- `token` (**required**) - a valid ActivityHub authentication token for this user
- `account_id` (**required**) - ID of account_linked object
- `search_term` (**required**) - String value to search by
- `ignore_retry` (**required**) - boolean used internally to force the function not to try refreshing an OAuth token

**Supports internal override?** 
Yes

**Example request body**
```
{
  "token": "XXXXX",
  "account_id": "42lGnp2Mwt",
  "ignore_retry": "false"
}
```

**Example response**
```
{
  "result": {
    "Results": [
      {
        "Email": "johndow@gmail.com",
        "Name": "John Dow",
        "Phone": "(800) 888-8888",
        "ServiceID": "0033200001YsFVTAA3"
      }
    ]
  }
}
```
***
 
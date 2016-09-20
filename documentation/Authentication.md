# Authentication

## Table of contents
#### Introduction & usage
- [Introduction](#introduction)
- [Obtain client keys](#obtain-client-keys)
- [Register and login](#register-and-login)
- [Add an additional linked account](#add-an-additional-linked-account)
- [Re-authenticate broken accounts](#re-authenticate-broken-accounts)

#### Endpoints (internal use)
- [Request permission to sign a user in](#request-permission-to-sign-a-user-in)
- [Complete a registration/login](#complete-a-registrationlogin)

***
## Introduction
In order to provide an easy-to-use authentication mechanism while keeping user information secure, we use third party identity providers to log users in to ActivityHub (as well as to sign them up). 

That being said, we provide our own OAuth wrapper which allows you to send users to a single ActivityHub URL which will then allow the user to sign in with any service in our myriad of supported accounts. 

Below, we have outlined information on how to construct and make various authentication requests. This includes signing up, logging in, adding a linked account, and re-authenticating broken accounts.

***
## Obtain client keys
Just like any other authentication provider, we require you (as the client application) to have your own set of keys to make requests to our servers with. These keys allow us to identify your application, and eventually serve requests to your users through our service.

To get a set of client keys, please contact us [via email](mailto:michael@nexmachine.com?subject=ActivityHub API Client Keys Request). When doing so, please be sure to **specify any redirect URIs that you'd like to configure for your application's callback**.

Once obtained, your key set will include:
- Client ID
- Client secret
- Redirect URI(s)

***
## Register and login
Once you have your client keys, you're ready to make a request to log a user in to ActivityHub! To do this, you'll construct a `GET` request that the user will **load in their browser, eventually redirecting to an OAuth login/consent screen**.

**Before making this request**, you'll want to call the [`get_available_acct_types`](../documentation/User%20Management.md#get-available-account-types) endpoint to get a list of services that the user can log in with. The ID selected from the list will serve as the `type` parameter below.

***This endpoint can be used for both signing a user up and logging a user in. Once the user authenticates with an identity provider, ActivityHub will search our database for a matching user profile. If none is found, one is created automatically for the user.***

URL: `https://auth.activityhub.io/authorize`

Parameters:
- **`type`** - the `ID` of the account type/service that the user will be signing in with. This ID is obtained from the `get_available_acct_types` call mentioned above
- **`client_id`** - the client ID for your app that we provided you with
- **`client_secret`** - the client secret for your app that we provided you with
- **`redirect_uri`** - a **URL-encoded** redirect URL that has been registered with your client app, to which the user will be redirected to after a successful or failed authentication
- **`sandbox`** - an **optional** parameter which should only be `true` for **Salesforce accounts using sandbox instead of production**

Example login URL:
```
https://auth.activityhub.io/authorize?type=AAAAA&client_id=BBBBB&client_secret=CCCCC&redirect_uri=https%3A%2F%2Flocalhost&sandbox=false
```

Once you've constructed this URL, simply request it in a browser/web view for your user. ActivityHub will then validate your request and send them to the appropriate login page. Upon a successful or failed authentication, your user will be redirected to the `redirect_uri`, sending the following parameters:

If **successful**...
- **`access_token`** - a **URL-encoded** access token to be used for making requests on behalf of this user. **This must be decoded before use**
- **`is_new`** - this value will be `true` if the system did not find this user's account in our system and created one automatically. Otherwise, `false`
- **`username`** - the user's ActivityHub username

If **unsuccessful**...
- **`error`** - a **URL-encoded** description of what went wrong, in JSON format

If your request was successful, you can now use your provided `access_token` for making authenticated requests to ActivityHub (after URL-decoding it).

***
## Add an additional linked account
The real power of ActivityHub becomes apparent once a user has added more than one external account to their profile. (For example, a user has a Salesforce account and 2 Google accounts associated with their ActivityHub profile.) 

Adding a linked account is very simple, and has an *almost identical* process as signing a user in, as shown above. The only difference is that you'll need to provide a `token` parameter to indicate that you'd like to link this account to the user's profile instead of attempt to sign up with it.

*A given external account (i.e., john.doe@gmail.com) can only be associated with a single ActivityHub user. Attempting to link an account already associated with another user will result in an error callback.*

URL: `https://auth.activityhub.io/authorize`

Parameters:
- All parameters mentioned above for [logging in](#register-and-login)
- **`token`** - an ActivityHub access token for this user. This will allow our servers to identify the user to associate this new account with

Example URL for adding a linked account:
```
https://auth.activityhub.io/authorize?type=AAAAA&client_id=BBBBB&client_secret=CCCCC&redirect_uri=https%3A%2F%2Flocalhost&sandbox=false&token=DDDDD
```

Once you've constructed this URL, simply request it in a browser/web view for your user. ActivityHub will then validate your request and send them to the appropriate login page. Upon a successful or failed authentication, your user will be redirected to the `redirect_uri`, sending the same parameters as the [login responses mentioned above](#register-and-login). **Be sure to use the new access token that is provided with the success callback for future requests.**

***
## Re-authenticate broken accounts
Sometimes, an OAuth refresh token for a given account's 3rd party service will expire or be revoked. In this case, ActivityHub will mark the account as `NeedsAuth` being `true` (visible when [requesting the user's profile or list of accounts](../master/documentation/User%20Management.md)). 

In this case, the user will need to re-authenticate their account by signing in with it again. This is done very similarly to [how users add new linked accounts](#add-an-additional-linked-account), with the addition of an `account_id` parameter.

 URL: `https://auth.activityhub.io/authorize`

Parameters:
- All parameters mentioned above for [adding a new linked account](#add-an-additional-linked-account)
- **`account_id`** - the ID of the linked account that's being re-authenticated. ActivityHub will make sure that the user signs in with the same account

Example URL for adding a linked account:
```
https://auth.activityhub.io/authorize?type=AAAAA&client_id=BBBBB&client_secret=CCCCC&redirect_uri=https%3A%2F%2Flocalhost&sandbox=false&token=DDDDD&account_id=EEEEE
```

Once you've constructed this URL, simply request it in a browser/web view for your user. ActivityHub will then validate your request and send them to the appropriate login page. Upon a successful or failed authentication, your user will be redirected to the `redirect_uri`, sending the same parameters as the [login responses mentioned above](#register-and-login). **Be sure to use the new access token that is provided with the success callback for future requests.**

***
## Internal endpoints 

### Request permission to sign a user in
**Discussion**

*For internal use only.* This endpoint is used as part of the above authentication flow after receiving a request for a sign-in from an authorized client. It can be used for the following tasks:
1. Sign a user up (new user)
2. Log a user in (existing user)
3. Add a linked account to an existing user
4. Re-authenticate a broken linked account

The first step to sign a user in (or sign them up) is to ask ActivityHub's servers for a token and login URL. For this, you'll need to know the ActivityHub service ID that the user will use to sign in with (for example, Google may be `T3amarz741` - these can be fetched in the `get_available_acct_types` endpoint).

See documentation on the `use_external_account` endpoint for what to do after calling this function.

**File**: `/src/manage_users.js`

**Function**: `get_oauth_for_login`

**Parameters**
- `key` (**required**) - internal key for authorizing this endpoint
- `client_id` (**required**) - the client ID of the app this request is being made on behalf of
- `client_secret` (**required**) - the client secret of the app this request is being made on behalf of
- `redirect_uri` (**required**) - URL of the registered app to redirect to after a request is made
- `token` (**optional**) - a valid ActivityHub authentication token for a user, **needed if adding the account to an existing user**
- `account_type_id` (**required**) - the account type (a value from the `get_available_acct_types` endpoint) that the user will be logging in with
- `sf_use_sandbox` (**optional**) - for Salesforce accounts, passing `true` for this value will have the user sign in to a Salesforce sandbox instead of production instance. Defaults to `false`

**Supports internal override?**
Yes

**Example request body**
```
{
  "key": "AAAAA",
  "account_type_id": "d7dhH8bd9N",
  "client_id": "BBBBB",
  "client_secret": "CCCCC",
  "redirect_uri": "localhost://callback",
  "sf_use_sandbox": false,
  "token": "DDDDD"
}
```

**Example response**
```
{
  "OAuth": {
    "LoginURL": "https://login.salesforce.com/auth..."
  },
  "Session": {
    "Token": "hegchx1tgkhgp66r0078r8ppKLS8k91466722721704",
    "ExpiresAt": 1466726321704
  }
}
```
***
### Complete a registration/login
**Discussion**

*For internal use only.* After calling the above `get_oauth_for_login` function and receiving a token and login URL, the next step is to send the user to that URL and allow them to log in. **The external service will automatically call an endpoint on our server as a callback, providing a `code` which should then be extracted to call this function.**

Once you have the code, call this function using that value as well as the `Token` returned in the previous function to finish validating the user. ActivityHub will then perform several validations and requests to this service, and will ultimately return with one of 3 results:
1. Account was recognized, signed the user in and returned a token
2. Account was not recognized, created a new user account and returned a token
3. Error encountered

*Upon receiving a successful return from this function, you'll be able to make calls to all other ActivityHub endpoints using the newly provided token for this user.* Here are some more details on the values returned:
- If a new user was created, `Response.IsNew` will be `true`
- `Response.AccessToken` can be used to make authenticated requests on behalf of this user
- The `Response.RedirectURL` is the URL that the user should be redirected to in order for the original client to receive a callback

**File**: `/src/manage_users.js`

**Function**: `use_external_account`

**Parameters**
- `key` (**required**) - internal key for authorizing this endpoint
- `oauth_token` (**required**) - a valid `Token` returned in the `get_oauth_for_login` function
- `code` (**required**) - a valid code retrieved by the client during the login process with the external service (see discussion above)

**Supports internal override?**
Yes

**Example request body**
```
{
  "key": "XXXXX",
  "oauth_token": "YYYYY",
  "code": "ZZZZZ"
}
```

**Example response**
```
{
  "Response": {
    "IsNew": false,
    "AccessToken": "XXXXX",
    "RedirectURL": "localhost://callback",
    "Username": "test-028"
  }
}
```
***
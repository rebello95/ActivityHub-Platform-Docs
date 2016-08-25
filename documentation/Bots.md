# Bots!
Bots are truly becoming a bigger part of our world with every day, which is why ActivityHub's platform includes some of our own. Our bots are unique, though - they're not just things you have to work to manage, they're just a part of your conversations. For example, starting a conversation via SMS ([documentation](../documentation/Phones%20and%20SMS.md)) will allow users to tag bots in messages. They can also communicate directly with them. 

This document should provide some information on how bots function internally.
***

## Table of contents
#### Internal functions
- [`understand()`](../documentation/Bots.md#understand)
- [`createContext()`](../documentation/Bots.md#createcontext)
- [`processContextData()`](../documentation/Bots.md#processcontextdata)
- [`continueContextNewEvent()`](../documentation/Bots.md#continuecontextnewevent)
- [`simplifyMessageReceived()`](../continueContextNewEvent/Bots.md#simplifymessagereceived)
- [`isGreeting()`](../documentation/Bots.md#isgreeting)
- [`isNegativeResponse()`](../documentation/Bots.md#isnegativeresponse)
- [`isPositiveResponse()`](../documentation/Bots.md#ispositiveresponse)
- [`isCancellationResponse()`](../documentation/Bots.md#iscancellationresponse)
- [`greetingResponse()`](../documentation/Bots.md#greetingresponse)
- [`misunderstoodResponse()`](../documentation/Bots.md#misunderstoodresponse)
- [`affirmativeResponse()`](../documentation/Bots.md#affirmativeresponse)
- [`problemResponse()`](../documentation/Bots.md#problemresponse)
- [`cancellingResponse()`](../documentation/Bots.md#cancellingresponse)
- [`cancelContext()`](../documentation/Bots.md#cancelcontext)

#### beforeSave triggers
- [`bot_context`](../documentation/Bots.md#bot_context)
- [`bot_message`](../documentation/Bots.md#bot_message)

***
## Internal functions
### understand()
**Discussion**

Key to any bot is natural language understanding. Calling this function with a given message string will communicate with Wit.ai to determine what exactly the user is trying to say, and will return a set of filtered data points using a confidence percentage requirement.

**File**: `/src/bots.js`

**Function**: `understand(message, confidence)`

**Arguments**
- `message` - content of the message. Things like `@` bot tags and extraneous spaces will be automatically removed before attempting parsing
- `confidence` - optional, a number between `0.0 - 1.0` indicating the minimum confidence requirement for any entity to match. Defaults to `0.65`

**Returns**: A data structure similar to the following. All possible data points are declared in `constants.js`
```
{
  "ah_event": [
    {
      "confidence": 0.6820028832761513,
      "value": "event"
    }
  ]
}
```
***
### createContext()
**Discussion**

The second critical part to creating a bot is the presence of contexts. For example, what was the last message the user sent? Are they referring to something else? How can I aggregate data across multiple messages in one session with a user to provide a useful outcome?

Context awareness allows for this. In our case, a `bot_context` record is updated every time a user or a bot says anything in a conversation to one another, and stores things like what action the user is trying to accomplish, who the user is, what values we need to resolve the user's request, and what values we already have (or partially have).

**File**: `/src/bots.js`

**Function**: `createContext(dataPoints, user, platform, smsGroup)`

**Arguments**
- `dataPoints` - map of information returned from a call to the `understand()` function
- `user` - `_User` who the bot is talking to
- `platform` - a platform type string, specified in `constants.js`
- `smsGroup` - optional, the `sms_group` record that should be associated with this context if the user is sidebarring with the bot

**Returns**: A promise containing a new `bot_context` object when fulfilled

**Context data example**
Here's an example of what a JSON value stored in a `bot_context` record's `contextData` might look like:
```
{
  "invitees": {
    "value": [
      {
        "Name": "John Doe",
        "Email": "john@test.com",
        "ServiceID": "003xxxxxxxxxxxxxxx",
        "AccountID": "Yxjrz6K0l6",
        "AccountNickname": "Salesforce"
      }
    ],
    "active": false,
    "required": true,
    "question": "Who are you meeting with?",
    "temp": {
      "action": "select"
    }
  },
  "location": {
    "value": "Starbucks",
    "active": false,
    "required": false,
    "question": "Where are you meeting?",
    "temp": {}
  },
  "start": {
    "value": "2016-07-09T15:00:00.000-07:00",
    "active": false,
    "required": true,
    "question": "When should I schedule it for?",
    "temp": {}
  }
}
```
***
### cancelContext()
**Discussion**

This function can be used to easily cancel an existing `bot_context` record (marking its `status` as cancelled). It returns a promise that is resolved upon successfully cancelling the context.

**File**: `/src/bots.js`

**Function**: `cancelContext(context)`

**Arguments**
- `context` - a `bot_context` object to be cancelled

**Returns**: A resolved promise upon cancellation of the context

***
### processContextData()
**Discussion**

This function essentially acts as a router given a user's context and most recent message data points. It will properly call the necessary function(s) to do the next step in the given context and provide the caller function with details on what to tell the user via their current means of communication.

**File**: `/src/bots.js`

**Function**: `processContextData(context, dataPoints, user, message)`

**Arguments**
- `context` - the `bot_context` object for the current context
- `dataPoints` - map of information returned from a call to the `understand()` function for the user's last message
- `user` - the `_User` who sent this message
- `message` - the message's text

**Returns**: A promise, which contains a map with the following key/values when fulfilled:
- `msg_individual` - a string that should be messaged back to the individual, or `null`
- `msg_group` - a string that should be messaged back to either the individual or a group, or `null`
- `context` - the updated `bot_context`

***
### continueContextNewEvent()
**Discussion**

This is the function that is routed to from the `processContextData()` function for creating new events. It is a very powerful and thorough function, which is capable of taking a user through the entire process of setting the time, location, and invitees for a given event. It also provides a variety of dynamic response options in order to make the bot seem more conversational.

Since this function runs the entire process of creating a new event, it adheres by the `autoAddSFInvitees` setting on every given `account_linked`, and will create the event in all of a user's accounts that are marked with `isDefault` being `true`, just as the user would expect. It will also always send email invitations.

**File**: `/src/bots.js`

**Function**: `continueContextNewEvent(context, dataPoints, user, message, config)`

**Arguments**
- `context` - the `bot_context` object for the current context
- `dataPoints` - map of information returned from a call to the `understand()` function for the user's last message
- `user` - the `_User` who sent this message
- `message` - the message's text
- `config` - instance of `Parse.Config` to get values from

**Returns**: A promise, which contains a map with the following key/values when fulfilled:
- `msg_individual` - a string that should be messaged back to the individual, or `null`
- `msg_group` - a string that should be messaged back to either the individual or a group, or `null`
- `context` - the updated `bot_context`

***
### simplifyMessageReceived()
**Discussion**

This function can be used to strip out `@` bot tags, leading and trailing spaces, and other useless information in order to provide more accurate natural language understanding. Resulting strings will also be in all lowercase.

**File**: `/src/bots.js`

**Function**: `simplifyMessageReceived(message)`

**Arguments**
- `message` - content of the message to simplify

**Returns**: A simplified string

***
### isGreeting()
**Discussion**

This is an easy way to determine if a message from a user is essentially saying some variant of "hello".

**File**: `/src/bots.js`

**Function**: `isGreeting(message)`

**Arguments**
- `message` - content of the message to evaluate

**Returns**: A boolean value

***
### isNegativeResponse()
**Discussion**

This is an easy way to determine if a message from a user is essentially saying some variant of "no" or "nothing".

**File**: `/src/bots.js`

**Function**: `isNegativeResponse(message)`

**Arguments**
- `message` - content of the message to evaluate

**Returns**: A boolean value

***
### isPositiveResponse()
**Discussion**

This is an easy way to determine if a message from a user is essentially saying some variant of "yes".

**File**: `/src/bots.js`

**Function**: `isPositiveResponse(message)`

**Arguments**
- `message` - content of the message to evaluate

**Returns**: A boolean value

***
### isCancellationResponse()
**Discussion**

This is an easy way to determine if a user is trying to cancel an existing action or context based on their response.

**File**: `/src/bots.js`

**Function**: `isCancellationResponse(message)`

**Arguments**
- `message` - content of the message to evaluate

**Returns**: A boolean value

***
### greetingResponse()
**Discussion**

This function will return a variant of a greeting, such as `Hi there!`, `Hello!`, etc., in order to respond to a user while sounding less automated.

**File**: `/src/bots.js`

**Function**: `greetingResponse()`

**Arguments**
None

**Returns**: A greeting string

***
### misunderstoodResponse()
**Discussion**

This function will return a some sort of variant of the phrase `I'm not sure what you mean.`, which should be used when no decisive data could be pulled from a user's response.

**File**: `/src/bots.js`

**Function**: `misunderstoodResponse()`

**Arguments**
None

**Returns**: A response string

***
### affirmativeResponse()
**Discussion**

This function will return a variant of a greeting, such as `Got it.`, `Alrighty.`, etc., in order to respond to a user while sounding less automated.

**File**: `/src/bots.js`

**Function**: `affirmativeResponse()`

**Arguments**
None

**Returns**: A response string

***
### problemResponse()
**Discussion**

This function will return a some sort of variant of the phrase `I seem to be having a problem. Could you rephrase that?`, which should be used when the function encounters an error.

**File**: `/src/bots.js`

**Function**: `problemResponse()`

**Arguments**
None

**Returns**: A response string

***
### cancellingResponse()
**Discussion**

This function will return a **promise** with some sort of variant of the phrase `Cancelled.`, which should be used when a user's current context is being cancelled. If `context` is defined, it will potentially suggest a link into the app to allow the user to manually try this action. 

Note: The reason this function returns a promise is so that it has the capability of shortening a continuation URL generated from the specified `context`, if any. For example, if the user is creating an event, the URL would redirect the user to the app and auto-fill the information the bot already gathered.

**File**: `/src/bots.js`

**Function**: `cancellingResponse(context)`

**Arguments**
- `context` - optional, a `bot_context` that, if specified, may add a link into the app which will allow the user to continue this action again manually

**Returns**: A promise containing a response string upon success

***
## beforeSave triggers
### `bot_context`
**Discussion**

For **new** objects, this will set the ACL on the `bot_context` record to `Master key only`. It will also update the `expiresAt` value of the context to the `BOT_CONTEXT_LENGTH` specified in `constants.js` to properly allow for context expiration.
***
### `bot_message`
**Discussion**

For **new** objects, this will set the ACL on the `bot_message` record to `Master key only`.
***
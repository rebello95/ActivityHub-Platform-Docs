# Phones and SMS

## Table of contents
#### Endpoints
- [Add a phone number](../documentation/Phones%20and%20SMS.md#add-a-phone-number)
- [Verify a phone number](../documentation/Phones%20and%20SMS.md#verify-a-phone-number)
- [Starting a conversation](../documentation/Phones%20and%20SMS.md#starting-a-conversation)
- [Receiving SMS messages](../documentation/Phones%20and%20SMS.md#receiving-sms-messages)

#### Internal functions
- [`sendSMS()`](../documentation/Phones%20and%20SMS.md#sendsms)
- [`storeSentGroupSMS()`](../documentation/Phones%20and%20SMS.md#storesentgroupsms)
- [`sendAccountNotLinkedSMS()`](../documentation/Phones%20and%20SMS.md#sendaccountnotlinkedsms)
- [`formatPhone()`](../documentation/Phones%20and%20SMS.md#formatphone)
- [`friendlyPhone()`](../documentation/Phones%20and%20SMS.md#friendlyphone)
- [`provisionTwilioNumber()`](../documentation/Phones%20and%20SMS.md#provisiontwilionumber)

***
## Endpoints
### Add a phone number
**Discussion**

Before ActivityHub can interact with a user via SMS, they must add a phone number to their profile. This can be done by calling this endpoint, then using the verification code provided to confirm the phone number.
- The `number` parameter must be a valid phone number, **including the country code**. No particular format is necessary as long as every digit is included
- Upon calling this endpoint, **ActivityHub will text a verification code the phone number provided, which should then be used to call the `confirm_phone_number` endpoint**
- You will have 3 hours to verify the phone number after calling this endpoint, as indicated by the `VerifyBefore` response value

**File**: `/src/sms_handling.js`

**Function**: `add_phone_number`

**Parameters**
- `token` (**required**) - a valid ActivityHub authentication token for this user
- `number` (**required**) - a valid cell phone number (see discussion above)

**Supports internal override?** 
No

**Example request body**
```
{
  "token": "XXXXX",
  "number": "1 222 333-4444"
}
```

**Example response**
```
{
  "Number": "+12223334444",
  "Status": "Pending",
  "VerifyBefore": "2016-06-24T23:24:00Z"
}
```
***
### Verify a phone number
**Discussion**

After calling the `add_phone_number` endpoint, the user will have received a text. To finish verifying the number, have the user input the verification code, then call this function.
- The response will include the number that was just verified, as well as all other numbers the user has previously verified
- If the phone number being verified was previously tied to another user, it will be removed from that user's profile

**File**: `/src/sms_handling.js`

**Function**: `verify_phone_number`

**Parameters**
- `token` (**required**) - a valid ActivityHub authentication token for this user
- `verification` (**required**) - verification code ActivityHub texted to the user

**Supports internal override?** 
No

**Example request body**
```
{
  "token": "XXXXX",
  "verification": "34609"
}
```

**Example response**
```
{
  "AllNumbers": [
    "+12223334444",
    "+13334445555"
  ],
  "Response": {
    "Number": "+12223334444",
    "Status": "Verified"
  }
}
```
***
### Starting a conversation
**Discussion**

News flash: People text each other! ActivityHub makes this even better by managing an SMS chat between 2 or more people, allowing recipients who are ActivityHub users to make requests and issue commands to the platform directly within their chat. 

When this function is called, ActivityHub assigns the group a phone number to use for sending and receiving texts within this group (Twilio does a great job [explaining this](https://www.twilio.com/help/faq/sms/how-can-i-have-users-send-text-messages-to-each-other-over-twilio)). It then sends a text to all recipients of the message via normal SMS, and from there the chat acts like a normal group chat (like as you may have seen with Google Voice).

The key here is that once ActivityHub is managing the SMS group chat, our bots are seamless and automatically integrated. Read more about bots in the [documentation](../documentation/Bots.md).  

**File**: `/src/sms_handling.js`

**Function**: `start_conversation`

**Parameters**
- `token` (**required**) - a valid ActivityHub authentication token for this user
- `recipients` (**required**) - array of maps, each containing the following information:
	- `name` (**optional**) - name of the recipient
	- `number` (**required**) - phone number of the recipient
- `message` (**required**) - the message to send all recipients

**Supports internal override?** 
No

**Example request body**
```
{
  "token": "XXXXX",
  "recipients": [
    {
      "name": "",
      "number": "1222 333-4444"
    }
  ],
  "message": "What's up!"
}
```

**Example response**
```
{
  "Sent": 2
}
```
***
### Receiving SMS messages
**Discussion**

*For internal use only*. This function is called by Twilio when any of our group text/conversational phone numbers receive a text message. This complex function does a variety of things with text messages when called, including forwarding them, storing them, and responding to them with bot messages and contexts as necessary. **Only Twilio will be able to successfully call this function.**

**File**: `/src/sms_handling.js`

**Function**: `sms_convo_messaged`

**Parameters**
Normal SMS parameters passed in by Twilio

**Supports internal override?** 
No

**Example request body**
```
Hidden
```

**Example response**
```
{

}
```
***
## Internal functions
### sendSMS()
**Discussion**

This function can be called internally to send a text message via Twilio to any set of phone numbers.
- Phone number formats are corrected after being passed into the function. However, the numbers **must contain their respective country codes**
- Each text is sent individually (not as a group text), due to [Twilio restrictions](https://www.twilio.com/help/faq/sms/can-i-set-up-one-api-call-to-send-messages-to-a-list-of-people)

**File**: `/src/sms_handling.js`

**Function**: `sendSMS(message, to, from)`

**Arguments**
- `message` - a string containing the content of the SMS you're sending
- `to` - an array of strings representing phone numbers (see discussion above)
- `from` - a valid Twilio phone number to send the text from (defaults to the number specified in Parse configs)

**Returns**: A promise resolved upon sending all texts
***
### storeSentGroupSMS()
**Discussion**

This function should be used as a convenience method to store text messages being sent by a user or bot via SMS within our database in the `sms_message` table.

**File**: `/src/sms_handling.js`

**Function**: `storeSentGroupSMS(group, message, isBot, sender, sidebarTo, sidebarUser)`

**Arguments**
- `group` - the `sms_group` this message is associated with
- `message` - the text of the message that was sent
- `isBot` - indicates whether an ActivityHub bot sent the message
- `sender` - optional, the phone number string of the person sending the text
- `sidebarTo` - optional, the phone number the bot is sending a sidebar message to
- `sidebarUser` - optional, the `_User` that the bot is sidebarring with

**Returns**: A promise resolved with the `sms_message` record once saved
***
### sendAccountNotLinkedSMS()
**Discussion**

If a user texts an ActivityHub number using a phone number that has not yet been added to ActivityHub, this function should be used to send a text back to the user providing them with a notice as well as further instructions. The `storeSentGroupSMS()` should also be manually called when calling this function.

**File**: `/src/sms_handling.js`

**Function**: `sendAccountNotLinkedSMS(toNumber, fromNumber, group)`

**Arguments**
- `toNumber` - phone number string to send the text to
- `fromNumber` - a valid Twilio phone number string (from the `phone_provision` table)
- `group` - the `sms_group` record that this message is being sent into

**Returns**: A promise resolved upon sending the text
***
### formatPhone()
**Discussion**

This function is used to easily correct mis-formatted phone numbers (and convert numbers into strings if necessary).

**File**: `/src/sms_handling.js`

**Function**: `formatPhone(number)`

**Arguments**
- `number` - a number or string containing a phone number

**Returns**: A text-ready phone number string in the format of `+12223334444`
***
### friendlyPhone()
**Discussion**

Since phone numbers are stored as Twilio-friendly strings in our database, this function can be used to re-format phone numbers and make them look more friendly to humans.

**File**: `/src/sms_handling.js`

**Function**: `friendlyPhone(phoneString, parentheses)`

**Arguments**
- `phoneString` - a phone number string formatted as normal in the database (like `+12223334444`)
- `parentheses` - should be `true` if you'd like the first 3 digits to be enclosed in parentheses (like an area code). Defaults to `false`

**Returns**: A phone number string formatted like either `111 222 3333` or `(111) 222 3333`
***
### provisionTwilioNumber()
**Discussion**

Twilio allows developers to dynamically provision new phone numbers. This function takes advantage of that by providing a convenient way to do that internally. *Unfortunately, phone numbers will still have to be manually hooked up to services and webooks, as Twilio doesn't allow this via their API yet.*

**File**: `/src/sms_handling.js`

**Function**: `provisionTwilioNumber()`

**Arguments**
None

**Returns**: A promise containing a new `phone_provision` record on success, or `null`
***
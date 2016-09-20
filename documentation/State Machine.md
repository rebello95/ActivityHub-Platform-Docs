# Syncing & state machine

ActivityHub's event state machine is *the mechanism that pushes changes the user made outside of ActivityHub from one linked account into one or more others*. For example, if a user created an event in their Google account (and they have sync enabled on it), ActivityHub will automatically clone this event into their other linked accounts (i.e., Salesforce). Or, if the user changed the time of an event in Salesforce, it'll be reflected in Office 365 as well.

Here is a summary of how the state machine runs:

1. Iterate through users
2. Sync events for the user one account at a time
3. Sync tasks for the user's Salesforce account (if any)
4. For events, determine if updates need to be pushed to other services and execute those changes. (Note that in this step, it will first need a list of all pre-existing matches. The reason for this is if one event changed it must know what associated events to update)
5. For events, determine if any that were previously matched have been deleted in one of the matched services. If so, remove it from the other services it was matched with
6. Match events for the user. This is done at this time so that state machine updates/deletions that were made will be taken into consideration
7. Determine which events were created in services since the last refresh which are set to sync changes to others, and create events in the services that don't yet have the event. These events will then be matched together

Notes:
- By default, private events are not pushed from one account to others. This setting can be enabled on any given account, though
- ActivityHub will not sync Salesforce events that the user was invited to within their organization in order to prevent duplicates

***
## Table of contents
#### beforeSave triggers
- [event](#event)
- [event_match](#event_match)
- [event_invitee](#event_invitee)
- [task](#task)
- [task_invitee](#task_invitee)

#### afterDelete triggers
- [event](#event-1)
- [task](#task-1)

#### Internal functions
- [syncAllDataForUser()](#syncalldataforuser)
- [syncDataForUserNormally()](#syncdataforusernormally)
- [matchEventsForUser()](#matcheventsforuser)
- [deleteEventsForAccount()](#deleteeventsforaccount)
- [deleteTasksForAccount()](#deletetasksforaccount)

***
## beforeSave triggers
### `event`
**Discussion**

For **new** objects, this will ensure that there's a valid `subject` being saved. If not, it will set it to `(Unknown subject)`. It will also make sure that there is a `userStatus` provided, and will set it to the `INVITEE_STATUS_ORGANIZER` constant if not. If no `userAccess` is provided, it will be defaulted to `ACCESS_FULL_ACCESS`. Lastly, the ACL on the `event` record will be set to `Master key only`.
***
### `event_match`
**Discussion**

For **new** objects, this will ensure that the ACL on this matched event is set to `Master key only`.
***
### `event_invitee`
**Discussion**

For **new** objects, this will ensure that there's a valid `status` being saved. If not, it will set it to the `INVITEE_STATUS_DEFAULT` constant. Also, for ease of parsing at a later time, if the `name` provided is a blank string or `null`, it will be set to `undefined`. Lastly, the ACL on the `event_invitee` record will be set to `Master key only`.
***
### `task`
**Discussion**

For **new** objects, this will ensure that there's a valid `subject` being saved. If not, it will set it to `(Unknown subject)`. It will also make sure there are values (or blank strings) set for `notes`, `status`, and `priority`. Lastly, the ACL on the `task` record will be set to `Master key only`.
***
### `task_invitee`
**Discussion**

For **new** objects: For ease of parsing at a later time, if the `name` provided is a blank string or `null`, it will be set to `undefined`. The ACL on the `task_invitee` record will also be set to `Master key only`.
***
## afterDelete triggers
### `event`
**Discussion**

Since events have other related objects (such as invitees) relying on them, these related objects should be deleted upon deletion of the event. When an `event` is deleted, this trigger will look up all `event_invitee` records related to it and delete them as well.
***
### `task`
**Discussion**

Since tasks have other related objects (such as invitees) relying on them, these related objects should be deleted upon deletion of the task. When a `task` is deleted, this trigger will look up all `task_invitee` records related to it and delete them as well.
***
## Internal functions
### syncAllDataForUser()
**Discussion**

This function, called by the job that handles syncing all users' external accounts, is the backbone of the event state machine. It essentially performs the entire state machine process outlined [above](#syncing--state-machine), and is called at several key times (i.e., when the user signs up, their accounts change, a system refresh occurs, etc.).

**File**: `/src/syncing.js`

**Function**: `syncAllDataForUser(user, start, end, excludedTypes)`

**Arguments**
- `user` - the `_User` to sync data for
- `start` - the start date to sync data for
- `end` - the end date to sync data through
- `excludedTypes` - an array of `account_type` objects to skip

**Returns**:  A resolved promise upon completion
***
### syncDataForUserNormally()
**Discussion**

This is a convenience function that syncs all of a user's data using the default timeframe and account type exclusions.

**File**: `/src/syncing.js`

**Function**: `syncDataForUserNormally(user)`

**Arguments**
- `user` - the `_User` to sync data for

**Returns**:  A resolved promise upon completion
***
### matchEventsForUser()
**Discussion**

A key feature of the platform is the ability to match events together between external systems. For example, a Google event that starts and ends at the same time as a Salesforce event owned by the same user with a similar name should automatically be linked by the ActivityHub platform internally so when a user modifies one, they can automatically modify others. This function matches (and un-matches, as necessary) a user's events between their accounts.

**File**: `/src/syncing.js`

**Function**: `matchEventsForUser(user)`

**Arguments**
- `user` - the `_User` to match events for

**Returns**:  A resolved promise upon completion
***
### deleteEventsForAccount()
**Discussion**

This function should be primarily used by `afterDelete` triggers on `account_linked` to clean up our database of events belonging to an account that has been deleted. Because this is such a unique and powerful function, it is not publicly available outside of our system. It uses an account passed in to delete all associated `event` objects, then calls the function to re-match the user's events.

**File**: `/src/syncing.js`

**Function**: `deleteEventsForAccount(linkedAcct)`

**Arguments**
- `linkedAcct` - a pointer to an `account_linked` object for which `event` objects should be deleted

**Returns**:  Nothing
***
### deleteTasksForAccount()
**Discussion**

This function should be primarily used by `afterDelete` triggers on `account_linked` to clean up our database of tasks belonging to a Salesforce account that has been deleted. Because this is such a unique and powerful function, it is not publicly available outside of our system. It uses an account passed in to delete all associated `task` objects.

**File**: `/src/sync_tasks.js`

**Function**: `deleteTasksForAccount(linkedAcct)`

**Arguments**
- `linkedAcct` - a pointer to a Salesforce `account_linked` object for which `task` objects should be deleted

**Returns**:  Nothing
***
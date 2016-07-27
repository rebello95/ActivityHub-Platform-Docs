# Surveys

## Table of contents
#### Endpoints
- [Get a survey](../documentation/Surveys.md#get-a-survey)
- [Answer a survey](../documentation/GoToMeeting.md#answer-a-survey)

***
## Endpoints
### Get a survey
**Discussion**

Gets the next active survey they user has not responded to. `Survey` is `null` if no surveys are active or unresponded to.

**File**: `/src/surveys.js`

**Function**: `get_survey`

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
  "Survey": {
    "Answers": [
      "Yes",
      "No"
    ],
    "Description": "What is this even...",
    "ID": "cGe9dqfo14",
    "Title": "Yes... or No?"
  }
}
```
### Answer a survey
**Discussion**

Creates a new `survey_response` record.

**File**: `/src/surveys.js`

**Function**: `answer_survey`

**Parameters**
- `token` (**required**) - a valid ActivityHub authentication token for this user
-  `survey_id` (**required**) - the id of a survey
-  `answer` (**required**) - a valid answer to said survey

**Supports internal override?** 
No

**Example request body**
```
{
  "token": "XXXXX",
  "survey_id": "tfG63nsa3r",
  "answer": "Yes"
}
```

**Example response**
```
{
  "Success": true
}
```
***

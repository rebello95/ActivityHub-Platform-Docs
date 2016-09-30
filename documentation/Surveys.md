# Surveys

## Table of contents
#### Endpoints
- [Get a survey](../documentation/Surveys.md#get-a-survey)
- [Answer a survey](../documentation/Surveys.md#answer-a-survey)

#### afterDelete triggers
- [`survey_response`](../documentation/Surveys.md#survey_response)

***
## Endpoints
### Get a survey
**Discussion**

Gets the next active survey they user has not responded to. `Survey` is `null` if no surveys are active or unresponded to.

**File**: `/src/surveys.js`

**Function**: `get_survey`

**Parameters**
- `token` (**required**) - a valid ActivityHub authentication token for this user
- `survey_id` (**optional**) - ID of a specific survey to fetch info for

**Supports internal override?** 
No

**Example request body**
```
{
	"token": "XXXXX",
	"survey_id": "Als7dhe2sx"
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
***
### Answer a survey
**Discussion**

Creates a new `survey_response` record. Upon success, will return a `ChartData` value that will be `non-null` if sufficient data is available. When populated, this will contain data that can be used for drawing a pie chart with data stored as `{response title => percent chosen}`.

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
  "ChartData": {
	  "Option Title 1": 0.49,
	  "Option Title 2": 0.51
  }
}
```
***

## afterDelete triggers
### `survey_response`
**Discussion**

Ensures that a given user has not responded to the same `survey` multiple times. If they have, it will remove all `survey_responses` except for the most recently modified one.
***
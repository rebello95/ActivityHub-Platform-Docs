# Utilities

## Table of contents
#### Endpoints
- [Convert address to lat/lon](../documentation/Utilities.md#convert-address-to-latlon)
- [Get Google address autocompletions](../documentation/Utilities.md#get-google-address-autocompletions)
- [Shorten a URL](../documentation/Utilities.md#shorten-a-url)

***
##Endpoints
### Convert address to lat/lon
**Discussion**

This endpoint will take a given address and return a latitude/longitude value using Google's geocoder API.

**File**: `/src/utilities`

**Function**: `get_address_loc`

**Parameters**
- `token` (**required**) - a valid ActivityHub authentication token for this user
- `address` (**required**) - a street address 

**Supports internal override?** 
Yes

**Example request body**
```
{
  "token": "XXXXX",
  "address": "One Infinite Loop Cupertino, CA 95014"
}
```

**Example response**
```
{
  "lng": 42.021,
  "lat": 40.456
}
```
***
### Get Google address autocompletions
**Discussion**

This endpoint will take a partial address and return up to 5 results using Google's geocoder API.

**File**: `/src/utilities`

**Function**: `get_address_completions`

**Parameters**
- `token` (**required**) - a valid ActivityHub authentication token for this user
- `address` (**required**) - a street address 
- `location` (**optional**) - the following values
- `lat` (**required**) - latitude of current location
- `lng` (**required**) - longitude of current location

**Supports internal override?** 
Yes

**Example request body**
```
{
  "token": "XXXXX",
  "address": "801 w mai",
  "location": {
    "lat": 12.34567,
    "lng": 89.01234
  }
}
```

**Example response**
```
{
  "Addresses": [
    "801 W Main St, Purcellville, VA, United States",
    "801 W Main St, Charlottesville, VA, United States",
    "801 W Main St, Richmond, VA, United States",
    "801 Maine Avenue Southwest, Washington, DC, United States",
    "801 W Main St, Mount Joy, PA, United States"
  ]
}
```
***
### Shorten a URL
**Discussion**

This endpoint uses Google's URL-shortening API to condense a long URL into a shorter, more readable, URL.

**File**: `/src/utilities`

**Function**: `shorten_url`

**Parameters**
- `token` (**required**) - a valid ActivityHub authentication token for this user
- `url` (**required**) - the URL string you'd like to shorten

**Supports internal override?** 
Yes

**Example request body**
```
{
  "token": "XXXXX",
  "url": "https://www.google.com"
}
```

**Example response**
```
{
  "LongURL": "https://www.google.com/",
  "ShortURL": "https://goo.gl/Njku"
}
```
***
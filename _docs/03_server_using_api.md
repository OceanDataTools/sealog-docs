---
permalink: /server_using_api/
title: "Using the API"
layout: single
toc: true
toc_label: "Contents"
toc_icon: "list"
toc_sticky: true  # Makes the TOC stick on scroll
---

The sealog server architecture is intentially small and concise.  The sealog-server is simply an API that other programs, scripts and applications leverage to submit and retrieve event data.

![Architecture](../assets/images/sealog-architecture.svg)

### Server Authentication/Authorization
API calls to potentially restricted data require Javascript Web Token (JWT) based authentication.  The JWT contains all the information the server requires to correctly respond to the request.  The data included in the response will change depending on the requestor's authorization/role.

The JWT passed to the server API via an "Authorization" header field.  The token format adheres to the standard JWT convention of `Bearer: <token string>`.   

### Obtaining a JWT via the Command-line
To obtain the JWT for a particular user from the command-line, use the following cURL command (assumes cURL is installed, replace `<username>` and `<password>` with the appropriate information):
```
curl -H "Content-Type: application/json" -X POST -d '{"username":<username>,"password":<pasword>}' https://sealog-vessel.oceandatatools.org:8000/sealog-server/api/v1/auth/login
```

This will respond with:
```
{
    "token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6IjBjNDVkN2I0LTU4ODEtNGU2NC04ZmQzLTIwNTczMjVlMmFmZSIsInNjb3BlIjpbImV2ZW50X21hbmFnZXIiLCJldmVudF9sb2dnZXIiLCJldmVudF93YXRjaGVyIl0sImlhdCI6MTUwMDAzNTc1NX0.WoOLfXxCIxIZEswy1lsbjm7XxDcbfd_NuZsL2-NB_Qw",
    "id":"6ad467beb0f844c49dcc5078"
}
```

The `"token"` value is the JWT.

**NOTES:**

- Sealog Server uses role-based permissions to limit what API calls a user can successfully make.  If an API request is made using a JWT who's associated user does not have permission to make the request the Sealog Server will respond with a status code of 401 (not authorized).

- A new JWT must be requested anytime the role permissions associated with an account are altered.

### Submitting an event to Sealog Server via the Command-line
Submitting an event to the Sealog Server requires a JWT who's associated user includes the role of 'event_logger'. Use the following cURL command to submit an event to the Sealog Server API (again assumes cURL is installed):

```
curl -X POST --header 'Content-Type: application/json' --header 'Accept: application/json' --header 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6IjBjNDVkN2I0LTU4ODEtNGU2NC04ZmQzLTIwNTczMjVlMmFmZSIsInNjb3BlIjpbImV2ZW50X21hbmFnZXIiLCJldmVudF9sb2dnZXIiLCJldmVudF93YXRjaGVyIl0sImlhdCI6MTUwMDAzNTc1NX0.WoOLfXxCIxIZEswy1lsbjm7XxDcbfd_NuZsL2-NB_Qw' -d '{"event_value": "TEST"}' 'https://sealog-vessel.oceandatatools.org:8000/sealog-server/api/v1/events'
```

This will respond with:
```
{
    "acknowledged": "true",
    "insertedId":"6ad467beb0f844c49dcc5078" 
}
```

Notice that the JWT was passed within the request header field as `'Authorization: Bearer <token string>'` 

The value associated with the variable `"insertedId"` is the UID for the event.  Event UIDs are unique and can be used to directly access the event for the purposes of reading/editing/deleting.

Using this technique scripts can be developed to allow scripts, software or even hardware like Arduinos to submit events to the Sealog Server without the need to hard-code in usernames and passwords.

### Submitting an event using python
Submitting an event to sealog using python is very similar to the submitting via the command-line.  First a JWT must be obtained, then the JWT is used to authenticate the event submission request.

```
import requests
import json

root_url = 'https://sealog-vessel.oceandatatools.org:8000/sealog-server'
api_path = '/api/v1/auth/login'

payload = {
  "username": "guest",
  "password": ""
}

r = requests.post(root_url + api_path, data=payload)

token = json.loads(r.text)['token']

api_path = '/api/v1/events'
headers = {'Authorization': 'Bearer ' + token}

payload = {
  "event_value": "HELLO_WORLD"
}

r = requests.post(root_url + api_path, headers=headers, data=payload)
```

### JWT Lifecycle
JWT remains valid for a user until the roles of that user change. When a user's roles change or the user is deleted, the server will reject any future API calls using that JWT.

### Subscribing to the websocket pub/sub streams using python

The Sealog Server includes several websocket-based pub/sub data streams for asynchorously pushing data to connected clients. The data streams use the [hapines websocket protocol](https://github.com/hapijs/nes) which all connecting clients must adhere to.

Please take a look at [this python script](https://github.com/oceandatatools/sealog-server/blob/master/misc/websockets_test.py) for a quick-n-dirty example of how to connect to the "events" stream.

Sealog Server provides the following pub/subs:
- **newEvents**: any events submitted to the server for the first time.
- **updateEvents**: any existing events submitted in the last 5 minutes that were updated
- **deleteEvents**: any events that were deleted
- **updateCustomVars**: any customVar variables that were updated
- **newCruises**: any cruise records submitted to the server for the first time.
- **updateCruises**: any existing cruise records that were updated
- **newLowerings**: any lowering records submitted to the server for the first time.
- **updateLowerings**: any existing lowering records that were updated

### API Documentation
Please refer to the [Sealog Server's RESTful API](<https://sealog-vessel.oceandatatools.org:8000/sealog-server/documentation>) for the complete list of available API calls.

![Sealog Server's RESTful API](../assets/images/sealog-swagger-screenshot.png)

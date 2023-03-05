# API-Server
**EN** | [PT](README.md)

The Holyrics program provides a communication interface to receive requests both over the local network and over the internet.

Incoming requests will be redirected to a JavaScript method in the program, which must be implemented by the user himself.
<br/>
That is, a request by itself does not perform actions in the program. What each request will do depends on the user's own defined implementation.
<br/>
The implementation of the JavaScript method must use the class [JSLib](https://github.com/holyrics/jslib) to perform actions in the program.
<br/>
It is also the responsibility of the user implementation to define, if necessary, access control to prevent unauthorized requests.
<br/>


To access API-Server settings:
<br/>
File menu > Settings > Advanced > JavaScript > API Server



# JavaScript implementation example

```javascript
function request(headers, content, info) {
    if (content.pwd !== 'password') {
        return {'status': 'unauthorized'};
    }
    switch (content.action) {
        case 'show_alert':
            if (content.message) {
                h.hly('SetAlert', {
                    text: content.message,
                    show: true
                });
                return {'status': 'ok'};
            }
            return {'status': 'Field "message" is required'};
        case 'hide_alert':
            h.hly('SetAlert', {show: false});
            return {'status': 'ok'};
        default:
            return {'status': 'action_not_found'};
    }
}

```

`headers` - Contains the request headers.
<br/>
Example: `headers.Authorization`

`content` - Object with the content extracted from the request.
<br/>
Example: `content.action`

`info` - Request information.
<br/>
`info.client_address` Request source address
<br/>
`info.local` **true** if the source of the request is from the local network
<br/>
`info.web` **true** if the origin of the request is from the internet
<br/>
# Example of a local network request
<br/>

```
Default URL
http://[IP]:[PORT]/api
```

It is possible to perform a GET or POST request.

Content-Type accepted in POST request:
<br/>
application/x-www-form-urlencoded
<br/>
application/json
<br/>

---

Request

```
http://[IP]:[PORT]/api?pwd=password&action=show_alert&message=Hello
```

Response

```
{
    "status": "ok",             <-  request status
    "data": { "status": "ok" }  <-  response received from javascript method
}
```

---

Request

```
http://[IP]:[PORT]/api?action=show_alert&message=Hello
```

Response

```
{
    "status": "ok",
    "data": { "status": "unauthorized" }
}
```

---

In case of error, **status** will return **error**, example:

```
{
    "status": "error",
    "error": "js unavailable"
}
```

# Example of internet request

Use the **API_KEY** value available in the API-Server settings to perform requests on the Holyrics server endpoint.
<br/>
File menu > Settings > Advanced > JavaScript > API Server
<br/>
```
https://api.holyrics.com.br/send.php?api_key=API_KEY
Just send the request without waiting for a response

https://api.holyrics.com.br/request.php?api_key=API_KEY
Send the request and wait for the response
```

It is possible to perform a GET or POST request.

Content-Type accepted in POST request:
<br/>
application/x-www-form-urlencoded
<br/>
application/json
<br/>
Maximum request body size: 20KB

The API_KEY value must be passed in the URL even in POST requests


Request - send.php

```
https://api.holyrics.com.br/send.php?api_key=API_KEY&pwd=password&action=show_alert&message=Hello
```

Response

```
{ "status": "ok" }
```

The status of **send.php** requests informs only if the request was sent to Holyrics open on the computer.

---

Request - request.php

```
https://api.holyrics.com.br/request.php?api_key=API_KEY&pwd=password&action=show_alert&message=Hello
```

Response

```
{
  "status": "ok",           <-  status of sending the request to the Holyrics
  "response_status": "ok",  <-  response status of sent request
  "response": {             <-  request response
    "status": "ok",         <-  request status
    "data": {               <-  javascript method response
      "status": "ok"
    }
  }
}
```

---

### Examples of error sending the request:

```
{
    "status": "error",
    "error": {
        "code": 9,
        "key": "device_disconnected",
        "message": "Device disconnected"
    }
}
```

```
{
    "status": "error",
    "error": {
        "code": 403,
        "key": "invalid_token",
        "message": "Invalid token"
    }
}
```

### Examples of request response error:

```
{
  "status": "ok",           <-  the request was sent to the computer
  "response_status": "ok",  <-  the response has been received
  "response": {             <-  response received from computer
      "status": "error",
      "error": "js unavailable"
    }
  }
}
```

```
{
    "status": "ok",               <-  the request was sent to the computer
    "response_status": "timeout"  <-  the time waiting for the response has expired
}
```

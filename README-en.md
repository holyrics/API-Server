# API-Server
**EN** | [PT](README.md)

The Holyrics program provides a communication interface to receive requests both over the local network and over the internet.
<br/>
When making a request, the default implementation of this documentation will be used.
<br/>
But it is also possible to implement a JavaScript method in the Holyrics program itself so that requests are redirected to this method and execute customized actions as needed.
<br/>
Use the `POST` method and the `Content-Type: application/json` to make the requests.
<br/>
It is necessary to add the `token` parameter to the request URL, which is created in the API Server settings, 'manage permissions' option.
<br/>
You can create multiple tokens and define the permissions of each token.
<br/>

To access the API Server settings:
<br/>
File menu > Settings > API Server

`v2.25.0+` Added compatibility with [ETag](https://developer.mozilla.org/pt-BR/docs/Web/HTTP/Headers/ETag)

# Example of a request over the local network

You can pass the token in the URL to make the request.
<br/>
However, as it is a local connection without SSL, if you want more security, use the "hash" method to send requests without having to inform the access token in the request.

default URL - Using token

```
http://[IP]:[PORT]/api/{action}?token=abcdef
```

default URL - Using hash

```
http://[IP]:[PORT]/api/{action}?dtoken=xyz123&sid=456&rid=3

dtoken
hash generated for each request from a 'nonce' (temporary token) obtained by the server

sid
Session ID, obtained along with the nonce

rid: request id
Sent by the client, a positive number that should always be greater than the one used in the previous request
```

Request

```
Using token
curl -X 'POST' \
  'http://ip:port/api/GetCPInfo?token=abcdef' \
  -H 'Content-Type: application/json' \
  -d '{}'
```

```
Using hash
nonce = '1a2b3c';  <- temporary token
rid   = 4;         <- request id
token = 'abcdef';  <- access token
data  = '{}';      <- request content

dtoken is the sha256 summary of the concatenation of the request information as example
dtoken = sha256(nonce + ':' + rid + ':' + token + ':' + data);
result: f3391f69cbe03940bd0d4a63ee191092aab2f3573f56b410a9cf94da05d4cdb5

curl -X 'POST' \
  'http://ip:port/api/GetCPInfo?sid=abc&rid=4&dtoken=f3391f69cbe03940bd0d4a63ee191092aab2f3573f56b410a9cf94da05d4cdb5' \
  -H 'Content-Type: application/json' \
  -d '{}'
```

Response

```
{
    "status": "ok",
    "data": {
        "text": "",
        "show": false,
        "display_ahead": false,
        "alert_text": "",
        "alert_show": false,
        "countdown_show": false,
        "countdown_time": 0
    }
}
```

---

Request

```
Using token
curl -X 'POST' \
  'http://ip:port/api/SetTextCP?token=abcdef' \
  -H 'Content-Type: application/json' \
  -d '{"text": "Example", "show": true, "display_ahead": true}'
```

```
Using hash
nonce = '1a2b3c';
rid   = 5;
token = 'abcdef';
data  = '{"text": "Example", "show": true, "display_ahead": true}';

dtoken = sha256(nonce + ':' + rid + ':' + token + ':' + data);
result: 02a7789759694c535cd032489bf101110837c972d76cec51c7ad7e797696749d

curl -X 'POST' \
  'http://ip:port/api/SetTextCP?sid=abc&rid=5&dtoken=02a7789759694c535cd032489bf101110837c972d76cec51c7ad7e797696749d' \
  -H 'Content-Type: application/json' \
  -d '{"text": "Example", "show": true, "display_ahead": true}'
```

Response

```
{ "status": "ok" }
```

---

In case of error, **status** will return **error**, example:

```
{
    "status": "error",
    "error": "invalid token"
}
```

### How to get a nonce

Use the 'Auth' action without including parameters in the URL to get a nonce
```
Request
http://ip:port/api/Auth

Response
{
    "status": "ok",
    "data": {
        "sid": "u80fbjbcknir",
        "nonce":"b58ba4f605bed27c40a20be53ee3cf3d"
    }
}
```

Use the 'Auth' action again to authenticate, passing the parameters in the URL

```
nonce = 'b58ba4f605bed27c40a20be53ee3cf3d';
rid   = 0;       <- should be zero for authentication
token = '1234';  <- access token
data  = 'auth';  <- should be 'auth' for authentication

dtoken = sha256(nonce + ':' + rid + ':' + token + ':' + data);
result: 5d632009dfde5e9771b4f98f1b28c88ac2f73ae1f9d81b62a9af241a304c4d7a

http://ip:port/api/Auth?sid=u80fbjbcknir&rid=0&dtoken=5d632009dfde5e9771b4f98f1b28c88ac2f73ae1f9d81b62a9af241a304c4d7a

Response
{ "status": "ok" }
```

# Example of a request over the internet

Use the **API_KEY** value available in the API Server settings to make requests to the Holyrics server endpoint along with the access token.

### Request - send
**Just sends the request without waiting for a return**

default URL
```
https://api.holyrics.com.br/send/{action}
```

Request
```
curl -X 'POST' \
  'https://api.holyrics.com.br/send/SetTextCP' \
  -H 'Content-Type: application/json' \
  -H 'api_key: API_KEY' \
  -H 'token: abcdef' \
  -d '{"text": "Example", "show": true, "display_ahead": true}'
```

Response

```
{ "status": "ok" }
```

The status of **send** requests only informs if the request was sent to Holyrics open on the computer.

---

### Request - request
Sends the request and waits for the response

default URL
```
https://api.holyrics.com.br/request/{action}
```

Request
```
curl -X 'POST' \
  'https://api.holyrics.com.br/request/GetCPInfo' \
  -H 'Content-Type: application/json' \
  -H 'api_key: API_KEY' \
  -H 'token: abcdef' \
  -d '{}'
```

Response

```
{
  "status": "ok",           <-  status of sending the request to Holyrics
  "response_status": "ok",  <-  status of the response of the sent request
  "response": {             <-  request response
    "status": "ok",
    "data": {
        "text": "Example",
        "show": true,
        "display_ahead": true,
        "alert_text": "",
        "alert_show": false,
        "countdown_show": false,
        "countdown_time": 0
    }
  }
}
```

---

### Examples of error in sending the request:

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
        "key": "invalid_api_key",
        "message": "Invalid API Key"
    }
}
```

### Examples of error in the request response:

```
{
  "status": "ok",           <-  the request was sent to the computer
  "response_status": "ok",  <-  the response was received
  "response": {             <-  response received from the computer
      "status": "error",
      "error": "invalid token"
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

# JavaScript implementation example

Use the methods of the [JSLIB](https://github.com/holyrics/jslib) class to create your own implementation.
<br/>
If you create your own implementation, you need to return 'true' if you want the program to consume the request using the default implementation.
<br/>
Any returned value that is different from 'true', the program will consider that the request has already been consumed and will respond to it with the returned value.

```javascript
function request(action, headers, content, info) {
    switch (action) {
        case 'my_custom_action':
            //execute your own action and response
            return {'status': 'ok'}; //  <-  Requisition response
    }
    //Returning 'true' for the program to consume this request with the default implementation
    return true;
}

```

### Method parameters

`action` - Action name

`headers` - Contains the request headers. Example: `headers.Authorization`

`content` - Object with the content extracted from the request. Example: `content.theme.id`

`info` - Request information.
<br/>
`info.client_address` Address of the request origin
<br/>
`info.token` access token used in the request
<br/>
`info.local` **true** if the request origin is from the local network
<br/>
`info.web` **true** if the request origin is from the internet
<br/>

# Available actions 
  - [GetTokenInfo](#gettokeninfo)
  - [CheckPermissions](#checkpermissions)
  - [GetLyrics](#getlyrics)
  - [GetSongs](#getsongs)
  - [SearchLyrics](#searchlyrics)
  - [ShowLyrics](#showlyrics)
  - [GetText](#gettext)
  - [GetTexts](#gettexts)
  - [SearchText](#searchtext)
  - [ShowText](#showtext)
  - [ShowVerse](#showverse)
  - [GetAudios](#getaudios)
  - [GetAudio](#getaudio)
  - [SetAudioItemProperty](#setaudioitemproperty)
  - [PlayAudio](#playaudio)
  - [PlayVideo](#playvideo)
  - [ShowImage](#showimage)
  - [ExecuteFile](#executefile)
  - [AudioExists](#audioexists)
  - [ShowAnnouncement](#showannouncement)
  - [GetCustomMessages](#getcustommessages)
  - [ShowCustomMessage](#showcustommessage)
  - [ShowQuickPresentation](#showquickpresentation)
  - [ShowCountdown](#showcountdown)
  - [ShowQuiz](#showquiz)
  - [QuizAction](#quizaction)
  - [GetAutomaticPresentations](#getautomaticpresentations)
  - [GetAutomaticPresentation](#getautomaticpresentation)
  - [PlayAutomaticPresentation](#playautomaticpresentation)
  - [GetAutomaticPresentationPlayerInfo](#getautomaticpresentationplayerinfo)
  - [AutomaticPresentationPlayerAction](#automaticpresentationplayeraction)
  - [GetMediaPlayerInfo](#getmediaplayerinfo)
  - [MediaPlayerAction](#mediaplayeraction)
  - [GetLyricsPlaylist](#getlyricsplaylist)
  - [AddLyricsToPlaylist](#addlyricstoplaylist)
  - [RemoveFromLyricsPlaylist](#removefromlyricsplaylist)
  - [SetLyricsPlaylistItem](#setlyricsplaylistitem)
  - [GetMediaPlaylist](#getmediaplaylist)
  - [SetMediaPlaylistItem](#setmediaplaylistitem)
  - [MediaPlaylistAction](#mediaplaylistaction)
  - [GetNextSongPlaylist](#getnextsongplaylist)
  - [GetNextMediaPlaylist](#getnextmediaplaylist)
  - [ShowNextSongPlaylist](#shownextsongplaylist)
  - [ShowNextMediaPlaylist](#shownextmediaplaylist)
  - [GetPreviousSongPlaylist](#getprevioussongplaylist)
  - [GetPreviousMediaPlaylist](#getpreviousmediaplaylist)
  - [ShowPreviousSongPlaylist](#showprevioussongplaylist)
  - [ShowPreviousMediaPlaylist](#showpreviousmediaplaylist)
  - [AddToPlaylist](#addtoplaylist)
  - [RemoveFromMediaPlaylist](#removefrommediaplaylist)
  - [SetPlaylistItemDuration](#setplaylistitemduration)
  - [GetSlideDescriptions](#getslidedescriptions)
  - [GetFavorites](#getfavorites)
  - [FavoriteAction](#favoriteaction)
  - [GetApis](#getapis)
  - [GetScripts](#getscripts)
  - [ApiAction](#apiaction)
  - [ScriptAction](#scriptaction)
  - [ApiRequest](#apirequest)
  - [GetCurrentPresentation](#getcurrentpresentation)
  - [CloseCurrentPresentation](#closecurrentpresentation)
  - [GetF8](#getf8)
  - [SetF8](#setf8)
  - [ToggleF8](#togglef8)
  - [ActionNext](#actionnext)
  - [ActionPrevious](#actionprevious)
  - [ActionGoToIndex](#actiongotoindex)
  - [ActionGoToSlideDescription](#actiongotoslidedescription)
  - [GetCurrentBackground](#getcurrentbackground)
  - [GetCurrentTheme](#getcurrenttheme)
  - [GetBackgrounds](#getbackgrounds)
  - [SetCurrentBackground](#setcurrentbackground)
  - [GetThumbnail](#getthumbnail)
  - [GetColorMap](#getcolormap)
  - [GetAlert](#getalert)
  - [SetAlert](#setalert)
  - [GetCurrentSchedule](#getcurrentschedule)
  - [GetSchedules](#getschedules)
  - [GetSavedPlaylists](#getsavedplaylists)
  - [LoadSavedPlaylist](#loadsavedplaylist)
  - [GetHistory](#gethistory)
  - [GetHistories](#gethistories)
  - [GetNearestHistory](#getnearesthistory)
  - [GetSongGroup](#getsonggroup)
  - [GetSongGroups](#getsonggroups)
  - [GetTeams](#getteams)
  - [GetMembers](#getmembers)
  - [GetRoles](#getroles)
  - [GetServices](#getservices)
  - [GetEvents](#getevents)
  - [GetAnnouncement](#getannouncement)
  - [GetAnnouncements](#getannouncements)
  - [GetCommunicationPanelInfo](#getcommunicationpanelinfo)
  - [SetCommunicationPanelSettings](#setcommunicationpanelsettings)
  - [StartCountdownCommunicationPanel](#startcountdowncommunicationpanel)
  - [StopCountdownCommunicationPanel](#stopcountdowncommunicationpanel)
  - [StartTimerCommunicationPanel](#starttimercommunicationpanel)
  - [StopTimerCommunicationPanel](#stoptimercommunicationpanel)
  - [SetTextCommunicationPanel](#settextcommunicationpanel)
  - [SetAlertCommunicationPanel](#setalertcommunicationpanel)
  - [CommunicationPanelCallAttention](#communicationpanelcallattention)
  - [GetWallpaperSettings](#getwallpapersettings)
  - [SetWallpaperSettings](#setwallpapersettings)
  - [GetDisplaySettings](#getdisplaysettings)
  - [SetDisplaySettings](#setdisplaysettings)
  - [GetTransitionEffectSettings](#gettransitioneffectsettings)
  - [SetTransitionEffectSettings](#settransitioneffectsettings)
  - [GetBibleVersions](#getbibleversions)
  - [GetBibleVersionsV2](#getbibleversionsv2)
  - [GetBibleSettings](#getbiblesettings)
  - [SetBibleSettings](#setbiblesettings)
  - [GetPresentationFooterSettings](#getpresentationfootersettings)
  - [SetPresentationFooterSettings](#setpresentationfootersettings)
  - [GetBpm](#getbpm)
  - [SetBpm](#setbpm)
  - [GetHue](#gethue)
  - [SetHue](#sethue)
  - [GetRuntimeEnvironment](#getruntimeenvironment)
  - [SetRuntimeEnvironment](#setruntimeenvironment)
  - [SetLogo](#setlogo)
  - [GetSyncStatus](#getsyncstatus)
  - [GetInterfaceInput](#getinterfaceinput)
  - [SetInterfaceInput](#setinterfaceinput)
  - [SelectVerse](#selectverse)
  - [OpenDrawLots](#opendrawlots)
  - [GetMediaDuration](#getmediaduration)
  - [GetVersion](#getversion)
  - [GetRealTimeSongKey](#getrealtimesongkey)
  - [SetRealTimeSongKey](#setrealtimesongkey)
  - [ActionNextQuickPresentation](#actionnextquickpresentation)
  - [ActionPreviousQuickPresentation](#actionpreviousquickpresentation)
  - [CloseCurrentQuickPresentation](#closecurrentquickpresentation)
  - [GetCurrentQuickPresentation](#getcurrentquickpresentation)
  - [GetTriggers](#gettriggers)
  - [GetGlobalSettings](#getglobalsettings)
  - [SetGlobalSettings](#setglobalsettings)
  - [GetStyledModels](#getstyledmodels)
  - [GetStyledModelsAsMap](#getstyledmodelsasmap)
  - [CreateItem](#createitem)
  - [EditItem](#edititem)
  - [DeleteItem](#deleteitem)
  - [AddSongsToSongGroup](#addsongstosonggroup)
  - [RemoveSongsFromSongGroup](#removesongsfromsonggroup)


---

### GetTokenInfo
- v2.25.0

Gets the token information



**Response:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `data.version` | _String_ | Version in the format: X.Y.Z |
| `data.permissions` | _String_ | Allowed actions for the token, separated by commas |


**Example:**
```
Response
{
  "status": "ok",
  "data": {
    "version": "2.25.0",
    "permissions": "GetSongs,GetFavorites"
  }
}
```


---

### CheckPermissions
- v2.25.0

Checks if the token has the required permissions.<br>Returns `status=ok` if the token has all the permissions required in the `actions` parameter.<br>Returns `code 401` if the token does not have all the required permissions.

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `actions` | _String_ | List of required actions for the token, separated by commas |


**Response:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| <br>Available if **status=error** |  |  |
| `error.unauthorized_actions` | _String (optional)_ | Actions not allowed, separated by comma |
| `error.request_status` | _String (optional)_ | `pending` a notification was created requesting permission from the user in the program interface<br> <br>`denied` the permission request was denied |


**Example:**
```
Request
{
  "actions": "GetSongs,GetFavorites"
}

Response
{
  "status": "error",
  "error": {
    "unauthorized_actions": "GetFavorites",
    "request_status": "pending"
  }
}
```


---

### GetLyrics
### GetSong
- v2.19.0

Returns a song.

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `id` | _String_ | Song ID |
| `fields` | _String (optional)_ | Comma-separated field names. If this field is declared, only the specified fields will be returned `v2.24.0+` |


**Response:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `data` | _[Lyrics](#lyrics)_ | Music or NULL if not found |


**Example:**
```
Request
{
  "id": "123"
}

Response
{
  "status": "ok",
  "data": {
    "id": "123",
    "title": "",
    "artist": "",
    "author": "",
    "note": "",
    "key": "",
    "bpm": 0,
    "time_sig": "",
    "groups": [
      {
        "name": "Group 1"
      },
      {
        "name": "Group 2"
      }
    ],
    "archived": false
  }
}
```


---

### GetSongs
- v2.21.0

Returns the list of songs

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `fields` | _String (optional)_ | Comma-separated field names. If this field is declared, only the specified fields will be returned `v2.24.0+` |


**Response:**

| Name | Type  |
| ---- | :---: |
| `data` | _Array&lt;[Lyrics](#lyrics)&gt;_| 


**Example:**
```
Request
{
  "fields": "id,title,artist,author"
}

Response
{
  "status": "ok",
  "data": [
    {
      "id": "0",
      "title": "",
      "artist": "",
      "author": "",
      "note": "",
      "copyright": "",
      "key": "",
      "bpm": 0.0,
      "time_sig": "",
      "groups": [],
      "extras": {
        "extra": ""
      },
      "archived": false
    },
    {
      "...": "..."
    },
    {
      "...": "..."
    }
  ]
}
```


---

### SearchLyrics
### SearchSong
- v2.19.0

Performs a search in the user's lyrics list

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `input` | _String_ | Filter |
| `text` | _String_ | Text to be searched |
| `title` | _Boolean (optional)_ |  `Default: true` |
| `artist` | _Boolean (optional)_ |  `Default: true` |
| `note` | _Boolean (optional)_ |  `Default: true` |
| `lyrics` | _Boolean (optional)_ |  `Default: false` |
| `group` | _String (optional)_ |  |
| `fields` | _String (optional)_ | Comma-separated field names. If this field is declared, only the specified fields will be returned `v2.24.0+` |


**Response:**

| Name | Type  |
| ---- | :---: |
| `data` | _Array&lt;[Lyrics](#lyrics)&gt;_| 


**Example:**
```
Request
{
  "text": "abc"
}

Response
{
  "status": "ok",
  "data": [
    {
      "id": "123",
      "title": "abc1",
      "artist": "",
      "author": "",
      "note": "",
      "key": "",
      "bpm": 0,
      "time_sig": "",
      "groups": [],
      "archived": false
    },
    {
      "id": "456",
      "title": "abc2",
      "artist": "",
      "author": "",
      "note": "",
      "key": "",
      "bpm": 0,
      "time_sig": "",
      "groups": [],
      "archived": false
    }
  ]
}
```


---

### ShowLyrics
### ShowSong
- v2.19.0

Starts a lyric show.

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `id` | _String_ | Item ID |
| `initial_index` | _Number (optional)_ | Initial index of the presentation `Default: 0` `v2.23.0+` |


_Method does not return value_

**Example:**
```
Request
{
  "id": "123"
}
```


---

### GetText
- v2.21.0

Returns a text.

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `id` | _String_ | Text ID |
| `fields` | _String (optional)_ | Comma-separated field names. If this field is declared, only the specified fields will be returned `v2.24.0+` |


**Response:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `data` | _[Text](#text)_ | Text or NULL if not found |


**Example:**
```
Request
{
  "id": "xyz"
}

Response
{
  "status": "ok",
  "data": {
    "id": "",
    "title": "",
    "folder": "",
    "theme": null,
    "slides": [
      {
        "text": "Slide 1 line 1\nSlide 1 line 2",
        "background_id": null
      },
      {
        "text": "Slide 2 line 1\nSlide 2 line 2",
        "background_id": null
      },
      {
        "text": "Slide 3 line 1\nSlide 3 line 2",
        "background_id": null
      }
    ],
    "extras": {}
  }
}
```


---

### GetTexts
- v2.21.0

Returns the list of texts

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `fields` | _String (optional)_ | Comma-separated field names. If this field is declared, only the specified fields will be returned `v2.24.0+` |


**Response:**

| Name | Type  |
| ---- | :---: |
| `data` | _Array&lt;[Text](#text)&gt;_| 


**Example:**
```
Request
{
  "fields": "id,title,folder"
}

Response
{
  "status": "ok",
  "data": [
    {
      "id": "",
      "title": "",
      "folder": "",
      "theme": null
    },
    {
      "...": "..."
    },
    {
      "...": "..."
    }
  ]
}
```


---

### SearchText
- v2.21.0

Performs a search in the user's text list

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `input` | _String_ | Filter |
| `text` | _String_ | Text to be searched |
| `fields` | _String (optional)_ | Comma-separated field names. If this field is declared, only the specified fields will be returned `v2.24.0+` |


**Response:**

| Name | Type  |
| ---- | :---: |
| `data` | _Array&lt;[Lyrics](#lyrics)&gt;_| 


**Example:**
```
Request
{
  "text": "example"
}

Response
{
  "status": "ok",
  "data": [
    {
      "id": "",
      "title": "",
      "folder": "",
      "theme": null
    },
    {
      "...": "..."
    },
    {
      "...": "..."
    }
  ]
}
```


---

### ShowText
- v2.19.0

Starts a presentation of a text tab item.

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `id` | _String_ | Item ID |
| `initial_index` | _Number (optional)_ | Initial index of the presentation `Default: 0` `v2.23.0+` |


_Method does not return value_

**Example:**
```
Request
{
  "id": "abc"
}
```


---

### ShowVerse
- v2.19.0

Starts a Bible verse presentation.<br>Note: It is possible to display a maximum of 100 different verses in a single request.

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `input` | _Object_ | **id**, **ids** or **references** |
| `id` | _String (optional)_ | To display a verse. Item ID in BBCCCVVV format.<br/>Example: '19023001' (book 19, chapter 023, verse 001) |
| `ids` | _Array&lt;String&gt; (optional)_ | To display a list of verses. List with the ID of each verse.<br/>Example: ['19023001', '43003016', '45012002'] |
| `references` | _String (optional)_ | References. Example: **John 3:16** or **Rm 12:2** or **Gn 1:1-3 Sl 23.1** |
| `version` | _String (optional)_ | Name or abbreviation of the translation used `v2.21.0+` |
| `quick_presentation` | _Boolean (optional)_ | `true` to display the verse through a quick presentation popup window.<br>Allows, for example, to start the presentation of a verse without ending the current presentation, returning to the current presentation when the verse presentation ends. `Default: false` `v2.24.0+` |


_Method does not return value_

**Example:**
```
Request
{
  "id": "19023001",
  "ids": [
    "19023001",
    "43003016",
    "45012002"
  ],
  "references": "Rm 12:2  Gn 1:1-3  Sl 23"
}
```


---

### GetAudios
### GetVideos
### GetImages
### GetFiles
- v2.19.0

Returns the list of files from the respective tab: audio, video, image, file

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `folder` | _String (optional)_ | Subfolder name to list files |
| `filter` | _String (optional)_ | Filter files by name |
| `include_metadata` | _Boolean (optional)_ | Add metadata to the response `Default: false` `v2.22.0+` |
| `include_thumbnail` | _Boolean (optional)_ | Add thumbnail to response (80x45) `Default: false` `v2.22.0+` |


**Response:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `data` | _Array&lt;Object&gt;_ |  |
| `data.*.name` | _String_ | Item name |
| `data.*.isDir` | _Boolean_ | Return **true** if it's a folder or **false** if it's a file. |
| `data.*.properties` | _Object_ | Map with the custom properties defined for the file `v2.24.0+` |
| <br>Available if **include_metadata=true** |  |  |
| `data.*.length` | _Number_ | File size (bytes). Available if **isDir=false** `v2.22.0+` |
| `data.*.modified_time` | _String_ | File modification date. Date and time format: YYYY-MM-DD HH:MM `v2.22.0+` |
| `data.*.modified_time_millis` | _String_ | File modification date. (timestamp) `v2.24.0+` |
| `data.*.duration_ms` | _Number_ | File duration. Available if the file is: audio or vídeo `v2.22.0+` |
| `data.*.width` | _Number_ | Width. Available if the file is: imagem or vídeo `v2.22.0+` |
| `data.*.height` | _Number_ | Height. Available if the file is: imagem or vídeo `v2.22.0+` |
| `data.*.position` | _String_ | Image adjustment. Available for images. Can be: `adjust` `extend` `fill` `v2.22.0+` |
| `data.*.blur` | _Boolean_ | Apply blur effect `v2.22.0+` |
| `data.*.transparent` | _Boolean_ | Display images with transparency `v2.22.0+` |
| `data.*.last_executed_time` | _String_ | Date of the last execution of the file. Date and time format: YYYY-MM-DD HH:MM `v2.24.0+` |
| `data.*.last_executed_time_millis` | _Number_ | Date of the last execution of the file. (timestamp) `v2.24.0+` |
| <br>Available if **include_thumbnail=true** |  |  |
| `data.*.thumbnail` | _String_ | Image in base64 format `v2.22.0+` |


**Example:**
```
Request
{
  "folder": "folder_name",
  "filter": "abc"
}

Response
{
  "status": "ok",
  "data": [
    {
      "name": "abcd",
      "isDir": true,
      "properties": {}
    },
    {
      "name": "abcd.jpg",
      "isDir": false,
      "properties": {}
    }
  ]
}
```


---

### GetAudio
### GetVideo
### GetImage
### GetFile
- v2.24.0

Returns the data of a file from the list of files in the respective tab: audio, video, image, file

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `name` | _String_ | File name (including subfolder) |
| `include_metadata` | _Boolean (optional)_ | Add metadata to the response `Default: false` |
| `include_thumbnail` | _Boolean (optional)_ | Add thumbnail to response (80x45) `Default: false` |


**Response:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `data` | _Object_ |  |
| `data.name` | _String_ | Item name |
| `data.isDir` | _Boolean_ | Return **true** if it's a folder or **false** if it's a file. |
| `data.properties` | _Object_ | Map with the custom information saved in the file |
| <br>Available if **include_metadata=true** |  |  |
| `data.length` | _Number_ | File size (bytes). Available if **isDir=false** |
| `data.modified_time` | _String_ | File modification date. Date and time format: YYYY-MM-DD HH:MM |
| `data.modified_time_millis` | _Number_ | File modification date. (timestamp) |
| `data.duration_ms` | _Number_ | File duration. Available if the file is: audio or vídeo |
| `data.width` | _Number_ | Width. Available if the file is: imagem or vídeo |
| `data.height` | _Number_ | Height. Available if the file is: imagem or vídeo |
| `data.position` | _String_ | Image adjustment. Available for images. Can be: `adjust` `extend` `fill` |
| `data.blur` | _Boolean_ | Apply blur effect |
| `data.transparent` | _Boolean_ | Display images with transparency |
| `data.last_executed_time` | _String_ | Date of the last execution of the file. Date and time format: YYYY-MM-DD HH:MM |
| `data.last_executed_time_millis` | _Number_ |  |
| <br>Available if **include_thumbnail=true** |  |  |
| `data.thumbnail` | _String_ | Image in base64 format |


**Example:**
```
Request
{
  "name": "filename.mp3",
  "include_metadata": true
}

Response
{
  "status": "ok",
  "data": {
    "name": "",
    "isDir": false,
    "length": 0,
    "modified_time": "",
    "modified_time_millis": 0,
    "duration_ms": 0,
    "width": 0,
    "height": 0,
    "position": "",
    "blur": false,
    "transparent": false,
    "last_executed_time": "",
    "last_executed_time_millis": 0,
    "properties": {}
  }
}
```


---

### SetAudioItemProperty
### SetVideoItemProperty
### SetImageItemProperty
### SetFileItemProperty
- v2.24.0

Changes the custom information of a file

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `name` | _String_ | File name (including subfolder) |
| `properties` | _Object_ | Key/value map with the information that will be changed. The passed values will be MERGED with the existing values. That is, it is not necessary to send parameters that will not be changed (or removed). |


_Method does not return value_

**Example:**
```
Request
{
  "name": "filename.mp3",
  "properties": {
    "key": "value 1",
    "abc": "value 2",
    "example": "value 3"
  }
}
```


---

### PlayAudio
- v2.19.0

Play an audio or a list of audios (folder)

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `file` | _String_ | File or folder name. Example: **file.mp3** or **folder** or **folder/file.mp3** |
| `settings` | _[PlayMediaSettings](#play-media-settings) (optional)_ | Settings for media execution `v2.21.0+` |


_Method does not return value_

**Example:**
```
Request
{
  "file": "audio.mp3"
}
```


---

### PlayVideo
- v2.19.0

Play a video or a list of videos (folder)

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `file` | _String_ | File or folder name. Example: **file.mp4** or **folder** or **folder/file.mp4** |
| `settings` | _[PlayMediaSettings](#play-media-settings) (optional)_ | Settings for media execution `v2.21.0+` |


_Method does not return value_

**Example:**
```
Request
{
  "file": "video.mp4"
}
```


---

### ShowImage
- v2.19.0

Displays an image or a list of images (folder)

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `file` | _String_ | File or folder name. Example: **file.jpg** or **folder** or **folder/file.jpg** |
| `automatic` | _[Automatic](#automatic) (optional)_ | If informed, the presentation of the items will be automatic |


_Method does not return value_

**Example:**
```
Request
{
  "file": "image.jpg"
}
```


---

### ExecuteFile
- v2.21.0

Executes a file. Only safe extensions or those added to the exception list.

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `file` | _String_ | File name |


_Method does not return value_

**Example:**
```
Request
{
  "file": "file.txt"
}
```


---

### AudioExists
### VideoExists
### ImageExists
### FileExists
- v2.21.0

Checks if there is a file with the informed name

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `file` | _String_ | File name |


**Response:**

| Name | Type  |
| ---- | :---: |
| `data` | _Boolean_| 


**Example:**
```
Request
{
  "file": "file.mp3"
}

Response
{
  "status": "ok",
  "data": true
}
```


---

### ShowAnnouncement
- v2.19.0

Displays an announcement or a list of announcements

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `id` | _String (optional)_ | Announcement ID. Can be **all** to display all |
| `ids` | _Array&lt;String&gt; (optional)_ | List with the ID of each announcement |
| `name` | _String (optional)_ | Announcement name |
| `names` | _Array&lt;String&gt; (optional)_ | List with the name of each announcement |
| `automatic` | _[Automatic](#automatic) (optional)_ | If informed, the presentation of the items will be automatic |


_Method does not return value_

**Example:**
```
Request
{
  "id": "all",
  "ids": [],
  "name": "",
  "names": [],
  "automatic": {
    "seconds": 10,
    "repeat": true
  }
}
```


---

### GetCustomMessages
- v2.19.0

List of custom messages



**Response:**

| Name | Type  |
| ---- | :---: |
| `data` | _Array&lt;[CustomMessage](#custom-message)&gt;_| 


**Example:**
```
Response
{
  "status": "ok",
  "data": [
    {
      "id": "123",
      "name": "Carro",
      "message_model": "Atenção proprietário do veículo    , placa   -  .",
      "message_example": "Atenção proprietário do veículo modelo cor, placa placa - motivo_carro.",
      "variables": [
        {
          "position": 32,
          "name": "modelo",
          "only_number": false,
          "uppercase": false,
          "suggestions": []
        },
        {
          "position": 34,
          "name": "cor",
          "only_number": false,
          "uppercase": false,
          "suggestions": []
        },
        {
          "position": 43,
          "name": "placa",
          "only_number": false,
          "uppercase": true,
          "suggestions": []
        },
        {
          "position": 47,
          "name": "motivo_carro",
          "only_number": false,
          "uppercase": false,
          "suggestions": [
            "Favor comparecer ao estacionamento",
            "Faróis acesos",
            "Alarme disparado",
            "Remover o veículo do local",
            "Comparecer ao estacionamento com urgência"
          ]
        }
      ]
    }
  ]
}
```


---

### ShowCustomMessage
- v2.19.0

Display a custom message. Note: A custom message is not displayed directly on the screen. a notification is created in the corner of the screen for the operator to accept and display.

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `name` | _String_ | custom message name |
| `position_?` | _Object (optional)_ | Variable added at the specified position according to the value returned in **variables.*.position** of the CustomMessage class. |
| `params` | _Object (optional)_ | Alternative method. Key/value map where the key is the field name **variables.*.name** of the CustomMessage class. If necessary to add the same parameter, add `*_n` at the end of the name, starting at 2<br>Example: **params.name, params.name_2, params.name_3** `v2.21.0+` |
| `note` | _String_ | Extra information displayed in popup window for operator |


_Method does not return value_

**Example:**
```
Request
{
  "name": "Carro",
  "position_32": "modelo",
  "position_34": "cor",
  "position_43": "placa",
  "position_47": "motivo",
  "note": "..."
}
```


---

### ShowQuickPresentation
- v2.19.0

Quick display of text

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `text` | _String_ | Text that will be displayed [Styled Text](https://github.com/holyrics/Scripts/blob/main/i18n/en/StyledText.md) from v2.19.0<br>Optional if `slides` is declared |
| `slides` | _Array&lt;[QuickPresentationSlide](#quick-presentation-slide)&gt;_ | Alternative parameter for more complex presentations<br>Optional if `text` is declared `v2.23.0+` |
| `theme` | _[ThemeFilter](#theme-filter) (optional)_ | Filter selected theme for display |
| `custom_theme` | _[Theme](#theme) (optional)_ | Custom theme used to display the text `v2.21.0+` |
| `automatic` | _[Automatic](#automatic) (optional)_ | If informed, the presentation of the items will be automatic |
| `initial_index` | _Number (optional)_ | Initial index of the presentation `Default: 0` `v2.23.0+` |


_Method does not return value_

**Example:**
```
Request
{
  "text": "Example",
  "theme": {
    "name": "Theme name"
  }
}
```


---

### ShowCountdown
- v2.20.0

Display a countdown on the public screen

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `time` | _String_ | HH:MM or MM:SS |
| `exact_time` | _Boolean (optional)_ | If **true**, `time` should be HH:MM (exact hour and minute). If **false**, `time` should be MM:SS (amount of minutes and seconds) `Default: false` |
| `text_before` | _String (optional)_ | Text displayed at the top of the countdown |
| `text_after` | _String (optional)_ | Text displayed at the bottom of the countdown |
| `zero_fill` | _Boolean (optional)_ | Fill in the 'minute' field with zero on the left `Default: false` |
| `hide_zero_minute` | _Boolean (optional)_ | Hide the display of minutes when it is zero `Default: false` `v2.25.0+` |
| `countdown_relative_size` | _Number (optional)_ | Relative size of the countdown `Default: 250` |
| `theme` | _[ThemeFilter](#theme-filter) (optional)_ | Filter selected theme for display `v2.21.0+` |
| `countdown_style` | _[FontSettings](#font-settings) (optional)_ | Custom font for countdown `v2.21.0+` |
| `custom_theme` | _[Theme](#theme) (optional)_ | Custom theme `v2.21.0+` |


_Method does not return value_

**Example:**
```
Request
{
  "time": "05:00",
  "zero_fill": true
}
```


---

### ShowQuiz
- v2.20.0

Start a multiple choice presentation

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `questions` | _Array&lt;[QuizQuestion](#quiz-question)&gt;_ | Questions to display |
| `settings` | _[QuizSettings](#quiz-settings)_ | Settings |


_Method does not return value_

**Example:**
```
Request
{
  "questions": [
    {
      "name": "Name",
      "title": "Example 1",
      "alternatives": [
        "Example 1a",
        "Example 2a",
        "Example 3a",
        "Example 4a"
      ],
      "correct_alternative_number": 2
    },
    {
      "title": "Example 2",
      "alternatives": [
        "Example 1b",
        "Example 2b",
        "Example 3b",
        "Example 4b"
      ],
      "correct_alternative_number": 2
    }
  ],
  "settings": {
    "display_alternatives_one_by_one": false,
    "alternative_char_type": "number"
  }
}
```


---

### QuizAction
- v2.20.0

Execute an action in a multiple choice presentation

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `action` | _String (optional)_ | One of the following values: `previous_slide`  `next_slide`  `previous_question`  `next_question`  `show_result`  `close` |
| `hide_alternative` | _Number (optional)_ | Hide an alternative. Starts at 1 |
| `select_alternative` | _Number (optional)_ | Select an alternative. Starts at 1 |
| `countdown` | _Number (optional)_ | Start a countdown. [1-120] |


_Method does not return value_

**Example:**
```
Request
{
  "action": "next_slide"
}
```


---

### GetAutomaticPresentations
### GetAPs
- v2.21.0

Returns the list of automatic presentations



**Response:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `data` | _Array&lt;Object&gt;_ |  |
| `data.*.name` | _String_ | File name. Example: **file.ap** |


**Example:**
```
Response
{
  "status": "ok",
  "data": [
    {
      "name": "file 1.ap"
    },
    {
      "name": "file 2.ap"
    },
    {
      "name": "file 3.ap"
    }
  ]
}
```


---

### GetAutomaticPresentation
### GetAP
- v2.21.0

Returns an automatic presentation

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `file` | _String_ | File name. Example: **file.ap** |


**Response:**

| Name | Type  |
| ---- | :---: |
| `data` | _[AutomaticPresentation](#automatic-presentation)_| 


**Example:**
```
Request
{
  "file": "filename.ap"
}

Response
{
  "status": "ok",
  "data": {
    "name": "filename.ap",
    "duration": 300000,
    "starts_with": "title",
    "song": {},
    "timeline": []
  }
}
```


---

### PlayAutomaticPresentation
### PlayAP
- v2.19.0

Play an automatic presentation item

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `file` | _String_ | File name. Example: **file.ap** |
| `theme` | _Object (optional)_ | Filter selected theme for display |
| `theme.id` | _String (optional)_ | Theme ID |
| `theme.name` | _String (optional)_ | Theme name |
| `theme.edit` | _[Theme](#theme) (optional)_ | Settings to modify the selected Theme for display `v2.21.0+` |
| `custom_theme` | _[Theme](#theme) (optional)_ | Custom theme used to display the automatic presentation `v2.21.0+` |


_Method does not return value_

**Example:**
```
Request
{
  "file": "filename"
}
```


---

### GetAutomaticPresentationPlayerInfo
### GetAPPlayerInfo
- v2.20.0

Returns the information of the automatic presentation on display



**Response:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `data.name` | _String_ | Item name |
| `data.playing` | _Boolean_ | Checks if the player is running |
| `data.time_ms` | _Number_ | Current media time in milliseconds |
| `data.volume` | _Number_ | Current player volume. Minimum=0, Maximum=100 |
| `data.mute` | _Boolean_ | Checks if the **mute** option is enabled |
| `data.duration_ms` | _Number_ | Total time in milliseconds `v2.21.0+` |


**Example:**
```
Response
{
  "status": "ok",
  "data": {
    "data": {
      "name": "example",
      "playing": true,
      "time_ms": 34552,
      "volume": 90,
      "mute": false
    }
  }
}
```


---

### AutomaticPresentationPlayerAction
### APPlayerAction
- v2.20.0

Perform actions in the player

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `action` | _String (optional)_ | Name of the action that will be performed on the player. play, pause, stop |
| `volume` | _Number (optional)_ | Change the volume of the player. Minimum=0, Maximum=100 |
| `mute` | _Boolean (optional)_ | Change the **mute** option |
| `time_ms` | _Boolean (optional)_ | Change the current media time in milliseconds |


_Method does not return value_

**Example:**
```
Request
{
  "action": "play",
  "volume": 80
}
```


---

### GetMediaPlayerInfo
- v2.19.0

Returns player information



**Response:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `data.name` | _String_ | Name of current media in player |
| `data.path` | _String_ | Full path of media in player |
| `data.relative_path` | _String_ | Relative path of the media in the player. Can be null. `v2.24.0+` |
| `data.playing` | _Boolean_ | Checks if the player is running |
| `data.duration_ms` | _Number_ | Total time in milliseconds |
| `data.time_ms` | _Number_ | Current media time in milliseconds |
| `data.time_elapsed` | _String_ | Elapsed time in format HH:MM:SS |
| `data.time_remaining` | _String_ | Remaining time in format HH:MM:SS |
| `data.volume` | _Number_ | Current player volume. Minimum=0, Maximum=100 |
| `data.mute` | _Boolean_ | Checks if the **mute** option is enabled |
| `data.repeat` | _Boolean_ | Checks if the **repeat** option is enabled |
| `data.execute_single` | _Boolean_ | Checks if the player is set to play only the current list item |
| `data.shuffle` | _Boolean_ | Checks if the **random** option is enabled |
| `data.fullscreen` | _Boolean_ | Checks if the **full screen** option is enabled |


**Example:**
```
Response
{
  "status": "ok",
  "data": {
    "name": "video.mp4",
    "path": "C:\\Holyrics\\Holyrics\\files\\media\\video\\video.mp4",
    "relative_path": "video\\video.mp4",
    "playing": false,
    "duration_ms": 321456,
    "time_ms": -1,
    "time_elapsed": "00:00",
    "time_remaining": "00:00",
    "volume": 80,
    "mute": false,
    "repeat": true,
    "execute_single": true,
    "shuffle": false,
    "fullscreen": false
  }
}
```


---

### MediaPlayerAction
- v2.19.0

Perform actions in the player

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `action` | _String (optional)_ | Name of the action that will be performed on the player. play, pause, stop, next, previous |
| `volume` | _Number (optional)_ | Change the volume of the player. Minimum=0, Maximum=100 |
| `mute` | _Boolean (optional)_ | Change the **mute** option |
| `repeat` | _Boolean (optional)_ | Change the **repeat** option |
| `shuffle` | _Boolean (optional)_ | Change the **random** option |
| `execute_single` | _Boolean (optional)_ | Change player setting to play only current list item |
| `fullscreen` | _Boolean (optional)_ | Change player **full screen** option |
| `time_ms` | _Boolean (optional)_ | Change the current media time in milliseconds `v2.20.0+` |


_Method does not return value_

**Example:**
```
Request
{
  "action": "stop",
  "volume": 80,
  "mute": false,
  "repeat": false,
  "shuffle": false,
  "execute_single": false,
  "fullscreen": false
}
```


---

### GetLyricsPlaylist
### GetSongPlaylist
- v2.19.0

Lyrics playlist



**Response:**

| Name | Type  |
| ---- | :---: |
| `data` | _Array&lt;[Lyrics](#lyrics)&gt;_| 


**Example:**
```
Response
{
  "status": "ok",
  "data": [
    {
      "id": "123",
      "title": "",
      "artist": "",
      "author": "",
      "note": "",
      "key": "",
      "bpm": 0,
      "time_sig": "",
      "groups": [],
      "archived": false
    },
    {
      "id": "456",
      "title": "",
      "artist": "",
      "author": "",
      "note": "",
      "key": "",
      "bpm": 0,
      "time_sig": "",
      "groups": [],
      "archived": false
    }
  ]
}
```


---

### AddLyricsToPlaylist
### AddSongsToPlaylist
- v2.19.0

Add song lyrics to playlist

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `id` | _String (optional)_ | Lyrics ID |
| `ids` | _Array&lt;String&gt; (optional)_ | List with id of each lyics |
| `index` | _Number (optional)_ | Position in the list where the item will be added (starts at zero). Items are added to the end of the list by default. `Default: -1` |
| `media_playlist` | _Boolean (optional)_ | Add the lyrics to the media playlist `Default: false` |


_Method does not return value_

**Example:**
```
Request
{
  "id": "123",
  "ids": [
    123,
    456,
    789
  ],
  "index": 3,
  "media_playlist": false
}
```


---

### RemoveFromLyricsPlaylist
### RemoveFromSongPlaylist
- v2.19.0

Remove lyrics from playlist

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `id` | _String (optional)_ | Lyrics ID |
| `ids` | _Array&lt;String&gt; (optional)_ | List with id of each lyics |
| `index` | _Number (optional)_ | Position of the item in the list to be removed (starts at zero). |
| `indexes` | _Array&lt;Number&gt; (optional)_ | List with the position of each item in the list that will be removed. (Starts at zero) |


_Method does not return value_

**Example:**
```
Request
{
  "id": "123",
  "ids": [
    "123",
    "456",
    "789"
  ],
  "index": 3,
  "indexes": [
    3,
    4,
    5
  ]
}
```


---

### SetLyricsPlaylistItem
### SetSongPlaylistItem
- v2.22.0

Change a Lyrics Playlist Item

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `index` | _Number_ | Index of the item in the list |
| `song_id` | _String_ | New item |


_Method does not return value_

**Example:**
```
Request
{
  "index": 2,
  "song_id": "123"
}
```


---

### GetMediaPlaylist
- v2.19.0

Media playlist



**Response:**

| Name | Type  |
| ---- | :---: |
| `data` | _Array&lt;[Item](#item)&gt;_| 


**Example:**
```
Response
{
  "status": "ok",
  "data": [
    {
      "id": "xyz",
      "type": "image",
      "name": "image.jpg"
    },
    {
      "id": "abc",
      "type": "video",
      "name": "video.mp4"
    }
  ]
}
```


---

### SetMediaPlaylistItem
- v2.22.0

Change a media playlist item

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `index` | _Number_ | Index of the item in the list |
| `item` | _[AddItem](#add-item)_ | New item |


_Method does not return value_

**Example:**
```
Request
{
  "index": 0,
  "item": {
    "type": "song",
    "id": "123"
  }
}
```


---

### MediaPlaylistAction
- v2.19.0

Play a media playlist item

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `id` | _String_ | Item ID |


_Method does not return value_

**Example:**
```
Request
{
  "id": "abc"
}
```


---

### GetNextSongPlaylist
- v2.22.0

Returns the next song in the playlist. Can be null



**Response:**

| Name | Type  |
| ---- | :---: |
| `data` | _[Lyrics](#lyrics)_| 


**Example:**
```
Response
{
  "status": "ok",
  "data": {
    "id": "123",
    "title": "",
    "artist": "",
    "author": "",
    "note": "",
    "key": "",
    "bpm": 0,
    "time_sig": "",
    "groups": [],
    "archived": false
  }
}
```


---

### GetNextMediaPlaylist
- v2.22.0

Returns the next executable item from the media playlist. Can be null



**Response:**

| Name | Type  |
| ---- | :---: |
| `data` | _[Item](#item)_| 


**Example:**
```
Response
{
  "status": "ok",
  "data": {
    "id": "abc123",
    "type": "song",
    "name": ""
  }
}
```


---

### ShowNextSongPlaylist
- v2.22.0

Play the next song in the playlist



_Method does not return value_



---

### ShowNextMediaPlaylist
- v2.22.0

Plays the next item in the media playlist



_Method does not return value_



---

### GetPreviousSongPlaylist
- v2.22.0

Returns the previous song in the playlist. Can be null



**Response:**

| Name | Type  |
| ---- | :---: |
| `data` | _[Lyrics](#lyrics)_| 


**Example:**
```
Response
{
  "status": "ok",
  "data": {
    "id": "123",
    "title": "",
    "artist": "",
    "author": "",
    "note": "",
    "key": "",
    "bpm": 0,
    "time_sig": "",
    "groups": [],
    "archived": false
  }
}
```


---

### GetPreviousMediaPlaylist
- v2.22.0

Returns the previous executable item from the media playlist. Can be null



**Response:**

| Name | Type  |
| ---- | :---: |
| `data` | _[Item](#item)_| 


**Example:**
```
Response
{
  "status": "ok",
  "data": {
    "id": "abc123",
    "type": "song",
    "name": ""
  }
}
```


---

### ShowPreviousSongPlaylist
- v2.22.0

Play the previous song in the playlist



_Method does not return value_



---

### ShowPreviousMediaPlaylist
- v2.22.0

Plays the previous item in the media playlist



_Method does not return value_



---

### AddToPlaylist
- v2.20.0

Add items to the media playlist

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `items` | _Array&lt;[AddItem](#add-item)&gt;_ | List with the items that will be added |
| `index` | _Number (optional)_ | Position in the list where the item will be added (starts at zero). Items are added to the end of the list by default. `Default: -1` |
| `ignore_duplicates` | _Boolean (optional)_ | Do not duplicate items when adding new items, that is, do not add an item if it is already on the list. `Default: false` |


_Method does not return value_

**Example:**
```
Request
{
  "items": [
    {
      "type": "title",
      "name": "Título",
      "background_color": "000080"
    },
    {
      "type": "song",
      "id": 12345678
    },
    {
      "type": "verse",
      "id": "19023001"
    },
    {
      "type": "verse",
      "ids": [
        "19023001",
        "43003016"
      ]
    },
    {
      "type": "verse",
      "references": "Sl 23.1 Rm 12:2"
    },
    {
      "type": "text",
      "id": "abcxyz"
    },
    {
      "type": "audio",
      "name": "example.mp3"
    },
    {
      "type": "video",
      "name": "example.mp4"
    },
    {
      "type": "image",
      "name": "example.jpg"
    },
    {
      "type": "automatic_presentation",
      "name": "example.ap"
    },
    {
      "type": "title",
      "name": "Título 2"
    },
    {
      "type": "announcement",
      "id": 12345678
    },
    {
      "type": "announcement",
      "ids": [
        123,
        456
      ]
    },
    {
      "type": "announcement",
      "name": "example"
    },
    {
      "type": "announcement",
      "names": [
        "example 2",
        "example 3"
      ]
    },
    {
      "type": "announcement",
      "id": "all",
      "automatic": {
        "seconds": 8,
        "repeat": false
      }
    },
    {
      "type": "title",
      "name": "Título 3"
    },
    {
      "type": "countdown",
      "time": "03:15"
    },
    {
      "type": "countdown_cp",
      "minutes": 15,
      "stop_at_zero": true
    },
    {
      "type": "cp_text",
      "text": "example"
    },
    {
      "type": "script",
      "id": "abcxyz"
    },
    {
      "type": "api",
      "id": "abcxyz"
    }
  ]
}
```


---

### RemoveFromMediaPlaylist
- v2.19.0

Remove items from media playlist

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `id` | _String (optional)_ | Item ID |
| `ids` | _Array&lt;String&gt; (optional)_ | List with id of each item |
| `index` | _Number (optional)_ | Position of the item in the list to be removed (starts at zero). |
| `indexes` | _Array&lt;Number&gt; (optional)_ | List with the position of each item in the list that will be removed. (Starts at zero) |


_Method does not return value_

**Example:**
```
Request
{
  "id": "abc",
  "ids": [
    "abc",
    "xyz"
  ],
  "index": 3,
  "indexes": [
    2,
    3
  ]
}
```


---

### SetPlaylistItemDuration
- v2.21.0

Change duration of an item in the media playlist.

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `id` | _String (optional)_ | Item ID |
| `index` | _Number (optional)_ | Item position in the list (starts at zero). |
| `duration` | _Number (optional)_ | Item duration (in seconds) |


_Method does not return value_

**Example:**
```
Request
{
  "id": "xyz",
  "duration": 300
}
```


---

### GetSlideDescriptions
- v2.21.0

List of available slide descriptions



**Response:**

| Name | Type  |
| ---- | :---: |
| `data` | _Array&lt;[SlideDescription](#slide-description)&gt;_| 


**Example:**
```
Response
{
  "status": "ok",
  "data": [
    {
      "name": "Chorus",
      "tag": "C",
      "aliases": [],
      "font_color": "FFFFFF",
      "bg_color": "000080",
      "background": null
    },
    {
      "...": "..."
    },
    {
      "...": "..."
    }
  ]
}
```


---

### GetFavorites
- v2.19.0

Favorites bar items



**Response:**

| Name | Type  |
| ---- | :---: |
| `data` | _Array&lt;[FavoriteItem](#favorite-item)&gt;_| 


**Example:**
```
Response
{
  "status": "ok",
  "data": [
    {
      "id": "1",
      "name": "abc"
    },
    {
      "id": "2",
      "name": "xyz"
    },
    {
      "id": "3",
      "name": "123"
    }
  ]
}
```


---

### FavoriteAction
- v2.19.0

Execute an item from the bookmarks bar

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `id` | _String_ | Item ID |


_Method does not return value_

**Example:**
```
Request
{
  "id": "abcxyz"
}
```


---

### GetApis
- v2.21.0

Returns the list of APIs



**Response:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `data` | _Array&lt;Object&gt;_ |  |
| `data.*.id` | _String_ | Item ID |
| `data.*.name` | _String_ | Item name |


**Example:**
```
Response
{
  "status": "ok",
  "data": [
    {
      "id": "xyz1",
      "name": "abc1"
    },
    {
      "id": "xyz2",
      "name": "abc2"
    }
  ]
}
```


---

### GetScripts
- v2.21.0

Returns the list of scripts



**Response:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `data` | _Array&lt;Object&gt;_ |  |
| `data.*.id` | _String_ | Item ID |
| `data.*.name` | _String_ | Item name |


**Example:**
```
Response
{
  "status": "ok",
  "data": [
    {
      "id": "xyz1",
      "name": "abc1"
    },
    {
      "id": "xyz2",
      "name": "abc2"
    }
  ]
}
```


---

### ApiAction
- v2.19.0

Executes the action of an existing API item in the program

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `id` | _String_ | Item ID |


_Method does not return value_

**Example:**
```
Request
{
  "id": "abcxyz"
}
```


---

### ScriptAction
- v2.19.0

Executes the action of an existing **Script** item in the program

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `id` | _String_ | Item ID |


_Method does not return value_

**Example:**
```
Request
{
  "id": "abc"
}
```


---

### ApiRequest
- v2.19.0

Executes a request to the associated receiver and returns the receiver's response.<br>
From `v2.23.0` it is possible to pass the host or IP directly, but it is necessary to add the host/IP to the list of allowed requests.<br>
file menu > settings > advanced > javascript > settings > allowed requests

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `id` | _String_ | receiver id |
| `raw` | _Object_ | requisition data |


**Response:**

| Type  | Description |
| :---: | ------------|
| _Object_ | Request return or NULL for error/timeout |


**Example:**
```
Request
{
  "id": "abcxyz",
  "raw": {
    "request-type": "GetSourceActive",
    "sourceName": "example"
  }
}

Response
{
  "status": "ok",
  "data": {
    "sourceActive": "example"
  }
}
```


---

### GetCurrentPresentation
- v2.19.0

Item currently being presented or **null** if no presentation is being displayed

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `include_slides` | _Boolean (optional)_ | Return the list of slides from the current presentation. Unavailable for verse presentation. `Default: false` `v2.21.0+` |
| `include_slide_comment` | _Boolean (optional)_ | Include comments (if any) in the slide text. Available if **include_slides=true**. `Default: false` `v2.21.0+` |
| `include_slide_preview` | _Boolean (optional)_ | Include slide preview image. Available if **include_slides=true**. `Default: false` `v2.21.0+` |
| `slide_preview_size` | _String (optional)_ | Preview size in WxH format (ex. 320x180). (max 640x360)<br>Available if **include_slide_preview=true** `Default: false` `v2.21.0+` |


**Response:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `data.id` | _String_ | Item ID |
| `data.type` | _String_ | Type of item. It can be: `song` `verse` `text` `audio` `image` `announcement` `automatic_presentation` `quick_presentation` |
| `data.name` | _String_ | Item name |
| `data.slide_number` | _Number_ | Starts at 1 `v2.20.0+` |
| `data.total_slides` | _Number_ | Total slides `v2.20.0+` |
| `data.slide_type` | _String_ | One of the following values: `default`  `wallpaper`  `blank`  `black`  `final_slide` `v2.20.0+` |
| `data.slides` | _Array&lt;[PresentationSlideInfo](#presentation-slide-info)&gt;_ | List with the slides of the current presentation. Available if **include_slides=true** `v2.21.0+` |


**Example:**
```
Request
{}

Response
{
  "status": "ok",
  "data": {
    "id": "abc123",
    "type": "song",
    "name": "",
    "song_id": "123"
  }
}
```


---

### CloseCurrentPresentation
- v2.19.0

End current presentation



_Method does not return value_



---

### GetF8
### GetF9
### GetF10
- v2.19.0

Returns the current state of the respective option **F8 (wallpaper), F9 (empty screen) or F10 (black screen)**



**Response:**

| Type  | Description |
| :---: | ------------|
| _Boolean_ | **true** or **false** |


**Example:**
```
Response
{
  "status": "ok",
  "data": false
}
```


---

### SetF8
### SetF9
### SetF10
- v2.19.0

Changes the current state of the respective option **F8 (wallpaper), F9 (empty screen) or F10 (black screen)**

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `enable` | _Boolean_ | **true** or **false** |


_Method does not return value_

**Example:**
```
Request
{
  "enable": true
}
```


---

### ToggleF8
### ToggleF9
### ToggleF10
- v2.19.0

Changes the current state of the respective option **F8 (wallpaper), F9 (empty screen) or F10 (black screen)**



_Method does not return value_



---

### ActionNext
- v2.19.0

Performs a **next** command on the current presentation



_Method does not return value_



---

### ActionPrevious
- v2.19.0

Performs a **back** command on the current presentation



_Method does not return value_



---

### ActionGoToIndex
- v2.19.0

Changes the slide in view from the slide index

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `index` | _Number_ | Slide index in presentation (starts at zero) |


_Method does not return value_

**Example:**
```
Request
{
  "index": 3
}
```


---

### ActionGoToSlideDescription
- v2.19.0

Changes the slide in view from the name of the slide description

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `name` | _String_ | Slide description name |


_Method does not return value_

**Example:**
```
Request
{
  "name": "Verse 1"
}
```


---

### GetCurrentBackground
- v2.19.0

Returns the background of the currently displayed presentation.



**Response:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `data` | _[Background](#background)_ | Current background or NULL if no presentation is displayed |


**Example:**
```
Response
{
  "status": "ok",
  "data": {
    "id": "abc",
    "type": "my_video",
    "name": "Video Name",
    "tags": [
      "circle",
      "blue"
    ],
    "bpm": 80,
    "midi": {
      "code": 85,
      "velocity": 81
    }
  }
}
```


---

### GetCurrentTheme
- v2.22.0

Returns the theme of the presentation currently displayed.



**Response:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `data` | _[Background](#background)_ | Current theme or NULL if there is no presentation displayed |


**Example:**
```
Response
{
  "status": "ok",
  "data": {
    "id": "123",
    "type": "theme",
    "name": "Theme Name",
    "tags": [
      "circle",
      "blue"
    ],
    "bpm": 80
  }
}
```


---

### GetBackgrounds
- v2.19.0

List of themes and backgrounds

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `input` | _Object (optional)_ | Filter |
| `type` | _String (optional)_ | Can be: `theme` `my_video` `my_image` `video` `image` |
| `tag` | _String (optional)_ |  |
| `tags` | _Array&lt;String&gt; (optional)_ |  |
| `intersection` | _Boolean (optional)_ | If the **tags** field is filled with multiple items, the **intersection** option defines the type of join. If **true**, the filter will return only items that contain **all** the informed tags, if **false**, the filter will return the items that have at least one of the informed tags `Default: false` |


**Response:**

| Name | Type  |
| ---- | :---: |
| `data` | _Array&lt;[Background](#background)&gt;_| 


**Example:**
```
Request
{
  "type": "my_video",
  "tag": "circle",
  "tags": [],
  "intersection": true
}

Response
{
  "status": "ok",
  "data": [
    {
      "id": "xyz",
      "type": "theme",
      "name": "Theme Name",
      "tags": [
        "circle",
        "blue"
      ],
      "midi": {
        "code": 85,
        "velocity": 80
      }
    },
    {
      "id": "abc",
      "type": "my_video",
      "name": "Video Name",
      "tags": [
        "circle",
        "blue"
      ],
      "bpm": 80,
      "midi": {
        "code": 85,
        "velocity": 81
      }
    }
  ]
}
```


---

### SetCurrentBackground
- v2.19.0

Changes the background (or theme) of the current presentation. If more than one item is found according to the filters, random item will be chosen from the list

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `input` | _Object (optional)_ | Filter |
| `id` | _String (optional)_ | Theme or background ID |
| `name` | _String (optional)_ | Theme name or background |
| `type` | _String (optional)_ | Can be: `theme` `my_video` `my_image` `video` `image` |
| `tag` | _String (optional)_ |  |
| `tags` | _Array&lt;String&gt; (optional)_ |  |
| `intersection` | _Boolean (optional)_ | If the **tags** field is filled with multiple items, the **intersection** option defines the type of join. If **true**, the filter will return only items that contain **all** the informed tags, if **false**, the filter will return the items that have at least one of the informed tags `Default: false` |
| `edit` | _[Theme](#theme) (optional)_ | Settings to modify the selected Theme for display `v2.21.0+` |
| `custom_theme` | _[Theme](#theme) (optional)_ | Custom theme `v2.21.0+` |


_Method does not return value_

**Example:**
```
Request
{
  "id": "123",
  "name": "",
  "type": "my_video",
  "tag": "blue",
  "tags": [],
  "intersection": false
}
```


---

### GetThumbnail
- v2.21.0

Returns the thumbnail image of an item in the program

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `id` | _String (optional)_ | Item ID |
| `ids` | _Array&lt;String&gt; (optional)_ | Item IDs |
| `type` | _String_ | Type of item. Can be: `video` `image` `announcement` `theme` `background` `api` `script` |


**Response:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `data` | _Array&lt;Object&gt;_ |  |
| `data.*.type` | _String_ | Type of item |
| `data.*.id` | _String_ | Item ID |
| `data.*.image` | _String_ | Image in base64 format |


**Example:**
```
Request
{
  "id": "image.jpg",
  "type": "image"
}

Response
{
  "status": "ok",
  "data": [
    {
      "type": "image",
      "id": "image.jpg",
      "image": "..."
    }
  ]
}
```


---

### GetColorMap
- v2.20.0

Returns the information of the predominant color of a respective type of item.<br/>The returned array contains 8 indices, and each index corresponds to the section according to the following example image.<br/> <br/>![Color Map Example](https://holyrics.com.br/images/color_map_item_example.png)<br/>

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `type` | _String_ | One of the following values:<br/>**background** - um item de tema ou plano de fundo<br/>**presentation** - apresentação atual em exibição<br/>**image** - uma imagem da aba 'imagens'<br/>**video** - um vídeo da aba 'vídeos'<br/>**printscreen** - um printscreen atual de uma tela do sistema<br/> |
| `source` | _Object (optional)_ | The item according to the type informed:<br/>**background** - ID do tema ou plano de fundo<br/>**presentation** - não é necessário informar um valor, a apresentação da tela público será retornada<br/>**image** - o nome do arquivo da aba 'imagens'<br/>**video** - o nome do arquivo da aba 'vídeos'<br/>**printscreen** `opcional` -  the name of the screen (public, screen_2, screen_3, ...); o padrão é `public`<br/> |


**Response:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `data` | _Array&lt;Object&gt;_ |  |
| `data.*.hex` | _String_ | Color in hexadecimal format |
| `data.*.red` | _Number_ | Red  `0 ~ 255` |
| `data.*.green` | _Number_ | Green  `0 ~ 255` |
| `data.*.blue` | _Number_ | Blue  `0 ~ 255` |


**Example:**
```
Request
{
  "type": "background",
  "source": 12345678
}

Response
{
  "status": "ok",
  "data": [
    {
      "hex": "0000FF",
      "red": 0,
      "green": 0,
      "blue": 255
    },
    {
      "...": "..."
    },
    {
      "...": "..."
    },
    {
      "...": "..."
    }
  ]
}
```


---

### GetAlert
- v2.20.0

Returns alert message settings



**Response:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `data.text` | _String_ | Current alert text |
| `data.show` | _Boolean_ | Whether the alert display is enabled |


**Example:**
```
Response
{
  "status": "ok",
  "data": {
    "text": "Example",
    "show": false
  }
}
```


---

### SetAlert
- v2.19.0

Change alert message settings

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `text` | _String (optional)_ | Change alert text |
| `show` | _Boolean (optional)_ | Show/hide the alert |


_Method does not return value_

**Example:**
```
Request
{
  "text": "",
  "show": false
}
```


---

### GetCurrentSchedule
- v2.19.0

Current schedule (selected in the main program window)

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `include_lyrics_slides` | _Boolean (optional)_ |  `v2.24.0+` |
| `include_lyrics_history` | _Boolean (optional)_ |  `v2.24.0+` |


**Response:**

| Name | Type  |
| ---- | :---: |
| `data` | _Array&lt;[Schedule](#schedule)&gt;_| 


**Example:**
```
Request
{}

Response
{
  "status": "ok",
  "data": {
    "type": "event",
    "name": "",
    "datetime": "2016-05-10 12:00",
    "lyrics_playlist": [
      {
        "id": "123",
        "title": "",
        "artist": "",
        "author": "",
        "note": "",
        "key": "",
        "bpm": 0,
        "time_sig": "",
        "groups": [
          {
            "name": "Group 1"
          },
          {
            "name": "Group 2"
          }
        ],
        "archived": false
      }
    ],
    "media_playlist": [
      {
        "id": "xyz",
        "type": "image",
        "name": "image.jpg"
      },
      {
        "id": "abc",
        "type": "video",
        "name": "video.mp4"
      }
    ],
    "responsible": null,
    "members": [],
    "roles": [],
    "notes": ""
  }
}
```


---

### GetSchedules
- v2.19.0

Returns the schedule list for a specific month

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `month` | _Number_ | Month (1-12) |
| `year` | _Number_ | Year |


**Response:**

| Name | Type  |
| ---- | :---: |
| `data` | _Array&lt;[Schedule](#schedule)&gt;_| 


**Example:**
```
Request
{
  "month": 3,
  "year": 2022
}

Response
{
  "status": "ok",
  "data": [
    {
      "type": "service",
      "name": "",
      "datetime": "2022-03-06 19:00",
      "lyrics_playlist": [],
      "media_playlist": [],
      "responsible": null,
      "members": [],
      "roles": [],
      "notes": ""
    },
    {
      "type": "event",
      "name": "",
      "datetime": "2022-03-12 20:00",
      "lyrics_playlist": [],
      "media_playlist": [],
      "responsible": null,
      "members": [],
      "roles": [],
      "notes": ""
    },
    {
      "type": "service",
      "name": "",
      "datetime": "2022-03-13 19:00",
      "lyrics_playlist": [],
      "media_playlist": [],
      "responsible": null,
      "members": [],
      "roles": [],
      "notes": ""
    },
    {
      "type": "service",
      "name": "",
      "datetime": "2022-03-20 19:00",
      "lyrics_playlist": [],
      "media_playlist": [],
      "responsible": null,
      "members": [],
      "roles": [],
      "notes": ""
    },
    {
      "type": "service",
      "name": "",
      "datetime": "2022-03-27 19:00",
      "lyrics_playlist": [],
      "media_playlist": [],
      "responsible": null,
      "members": [],
      "roles": [],
      "notes": ""
    }
  ]
}
```


---

### GetSavedPlaylists
- v2.19.0

Returns saved playlists



**Response:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `data` | _Array&lt;Object&gt;_ |  |
| `data.*.id` | _String_ | Item ID |
| `data.*.name` | _String_ | Item name |
| `data.*.items` | _Array&lt;[Item](#item)&gt;_ | Items saved in the list |


**Example:**
```
Response
{
  "status": "ok",
  "data": [
    {
      "id": "xyzabc",
      "name": "",
      "items": [
        {
          "id": "xyz",
          "type": "image",
          "name": "image.jpg"
        },
        {
          "id": "abc",
          "type": "video",
          "name": "video.mp4"
        }
      ]
    },
    {
      "id": "abcdef",
      "name": "",
      "items": [
        {
          "id": "abc",
          "type": "audio",
          "name": "audio.mp3"
        },
        {
          "id": "xyz",
          "type": "song",
          "name": "",
          "song_id": "123"
        }
      ]
    }
  ]
}
```


---

### LoadSavedPlaylist
- v2.19.0

Populates the media list of the currently selected playlist in the program with the given list

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `name` | _String_ | Saved playlist name |


_Method does not return value_

**Example:**
```
Request
{
  "name": ""
}
```


---

### GetHistory
- v2.19.0

"Music played" history

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `id` | _String_ | Song lyric ID |
| `in_millis` | _Boolean (optional)_ | `true` to return the value in Timestamp `v2.24.0+` |


**Response:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `data` | _Array&lt;String&gt;_ | Date and time in format: YYYY-MM-DD HH:MM |


**Example:**
```
Request
{
  "id": "123"
}

Response
{
  "status": "ok",
  "data": [
    "2022-04-03 19:32",
    "2022-07-10 20:10",
    "2023-01-01 19:15"
  ]
}
```


---

### GetHistories
- v2.19.0

History of all tags for "Music played"

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `in_millis` | _Boolean (optional)_ | `true` to return the value in Timestamp `v2.24.0+` |


**Response:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `data` | _Array&lt;Object&gt;_ |  |
| `data.*.music_id` | _String_ | Song ID |
| `data.*.history` | _Array&lt;String&gt;_ | Date and time in format: YYYY-MM-DD HH:MM |


**Example:**
```
Request
{}

Response
{
  "status": "ok",
  "data": [
    {
      "music_id": "123",
      "history": [
        "2022-04-03 19:32",
        "2022-07-10 20:10",
        "2023-01-01 19:15"
      ]
    },
    {
      "music_id": "456",
      "history": [
        "2022-05-05 19:20",
        "2022-08-08 20:30",
        "2023-01-10 20:02"
      ]
    }
  ]
}
```


---

### GetNearestHistory
- v2.24.0

Gets the date of the "Song played" history closest to a date and time passed as a parameter (or the current system date and time)

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `id` | _String_ | Song lyric ID |
| `datetime` | _String (optional)_ | Accepted formats: `timestamp` `YYYY-MM-DD` `YYYY/MM/DD` `YYYY-MM-DD HH:MM:SS` `YYYY/MM/DD HH:MM:SS` `DD-MM-YYYY` `DD/MM/YYYY` `DD-MM-YYYY HH:MM:SS` `DD/MM/YYYY HH:MM:SS` `Default: Date.now()` |
| `type` | _String (optional)_ | Search filter. Can be:<br>`any` any value closest to the specified date<br>`before_datetime` The closest value that is earlier than or equal to the specified date (value <= datetime)<br>`after_datetime` The closest value that is equal to or later than the specified date (value >= datetime) `Default: any` |


**Response:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `data` | _Object_ | Can be null |
| `data.datetime` | _String_ | Date and time in format: YYYY-MM-DD HH:MM |
| `data.datetime_millis` | _Number_ | Timestamp |


**Example:**
```
Request
{
  "id": "123",
  "datetime": "2024-12-16",
  "type": "after_datetime"
}

Response
{
  "status": "ok",
  "data": {
    "datetime": "2024-12-16 19:30:00",
    "datetime_millis": 1734388200000
  }
}
```


---

### GetSongGroup
- v2.25.0

Music group

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `name` | _String_ |  |


**Response:**

| Name | Type  |
| ---- | :---: |
| `data` | _[Group](#group)_| 


**Example:**
```
Request
{
  "name": "Example"
}

Response
{
  "status": "ok",
  "data": {
    "id": "example",
    "name": "example",
    "songs": [
      "123",
      "456"
    ],
    "add_chorus_between_verses": false,
    "hide_in_interface": false,
    "metadata": {
      "modified_time_millis": 1234
    }
  }
}
```


---

### GetSongGroups
- v2.24.0

List of music groups



**Response:**

| Name | Type  |
| ---- | :---: |
| `data` | _Array&lt;[Group](#group)&gt;_| 


**Example:**
```
Response
{
  "status": "ok",
  "data": [
    {
      "id": "example 1",
      "name": "example 1",
      "songs": [
        "123",
        "456"
      ],
      "add_chorus_between_verses": false,
      "hide_in_interface": false,
      "metadata": {
        "modified_time_millis": 1234
      }
    },
    {
      "id": "example 2",
      "name": "example 2",
      "songs": [
        "123",
        "456"
      ],
      "add_chorus_between_verses": false,
      "hide_in_interface": false,
      "metadata": {
        "modified_time_millis": 1234
      }
    }
  ]
}
```


---

### GetTeams
- v2.22.0

Team list



**Response:**

| Name | Type  |
| ---- | :---: |
| `data` | _Array&lt;[Team](#team)&gt;_| 


**Example:**
```
Response
{
  "status": "ok",
  "data": [
    {
      "id": "abc",
      "name": "abc",
      "description": ""
    },
    {
      "id": "xyz",
      "name": "xyz",
      "description": ""
    }
  ]
}
```


---

### GetMembers
- v2.19.0

List of members

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `only_active` | _Boolean_ |  `Default: true` `v2.25.0+` |


**Response:**

| Name | Type  |
| ---- | :---: |
| `data` | _Array&lt;[Member](#member)&gt;_| 


**Example:**
```
Request
{
  "only_active": true
}

Response
{
  "status": "ok",
  "data": [
    {
      "id": "abc",
      "name": ""
    },
    {
      "id": "xyz",
      "name": ""
    }
  ]
}
```


---

### GetRoles
- v2.19.0

List of functions

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `only_active` | _Boolean_ |  `Default: true` `v2.25.0+` |


**Response:**

| Name | Type  |
| ---- | :---: |
| `data` | _Array&lt;[Role](#role)&gt;_| 


**Example:**
```
Request
{
  "only_active": true
}

Response
{
  "status": "ok",
  "data": [
    {
      "id": "abc",
      "name": ""
    },
    {
      "id": "xyz",
      "name": ""
    }
  ]
}
```


---

### GetServices
- v2.22.0

Service list

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `only_active` | _Boolean_ |  `Default: true` `v2.25.0+` |


**Response:**

| Name | Type  |
| ---- | :---: |
| `data` | _Array&lt;[Service](#service)&gt;_| 


**Example:**
```
Request
{
  "only_active": true
}

Response
{
  "status": "ok",
  "data": [
    {
      "name": "",
      "week": "all",
      "day": "sun",
      "hour": 19,
      "minute": 0,
      "description": "",
      "hide_week": [
        "second"
      ]
    },
    {
      "name": "Supper Worship",
      "week": "second",
      "day": "sun",
      "hour": 19,
      "minute": 0,
      "description": ""
    }
  ]
}
```


---

### GetEvents
- v2.22.0

Event list

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `month` | _Number_ | Month (1-12) |
| `year` | _Number_ | Year |


**Response:**

| Name | Type  |
| ---- | :---: |
| `data` | _Array&lt;[Event](#event)&gt;_| 


**Example:**
```
Request
{
  "month": 8,
  "year": 2022
}

Response
{
  "status": "ok",
  "data": [
    {
      "name": "Event name",
      "datetime": "2022-08-19 18:00",
      "wallpaper": ""
    },
    {
      "name": "Event name",
      "datetime": "2022-08-20 18:00",
      "wallpaper": ""
    }
  ]
}
```


---

### GetAnnouncement
- v2.22.0

Announcement

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `id` | _String (optional)_ | Announcement ID |
| `name` | _String (optional)_ | Announcement name |


**Response:**

| Name | Type  |
| ---- | :---: |
| `data` | _[Announcement](#announcement)_| 


**Example:**
```
Request
{
  "id": "123"
}

Response
{
  "status": "ok",
  "data": {
    "id": "abc",
    "name": "",
    "text": "",
    "archived": false
  }
}
```


---

### GetAnnouncements
- v2.22.0

Announcement list



**Response:**

| Name | Type  |
| ---- | :---: |
| `data` | _Array&lt;[Announcement](#announcement)&gt;_| 


**Example:**
```
Response
{
  "status": "ok",
  "data": [
    {
      "id": "abc",
      "name": "",
      "text": "",
      "archived": false
    },
    {
      "id": "xyz",
      "name": "",
      "text": "",
      "archived": false
    }
  ]
}
```


---

### GetCommunicationPanelInfo
### GetCPInfo
### GetCPSettings
- v2.19.0

Current communication panel configuration



**Response:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `data.text` | _String_ | Current text |
| `data.show` | _Boolean_ | Whether the current text is displayed |
| `data.display_ahead` | _Boolean_ | Whether the *'display in front of all'* option is enabled |
| `data.alert_text` | _String_ | Current alert text |
| `data.alert_show` | _Boolean_ | Whether the alert display is enabled |
| `data.countdown_show` | _Boolean_ | If a countdown is on display |
| `data.countdown_time` | _Number_ | The current countdown time displayed (in seconds) |
| `data.stopwatch_show` | _Boolean_ | If a timer is on display `v2.20.0+` |
| `data.stopwatch_time` | _Number_ | The current time of the timer on display (in seconds) `v2.20.0+` |
| `data.theme` | _String_ | Theme ID `v2.20.0+` |
| `data.countdown_font_relative_size` | _Number_ | Relative size of the countdown `v2.20.0+` |
| `data.countdown_font_color` | _String_ | Color of the countdown font `v2.20.0+` |
| `data.stopwatch_font_color` | _String_ | Color of the stopwatch font `v2.20.0+` |
| `data.time_font_color` | _String_ | Color of the time font `v2.20.0+` |
| `data.display_clock_as_background` | _Boolean_ | Display clock as background `v2.20.0+` |
| `data.display_clock_on_alert` | _Boolean_ | Display clock in the alert `v2.20.0+` |
| `data.countdown_display_location` | _String_ | Location of the countdown or stopwatch display. `FULLSCREEN`  `FULLSCREEN_OR_ALERT`  `ALERT` `v2.20.0+` |
| `data.display_clock_with_countdown_fullscreen` | _Boolean_ | Display clock along with the countdown or stopwatch when displayed in full screen `v2.20.0+` |
| `data.display_vlc_player_remaining_time` | _Boolean_ | Display remaining time of the media playing in VLC Player `v2.20.0+` |
| `data.attention_icon_color` | _String_ | Icon color of the **Attention** button `v2.23.0+` |
| `data.attention_background_color` | _String_ | Background color of the **Attention** button icon `v2.23.0+` |
| `data.countdown_hide_zero_minute` | _Boolean_ | Hide the display of minutes when it is zero `v2.25.0+` |
| `data.countdown_hide_zero_hour` | _Boolean_ | Hide the display of hours when it is zero `v2.25.0+` |
| `data.stopwatch_hide_zero_minute` | _Boolean_ | Hide the display of minutes when it is zero `v2.25.0+` |
| `data.stopwatch_hide_zero_hour` | _Boolean_ | Hide the display of hours when it is zero `v2.25.0+` |


**Example:**
```
Response
{
  "status": "ok",
  "data": {
    "text": "",
    "show": false,
    "display_ahead": false,
    "alert_text": "",
    "alert_show": false,
    "countdown_show": false,
    "countdown_time": 0
  }
}
```


---

### SetCommunicationPanelSettings
### SetCPSettings
- v2.20.0

Change the current setting of the communication panel.

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `text` | _String_ | Current text |
| `show` | _Boolean_ | Display the current text |
| `display_ahead` | _Boolean_ | Option *'display ahead of everything'* |
| `theme` | _Object_ | ID or name of the default theme |
| `theme.id` | _String_ |  |
| `theme.name` | _String_ |  |
| `custom_theme` | _[Theme](#theme) (optional)_ | Custom theme `v2.21.0+` |
| `alert_text` | _String_ | Current alert text |
| `alert_show` | _Boolean_ | Enable the display of the alert |
| `countdown_font_relative_size` | _Number_ | Relative size of the countdown |
| `countdown_font_color` | _String_ | Color of the countdown font |
| `stopwatch_font_color` | _String_ | Color of the stopwatch font |
| `time_font_color` | _String_ | Color of the time font |
| `display_clock_as_background` | _Boolean_ | Display clock as background |
| `display_clock_on_alert` | _Boolean_ | Display clock in the alert |
| `countdown_display_location` | _String_ | Location of the countdown or stopwatch display. `FULLSCREEN`  `FULLSCREEN_OR_ALERT`  `ALERT` |
| `display_clock_with_countdown_fullscreen` | _Boolean_ | Display clock along with the countdown or stopwatch when displayed in full screen |
| `display_vlc_player_remaining_time` | _Boolean_ | Display remaining time of the media playing in VLC Player |
| `attention_icon_color` | _String_ | Icon color of the **Attention** button `v2.23.0+` |
| `attention_background_color` | _String_ | Background color of the **Attention** button icon `v2.23.0+` |
| `countdown_hide_zero_minute` | _Boolean_ | Hide the display of minutes when it is zero `v2.25.0+` |
| `countdown_hide_zero_hour` | _Boolean_ | Hide the display of hours when it is zero `v2.25.0+` |
| `stopwatch_hide_zero_minute` | _Boolean_ | Hide the display of minutes when it is zero `v2.25.0+` |
| `stopwatch_hide_zero_hour` | _Boolean_ | Hide the display of hours when it is zero `v2.25.0+` |


_Method does not return value_

**Example:**
```
Request
{
  "display_clock_as_background": false,
  "display_clock_on_alert": true
}
```


---

### StartCountdownCommunicationPanel
### StartCountdownCP
- v2.19.0

Starts a countdown on the communication panel

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `minutes` | _Number_ | Number of minutes |
| `seconds` | _Number_ | Number of seconds |
| `yellow_starts_at` | _Number (optional)_ | Value in seconds to define how long the countdown will be yellow from |
| `stop_at_zero` | _Boolean (optional)_ | Stop the countdown when it reaches zero `Default: false` |
| `text` | _String (optional)_ | Text for display. By default, the text is displayed before the numeric part. For special formatting, use the variable `@cp_countdown` in the middle of the text to indicate the location of the numeric part. `v2.24.0+` |
| `alert_text` | _String (optional)_ | Alternative text to be displayed when the display is in the alert. By default, the text is displayed before the numeric part. For special formatting, use the variable `@cp_countdown` in the middle of the text to indicate the location of the numeric part. `v2.24.0+` |


_Method does not return value_

**Example:**
```
Request
{
  "minutes": 5,
  "seconds": 0,
  "yellow_starts_at": 60,
  "stop_at_zero": false
}
```


---

### StopCountdownCommunicationPanel
### StopCountdownCP
- v2.19.0

Ends current communication panel countdown



_Method does not return value_



---

### StartTimerCommunicationPanel
### StartTimerCP
- v2.20.0

Start a stopwatch on the communication panel

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `text` | _String (optional)_ | Text for display. By default, the text is displayed before the numeric part. For special formatting, use the variable `@cp_countdown` in the middle of the text to indicate the location of the numeric part. `v2.24.0+` |
| `alert_text` | _String (optional)_ | Alternative text to be displayed when the display is in the alert. By default, the text is displayed before the numeric part. For special formatting, use the variable `@cp_countdown` in the middle of the text to indicate the location of the numeric part. `v2.24.0+` |


_Method does not return value_

**Example:**
```
Request
{}
```


---

### StopTimerCommunicationPanel
### StopTimerCP
- v2.20.0

End the current stopwatch of the communication panel



_Method does not return value_



---

### SetTextCommunicationPanel
### SetTextCP
- v2.19.0

Change communication panel text

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `text` | _String (optional)_ | Change the text of the communication panel. [Styled Text](https://github.com/holyrics/Scripts/blob/main/i18n/en/StyledText.md) from v2.19.0 |
| `show` | _Boolean (optional)_ | Show/hide the text |
| `display_ahead` | _Boolean (optional)_ | Change the *'display in front of all'* option |
| `theme` | _Object (optional)_ | ID or name of the Theme used to display the text `v2.21.0+` |
| `custom_theme` | _[Theme](#theme) (optional)_ | Custom theme to display the text `v2.21.0+` |


_Method does not return value_

**Example:**
```
Request
{
  "text": "",
  "show": true,
  "display_ahead": true
}
```


---

### SetAlertCommunicationPanel
### SetAlertCP
- v2.19.0

Change communication panel alert settings

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `text` | _String (optional)_ | Change alert text |
| `show` | _Boolean (optional)_ | Show/hide the alert |


_Method does not return value_

**Example:**
```
Request
{
  "text": "",
  "show": false
}
```


---

### CommunicationPanelCallAttention
### CPCallAttention
- v2.20.0

Execute the 'call attention' option available in the communication panel



_Method does not return value_



---

### GetWallpaperSettings
- v2.19.0

Wallpaper settings



**Response:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `data.image_base64` | _String_ | Wallpaper image in base 64 |
| `data.enabled` | _Boolean_ | Show wallpaper |
| `data.fill_color` | _String_ | Color in hexadecimal defined in the **fill** option. |
| `data.extend` | _Boolean_ | `deprecated` Replaced for `adjust_type`<br>Extend wallpaper |
| `data.adjust_type` | _String_ | Image adjustment: Can be: `ADJUST` `EXTEND` `FILL` `ADJUST_BLUR` `v2.22.0+` |
| `data.show_clock` | _Boolean_ | Show clock |
| `data.by_screen` | _Object_ | Independent configuration per screen `v2.23.0+` |
| `data.by_screen.default` | _[WallpaperSettings](#wallpaper-settings)_ | Default configuration `v2.23.0+` |
| `data.by_screen.public` | _[WallpaperSettings](#wallpaper-settings)_ | Custom configuration for the screen or **null** if using the default configuration `v2.23.0+` |
| `data.by_screen.screen_n` | _[WallpaperSettings](#wallpaper-settings)_ | n >= 2  `v2.23.0+` |


**Example:**
```
Response
{
  "status": "ok",
  "data": {
    "image_base64": "",
    "enabled": false,
    "fill_color": "#000000",
    "extend": false,
    "show_clock": false
  }
}
```


---

### SetWallpaperSettings
- v2.19.0

Change wallpaper settings

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `file` | _String (optional)_ | File location in the **Images** tab |
| `enabled` | _Boolean (optional)_ | Show wallpaper |
| `fill_color` | _String (optional)_ | Color in hexadecimal defined in the **fill** option. **NULL** to disable |
| `extend` | _Boolean (optional)_ | `deprecated` Replaced for `adjust_type`<br>Extend wallpaper |
| `adjust_type` | _String_ | Image adjustment: Can be: `ADJUST` `EXTEND` `FILL` `ADJUST_BLUR` `v2.22.0+` |
| `show_clock` | _Boolean (optional)_ | Show clock |
| `by_screen` | _Object (optional)_ | Independent configuration per screen `v2.23.0+` |
| `by_screen.default` | _[WallpaperSettings](#wallpaper-settings) (optional)_ | Default configuration `v2.23.0+` |
| `by_screen.public` | _[WallpaperSettings](#wallpaper-settings) (optional)_ | Custom configuration for the screen or **null** if using the default configuration `v2.23.0+` |
| `by_screen.screen_n` | _[WallpaperSettings](#wallpaper-settings) (optional)_ | n >= 2 `v2.23.0+` |


**Response:**

| Type  | Description |
| :---: | ------------|
| _Object_ | Return **true** or a list of errors that occurred |


**Example:**
```
Request
{
  "file": "wallpapers/image.jpg",
  "enabled": true,
  "fill_color": "#000000",
  "extend": true,
  "show_clock": false
}

Response
{
  "status": "ok",
  "data": true
}
```


---

### GetDisplaySettings
- v2.19.0

List of display settings for each screen



**Response:**

| Name | Type  |
| ---- | :---: |
| `data` | _Array&lt;[DisplaySettings](#display-settings)&gt;_| 


**Example:**
```
Response
{
  "status": "ok",
  "data": [
    {
      "id": "public",
      "name": "Public",
      "slide_info": {
        "info_1": {
          "show_page_count": false,
          "show_slide_description": false,
          "horizontal_align": "right",
          "vertical_align": "bottom"
        },
        "info_2": {
          "show": false,
          "layout_row_1": "<title>< (%author_or_artist%)>",
          "horizontal_align": "right",
          "vertical_align": "bottom"
        },
        "font": {
          "name": null,
          "bold": null,
          "italic": null,
          "color": null
        },
        "height": 7,
        "paint_theme_effect": true
      },
      "slide_translation": "",
      "margin": {
        "top": 0.0,
        "right": 0.0,
        "bottom": 0.0,
        "left": 0.0
      },
      "area": {
        "x": 1920,
        "y": 0,
        "width": 1920,
        "height": 1080
      },
      "total_area": {
        "x": 1920,
        "y": 0,
        "width": 1920,
        "height": 1080
      },
      "hide": false
    },
    {
      "id": "screen_2",
      "name": "Screen 2",
      "stage_view": {
        "enabled": true,
        "preview_mode": "FIRST_LINE_OF_THE_NEXT_SLIDE_WITH_SEPARATOR",
        "uppercase": false,
        "remove_line_break": false,
        "show_comment": true,
        "show_advanced_editor": false,
        "show_communication_panel": true,
        "custom_theme": 123,
        "apply_custom_theme_to_bible": true,
        "apply_custom_theme_to_text": true
      },
      "slide_info": {
        "info_1": {
          "show_page_count": false,
          "show_slide_description": false,
          "horizontal_align": "right",
          "vertical_align": "bottom"
        },
        "info_2": {
          "show": false,
          "layout_row_1": "<title>< (%author_or_artist%)>",
          "horizontal_align": "right",
          "vertical_align": "bottom"
        },
        "font": {
          "name": null,
          "bold": null,
          "italic": null,
          "color": null
        },
        "height": 7,
        "paint_theme_effect": true
      },
      "slide_translation": "",
      "bible_version_tab": 1,
      "margin": {
        "top": 0.0,
        "right": 0.0,
        "bottom": 0.0,
        "left": 0.0
      },
      "area": {
        "x": 3840,
        "y": 0,
        "width": 1920,
        "height": 1080
      },
      "total_area": {
        "x": 3840,
        "y": 0,
        "width": 1920,
        "height": 1080
      },
      "hide": false
    },
    {
      "id": "stream_image",
      "name": "Stream - Image",
      "stage_view": {
        "enabled": true,
        "preview_mode": "FIRST_LINE_OF_THE_NEXT_SLIDE_WITH_SEPARATOR",
        "uppercase": false,
        "remove_line_break": false,
        "show_comment": true,
        "show_advanced_editor": false,
        "show_communication_panel": true,
        "custom_theme": null,
        "apply_custom_theme_to_bible": true,
        "apply_custom_theme_to_text": true
      },
      "slide_info": {
        "info_1": {
          "show_page_count": false,
          "show_slide_description": false,
          "horizontal_align": "right",
          "vertical_align": "bottom"
        },
        "info_2": {
          "show": false,
          "layout_row_1": "<title>< (%author_or_artist%)>",
          "horizontal_align": "right",
          "vertical_align": "bottom"
        },
        "font": {
          "name": null,
          "bold": null,
          "italic": null,
          "color": null
        },
        "height": 7,
        "paint_theme_effect": true
      },
      "slide_translation": "",
      "bible_version_tab": 1,
      "show_items": {
        "lyrics": true,
        "text": true,
        "verse": true,
        "image": true,
        "alert": true,
        "announcement": true
      }
    },
    {
      "id": "stream_html_1",
      "name": "Stream - HTML 1",
      "stage_view": {
        "enabled": true,
        "preview_mode": "FIRST_LINE_OF_THE_NEXT_SLIDE_WITH_SEPARATOR",
        "uppercase": false,
        "remove_line_break": false,
        "show_comment": true,
        "show_advanced_editor": false,
        "show_communication_panel": true,
        "custom_theme": null,
        "apply_custom_theme_to_bible": true,
        "apply_custom_theme_to_text": true
      },
      "slide_translation": "",
      "bible_version_tab": 1,
      "show_items": {
        "lyrics": true,
        "text": true,
        "verse": true,
        "image": true,
        "alert": true,
        "announcement": true
      }
    }
  ]
}
```


---

### SetDisplaySettings
- v2.19.0

Change a screen's display settings

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `input` | _[DisplaySettings](#display-settings)_ | New settings. Settings are individually optional. Fill in only the fields you want to change. |


**Response:**

| Type  | Description |
| :---: | ------------|
| _Object_ | Return **true** or a list of errors that occurred |


**Example:**
```
Request
{
  "id": "screen_2",
  "stage_view": {
    "enabled": true,
    "preview_mode": "FIRST_LINE_OF_THE_NEXT_SLIDE_WITH_SEPARATOR",
    "uppercase": true
  },
  "margin": {
    "top": 0,
    "right": 0,
    "bottom": 10,
    "left": 0
  }
}

Response
{
  "status": "ok",
  "data": true
}
```


---

### GetTransitionEffectSettings
- v2.21.0

List of transition effect configuration



**Response:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `music` | _Array&lt;[TransitionEffectSettings](#transition-effect-settings)&gt;_ |  |
| `bible` | _Array&lt;[TransitionEffectSettings](#transition-effect-settings)&gt;_ |  |
| `image` | _Array&lt;[TransitionEffectSettings](#transition-effect-settings)&gt;_ |  |
| `announcement` | _Array&lt;[TransitionEffectSettings](#transition-effect-settings)&gt;_ |  |


**Example:**
```
Response
{
  "status": "ok",
  "data": {
    "music": {
      "enabled": true,
      "type": "fade",
      "duration": 500,
      "only_area_within_margin": false,
      "merge": false,
      "division_point": 30,
      "increase_duration_blank_slides": false
    },
    "bible": {
      "...": "..."
    },
    "image": {
      "...": "..."
    },
    "announcement": {
      "...": "..."
    }
  }
}
```


---

### SetTransitionEffectSettings
- v2.21.0

Change the settings of a transition effect

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `id` | _Object_ | Item ID |
| `settings` | _[TransitionEffectSettings](#transition-effect-settings)_ | New settings. Settings are individually optional. Fill in only the fields you want to change. |


**Response:**

| Type  | Description |
| :---: | ------------|
| _Object_ | Return **true** or a list of errors that occurred |


**Example:**
```
Request
{
  "id": "music",
  "settings": {
    "enabled": true,
    "type": "fade",
    "duration": 500,
    "only_area_within_margin": false,
    "merge": false,
    "division_point": 30,
    "increase_duration_blank_slides": false
  }
}

Response
{
  "status": "ok",
  "data": true
}
```


---

### GetBibleVersions
- v2.21.0

`deprecated` Replaced for: hly('GetBibleVersionsV2')<br>Returns the list of available versions of the Bible, and also the associated shortcuts



**Response:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `data` | _Array&lt;Object&gt;_ |  |
| `data.*.key` | _String_ | Abbreviation of the version or the name of the shortcut, if it starts with '#shortcut ' |
| `data.*.title` | _String_ | Version name |
| `data.*.version` | _String (optional)_ | Abbreviation of the version. Available if the item is a shortcut, that is if 'key' starts with '#shortcut ' |
| `data.*.language` | _Object_ | Language `v2.24.0+` |
| `data.*.language.id` | _String_ | Item ID `v2.24.0+` |
| `data.*.language.iso` | _String_ | ISO 639 two-letter language code `v2.24.0+` |
| `data.*.language.name` | _String_ | Name in English `v2.24.0+` |
| `data.*.language.alt_name` | _String_ | Name in the language defined in `language`. Can be null. `v2.24.0+` |


**Example:**
```
Response
{
  "status": "ok",
  "data": [
    {
      "key": "en_kjv",
      "title": "King James Version"
    },
    {
      "key": "en_akjv",
      "title": "American King James Version"
    },
    {
      "key": "#shortcut Example",
      "title": "King James Version",
      "version": "en_kjv"
    }
  ]
}
```


---

### GetBibleVersionsV2
- v2.23.0

Returns the list of available versions of the Bible, and also the associated shortcuts



**Response:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `data` | _Array&lt;Object&gt;_ |  |
| `data.*.key` | _String_ | Item ID |
| `data.*.version` | _String_ | Bible version ID |
| `data.*.title` | _String_ | Version name or shortcut name |
| `data.*.language` | _Object_ | Language `v2.24.0+` |
| `data.*.language.id` | _String_ | Item ID `v2.24.0+` |
| `data.*.language.iso` | _String_ | ISO 639 two-letter language code `v2.24.0+` |
| `data.*.language.name` | _String_ | Name in English `v2.24.0+` |
| `data.*.language.alt_name` | _String_ | Name in the language defined in `language`. Can be null. `v2.24.0+` |


**Example:**
```
Response
{
  "status": "ok",
  "data": [
    {
      "key": "en_kjv",
      "version": "en_kjv",
      "title": "King James Version",
      "language": {
        "id": "en",
        "iso": "en",
        "name": "English",
        "alt_name": "English"
      }
    },
    {
      "key": "pt_acf",
      "version": "pt_acf",
      "title": "Almeida Corrigida Fiel",
      "language": {
        "id": "pt",
        "iso": "pt",
        "name": "Portuguese",
        "alt_name": "Português"
      }
    }
  ]
}
```


---

### GetBibleSettings
- v2.21.0

Bible module settings



**Response:**

| Name | Type  |
| ---- | :---: |
| `data` | _Array&lt;[BibleSettings](#bible-settings)&gt;_| 


**Example:**
```
Response
{
  "status": "ok",
  "data": {
    "tab_version_1": "pt_???",
    "tab_version_2": "es_???",
    "tab_version_3": "en_???",
    "show_x_verses": 1,
    "uppercase": false,
    "show_only_reference": false,
    "show_second_version": false,
    "show_third_version": false,
    "book_panel_type": "grid",
    "book_panel_order": "automatic",
    "book_panel_order_available_items": [
      "automatic",
      "standard",
      "ru",
      "tyv"
    ],
    "multiple_verses_separator_type": "double_line_break",
    "versification": true,
    "theme": {
      "public": 123,
      "screen_n": null
    }
  }
}
```


---

### SetBibleSettings
- v2.21.0

Change the settings of the Bible module

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `input` | _[BibleSettings](#bible-settings)_ | New settings. Settings are individually optional. Fill in only the fields you want to change. |


**Response:**

| Type  | Description |
| :---: | ------------|
| _Object_ | Return **true** or a list of errors that occurred |


**Example:**
```
Request
{
  "tab_version_1": "pt_acf",
  "show_x_verses": 1,
  "theme": {
    "public": "123"
  }
}

Response
{
  "status": "ok",
  "data": true
}
```


---

### GetPresentationFooterSettings
- v2.23.0

Presentation footer settings



**Response:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `data.rows` | _Number_ | Number of lines. `1 ~ 4` |
| `data.preview_mode` | _String_ | Accepted values: `text` `image` |
| `data.minimized` | _Boolean_ |  |


**Example:**
```
Response
{
  "status": "ok",
  "data": {
    "rows": 2,
    "preview_mode": "text",
    "minimized": false
  }
}
```


---

### SetPresentationFooterSettings
- v2.23.0

Change the presentation footer settings

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `rows` | _Number_ | Number of lines. `1 ~ 4` |
| `preview_mode` | _String_ | Accepted values: `text` `image` |
| `minimized` | _Boolean_ |  |


**Response:**

| Type  | Description |
| :---: | ------------|
| _Object_ | Return **true** or a list of errors that occurred |


**Example:**
```
Request
{
  "rows": 2,
  "preview_mode": "text",
  "minimized": false
}

Response
{
  "status": "ok",
  "data": {}
}
```


---

### GetBpm
- v2.19.0

Returns the current BPM value set in the program



**Response:**

| Type  | Description |
| :---: | ------------|
| _Number_ | Current BPM value |


**Example:**
```
Response
{
  "status": "ok",
  "data": 80
}
```


---

### SetBpm
- v2.19.0

Change the current BPM value of the program

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `bpm` | _Number_ | BPM value |


_Method does not return value_

**Example:**
```
Request
{
  "bpm": 80
}
```


---

### GetHue
- v2.19.0

Returns the current hue value defined in the program



**Response:**

| Type  | Description |
| :---: | ------------|
| _Number_ | Current hue value. Minimum=0, Maximum=360. Returns **null** if disabled. |


**Example:**
```
Response
{
  "status": "ok",
  "data": 120
}
```


---

### SetHue
- v2.19.0

Changes the program's current hue value

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `hue` | _Number_ | Hue value. Minimum=0, Maximum=360 or **null** to disable. |


_Method does not return value_

**Example:**
```
Request
{
  "hue": null
}
```


---

### GetRuntimeEnvironment
### GetRE
- v2.19.0

Returns the name of the currently defined runtime environment in the program settings.



**Response:**

| Type  | Description |
| :---: | ------------|
| _String_ | Runtime environment name |


**Example:**
```
Response
{
  "status": "ok",
  "data": "runtime environment name"
}
```


---

### SetRuntimeEnvironment
### SetRE
- v2.19.0

Change the current runtime environment.

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `name` | _String_ | Runtime environment name |


_Method does not return value_

**Example:**
```
Request
{
  "name": "runtime environment name"
}
```


---

### SetLogo
- v2.19.0

Change the settings for the program's *Logo* functionality (tools menu)

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `enable` | _Boolean (optional)_ | Enable/disable functionality |
| `hide` | _Boolean (optional)_ | Show/hide functionality |


_Method does not return value_

**Example:**
```
Request
{
  "enable": true,
  "hide": true
}
```


---

### GetSyncStatus
- v2.19.0

Returns the current state of online synchronization via Google Drive™



**Response:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `data.enabled` | _Boolean_ | If sync is turned on |
| `data.started` | _Boolean_ | Whether synchronization has started (internet available for example) |
| `data.progress` | _Number_ | Sync progress from 0 to 100 |


**Example:**
```
Response
{
  "status": "ok",
  "data": {
    "enabled": true,
    "started": true,
    "progress": 99
  }
}
```


---

### GetInterfaceInput
- v2.21.0

Returns the value of a field from the program interface

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `id` | _String_ | Item ID. Can be: <br>`main_lyrics_tab_search`<br>`main_text_tab_search`<br>`main_audio_tab_search`<br>`main_video_tab_search`<br>`main_image_tab_search`<br>`main_file_tab_search`<br>`main_automatic_presentation_tab_search`<br>`main_selected_theme` |


**Response:**

| Type  | Description |
| :---: | ------------|
| _String_ | Item content |


**Example:**
```
Request
{
  "id": "main_lyrics_tab_search"
}

Response
{
  "status": "ok",
  "data": "input value"
}
```


---

### SetInterfaceInput
- v2.21.0

Change the value of a field in the program interface

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `id` | _String_ | Item ID. Can be: <br>`main_lyrics_tab_search`<br>`main_text_tab_search`<br>`main_audio_tab_search`<br>`main_video_tab_search`<br>`main_image_tab_search`<br>`main_file_tab_search`<br>`main_automatic_presentation_tab_search`<br>`main_selected_theme` |
| `value` | _String_ | New value |
| `focus` | _Boolean (optional)_ | Make the component receive system focus |


_Method does not return value_

**Example:**
```
Request
{
  "id": "main_lyrics_tab_search",
  "value": "new input value",
  "focus": true
}
```


---

### SelectVerse
- v2.21.0

Selects a verse in the Bible window

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `id` | _String (optional)_ | Verse ID |
| `reference` | _String (optional)_ | Verse reference |


_Method does not return value_

**Example:**
```
Request
{
  "id": "43003016"
}
```


---

### OpenDrawLots
- v2.21.0

Opens the draw window from a list of items

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `items` | _Array&lt;String&gt;_ | List with the items to be drawn |


_Method does not return value_

**Example:**
```
Request
{
  "items": [
    "Item 1",
    "Item 2",
    "Item 3"
  ]
}
```


---

### GetMediaDuration
- v2.21.0

Returns the duration of the media

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `type` | _String_ | Type of item. Can be: `video`, `audio`, `automatic_presentation` |
| `name` | _String_ | Item name |


**Response:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `data.type` | _String_ |  |
| `data.name` | _String_ |  |
| `data.duration` | _Number_ | Duration in seconds |
| `data.duration_ms` | _Number_ | Duration in milliseconds |


**Example:**
```
Request
{
  "type": "audio",
  "name": "file.mp3"
}

Response
{
  "status": "ok",
  "data": {
    "type": "audio",
    "name": "file.mp3",
    "duration": 128,
    "duration_ms": 128320
  }
}
```


---

### GetVersion
- v2.22.0

Returns information about the version of the running program



**Response:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `data.version` | _String_ | Program version |
| `data.platform` | _String_ | Operational system. Can be: `win` `uni` `osx` |
| `data.platformDescription` | _String_ | Detailed operating system name |
| `data.baseDir` | _String_ |  `v2.24.0+` |
| `data.language` | _String_ |  `v2.24.0+` |
| `data.theme` | _String_ | One of the following values: `DEFAULT` `DARK_SOFT` `DARK_MEDIUM` `DARK_STRONG` `v2.24.0+` |
| `data.jscVersion` | _String_ | JS Community Version y.m.d `v2.24.0+` |


**Example:**
```
Response
{
  "status": "ok",
  "data": {
    "data": {
      "version": "2.22.0",
      "platform": "win",
      "platformDescription": "Windows 10",
      "baseDir": "C:\\Holyrics",
      "language": "pt",
      "theme": "DARK_STRONG",
      "jscVersion": "24.10.12"
    }
  }
}
```


---

### GetRealTimeSongKey
- v2.24.0



**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `id` | _String_ | Song ID |


**Response:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `data.key` | _String_ | `C` `C#` `Db` `D` `D#` `Eb` `E` `F` `F#` `Gb` `G` `G#` `Ab` `A` `A#` `Bb` `B` `Cm` `C#m` `Dbm` `Dm` `D#m` `Ebm` `Em` `Fm` `F#m` `Gbm` `Gm` `G#m` `Abm` `Am` `A#m` `Bbm` `Bm` |


**Example:**
```
Request
{
  "id": "123"
}

Response
{
  "status": "ok",
  "data": {
    "key": "Em"
  }
}
```


---

### SetRealTimeSongKey
- v2.24.0



**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `id` | _String_ | Song ID |
| `key` | _String_ | `C` `C#` `Db` `D` `D#` `Eb` `E` `F` `F#` `Gb` `G` `G#` `Ab` `A` `A#` `Bb` `B` `Cm` `C#m` `Dbm` `Dm` `D#m` `Ebm` `Em` `Fm` `F#m` `Gbm` `Gm` `G#m` `Abm` `Am` `A#m` `Bbm` `Bm` |


_Method does not return value_

**Example:**
```
Request
{
  "id": "123",
  "key": "Am"
}
```


---

### ActionNextQuickPresentation
- v2.24.0





_Method does not return value_



---

### ActionPreviousQuickPresentation
- v2.24.0





_Method does not return value_



---

### CloseCurrentQuickPresentation
- v2.24.0





_Method does not return value_



---

### GetCurrentQuickPresentation
- v2.24.0





**Response:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `data.id` | _String_ | Current verse ID |
| `data.slide_number` | _Number_ | Starts at 1 |
| `data.total_slides` | _Number_ | Total verses |
| `data.slide_type` | _String_ | One of the following values: `default`  `wallpaper`  `blank`  `black`  `final_slide` |
| `data.slides` | _Array&lt;Object&gt;_ | List of verses from the current presentation |
| `data.slides.*.number` | _Number_ | Slide number. Starts at 1. |
| `data.slides.*.reference` | _String_ | Verse reference. Example: **John 3:16** |


**Example:**
```
Response
{
  "status": "ok",
  "data": {
    "id": "43003016",
    "slide_number": 1,
    "total_slides": 3,
    "slide_type": "default",
    "slides": [
      {
        "number": 1,
        "reference": "43003016"
      },
      {
        "number": 2,
        "reference": "43003017"
      },
      {
        "number": 3,
        "reference": "43003018"
      }
    ]
  }
}
```


---

### GetTriggers
- v2.24.0

Returns the list of saved triggers



**Response:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `data` | _Array&lt;Object&gt;_ |  |
| `data.*.id` | _String_ | Item ID |
| `data.*.enabled` | _Boolean_ |  |
| `data.*.when` | _String_ | Can be: `displaying` `closing` `change` `event` |
| `data.*.type` | _String_ | Type of item. Can be:<br>**when=displaying**: `any_song` `any_text` `any_verse` `any_announcement` `any_audio` `any_video` `any_image` `any_automatic_presentation` `any_song_slide` `any_text_slide` `any_ppt_slide` `any_theme` `any_background` `any_title_subitem` `any_webcam` `any_audio_folder` `any_video_folder` `any_image_folder` `any_ppt` `any_countdown` `any_automatic_presentation_slide` `f8` `f9` `f10`<br><br>**when=closing**: `any_song` `any_text` `any_verse` `any_announcement` `any_audio` `any_video` `any_image` `any_automatic_presentation` `any_webcam` `any_audio_folder` `any_video_folder` `any_image_folder` `any_ppt` `f8` `f9` `f10`<br><br>**when=change**: `countdown_seconds_public` `countdown_seconds_communication_panel` `timer_seconds_communication_panel` `wallpaper` `wallpaper_service` `stage` `playlist` `bpm` `hue` `player_volume` `player_mute` `player_pause` `player_repeat` `player_list_or_single` `player_shuffle`<br><br>**when=event**: `new_message_chat` `verse_presentation_changed` `playlist_changed` `file_modified` `player_progress` |
| `data.*.item.title` | _String_ |  |
| `data.*.item.reference` | _Object_ |  |
| `data.*.receiver.type` | _String_ | Can be: `get` `post` `ws` `tcp` `udp` `midi` `javascript` `community` `multiple_actions` `obs_v4` `obs_v5` `lumikit` `vmix` `osc` `soundcraft` `ha` `ptz` `tbot` `openai` |
| `data.*.description` | _String_ |  |
| `data.*.tags` | _Array&lt;String&gt;_ |  |


**Example:**
```
Response
{
  "status": "ok",
  "data": [
    {
      "id": "xyz",
      "enabled": true,
      "when": "displaying",
      "type": "any_song",
      "item": {
        "title": "",
        "reference": {}
      },
      "receiver": {
        "type": "obs_v5"
      },
      "description": "",
      "tags": []
    }
  ]
}
```


---

### GetGlobalSettings
- v2.25.0



**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `filter` | _String (optional)_ | Name of the settings separated by comma |


**Response:**

| Type  |
| :---: |
| _[GlobalSettings](#global-settings)_ | 


**Example:**
```
Request
{
  "filter": "show_favorite_bar_main_window,fade_in_out_duration"
}

Response
{
  "status": "ok",
  "data": {
    "show_favorite_bar_main_window": true,
    "fade_in_out_duration": 500
  }
}
```


---

### SetGlobalSettings
- v2.25.0



**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `input` | _[GlobalSettings](#global-settings)_ |  |


**Response:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `data` | _Object_ | Key/value pair with the result of the change of each item.<br>`true` if the value was successfully changed, or a `string` with the reason for the error. |


**Example:**
```
Request
{
  "show_favorite_bar_main_window": true,
  "fade_in_out_duration": 100
}

Response
{
  "status": "ok",
  "data": {
    "show_favorite_bar_main_window": true,
    "fade_in_out_duration": "invalid value: 100"
  }
}
```


---

### GetStyledModels
- v2.25.0





**Response:**

| Type  |
| :---: |
| _[StyledModel](#styled-model)_ | 


**Example:**
```
Response
{
  "status": "ok",
  "data": [
    {
      "key": "title",
      "properties": {
        "b": "true",
        "size": "120"
      }
    },
    {
      "key": "footer",
      "properties": {
        "i": "true",
        "size": "80"
      }
    }
  ]
}
```


---

### GetStyledModelsAsMap
- v2.25.0





**Response:**

| Type  | Description |
| :---: | ------------|
| _Object_ | Key/value pair<br>`StyledModel#key : StyledModel#properties` |


**Example:**
```
Response
{
  "status": "ok",
  "data": {
    "title": {
      "b": "true",
      "size": "120"
    },
    "footer": {
      "i": "true",
      "size": "80"
    }
  }
}
```


---

### CreateItem
- v2.25.0

Create a new item<br> <br>This action requires a [Holyrics Plan](https://holyrics.com.br/holyrics_plan.html) subscription to be executed<br>To use this action, you need to enable permission in the settings<br>`file menu > settings > advanced > javascript > settings > advanced permissions`<br>Or if you are using the implementation of a module, grant permission in the module settings and use the `hly` method of the **Module** class `module.hly(action, input)`<br> <br>The structure of the object passed as a parameter must be according to the table below<br><table><tr><td><p align="right">**Action**</p></td><td>Type</td></tr><tr><td><p align="right">CreateSong</p></td><td>[Lyrics](#lyrics)</td></tr><tr><td><p align="right">CreateText</p></td><td>[Text](#text)</td></tr><tr><td><p align="right">CreateTheme</p></td><td>[Theme](#theme)</td></tr><tr><td><p align="right">CreateTeam</p></td><td>[Team](#team)</td></tr><tr><td><p align="right">CreateRole</p></td><td>[Role](#role)</td></tr><tr><td><p align="right">CreateMember</p></td><td>[Member](#member)</td></tr><tr><td><p align="right">CreateEvent</p></td><td>[Event](#event)</td></tr><tr><td><p align="right">CreateSongGroup</p></td><td>[Group](#group)</td></tr></table>



**Response:**

| Type  | Description |
| :---: | ------------|
| _Object_ | Returns the created item |


**Example:**
```
Request
{
  "status": "ok",
  "data": {
    "title": "Title",
    "artist": "Artist",
    "author": "Author",
    "slides": [
      {
        "text": "Example",
        "slide_description": "Verse 1",
        "translations": {
          "pt": "Exemplo"
        }
      },
      {
        "text": "Example",
        "slide_description": "Chorus",
        "translations": {
          "pt": "Exemplo"
        }
      },
      {
        "text": "Example",
        "slide_description": "Verse 2",
        "translations": {
          "pt": "Exemplo"
        }
      }
    ],
    "title_translations": {
      "pt": "Título"
    },
    "orde": "1,2,3,2,2",
    "key": "G",
    "bpm": 80,
    "time_sig": "4/4"
  }
}

Response
{
  "status": "ok",
  "data": {
    "id": "123",
    "title": "Title",
    "artist": "Artist",
    "author": "Author",
    "slides": [
      {
        "text": "Example",
        "slide_description": "Verse 1",
        "translations": {
          "pt": "Exemplo"
        }
      },
      {
        "text": "Example",
        "slide_description": "Chorus",
        "translations": {
          "pt": "Exemplo"
        }
      },
      {
        "text": "Example",
        "slide_description": "Verse 2",
        "translations": {
          "pt": "Exemplo"
        }
      }
    ],
    "title_translations": {
      "pt": "Título"
    },
    "orde": "1,2,3,2,2",
    "key": "G",
    "bpm": 80,
    "time_sig": "4/4"
  }
}
```


---

### EditItem
- v2.25.0

Edit an existing item<br> <br>This action requires a [Holyrics Plan](https://holyrics.com.br/holyrics_plan.html) subscription to be executed<br>To use this action, you need to enable permission in the settings<br>`file menu > settings > advanced > javascript > settings > advanced permissions`<br>Or if you are using the implementation of a module, grant permission in the module settings and use the `hly` method of the **Module** class `module.hly(action, input)`<br> <br>All parameters are optional except: `id`<br>Only the declared parameters will be changed that is, it is not necessary to provide the complete object to change just one parameter.<br>Parameters defined as `read-only` are not editable <br>The structure of the object passed as a parameter must be according to the table below<br><table><tr><td><p align="right">**Action**</p></td><td>Type</td></tr><tr><td><p align="right">EditSong</p></td><td>[Lyrics](#lyrics)</td></tr><tr><td><p align="right">EditText</p></td><td>[Text](#text)</td></tr><tr><td><p align="right">EditTheme</p></td><td>[Theme](#theme)</td></tr><tr><td><p align="right">EditTeam</p></td><td>[Team](#team)</td></tr><tr><td><p align="right">EditRole</p></td><td>[Role](#role)</td></tr><tr><td><p align="right">EditMember</p></td><td>[Member](#member)</td></tr><tr><td><p align="right">EditEvent</p></td><td>[Event](#event)</td></tr><tr><td><p align="right">EditSongGroup</p></td><td>[Group](#group)</td></tr></table>



_Method does not return value_



---

### DeleteItem
- v2.25.0

Deletes an existing item<br> <br>This action requires a [Holyrics Plan](https://holyrics.com.br/holyrics_plan.html) subscription to be executed<br>To use this action, you need to enable permission in the settings<br>`file menu > settings > advanced > javascript > settings > advanced permissions`<br>Or if you are using the implementation of a module, grant permission in the module settings and use the `hly` method of the **Module** class `module.hly(action, input)`<br> <br>Provide the id of the respective item to remove it.<br> <br><table><tr><td><p align="right">**Action**</p></td></tr><tr><td><p align="right">DeleteSong</p></td></tr><tr><td><p align="right">DeleteText</p></td></tr><tr><td><p align="right">DeleteTheme</p></td></tr><tr><td><p align="right">DeleteTeam</p></td></tr><tr><td><p align="right">DeleteRole</p></td></tr><tr><td><p align="right">DeleteMember</p></td></tr><tr><td><p align="right">DeleteEvent</p></td></tr><tr><td><p align="right">DeleteSongGroup</p></td></tr></table>

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `id` | _String_ | ID of the respective item |


_Method does not return value_

**Example:**
```
Request
{
  "id": "123"
}
```


---

### AddSongsToSongGroup
- v2.25.0

Adiciona músicas a um grupo<br> <br>This action requires a [Holyrics Plan](https://holyrics.com.br/holyrics_plan.html) subscription to be executed<br>To use this action, you need to enable permission in the settings<br>`file menu > settings > advanced > javascript > settings > advanced permissions`<br>Or if you are using the implementation of a module, grant permission in the module settings and use the `hly` method of the **Module** class `module.hly(action, input)`

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `group` | _String_ | Group name |
| `songs` | _String_ | List with the id of the songs separated by comma |


_Method does not return value_

**Example:**
```
Request
{
  "group": "Name",
  "songs": "123,456"
}
```


---

### RemoveSongsFromSongGroup
- v2.25.0

Remove músicas de um grupo<br> <br>This action requires a [Holyrics Plan](https://holyrics.com.br/holyrics_plan.html) subscription to be executed<br>To use this action, you need to enable permission in the settings<br>`file menu > settings > advanced > javascript > settings > advanced permissions`<br>Or if you are using the implementation of a module, grant permission in the module settings and use the `hly` method of the **Module** class `module.hly(action, input)`

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `group` | _String_ | Group name |
| `songs` | _String_ | List with the id of the songs separated by comma |


_Method does not return value_

**Example:**
```
Request
{
  "group": "Name",
  "songs": "123,456"
}
```


---



# Classes

## Lyrics
| Name | Type  | Description |
| ---- | :---: | ------------|
| `id` | _String_ | Song ID |
| `title` | _String_ | Song title |
| `artist` | _String_ | Music artist |
| `author` | _String_ | Music author |
| `note` | _String_ | Music annotation |
| `copyright` | _String_ | Music copyright |
| `slides` | _Array&lt;Object&gt;_ |  `v2.21.0+` |
| `slides.*.text` | _String_ | Slide text `v2.21.0+` |
| `slides.*.styled_text` | _String_ | Slide text with **styled** formatting (when available) `v2.24.0+` |
| `slides.*.slide_description` | _String_ | Slide description `v2.21.1+` |
| `slides.*.background_id` | _String_ | ID of the theme or background saved for the slide `v2.21.0+` |
| `slides.*.translations` | _Object_ | Translations for the slide.<br>Key/value pair. `v2.25.0+` |
| `formatting_type` | _String_ | `basic`  `styled`  `advanced`<br> <br>When using this object in creation or editing methods, if `formatting_type=basic` is used, the value of the variable `slides.*.text` will be used; otherwise, the value of the variable `slides.*.styled_text` will be used `Default: basic` `v2.25.0+` |
| `order` | _String_ | Order of slides (index from 1), separated by comma `v2.21.0+` |
|  | _Array&lt;[SongArrangement](#song-arrangement)&gt;_ |  `v2.25.1+` |
| `title_translations` | _Object_ | Translations for the title slide.<br>Key/value pair. `v2.25.0+` |
| `key` | _String_ | Tone of music.<br>Can be: `C` `C#` `Db` `D` `D#` `Eb` `E` `F` `F#` `Gb` `G` `G#` `Ab` `A` `A#` `Bb` `B` `Cm` `C#m` `Dbm` `Dm` `D#m` `Ebm` `Em` `Fm` `F#m` `Gbm` `Gm` `G#m` `Abm` `Am` `A#m` `Bbm` `Bm` |
| `bpm` | _Number_ | BPM of the song |
| `time_sig` | _String_ | Music time.<br>Can be: `2/2` `2/4` `3/4` `4/4` `5/4` `6/4` `3/8` `6/8` `7/8` `9/8` `12/8` |
| `groups` | _Array&lt;[Group](#group)&gt;_ | Groups where music is added `read-only` |
| `linked_audio_file` | _String_ | Path of the audio file linked to the song `v2.22.0+` |
| `linked_backing_track_file` | _String_ | Path of the audio file (backing track) linked to the song `v2.22.0+` |
| `streaming` | _Object_ | URI or ID of the streamings `v2.22.0+` |
| `streaming.audio` | _Object_ | Audio `v2.22.0+` |
| `streaming.audio.spotify` | _String_ |  `v2.22.0+` |
| `streaming.audio.youtube` | _String_ |  `v2.22.0+` |
| `streaming.audio.deezer` | _String_ |  `v2.22.0+` |
| `streaming.backing_track` | _Object_ | Backing track `v2.22.0+` |
| `streaming.backing_track.spotify` | _String_ |  `v2.22.0+` |
| `streaming.backing_track.youtube` | _String_ |  `v2.22.0+` |
| `streaming.backing_track.deezer` | _String_ |  `v2.22.0+` |
| `midi` | _[Midi](#midi)_ | Item MIDI shortcut |
| `extras` | _Object_ | Map of extra objects (added by the user) `v2.21.0+` |
| `theme` | _String_ | Saved theme ID for the song `v2.25.0+` |
| `archived` | _Boolean_ | If the song is archived |
| `metadata.modified_time_millis` | _Number_ | File modification date. (timestamp) `v2.25.0+` `read-only` |
<details>
  <summary>See example</summary>

```json
{
  "id": "0",
  "title": "",
  "artist": "",
  "author": "",
  "note": "",
  "copyright": "",
  "slides": [
    {
      "text": "Slide 1 line 1\nSlide 1 line 2",
      "styled_text": "Slide 1 line 1\nSlide 1 line 2",
      "slide_description": "Verse 1",
      "background_id": null,
      "translations": null
    },
    {
      "text": "Slide 2 line 1\nSlide 2 line 2",
      "styled_text": "Slide 2 line 1\nSlide 2 line 2",
      "slide_description": "Chorus",
      "background_id": null,
      "translations": null
    },
    {
      "text": "Slide 3 line 1\nSlide 3 line 2",
      "styled_text": "Slide 3 line 1\nSlide 3 line 2",
      "slide_description": "Verse 2",
      "background_id": null,
      "translations": null
    }
  ],
  "formatting_type": "basic",
  "order": "1,2,3,2,2",
  "title_translations": null,
  "key": "",
  "bpm": 0.0,
  "time_sig": "",
  "linked_audio_file": "",
  "linked_backing_track_file": "",
  "streaming": {
    "audio": {
      "spotify": "",
      "youtube": "",
      "deezer": ""
    },
    "backing_track": {
      "spotify": "",
      "youtube": "",
      "deezer": ""
    }
  },
  "extras": {
    "extra": ""
  },
  "theme": null,
  "archived": false
}
```
</details>

## Text
| Name | Type  | Description |
| ---- | :---: | ------------|
| `id` | _String_ | Text ID |
| `title` | _String_ | Text title |
| `folder` | _String_ | Path of the location folder |
| `theme` | _String_ | ID of the theme saved for the text |
| `slides` | _Array&lt;Object&gt;_ |  |
| `slides.*.text` | _String_ | Slide text |
| `slides.*.styled_text` | _String_ | Slide text with **styled** formatting (when available) `v2.24.0+` |
| `slides.*.background_id` | _String_ | ID of the theme or background saved for the slide |
| `slides.*.translations` | _Object_ | Translations for the slide.<br>Key/value pair. `v2.25.0+` |
| `formatting_type` | _String_ | `basic`  `styled`  `advanced`<br> <br>When using this object in creation or editing methods, if `formatting_type=basic` is used, the value of the variable `slides.*.text` will be used; otherwise, the value of the variable `slides.*.styled_text` will be used `Default: basic` `v2.25.0+` |
| `extras` | _Object_ | Map of extra objects (added by the user) `v2.24.0+` |
| `metadata.modified_time_millis` | _Number_ | File modification date. (timestamp) `v2.25.0+` `read-only` |
<details>
  <summary>See example</summary>

```json
{
  "id": "",
  "title": "",
  "folder": "",
  "theme": null,
  "slides": [
    {
      "text": "Slide 1 line 1\nSlide 1 line 2",
      "styled_text": "Slide 1 line 1\nSlide 1 line 2",
      "background_id": null,
      "translations": null
    },
    {
      "text": "Slide 2 line 1\nSlide 2 line 2",
      "styled_text": "Slide 2 line 1\nSlide 2 line 2",
      "background_id": null,
      "translations": null
    },
    {
      "text": "Slide 3 line 1\nSlide 3 line 2",
      "styled_text": "Slide 3 line 1\nSlide 3 line 2",
      "background_id": null,
      "translations": null
    }
  ],
  "formatting_type": "basic"
}
```
</details>

## Theme
| Name | Type  | Description |
| ---- | :---: | ------------|
| `copy_from_id` | _String (optional)_ | ID of an existing Theme to use as initial copy when creating a new item |
| `id` | _String_ | Item ID |
| `name` | _String_ | Item name |
| <br>**background** |  | <br>Background |
| `background.type` | _String_ | Type of background. Can be: `color`  `my_video`  `my_image`  `video`  `image`  `pattern`  `transparent`  `image_file`  `video_file` |
| `background.id` | _String_ | <table><tr><td><p align="right">**Type**</p></td><td>Value</td></tr><tr><td><p align="right">color</p></td><td>Color in hexadecimal format</td></tr><tr><td><p align="right">my_video</p></td><td>Item ID</td></tr><tr><td><p align="right">my_image</p></td><td>Item ID</td></tr><tr><td><p align="right">video</p></td><td>Item ID</td></tr><tr><td><p align="right">image</p></td><td>Item ID</td></tr><tr><td><p align="right">pattern</p></td><td>Item ID</td></tr><tr><td><p align="right">transparent</p></td><td>"transparent"</td></tr><tr><td><p align="right">image_file</p></td><td>File name in the library</td></tr><tr><td><p align="right">video_file</p></td><td>File name in the library</td></tr></table> |
| `background.adjust_type` | _String_ | `fill` `extend` `adjust` `side_by_side` `center`<br>Available for: **type=my_image**, **type=image** |
| `background.opacity` | _Number_ | Opacity. `0 ~ 100` |
| `background.velocity` | _Number_ | Available for: **type=my_video**, **type=video**<br>`0.25 ~ 4.0` |
| `base_color` | _String_ | Color in hexadecimal format. Base color of the background when reducing opacity. |
| <br>**font** |  | <br>Source |
| `font.name` | _String_ | Font name |
| `font.bold` | _Boolean_ | Bold |
| `font.italic` | _Boolean_ | Italic |
| `font.size` | _Number_ | Size `0.0 ~ 0.4`<br>Value in percentage, based on the slide height. |
| `font.color` | _String_ | Color in hexadecimal format |
| `font.line_spacing` | _Number_ | Line spacing. `-0.5 ~ 1.0`<br>Value in percentage based on the line height. |
| `font.char_spacing` | _Number_ | Spacing between characters. `-40 ~ 120` |
| <br>**align** |  | <br>Alignment |
| `align.horizontal` | _String_ | `left`  `center`  `right`  `justify` |
| `align.vertical` | _String_ | `top`  `middle`  `bottom` |
| `align.margin.top` | _Number_ | `0 ~ 90` |
| `align.margin.right` | _Number_ | `0 ~ 90` |
| `align.margin.bottom` | _Number_ | `0 ~ 90` |
| `align.margin.left` | _Number_ | `0 ~ 90` |
| <br>**effect** |  | <br>Font effects |
| `effect.outline_color` | _String_ | Color in hexadecimal format |
| `effect.outline_weight` | _Number_ | `0.0 ~ 100.0` |
| `effect.brightness_color` | _String_ | Color in hexadecimal format |
| `effect.brightness_weight` | _Number_ | `0.0 ~ 100.0` |
| `effect.shadow_color` | _String_ | Color in hexadecimal format |
| `effect.shadow_x_weight` | _Number_ | `-100.0 ~ 100.0` |
| `effect.shadow_y_weight` | _Number_ | `-100.0 ~ 100.0` |
| `effect.blur` | _Boolean_ | Apply 'blur' effect on brightness and shadow |
| <br>**shape_fill** |  | <br>Background color |
| `shape_fill.type` | _String_ | `box`  `line`  `line_fill`  `theme_margin` |
| `shape_fill.enabled` | _Boolean_ |  |
| `shape_fill.color` | _String_ | Color in hexadecimal format (RGBA) |
| `shape_fill.margin.top` | _Number_ | `0 ~ 100` |
| `shape_fill.margin.right` | _Number_ | `0 ~ 100` |
| `shape_fill.margin.bottom` | _Number_ | `0 ~ 100` |
| `shape_fill.margin.left` | _Number_ | `0 ~ 100` |
| `shape_fill.corner` | _Number_ | `0 ~ 100` |
| <br>**shape_outline** |  | <br>Outline |
| `shape_outline.type` | _String_ | `box`  `line`  `line_fill`  `theme_margin` |
| `shape_outline.enabled` | _Boolean_ |  |
| `shape_outline.color` | _String_ | Color in hexadecimal format (RGBA) |
| `shape_outline.outline_thickness` | _Number_ | `0 ~ 25` |
| `shape_outline.margin.top` | _Number_ | `0 ~ 100` |
| `shape_outline.margin.right` | _Number_ | `0 ~ 100` |
| `shape_outline.margin.bottom` | _Number_ | `0 ~ 100` |
| `shape_outline.margin.left` | _Number_ | `0 ~ 100` |
| `shape_outline.corner` | _Number_ | `0 ~ 100` |
| <br>**comment** |  | <br>Comment |
| `comment.font_name` | _String_ | Font name |
| `comment.bold` | _Boolean_ | Bold |
| `comment.italic` | _Boolean_ | Italic |
| `comment.relative_size` | _Number_ | relative font size. `40 ~ 100` |
| `comment.color` | _String_ | Color in hexadecimal format |
| <br>**settings** |  | <br>Settings |
| `settings.uppercase` | _Boolean_ | Display the text in uppercase |
| `settings.line_break` | _String_ | Apply line break. `system`  `true`  `false`<br> `Default: system` |
| <br>**metadata** |  | <br> |
| `metadata.modified_time_millis` | _Number_ | File modification date. (timestamp) `v2.25.0+` `read-only` |
<details>
  <summary>See example</summary>

```json
{
  "id": "123",
  "name": "",
  "background": {
    "type": "color",
    "id": "212121",
    "opacity": 100
  },
  "base_color": "FFFFFF",
  "font": {
    "name": "CMG Sans",
    "bold": true,
    "italic": false,
    "size": 10.0,
    "color": "F5F5F5",
    "line_spacing": 0.3,
    "char_spacing": 0
  },
  "align": {
    "horizontal": "center",
    "vertical": "middle",
    "margin": {
      "top": 3.0,
      "right": 3.0,
      "bottom": 3.0,
      "left": 3.0
    }
  },
  "effect": {
    "outline_color": "404040",
    "outline_weight": 0.0,
    "brightness_color": "C0C0C0",
    "brightness_weight": 0.0,
    "shadow_color": "404040",
    "shadow_x_weight": 0.0,
    "shadow_y_weight": 0.0,
    "blur": true
  },
  "shape_fill": {
    "type": "box",
    "enabled": false,
    "color": "000000",
    "margin": {
      "top": 5.0,
      "right": 30.0,
      "bottom": 10.0,
      "left": 30.0
    },
    "corner": 0
  },
  "shape_outline": {
    "type": "box",
    "enabled": false,
    "color": "000000",
    "outline_thickness": 10,
    "margin": {
      "top": 5.0,
      "right": 30.0,
      "bottom": 10.0,
      "left": 30.0
    },
    "corner": 0
  },
  "comment": {
    "font_name": "Arial",
    "bold": false,
    "italic": true,
    "relative_size": 100,
    "color": "A0A0A0"
  },
  "settings": {
    "uppercase": false,
    "line_break": "system"
  }
}
```
</details>

## Background
| Name | Type  | Description |
| ---- | :---: | ------------|
| `id` | _String_ | Item ID |
| `type` | _String_ | Type of item. Can be: `theme` `my_video` `my_image` `video` `image` |
| `name` | _String_ | Item name |
| `width` | _Number (optional)_ |  |
| `height` | _Number (optional)_ |  |
| `duration` | _Number (optional)_ | Duration in milliseconds |
| `tags` | _Array&lt;String&gt;_ | Item tag list |
| `bpm` | _Number_ | BPM value of item |
| `midi` | _[Midi](#midi) (optional)_ | Item MIDI shortcut |
<details>
  <summary>See example</summary>

```json
{
  "id": "10",
  "type": "video",
  "name": "Hexagons",
  "duration": "29050",
  "width": "1280",
  "height": "720",
  "bpm": 0.0
}
```
</details>

## Slide Description
| Name | Type  | Description |
| ---- | :---: | ------------|
| `name` | _String_ | Item name |
| `tag` | _String_ | Short name of the item |
| `aliases` | _Array&lt;String&gt;_ | List with alternative names |
| `font_color` | _String_ | Font color in hexadecimal format |
| `bg_color` | _String_ | Background color in hexadecimal format |
| `background` | _Number_ | ID of the custom background |
| `midi` | _[Midi](#midi) (optional)_ | Item MIDI shortcut |
<details>
  <summary>See example</summary>

```json
{
  "name": "Chorus",
  "tag": "C",
  "font_color": "FFFFFF",
  "bg_color": "000080",
  "background": null,
  "midi": null
}
```
</details>

## Item
| Name | Type  | Description |
| ---- | :---: | ------------|
| `id` | _String_ | Item ID |
| `type` | _String_ | Type of item. It can be: `title`  `song`  `verse`  `text`  `audio`  `video`  `image`  `file`  `announcement`  `automatic_presentation`  `countdown`  `countdown_cp`  `cp_text`  `plain_text`  `uri`  `global_action`  `api`  `script` |
| `name` | _String_ | Item name |

## Group
| Name | Type  | Description |
| ---- | :---: | ------------|
| `id` | _String_ | Item ID. (The same value as `name`). `v2.25.0+` |
| `name` | _String_ | Item name `read-only` |
| `songs` | _Array&lt;String&gt;_ | List of song IDs |
| `add_chorus_between_verses` | _Boolean_ |  `v2.25.0+` |
| `hide_in_interface` | _Boolean_ |  `v2.25.0+` |
| `metadata.modified_time_millis` | _Number_ | File modification date. (timestamp) `v2.25.0+` `read-only` |

## Song Arrangement
| Name | Type  | Description |
| ---- | :---: | ------------|
| `name` | _String_ | Item name |
| `sequence` | _String_ | Order of slides (index from 1), separated by comma |
| `collections` | _Array&lt;String&gt;_ | Short name of the item |
<details>
  <summary>See example</summary>

```json
{
  "name": "",
  "sequence": "1,2,3,2,2"
}
```
</details>

## Announcement
| Name | Type  | Description |
| ---- | :---: | ------------|
| `id` | _String_ | Item ID |
| `name` | _String_ | Item name |
| `text` | _String_ | Announcement text |
| `archived` | _Boolean_ | If the item is archived |

## Midi
| Name | Type  | Description |
| ---- | :---: | ------------|
| `code` | _Number_ | MIDI code |
| `velocity` | _Number_ | Speed/intensity midi |
<details>
  <summary>See example</summary>

```json
{
  "code": 80,
  "velocity": 20
}
```
</details>

## Favorite Item
| Name | Type  | Description |
| ---- | :---: | ------------|
| `id` | _String_ | Item ID |
| `name` | _String_ | Item name |

## Service
| Name | Type  | Description |
| ---- | :---: | ------------|
| `id` | _String_ | Item ID `v2.25.0+` |
| `name` | _String_ | Item name |
| `disabled` | _Boolean_ | Returns **true** if the item is set to disabled `v2.25.0+` |
| `week` | _String_ | Week. Can be: `all` `first` `second` `third` `fourth` `last` |
| `day` | _String_ | Day of the week. Can be: `sun` `mon` `tue` `wed` `thu` `fri` `sat` |
| `hour` | _Number_ | Hour [0-23] |
| `minute` | _Number_ | Minute [0-59] |
| `type` | _String_ | Type of item. Can be: `service` `event` |
| `hide_week` | _Array&lt;String&gt;_ | List of hidden weeks. Available if `week=all` |
| `metadata.modified_time_millis` | _Number_ | File modification date. (timestamp) `v2.25.0+` `read-only` |

## Event
| Name | Type  | Description |
| ---- | :---: | ------------|
| `id` | _String_ | Item ID `v2.25.0+` |
| `name` | _String_ | Item name |
| `description` | _String_ | Item description `v2.25.0+` |
| `datetime` | _String_ | Date and time format: YYYY-MM-DD HH:MM |
| `datetime_millis` | _String_ | timestamp `v2.24.0+` `read-only` |
| `wallpaper` | _String_ | Relative path of the file used as the event wallpaper |
| `metadata.modified_time_millis` | _Number_ | File modification date. (timestamp) `v2.25.0+` `read-only` |
| `metadata.service` | _[Service](#service)_ | Regular service or event that gives rise to this event. It can be `null` if it is an individually created event. `v2.25.0+` `read-only` |

## Schedule
| Name | Type  | Description |
| ---- | :---: | ------------|
| `type` | _String_ | Playlist type. Can be: temporary, service, event |
| `name` | _String_ |  |
| `datetime` | _String_ | Date and time format: YYYY-MM-DD HH:MM |
| `lyrics_playlist` | _Array&lt;[Lyrics](#lyrics)&gt;_ | Lyrics list |
| `media_playlist` | _Array&lt;[Item](#item)&gt;_ | Media list |
| `responsible` | _[Member](#member)_ | Member defined as responsible for the event |
| `members` | _Array&lt;Object&gt;_ | List of members |
| `members.*.id` | _String_ | Member ID |
| `members.*.name` | _String_ | Name of the member |
| `members.*.scheduled` | _Boolean_ | Whether the integrand is scaled or defined for a function |
| `roles` | _Array&lt;Object&gt;_ | List of functions on the scale |
| `roles.*.id` | _String_ | Function ID |
| `roles.*.name` | _String_ | Function name |
| `roles.*.member` | _[Member](#member)_ | Member assigned to the role |
| `notes` | _String_ | Notes `v2.21.0+` |
| `metadata.modified_time_millis` | _Number_ | File modification date. (timestamp) `v2.25.0+` `read-only` |
| `metadata.event` | _[Event](#event)_ | Event that gives rise to this playlist. It can be `null` if `type=temporary`. `v2.25.0+` `read-only` |
<details>
  <summary>See example</summary>

```json
{
  "type": "temporary",
  "name": "",
  "datetime": "2024-01-16 20:00",
  "lyrics_playlist": [
    {
      "id": 1,
      "title": "Title 1",
      "artist": "",
      "author": "",
      "...": ".."
    },
    {
      "id": 2,
      "title": "Title 2",
      "artist": "",
      "author": "",
      "...": ".."
    },
    {
      "id": 3,
      "title": "Title 3",
      "artist": "",
      "author": "",
      "...": ".."
    }
  ],
  "media_playlist": [
    {
      "id": "a",
      "type": "video",
      "name": "file.mp4"
    },
    {
      "id": "b",
      "type": "audio",
      "name": "file.mp3"
    },
    {
      "id": "c",
      "type": "image",
      "name": "file.jpg"
    }
  ],
  "responsible": null,
  "notes": ""
}
```
</details>

## Team
| Name | Type  | Description |
| ---- | :---: | ------------|
| `id` | _String_ | Item ID |
| `name` | _String_ | Item name |
| `description` | _String_ | Item description |
| `metadata.modified_time_millis` | _Number_ | File modification date. (timestamp) `v2.25.0+` `read-only` |

## Member
| Name | Type  | Description |
| ---- | :---: | ------------|
| `id` | _String_ | Item ID |
| `name` | _String_ | Item name |
| `disabled` | _Boolean_ | Returns **true** if the item is set to disabled `v2.25.0+` |
| `skills` | _String_ | Skills |
| `roles` | _Array&lt;[Role](#role)&gt;_ | Roles |
| `metadata.modified_time_millis` | _Number_ | File modification date. (timestamp) `v2.25.0+` `read-only` |

## Role
| Name | Type  | Description |
| ---- | :---: | ------------|
| `id` | _String_ | Item ID |
| `name` | _String_ | Item name |
| `disabled` | _Boolean_ | Returns **true** if the item is set to disabled `v2.25.0+` |
| `team` | _[Team](#team)_ | Team |
| `metadata.modified_time_millis` | _Number_ | File modification date. (timestamp) `v2.25.0+` `read-only` |

## Automatic Presentation
| Name | Type  | Description |
| ---- | :---: | ------------|
| `name` | _String_ | Item name |
| `duration` | _Number_ | Duration in milliseconds |
| `starts_with` | _String_ | Accepted values: `title` `blank` |
| `song` | _[Lyrics](#lyrics)_ |  |
| `timeline` | _Array&lt;Object&gt;_ | Information about the start and duration of each slide in the presentation |
| `timeline.*.number` | _Number_ | number >= 0 |
| `timeline.*.start` | _Number_ | Start time of the presentation in milliseconds |
| `timeline.*.end` | _Number_ | End time of the presentation in milliseconds |
| `timeline.*.duration` | _Number_ | Duration in milliseconds |

## Automatic
| Name | Type  | Description |
| ---- | :---: | ------------|
| `seconds` | _Number_ | Time each item will be displayed |
| `repeat` | _Boolean_ | **true** to keep repeating the presentation (go back to the first item after the last one) |

## Presentation Slide Info
| Name | Type  | Description |
| ---- | :---: | ------------|
| `number` | _Number_ | Slide number (starts at 1) |
| `text` | _String_ | Slide text |
| `theme_id` | _String_ | Slide theme ID |
| `slide_description` | _String (optional)_ | Slide description name. Available if it is a music presentation. |
| `preview` | _String (optional)_ | Image in base64 format |

## Trigger Item
| Name | Type  | Description |
| ---- | :---: | ------------|
| `id` | _String (optional)_ | Item ID |
| `when` | _String_ | `displaying` `closing` `change` `event` |
| `item` | _String_ | Type of item. Can be:<br>**when=displaying**: `any_song` `any_text` `any_verse` `any_announcement` `any_audio` `any_video` `any_image` `any_automatic_presentation` `any_song_slide` `any_text_slide` `any_ppt_slide` `any_theme` `any_background` `any_title_subitem` `any_webcam` `any_audio_folder` `any_video_folder` `any_image_folder` `any_ppt` `any_countdown` `any_automatic_presentation_slide` `f8` `f9` `f10`<br><br>**when=closing**: `any_song` `any_text` `any_verse` `any_announcement` `any_audio` `any_video` `any_image` `any_automatic_presentation` `any_webcam` `any_audio_folder` `any_video_folder` `any_image_folder` `any_ppt` `f8` `f9` `f10`<br><br>**when=change**: `countdown_seconds_public` `countdown_seconds_communication_panel` `timer_seconds_communication_panel` `wallpaper` `wallpaper_service` `stage` `playlist` `bpm` `hue` `player_volume` `player_mute` `player_pause` `player_repeat` `player_list_or_single` `player_shuffle`<br><br>**when=event**: `new_message_chat` `verse_presentation_changed` `playlist_changed` `file_modified` `player_progress` |
| `action` | _Function_ | Action to be executed.<br>`function(obj) { /*  */ }`<br>Content of `obj` according to the item type:<br>[`any_song`](https://github.com/holyrics/jslib/blob/main/README-en.md#songinfo)  [`any_text`](https://github.com/holyrics/jslib/blob/main/README-en.md#textinfo)  [`any_verse`](https://github.com/holyrics/jslib/blob/main/README-en.md#verseinfo)  [`any_announcement`](https://github.com/holyrics/jslib/blob/main/README-en.md#announcementinfo)  [`any_audio`](https://github.com/holyrics/jslib/blob/main/README-en.md#audioinfo)  [`any_video`](https://github.com/holyrics/jslib/blob/main/README-en.md#videoinfo)  [`any_image`](https://github.com/holyrics/jslib/blob/main/README-en.md#imageinfo)  [`any_automatic_presentation`](https://github.com/holyrics/jslib/blob/main/README-en.md#automaticpresentationinfo)  [`any_song_slide`](https://github.com/holyrics/jslib/blob/main/README-en.md#songslideinfo)  [`any_text_slide`](https://github.com/holyrics/jslib/blob/main/README-en.md#textslideinfo)  [`any_ppt_slide`](https://github.com/holyrics/jslib/blob/main/README-en.md#pptslideinfo)  [`any_theme`](https://github.com/holyrics/jslib/blob/main/README-en.md#themeinfo)  [`any_background`](https://github.com/holyrics/jslib/blob/main/README-en.md#backgroundinfo)  [`any_title_subitem`](https://github.com/holyrics/jslib/blob/main/README-en.md#titleinfo)  [`any_webcam`](https://github.com/holyrics/jslib/blob/main/README-en.md#webcaminfo)  [`any_audio_folder`](https://github.com/holyrics/jslib/blob/main/README-en.md#audioinfo)  [`any_video_folder`](https://github.com/holyrics/jslib/blob/main/README-en.md#videoinfo)  [`any_image_folder`](https://github.com/holyrics/jslib/blob/main/README-en.md#imageinfo)  [`any_ppt`](https://github.com/holyrics/jslib/blob/main/README-en.md#pptinfo)  [`any_countdown`](https://github.com/holyrics/jslib/blob/main/README-en.md#countdowninfo)  [`any_automatic_presentation_slide`](https://github.com/holyrics/jslib/blob/main/README-en.md#automaticpresentationslideinfo)  [`f8`](https://github.com/holyrics/jslib/blob/main/README-en.md#presentationmodifierinfoinfo)  [`f9`](https://github.com/holyrics/jslib/blob/main/README-en.md#presentationmodifierinfoinfo)  [`f10`](https://github.com/holyrics/jslib/blob/main/README-en.md#presentationmodifierinfoinfo)  [`new_message_chat`](https://github.com/holyrics/jslib/blob/main/README-en.md#newchatmessageinfo)  [`verse_presentation_changed`](https://github.com/holyrics/jslib/blob/main/README-en.md#versepresentationchangedinfo)  [`playlist_changed`](https://github.com/holyrics/jslib/blob/main/README-en.md#playlistchangedinfo)  [`file_modified`](https://github.com/holyrics/jslib/blob/main/README-en.md#filemodifiedinfo)  [`player_progress`](https://github.com/holyrics/jslib/blob/main/README-en.md#playerprogressinfo)<br><br>All items with **when=change** contain: `obj.id` `obj.name` `obj.old_value` `obj.new_value` |
| `name` | _String (optional)_ | Item name. Compatible value for display in **JavaScript Monitor** `v2.23.0+` |
| `filter` | _Object (optional)_ | Execute action only if the object that triggered the event matches the filter object `v2.24.0+` |
<details>
  <summary>See example</summary>

```javascript
{
  "id": "",
  "when": "displaying",
  "item": "any_song",
  "action": function(obj) { /* TODO */ },
  "name": "name"
}
```
</details>

## Play Media Settings
Settings for media execution

| Name | Type  | Description |
| ---- | :---: | ------------|
| `volume` | _Number_ | Change the volume of the player |
| `repeat` | _Boolean_ | Change the **repeat** option |
| `shuffle` | _Boolean_ | Change the **random** option |
| `start_time` | _String_ | Start time for execution in SS, MM:SS or HH:MM:SS format |
| `stop_time` | _String_ | End time for execution in SS, MM:SS or HH:MM:SS format |
<details>
  <summary>See example</summary>

```json
{
  "volume": "80",
  "repeat": true,
  "shuffle": false,
  "start_time": "00:30",
  "stop_time": "02:00"
}
```
</details>

## Display Settings
Display settings

| Name | Type  | Description |
| ---- | :---: | ------------|
| `id` | _String_ | Item ID. `public` `screen_2` `screen_3` `screen_?` `stream_image` `stream_html_1` `stream_html_2` `stream_html_3` |
| `name` | _String_ | Item name |
| `stage_view` | _[StageView](#stage-view)_ | Stage view settings. (Unavailable for public screen) |
| `slide_info` | _[SlideAdditionalInfo](#slide-additional-info)_ | Additional slide info |
| `slide_translation` | _String_ | translation name |
| `slide_translation_custom_settings` | _[TranslationCustomSettings](#translation-custom-settings)_ | Custom translation settings |
| `bible_version_tab` | _Number_ | Tab number (1, 2 or 3) of the Bible translation displayed on the screen, as translations are loaded in the Bible window |
| `margin` | _Object_ | Margins set in **Edit screen position** option. margin.top, margin.right, margin.bottom, margin.left |
| `area` | _[Rectangle](#rectangle)_ | Screen area with margins applied (if available) |
| `total_area` | _[Rectangle](#rectangle)_ | Total screen area on the system |
| `hide` | _Boolean_ | Hide screen |
| `show_items` | _Object_ | Defines the presentation types that will be displayed (only available for stream screens - image and html) |
| `show_items.lyrics` | _Boolean_ | Song lyrics |
| `show_items.text` | _Boolean_ | Text |
| `show_items.verse` | _Boolean_ | Verse |
| `show_items.image` | _Boolean_ | Image |
| `show_items.alert` | _Boolean_ | Alert |
| `show_items.announcement` | _Boolean_ | Announcement |
| `media_player.show` | _Boolean_ | Display VLC Player `v2.20.0+` |
| `media_player.margin` | _[Rectangle](#rectangle)_ | Margin for displaying videos by VLC Player `v2.20.0+` |
<details>
  <summary>See example</summary>

```json
{
  "id": "public",
  "name": "Público",
  "screen": "1920,0",
  "slide_info": {
    "info_1": {
      "show_page_count": false,
      "show_slide_description": false,
      "horizontal_align": "right",
      "vertical_align": "bottom"
    },
    "info_2": {
      "show": false,
      "layout_row_1": "<title>< (%author_or_artist%)>",
      "layout_text_row_1": "",
      "horizontal_align": "right",
      "vertical_align": "bottom"
    },
    "font": {
      "name": null,
      "bold": null,
      "italic": null,
      "color": null
    },
    "height": 7,
    "paint_theme_effect": true
  },
  "slide_translation": null,
  "slide_translation_custom_settings": {
    "translation_1": {
      "name": "default",
      "style": "",
      "prefix": "",
      "suffix": ""
    },
    "translation_2": null,
    "translation_3": null,
    "translation_4": null,
    "merge": true,
    "uppercase": false,
    "blank_line_height": 40
  },
  "margin": {
    "top": 0.0,
    "right": 0.0,
    "bottom": 0.0,
    "left": 0.0
  },
  "area": {
    "x": 1920,
    "y": 0,
    "width": 1920,
    "height": 1080
  },
  "total_area": {
    "x": 1920,
    "y": 0,
    "width": 1920,
    "height": 1080
  },
  "hide": false,
  "media_player": {
    "margin": {
      "top": 0.0,
      "right": 0.0,
      "bottom": 0.0,
      "left": 0.0
    },
    "area": {
      "x": 1920,
      "y": 0,
      "width": 1920,
      "height": 1080
    }
  }
}
```
</details>

## Transition Effect Settings
| Name | Type  | Description |
| ---- | :---: | ------------|
| `type` | _String_ | Type of effect. Can be: `random` `fade` `slide` `accordion` `linear_fade` `zoom` `curtain` |
| `enabled` | _Boolean_ | Whether it is enabled or disabled |
| `duration` | _Number_ | Total duration of the transition (in milliseconds) `200 ~ 2400` |
| `only_area_within_margin` | _Number_ | Performs the transition effect only within the margin defined in the Theme. (Available only for text transition) |
| <br>**type=fade** |  |  |
| `merge` | _Object_ | Accepted values: true,&nbsp;false |
| `division_point` | _Object_ | Accepted values: min:&nbsp;10,&nbsp;max:&nbsp;100 |
| `increase_duration_blank_slides` | _Object_ | Accepted values: true,&nbsp;false |
| <br>**type=slide** |  |  |
| `direction` | _Object_ | Accepted values: random,&nbsp;left,&nbsp;up |
| `slide_move_type` | _Object_ | Accepted values: random,&nbsp;move_new,&nbsp;move_old,&nbsp;move_both |
| <br>**type=accordion** |  |  |
| `direction` | _Object_ | Accepted values: random,&nbsp;horizontal,&nbsp;vertical |
| <br>**type=linear_fade** |  |  |
| `direction` | _Object_ | Accepted values: random,&nbsp;horizontal,&nbsp;vertical,&nbsp;up,&nbsp;down,&nbsp;left,&nbsp;right |
| `distance` | _Object_ | Accepted values: min:&nbsp;5,&nbsp;max:&nbsp;90 |
| `fade` | _Object_ | Accepted values: min:&nbsp;2,&nbsp;max:&nbsp;90 |
| <br>**type=zoom** |  |  |
| `zoom_type` | _Object_ | Accepted values: random,&nbsp;increase,&nbsp;decrease |
| `directions` | _Object_ | Accepted values: {<br>&nbsp;&nbsp;top_left:&nbsp;boolean,<br>&nbsp;&nbsp;top_center:&nbsp;boolean,<br>&nbsp;&nbsp;top_right:&nbsp;boolean,<br>&nbsp;&nbsp;middle_left:&nbsp;boolean,<br>&nbsp;&nbsp;middle_center:&nbsp;boolean,<br>&nbsp;&nbsp;middle_right:&nbsp;boolean,<br>&nbsp;&nbsp;bottom_left:&nbsp;boolean,<br>&nbsp;&nbsp;bottom_center:&nbsp;boolean,<br>&nbsp;&nbsp;bottom_right:&nbsp;boolean<br>} |
| <br>**type=curtain** |  |  |
| `direction` | _Object_ | Accepted values: random,&nbsp;horizontal,&nbsp;vertical |
| `direction_lines` | _Object_ | Accepted values: random,&nbsp;down_right,&nbsp;up_left,&nbsp;alternate |
| `slide_move_type` | _Object_ | Accepted values: random,&nbsp;new,&nbsp;old,&nbsp;both |
| <br>**type=random** |  |  |
| `random_enabled_types` | _Object_ | Accepted values: {<br>&nbsp;&nbsp;fade:&nbsp;boolean,<br>&nbsp;&nbsp;slide:&nbsp;boolean,<br>&nbsp;&nbsp;accordion:&nbsp;boolean,<br>&nbsp;&nbsp;linear_fade:&nbsp;boolean,<br>&nbsp;&nbsp;zoom:&nbsp;boolean,<br>&nbsp;&nbsp;curtain:&nbsp;boolean<br>} |
<details>
  <summary>See example</summary>

```json
{
  "enabled": true,
  "type": "fade",
  "duration": 500,
  "only_area_within_margin": false,
  "merge": false,
  "division_point": 30,
  "increase_duration_blank_slides": false
}
```
</details>

## Bible Settings
| Name | Type  | Description |
| ---- | :---: | ------------|
| `tab_version_1` | _String_ | Bible version set in the first tab |
| `tab_version_2` | _String_ | Bible version set in the second tab |
| `tab_version_3` | _String_ | Bible version set in the third tab |
| `show_x_verses` | _Number_ | Number of verses displayed in the projection |
| `uppercase` | _Boolean_ | Display the verse text in uppercase |
| `show_only_reference` | _Boolean_ | Display only the verse reference |
| `show_two_versions` | _Boolean_ | `deprecated` Replaced for: `show_second_version` `show_third_version`<br>Display two versions. |
| `show_second_version` | _Boolean_ | Show second version `v2.22.0+` |
| `show_third_version` | _Boolean_ | Show third version `v2.22.0+` |
| `book_panel_type` | _String_ | Type of view of the books of the Bible `grid` `list` |
| `book_panel_order` | _String_ | Type of sorting of the books of the Bible |
| `book_panel_order_available_items` | _Array&lt;String&gt;_ |  |
| `multiple_verses_separator_type` | _String_ | Type of separation in the display of multiple verses. Can be: no_line_break, single_line_break, double_line_break |
| `multiple_versions_separator_type` | _String_ | Separation type in multiple version view. Can be: no_line_break, single_line_break, double_line_break `v2.22.0+` |
| `versification` | _Boolean_ | Apply verse mapping |
| `theme` | _Object_ | Display Theme ID for the different system screens |
| `theme.public` | _String_ |  |
| `theme.screen_n` | _String_ | n >= 2 |
<details>
  <summary>See example</summary>

```json
{
  "tab_version_1": "pt_???",
  "tab_version_2": "es_???",
  "tab_version_3": "en_???",
  "show_x_verses": 1,
  "uppercase": false,
  "show_only_reference": false,
  "show_two_versions": false,
  "show_second_version": false,
  "show_third_version": false,
  "book_panel_type": "grid",
  "book_panel_order": "automatic",
  "book_panel_order_available_items": [
    "automatic",
    "standard",
    "ru",
    "tyv"
  ],
  "multiple_verses_separator_type": "double_line_break",
  "multiple_versions_separator_type": "double_line_break",
  "versification": true,
  "theme": {
    "public": 123,
    "screen_n": null
  }
}
```
</details>

## Font Settings
| Name | Type  | Description |
| ---- | :---: | ------------|
| `font_name` | _String (optional)_ | Font name `Default: null` |
| `bold` | _Boolean (optional)_ | Bold `Default: null` |
| `italic` | _Boolean (optional)_ | Italic `Default: null` |
| `color` | _String (optional)_ | Color in hexadecimal `Default: null` |

## Stage View
| Name | Type  | Description |
| ---- | :---: | ------------|
| `enabled` | _Boolean_ | Stage view enabled |
| `preview_mode` | _String_ | Lyrics display mode. Available options:<br/>`CURRENT_SLIDE`<br>`FIRST_LINE_OF_THE_NEXT_SLIDE_WITH_SEPARATOR`<br>`FIRST_LINE_OF_THE_NEXT_SLIDE_WITHOUT_SEPARATOR`<br>`NEXT_SLIDE`<br>`CURRENT_AND_NEXT_SLIDE`<br>`ALL_SLIDES` |
| `uppercase` | _Boolean_ | Show in capital letters |
| `remove_line_break` | _Boolean_ | remove line break |
| `show_comment` | _Boolean_ | Show comments |
| `show_advanced_editor` | _Boolean_ | Show advanced editor |
| `show_communication_panel` | _Boolean_ | Show communication panel content |
| `show_next_image` | _Boolean_ | Display next image `v2.21.0+` |
| `custom_theme` | _String_ | Custom Theme ID used in presentations |
| `apply_custom_theme_to_bible` | _Boolean_ | Use custom theme in verses |
| `apply_custom_theme_to_text` | _Boolean_ | Use custom theme in texts |
| `apply_custom_theme_to_quick_presentation` | _Boolean_ | Use the custom theme in the **Quick Presentation** option `v2.21.0+` |
<details>
  <summary>See example</summary>

```json
{
  "enabled": false,
  "preview_mode": "FIRST_LINE_OF_THE_NEXT_SLIDE_WITH_SEPARATOR",
  "uppercase": false,
  "uppercase_mode": "text_and_comment",
  "remove_line_break": false,
  "show_comment": true,
  "show_advanced_editor": false,
  "show_communication_panel": true,
  "show_next_image": false,
  "custom_theme": null,
  "apply_custom_theme_to_bible": true,
  "apply_custom_theme_to_text": true,
  "apply_custom_theme_to_quick_presentation": false
}
```
</details>

## Slide Additional Info
| Name | Type  | Description |
| ---- | :---: | ------------|
| `info_1` | _Object_ |  |
| `info_1.show_page_count` | _Boolean_ | Show slide count |
| `info_1.show_slide_description` | _Boolean_ | Show slide description (chorus, i.e.) |
| `info_1.horizontal_align` | _String_ | Horizontal alignment of information on the slide. left, center, right |
| `info_1.vertical_align` | _String_ | Vertical alignment of information on the slide. top, bottom |
| `info_2` | _Object_ |  |
| `info_2.show` | _Boolean_ |  |
| `info_2.layout_row_1` | _String_ | Layout of the first line information **type=song** [Slide Additional Info Layout](#slide-additional-info-layout) |
| `info_2.layout_row_2` | _String (optional)_ | Layout of the second line information **type=song** [Slide Additional Info Layout](#slide-additional-info-layout) |
| `info_2.layout_text_row_1` | _String_ | Layout of the first line information **type=text** [Slide Additional Info Layout](#slide-additional-info-layout) `v2.24.0+` |
| `info_2.layout_text_row_2` | _String (optional)_ | Layout of the first line information **type=text** [Slide Additional Info Layout](#slide-additional-info-layout) `v2.24.0+` |
| `info_2.horizontal_align` | _String_ | Horizontal alignment of information on the slide. left, center, right |
| `info_2.vertical_align` | _String_ | Vertical alignment of information on the slide. top, bottom |
| `font` | _Object_ |  |
| `font.name` | _String_ | Font name. If it is **null**, it uses the theme's default font. |
| `font.bold` | _Boolean_ | Bold. If it is **null**, use the default theme setting |
| `font.italic` | _Boolean_ | Italid. If it is **null**, use the default theme setting |
| `font.color` | _String_ | Font color in hexadecimal. If it is **null**, use the theme's default font color |
| `height` | _Number_ | Height in percent of slide height `4 ~ 15` |
| `paint_theme_effect` | _String_ | Render text with theme outline, glow, and shadow effects, if available |
<details>
  <summary>See example</summary>

```json
{
  "info_1": {
    "show_page_count": false,
    "show_slide_description": false,
    "horizontal_align": "right",
    "vertical_align": "bottom"
  },
  "info_2": {
    "show": false,
    "layout_row_1": "<title>< (%author_or_artist%)>",
    "layout_text_row_1": "",
    "horizontal_align": "right",
    "vertical_align": "bottom"
  },
  "font": {
    "name": null,
    "bold": null,
    "italic": null,
    "color": null
  },
  "height": 7,
  "paint_theme_effect": true
}
```
</details>

## Rectangle
| Name | Type  | Description |
| ---- | :---: | ------------|
| `x` | _Number_ |  |
| `y` | _Number_ |  |
| `width` | _Number_ |  |
| `height` | _Number_ |  |

## Custom Message
| Name | Type  | Description |
| ---- | :---: | ------------|
| `id` | _String_ | Item ID |
| `name` | _String_ | Item name |
| `message_model` | _String_ | message without filling |
| `message_example` | _String_ | Example message with the name of the parameters filled in |
| `variables` | _Array&lt;[CustomMessageParam](#custom-message-param)&gt;_ | message parameters |
<details>
  <summary>See example</summary>

```json
{
  "id": "123",
  "name": "Chamar pessoa",
  "message_model": "   , favor comparecer  .",
  "message_example": "função nome, favor comparecer local.",
  "variables": [
    {
      "position": 0,
      "name": "função",
      "only_number": false,
      "uppercase": false,
      "suggestions": [
        "Diácono",
        "Presbítero",
        "Pastor",
        "Professor",
        "Ministro"
      ]
    },
    {
      "position": 2,
      "name": "nome",
      "only_number": false,
      "uppercase": false
    },
    {
      "position": 22,
      "name": "local",
      "only_number": false,
      "uppercase": false,
      "suggestions": [
        "ao estacionamento",
        "ao hall de entrada",
        "à mesa de som",
        "ao berçário"
      ]
    }
  ]
}
```
</details>

## Custom Message Param
| Name | Type  | Description |
| ---- | :---: | ------------|
| `position` | _Number_ | Parameter position in the message (in number of characters) |
| `name` | _String_ | Item name |
| `only_number` | _Boolean_ | Parameter accepts only numbers |
| `uppercase` | _Boolean_ | Parameter always displayed in uppercase |
| `suggestions` | _Array&lt;String&gt; (optional)_ | List with default values for the parameter |
<details>
  <summary>See example</summary>

```json
{
  "position": 0,
  "name": "",
  "only_number": false,
  "uppercase": false
}
```
</details>

## Quiz Question
| Name | Type  | Description |
| ---- | :---: | ------------|
| `name` | _String_ | Item name `v2.24.0+` |
| `title` | _String_ | Question |
| `alternatives` | _Array&lt;String&gt;_ | Alternatives |
| `correct_alternative_number` | _Number (optional)_ | Number of the correct alternative. Starts at 1 `Default: 1` |
| `source` | _String (optional)_ | Answer font |
<details>
  <summary>See example</summary>

```json
{
  "name": "",
  "title": "...",
  "alternatives": [
    "Item 1",
    "Item 2",
    "Item 3"
  ],
  "correct_alternative_number": 2,
  "source": ""
}
```
</details>

## Quiz Settings
| Name | Type  | Description |
| ---- | :---: | ------------|
| `correct_answer_color_font` | _String (optional)_ | Font color for the correct answer |
| `correct_answer_color_background` | _String (optional)_ | Background color for the correct answer |
| `incorrect_answer_color_font` | _String (optional)_ | Font color for the incorrect answer |
| `incorrect_answer_color_background` | _String (optional)_ | Background color for the incorrect answer |
| `question_and_alternatives_different_slides` | _Boolean (optional)_ | Display the question and alternatives on separate slides `Default: false` |
| `display_alternatives_one_by_one` | _Boolean (optional)_ | Display the alternatives one by one `Default: true` |
| `alternative_char_type` | _String (optional)_ | Type of character to list the alternatives `number (1, 2, 3...)`  `alpha (A, B, C...)` `Default: 'alpha'` |
| `alternative_separator_char` | _String (optional)_ | Separator character. Allowed values:  ` `  `.`  `)`  `-`  `:` `Default: '.'` |
<details>
  <summary>See example</summary>

```json
{
  "correct_answer_color_font": "00796B",
  "correct_answer_color_background": "CCFFCC",
  "incorrect_answer_color_font": "721C24",
  "incorrect_answer_color_background": "F7D7DB",
  "question_and_alternatives_different_slides": false,
  "display_alternatives_one_by_one": true,
  "alternative_separator_char": ".",
  "alternative_char_type": "alpha"
}
```
</details>

## Quick Presentation Slide
| Name | Type  | Description |
| ---- | :---: | ------------|
| `text` | _String_ | Slide text |
| `theme` | _[ThemeFilter](#theme-filter) (optional)_ | Filter selected theme for display |
| `custom_theme` | _[Theme](#theme) (optional)_ | Custom theme used to display the text |
| `translations` | _[Translations](#translations) (optional)_ |  |
| `duration` | _Number (optional)_ | Slide duration for use in automatic presentations |
<details>
  <summary>See example</summary>

```json
{
  "text": "text",
  "duration": 3,
  "translations": {
    "key1": "value1",
    "key2": "value2"
  },
  "theme": {
    "name": "...",
    "edit": {
      "font": {
        "name": "Arial",
        "size": 10,
        "bold": true,
        "color": "FFFFFF"
      },
      "background": {
        "type": "color",
        "id": "000000"
      }
    }
  }
}
```
</details>

## Theme Filter
| Name | Type  | Description |
| ---- | :---: | ------------|
| `id` | _String (optional)_ | Theme or background ID |
| `name` | _String (optional)_ | Theme name or background |
| `edit` | _[Theme](#theme) (optional)_ | Settings to modify the selected Theme for display |
<details>
  <summary>See example</summary>

```json
{
  "name": "...",
  "edit": {
    "font": {
      "name": "Arial",
      "size": 10,
      "bold": true,
      "color": "FFFFFF"
    },
    "background": {
      "type": "color",
      "id": "000000"
    }
  }
}
```
</details>

## Translations
Key/value pair

| Name | Type  | Description |
| ---- | :---: | ------------|
| `???` | _String_ | Translated value of the item, where the parameter name `???` is the name of the translation |
| `???` | _String_ |  |
| `...` | _String_ |  |
<details>
  <summary>See example</summary>

```json
{
  "key1": "value1",
  "key2": "value2"
}
```
</details>

## Wallpaper Settings
| Name | Type  | Description |
| ---- | :---: | ------------|
| `enabled` | _Boolean_ | Show wallpaper |
| `fill_color` | _String_ | Color in hexadecimal defined in the **fill** option. |
| `clock` | _[ClockSettings](#clock-settings)_ | Clock settings |
<details>
  <summary>See example</summary>

```json
{
  "enabled": null,
  "fill_color": null,
  "clock": {
    "enabled": false,
    "font_name": "",
    "bold": false,
    "italic": false,
    "color": "FF0000",
    "background": "000000",
    "height": 12,
    "position": "top_right",
    "corner": 0
  }
}
```
</details>

## Clock Settings
| Name | Type  | Description |
| ---- | :---: | ------------|
| `enabled` | _Boolean_ |  |
| `font_name` | _String_ | Font name |
| `bold` | _Boolean_ | Bold |
| `italic` | _Boolean_ | Italic |
| `color` | _String_ | Color in hexadecimal format (RGBA) |
| `background` | _String_ | Color in hexadecimal format (RGBA) |
| `height` | _Number_ | Value in percentage based on the line height.<br>Accepted values: `6` `7` `8` `9` `10` `12` `14` `15` `16` `18` `20` `25` `30` `35` `40` |
| `position` | _Boolean_ | Accepted values: `top_left` `top_center` `top_right` `middle_left` `middle_center` `middle_right` `bottom_left` `bottom_center` `bottom_right` |
| `corner` | _Number_ | `0 ~ 100` |
<details>
  <summary>See example</summary>

```json
{
  "enabled": false,
  "font_name": "",
  "bold": false,
  "italic": false,
  "color": "FF0000",
  "background": "000000",
  "height": 12,
  "position": "top_right",
  "corner": 0
}
```
</details>

## Bible Book List
| Name | Type  | Description |
| ---- | :---: | ------------|
| `id` | _String_ | Item ID |
| `name` | _String_ | Name in English |
| `language` | _String_ | ISO 639 two-letter language code `v2.24.0+` |
| `alt_name` | _String_ | Name in the language defined in `language`. Can be null. `v2.24.0+` |
<details>
  <summary>See example</summary>

```json
{
  "id": "en",
  "name": "English",
  "language": "en",
  "alt_name": "English"
}
```
</details>

## Bible Book Info
| Name | Type  | Description |
| ---- | :---: | ------------|
| `id` | _String_ | Book ID `01 ~ 66` |
| `name` | _String_ | Book name |
| `abbrev` | _String_ | Book abbreviation |
| `usfx_code` | _String_ |  `v2.24.0+` |
<details>
  <summary>See example</summary>

```json
{
  "id": "01",
  "name": "Genesis",
  "abbrev": "Gn"
}
```
</details>

## Verse Reference Group
| Name | Type  | Description |
| ---- | :---: | ------------|
| `reference` | _String_ | References. Example: **John 3:16** or **Rm 12:2** or **Gn 1:1-3 Sl 23.1** |
| `ids` | _Array&lt;String&gt;_ | Example:  ['19023001', '43003016', '45012002'] |
| `verses` | _Array&lt;[VerseReference](#verse-reference)&gt;_ | Detailed list of references |
<details>
  <summary>See example</summary>

```json
{
  "reference": "Ps 23.1-2",
  "ids": [
    "19023001",
    "19023002"
  ],
  "verses": [
    {
      "id": "19023001",
      "book": 19,
      "chapter": 23,
      "verse": 1,
      "reference": "Psalms 23.1"
    },
    {
      "id": "19023002",
      "book": 19,
      "chapter": 23,
      "verse": 2,
      "reference": "Psalms 23.2"
    }
  ]
}
```
</details>

## Verse Reference
| Name | Type  | Description |
| ---- | :---: | ------------|
| `id` | _String_ | Item ID |
| `book` | _Number_ | Book ID `1 ~ 66` |
| `chapter` | _Number_ | Chapter |
| `verse` | _Number_ | Verse |
| `reference` | _String_ |  |
<details>
  <summary>See example</summary>

```json
{
  "id": "19023001",
  "book": 19,
  "chapter": 23,
  "verse": 1,
  "reference": "Psalms 23.1"
}
```
</details>

## Translation Custom Settings
Custom translation settings

| Name | Type  | Description |
| ---- | :---: | ------------|
| `translation_1` | _[TranslationCustomSettingsItem](#translation-custom-settings-item)_ |  |
| `translation_2` | _[TranslationCustomSettingsItem](#translation-custom-settings-item)_ |  |
| `translation_3` | _[TranslationCustomSettingsItem](#translation-custom-settings-item)_ |  |
| `translation_4` | _[TranslationCustomSettingsItem](#translation-custom-settings-item)_ |  |
| `merge` | _Boolean_ |  |
| `uppercase` | _Boolean_ |  |
| `blank_line_height` | _Number_ | `0 ~ 100` |
<details>
  <summary>See example</summary>

```json
{
  "translation_1": {
    "name": "default",
    "style": "",
    "prefix": "",
    "suffix": ""
  },
  "translation_2": null,
  "translation_3": null,
  "translation_4": null,
  "merge": false,
  "uppercase": false,
  "blank_line_height": 0
}
```
</details>

## Translation Custom Settings Item
Custom translation settings (item)

| Name | Type  | Description |
| ---- | :---: | ------------|
| `name` | _String_ | translation name. Use 'default' to use the original text. |
| `style` | _String_ | Custom text formatting. [Styled Text](https://github.com/holyrics/Scripts/blob/main/i18n/en/StyledText.md) |
| `prefix` | _String_ | Text added at the beginning of each line |
| `suffix` | _String_ | Text added at the end of each line |
<details>
  <summary>See example</summary>

```json
{
  "name": "default",
  "style": "",
  "prefix": "",
  "suffix": ""
}
```
</details>

## Styled Model
| Name | Type  | Description |
| ---- | :---: | ------------|
| `key` | _String_ |  |
| `properties` | _Object_ | Key/value pair |
<details>
  <summary>See example</summary>

```json
{
  "key": "title",
  "properties": {
    "b": "true",
    "size": "120"
  }
}
```
</details>

## Initial Slide Settings
| Name | Type  | Description |
| ---- | :---: | ------------|
| `display_mode` | _String_ | Accepted values: `title_author` `title_author_or_artist` `title` `title_artist` `blank` `remove` |
| `uppercase` | _Boolean_ |  |
| `automatic_line_break` | _Boolean_ |  |
| `underlined_title` | _Boolean_ |  |
| `title_font_relative_size` | _Number_ | `40 ~ 160` |
| `author_or_artist_font_relative_size` | _Number_ | `40 ~ 160` |
| `keep_ratio` | _Boolean_ |  |
| `remove_final_slide` | _Boolean_ |  |
<details>
  <summary>See example</summary>

```json
{
  "display_mode": "title_author_or_artist",
  "uppercase": false,
  "automatic_line_break": true,
  "underlined_title": true,
  "title_font_relative_size": 130,
  "author_or_artist_font_relative_size": 110,
  "keep_ratio": true,
  "remove_final_slide": false
}
```
</details>

## Copyright Settings
| Name | Type  | Description |
| ---- | :---: | ------------|
| `display_mode` | _String_ | Accepted values: `disabled` `first_slide` `all_slides` `last_slide` `display_for_x_seconds` |
| `seconds` | _String_ | Available if `display_mode=display_for_x_seconds`<br>Accepted values: `5` `10` `15` `20` `30` `60` `120` |
| `layout` | _String_ | Accepted values: `t,a` `t;a` `t,a;c` `t;a;c` |
| `font.name` | _String_ | Font name |
| `font.bold` | _String_ | Bold |
| `font.italic` | _String_ | Italic |
| `font.color` | _String_ | Color in hexadecimal format |
| `line_height` | _Number_ | `2.0 ~ 10.0` |
| `align` | _String_ | Accepted values: `left` `center` `right` |
| `opaticy` | _Number_ | `30 ~ 100` |
| `position` | _String_ | Accepted values: `top_left` `top_center` `top_right` `bottom_left` `bottom_center` `bottom_right` |
<details>
  <summary>See example</summary>

```json
{
  "display_mode": "all_slides",
  "layout": "t;a;c",
  "font": {
    "name": "Arial",
    "bold": true,
    "italic": true,
    "color": "FFFF00"
  },
  "line_height": 3.0,
  "align": "left",
  "opacity": 70,
  "position": "top_left"
}
```
</details>

## Image Presentation Settings
| Name | Type  | Description |
| ---- | :---: | ------------|
| `adjust_type` | _String_ | `adjust` `extend` |
| `blur` | _Object_ | Used only if: `adjust_type=adjust` |
| `blur.enabled` | _Boolean_ |  |
| `blur.radius` | _Number_ | `1 ~ 20` |
| `blur.times` | _Number_ | `1 ~ 10` |
| `blur.opacity` | _Number_ | `10 ~ 100` |
<details>
  <summary>See example</summary>

```json
{
  "adjust_type": "adjust",
  "blur": {
    "enabled": true,
    "radius": 8,
    "times": 5,
    "opacity": 70
  }
}
```
</details>

## Non-Latin Alphabet Support Settings
| Name | Type  | Description |
| ---- | :---: | ------------|
| `enabled` | _Boolean_ |  |
| `font_or_script` | _String_ | `system` `lucida_sans` `arial_unicode_ms` `nirmala_ui` `arabic` `armenian` `bengali` `bopomofo` `cyrillic` `devanagari` `georgian` `gujarati` `gurmukhi` `han` `hebrew` `hiragana` `kannada` `katakana` `malayalam` `meetei_mayek` `ol_chiki` `oriya` `sinhala` `tamil` `telugu` `thai` |
<details>
  <summary>See example</summary>

```json
{
  "enabled": false,
  "font_or_script": "system"
}
```
</details>

## Global Settings
| Name | Type  | Description |
| ---- | :---: | ------------|
| `fade_in_out_enabled` | _Boolean_ |  |
| `fade_in_out_duration` | _Number_ | `200 ~ 1200` |
| `show_history_main_window` | _Boolean_ |  |
| `show_favorite_bar_main_window` | _Boolean_ |  |
| `show_favorite_bar_bible_window` | _Boolean_ |  |
| `show_module_bar_main_window` | _Boolean_ |  |
| `show_module_bar_bible_window` | _Boolean_ |  |
| `show_automatic_presentation_tab_main_window` | _Boolean_ |  |
| `text_editor_font_name` | _String_ |  |
| `show_comment_main_window` | _Boolean_ |  |
| `show_comment_presentation_footer` | _Boolean_ |  |
| `show_comment_app` | _Boolean_ |  |
| `initial_slide` | _[InitialSlideSettings](#initial-slide-settings)_ |  |
| `copyright` | _Object_ | Key/value pair<br>key: `public` `screen_2` `screen_3` `screen_?` `stream_image`<br>valor: [CopyrightSettings](#copyright-settings) |
| `image_presentation` | _[ImagePresentationSettings](#image-presentation-settings)_ |  |
| `black_screen_color` | _String_ | Color in hexadecimal format |
| `swap_f5` | _Boolean_ |  |
| `stage_view_modifier_enabled` | _Boolean_ |  |
| `disable_modifier_automatically` | _Boolean_ |  |
| `automatic_presentation_theme_chooser` | _Boolean_ |  |
| `automatic_presentation_execution_delay` | _String_ | Accepted values: `0` `1000` `1500` `2000` `2500` `3000` |
| `skip_slide_transition_if_equals` | _Boolean_ |  |
| `non_latin_alphabet_support` | _[NonLatinAlphabetSupportSettings](#non-latin-alphabet-support-settings)_ |  |
| `public_screen_expand_width` | _Number_ | `0 ~ 3840` |
| `public_screen_rounded_border` | _Boolean_ |  |
| `public_screen_rounded_border_size` | _Number_ | `0 ~ 540` |
| `display_custom_formatting_enabled` | _Boolean_ |  |
| `display_custom_background_enabled` | _Boolean_ |  |
| `display_advanced_editor_enabled` | _Boolean_ |  |
| `advanced_editor_block_line_break` | _Boolean_ |  |
| `slide_description_repeat_description_for_sequence` | _Boolean_ |  |
| `standardize_automatic_line_break` | _Boolean_ |  |
| `allow_main_window_and_bible_window_simultaneously` | _Boolean_ |  |
| `preferential_arrangement_collection` | _String_ |  |
<details>
  <summary>See example</summary>

```json
{
  "fade_in_out_enabled": true,
  "fade_in_out_duration": 500,
  "show_history_main_window": true,
  "show_favorite_bar_main_window": true,
  "show_favorite_bar_bible_window": false,
  "show_module_bar_main_window": false,
  "show_module_bar_bible_window": false,
  "show_automatic_presentation_tab_main_window": false,
  "text_editor_font_name": "Lucida Sans Unicode",
  "show_comment_main_window": false,
  "show_comment_presentation_footer": true,
  "show_comment_app": true,
  "initial_slide": {
    "display_mode": "title_author_or_artist",
    "uppercase": false,
    "automatic_line_break": true,
    "underlined_title": true,
    "title_font_relative_size": 130,
    "author_or_artist_font_relative_size": 110,
    "keep_ratio": true,
    "remove_final_slide": false
  },
  "copyright": {
    "display_mode": "all_slides",
    "layout": "t;a;c",
    "font": {
      "name": "Arial",
      "bold": true,
      "italic": true,
      "color": "FFFF00"
    },
    "line_height": 3.0,
    "align": "left",
    "opacity": 70,
    "position": "top_left"
  },
  "image_presentation": {
    "adjust_type": "adjust",
    "blur": {
      "enabled": true,
      "radius": 8,
      "times": 5,
      "opacity": 70
    }
  },
  "black_screen_color": "1E1E1E",
  "swap_f5": false,
  "stage_view_modifier_enabled": true,
  "disable_modifier_automatically": true,
  "automatic_presentation_theme_chooser": true,
  "automatic_presentation_execution_delay": 0,
  "skip_slide_transition_if_equals": false,
  "non_latin_alphabet_support": {
    "enabled": false,
    "font_or_script": "system"
  },
  "public_screen_expand_width": 0,
  "public_screen_rounded_border": false,
  "public_screen_rounded_border_size": 100,
  "display_custom_formatting_enabled": true,
  "display_custom_background_enabled": true,
  "display_advanced_editor_enabled": true,
  "advanced_editor_block_line_break": true,
  "slide_description_repeat_description_for_sequence": true,
  "standardize_automatic_line_break": false,
  "allow_main_window_and_bible_window_simultaneously": false,
  "preferential_arrangement_collection": ""
}
```
</details>

## AddItem
| Name | Type  | Description |
| ---- | :---: | ------------|
| `type` | _String_ | Type of item. It can be: `title`  `song`  `verse`  `text`  `audio`  `video`  `image`  `file`  `announcement`  `automatic_presentation`  `countdown`  `countdown_cp`  `cp_text`  `plain_text`  `uri`  `global_action`  `api`  `script` |

## AddItemTitle
| Name | Type  | Description |
| ---- | :---: | ------------|
| `type` | _String_ | title |
| `name` | _String_ | Item name |
| `background_color` | _String (optional)_ | Background color in hexadecimal, example: 000080 |
| `collapsed` | _Boolean (optional)_ |  `Default: false` |
<details>
  <summary>See example</summary>

```json
{
  "type": "title",
  "name": "Example",
  "background_color": "0000FF",
  "collapsed": false
}
```
</details>

## AddItemSong
| Name | Type  | Description |
| ---- | :---: | ------------|
| `type` | _String_ | song |
| `id` | _String_ | Item ID |
<details>
  <summary>See example</summary>

```json
{
  "type": "song",
  "id": "123"
}
```
</details>

## AddItemVerse
**id**, **ids** or **references**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `type` | _String_ | verse |
| `id` | _String (optional)_ | To display a verse. Item ID in BBCCCVVV format.<br/>Example: '19023001' (book 19, chapter 023, verse 001) |
| `ids` | _Array&lt;String&gt; (optional)_ | To display a list of verses. List with the ID of each verse.<br/>Example: ['19023001', '43003016', '45012002'] |
| `references` | _String (optional)_ | References. Example: **John 3:16** or **Rm 12:2** or **Gn 1:1-3 Sl 23.1** |
| `version` | _String (optional)_ | Name or abbreviation of the translation used `v2.21.0+` |
<details>
  <summary>See example</summary>

```json
{
  "type": "verse",
  "references": "Ps 23.1-6 Rm 12.2",
  "version": "en_kjv"
}
```
</details>

## AddItemText
| Name | Type  | Description |
| ---- | :---: | ------------|
| `type` | _String_ | text |
| `id` | _String_ | Item ID |
<details>
  <summary>See example</summary>

```json
{
  "type": "text",
  "id": "xyz"
}
```
</details>

## AddItemAudio
| Name | Type  | Description |
| ---- | :---: | ------------|
| `type` | _String_ | audio |
| `name` | _String_ | File name |
| `settings` | _[PlayMediaSettings](#play-media-settings) (optional)_ | Settings for media execution `v2.21.0+` |
<details>
  <summary>See example</summary>

```json
{
  "id": "",
  "type": "audio",
  "name": "file.mp3",
  "isDir": false
}
```
</details>

## AddItemVideo
| Name | Type  | Description |
| ---- | :---: | ------------|
| `type` | _String_ | video |
| `name` | _String_ | File name |
| `settings` | _[PlayMediaSettings](#play-media-settings) (optional)_ | Settings for media execution `v2.21.0+` |
<details>
  <summary>See example</summary>

```json
{
  "id": "",
  "type": "video",
  "name": "file.mp4",
  "isDir": false
}
```
</details>

## AddItemImage
| Name | Type  | Description |
| ---- | :---: | ------------|
| `type` | _String_ | image |
| `name` | _String_ | File name |
<details>
  <summary>See example</summary>

```json
{
  "type": "image",
  "name": "file.ext"
}
```
</details>

## AddItemFile
| Name | Type  | Description |
| ---- | :---: | ------------|
| `type` | _String_ | file |
| `name` | _String_ | File name |
<details>
  <summary>See example</summary>

```json
{
  "type": "file",
  "name": "file.ext"
}
```
</details>

## AddItemAutomaticPresentation
| Name | Type  | Description |
| ---- | :---: | ------------|
| `type` | _String_ | automatic_presentation |
| `name` | _String_ | File name |
<details>
  <summary>See example</summary>

```json
{
  "type": "automatic_presentation",
  "name": "filename.ap"
}
```
</details>

## AddItemAnnouncement
**id**, **ids**, **name** or **names**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `type` | _String_ | announcement |
| `id` | _String (optional)_ | Announcement ID. Can be **all** to display all |
| `ids` | _Array&lt;String&gt; (optional)_ | List with the ID of each announcement |
| `name` | _String (optional)_ | Announcement name |
| `names` | _Array&lt;String&gt; (optional)_ | List with the name of each announcement |
| `automatic` | _[Automatic](#automatic) (optional)_ | If informed, the presentation of the items will be automatic |
<details>
  <summary>See example</summary>

```json
{
  "type": "announcement",
  "names": [
    "Anúncio 1",
    "Anúncio 2",
    "Anúncio 3"
  ],
  "automatic": {
    "seconds": 10,
    "repeat": true
  }
}
```
</details>

## AddItemCountdown
| Name | Type  | Description |
| ---- | :---: | ------------|
| `type` | _String_ | countdown |
| `time` | _String_ | HH:MM or MM:SS |
| `exact_time` | _Boolean (optional)_ | If **true**, `time` should be HH:MM (exact hour and minute). If **false**, `time` should be MM:SS (amount of minutes and seconds) `Default: false` |
| `text_before` | _String (optional)_ | Text displayed at the top of the countdown |
| `text_after` | _String (optional)_ | Text displayed at the bottom of the countdown |
| `zero_fill` | _Boolean (optional)_ | Fill in the 'minute' field with zero on the left `Default: false` |
| `hide_zero_minute` | _Boolean (optional)_ | Hide the display of minutes when it is zero `Default: false` `v2.25.0+` |
| `countdown_relative_size` | _Number (optional)_ | Relative size of the countdown `Default: 250` |
| `theme` | _[ThemeFilter](#theme-filter) (optional)_ | Filter selected theme for display `v2.21.0+` |
| `countdown_style` | _[FontSettings](#font-settings) (optional)_ | Custom font for countdown `v2.21.0+` |
<details>
  <summary>See example</summary>

```json
{
  "type": "countdown",
  "time": "05:00",
  "exact_time": false,
  "text_before": "",
  "text_after": "",
  "zero_fill": false,
  "countdown_relative_size": 250,
  "theme": null,
  "countdown_style": {
    "font_name": null,
    "bold": null,
    "italic": true,
    "color": null
  },
  "hide_zero_minute": false
}
```
</details>

## AddItemCountdownCommunicationPanel
| Name | Type  | Description |
| ---- | :---: | ------------|
| `type` | _String_ | countdown_cp |
| `minutes` | _Number_ | Number of minutes |
| `seconds` | _Number_ | Number of seconds |
| `stop_at_zero` | _Boolean (optional)_ | Stop the countdown when it reaches zero `Default: false` |
| `description` | _String_ | Item description |
<details>
  <summary>See example</summary>

```json
{
  "type": "countdown_cp",
  "minutes": 5,
  "seconds": 0,
  "stop_at_zero": false,
  "description": ""
}
```
</details>

## AddItemPlainText
| Name | Type  | Description |
| ---- | :---: | ------------|
| `type` | _String_ | plain_text |
| `name` | _String_ | Item name |
| `text` | _String_ | Text |
<details>
  <summary>See example</summary>

```json
{
  "type": "plain_text",
  "name": "",
  "text": "Example"
}
```
</details>

## AddItemTextCommunicationPanel
| Name | Type  | Description |
| ---- | :---: | ------------|
| `type` | _String_ | cp_text |
| `name` | _String_ | Item name |
| `text` | _String_ | Text |
| `display_ahead` | _Boolean (optional)_ | Change the *'display in front of all'* option |
<details>
  <summary>See example</summary>

```json
{
  "type": "cp_text",
  "name": "",
  "text": "Example",
  "display_ahead": false
}
```
</details>

## AddItemAddItemScript
| Name | Type  | Description |
| ---- | :---: | ------------|
| `type` | _String_ | script |
| `id` | _String_ | Item ID |
| `description` | _String_ | Item description |
| `inputs` | _Object (optional)_ | Default value for [Function Input](https://github.com/holyrics/Scripts/blob/main/i18n/en/FunctionInput.md) |
<details>
  <summary>See example</summary>

```json
{
  "type": "script",
  "id": "xyz",
  "description": "",
  "inputs": {
    "message": "Example",
    "duration": 30
  }
}
```
</details>

## AddItemAddItemAPI
| Name | Type  | Description |
| ---- | :---: | ------------|
| `type` | _String_ | api |
| `id` | _String_ | Item ID |
| `description` | _String_ | Item description |
| `inputs` | _Object (optional)_ | Default value for [Function Input](https://github.com/holyrics/Scripts/blob/main/i18n/en/FunctionInput.md) |
<details>
  <summary>See example</summary>

```json
{
  "type": "api",
  "id": "xyz",
  "description": "",
  "inputs": {
    "message": "Example",
    "duration": 30
  }
}
```
</details>

## AddItemURI
| Name | Type  | Description |
| ---- | :---: | ------------|
| `type` | _String_ | uri |
| `title` | _String_ | Item title |
| `uri_type` | _String_ | Can be: `spotify` `youtube` `deezer` |
| `value` | _String_ | URI |
<details>
  <summary>See example</summary>

```json
{
  "type": "uri",
  "title": "Holyrics",
  "uri_type": "youtube",
  "value": "https://youtube.com/watch?v=umYQpAxL4dI"
}
```
</details>

## AddItemGlobalAction
| Name | Type  | Description |
| ---- | :---: | ------------|
| `type` | _String_ | global_action |
| `action` | _String_ | Can be: `slide_exit` `vlc_stop` `vlc_stop_fade_out` |

## SongInfo
| Name | Type  | Description |
| ---- | :---: | ------------|
| `id` | _String_ | Song ID |
| `title` | _String_ | Song title |
| `artist` | _String_ | Music artist |
| `author` | _String_ | Music author |
| `note` | _String_ | Music annotation |
| `copyright` | _String_ | Music copyright |
| `key` | _String_ | Tone of music.<br>Can be: `C` `C#` `Db` `D` `D#` `Eb` `E` `F` `F#` `Gb` `G` `G#` `Ab` `A` `A#` `Bb` `B` `Cm` `C#m` `Dbm` `Dm` `D#m` `Ebm` `Em` `Fm` `F#m` `Gbm` `Gm` `G#m` `Abm` `Am` `A#m` `Bbm` `Bm` |
| `bpm` | _Number_ | BPM of the song |
| `time_sig` | _String_ | Music time.<br>Can be: `2/2` `2/4` `3/4` `4/4` `5/4` `6/4` `3/8` `6/8` `7/8` `9/8` `12/8` |
<details>
  <summary>See example</summary>

```json
{
  "id": "0",
  "title": "",
  "artist": "",
  "author": "",
  "note": "",
  "copyright": "",
  "key": "",
  "bpm": 0.0,
  "time_sig": ""
}
```
</details>

## TextInfo
| Name | Type  | Description |
| ---- | :---: | ------------|
| `id` | _String_ | Text ID |
| `title` | _String_ | Text title |
<details>
  <summary>See example</summary>

```json
{
  "id": "",
  "title": ""
}
```
</details>

## VerseInfo
<details>
  <summary>See example</summary>

```json
{}
```
</details>

## AudioInfo
| Name | Type  | Description |
| ---- | :---: | ------------|
| `file_name` | _String_ |  |
| `file_fullname` | _String_ |  |
| `file_relative_path` | _String_ |  |
| `file_path` | _String_ |  |
| `is_dir` | _Boolean_ |  |
| `extension` | _String_ |  |
| `properties` | _Object_ |  |
<details>
  <summary>See example</summary>

```json
{
  "file_name": "file.mp3",
  "file_fullname": "folder\\file.mp3",
  "file_relative_path": "audio\\folder\\file.mp3",
  "file_path": "C:\\Holyrics\\Holyrics\\files\\media\\audio\\folder\\file.mp3",
  "is_dir": false,
  "extension": "mp3"
}
```
</details>

## VideoInfo
| Name | Type  | Description |
| ---- | :---: | ------------|
| `file_name` | _String_ |  |
| `file_fullname` | _String_ |  |
| `file_relative_path` | _String_ |  |
| `file_path` | _String_ |  |
| `is_dir` | _Boolean_ |  |
| `extension` | _String_ |  |
| `properties` | _Object_ |  |
<details>
  <summary>See example</summary>

```json
{
  "file_name": "file.mp4",
  "file_fullname": "folder\\file.mp4",
  "file_relative_path": "video\\folder\\file.mp4",
  "file_path": "C:\\Holyrics\\Holyrics\\files\\media\\video\\folder\\file.mp4",
  "is_dir": false,
  "extension": "mp4"
}
```
</details>

## ImageInfo
| Name | Type  | Description |
| ---- | :---: | ------------|
| `file_name` | _String_ |  |
| `file_fullname` | _String_ |  |
| `file_relative_path` | _String_ |  |
| `file_path` | _String_ |  |
| `is_dir` | _Boolean_ |  |
| `extension` | _String_ |  |
| `properties` | _Object_ |  |
<details>
  <summary>See example</summary>

```json
{
  "file_name": "file.jpg",
  "file_fullname": "folder\\file.jpg",
  "file_relative_path": "image\\folder\\file.jpg",
  "file_path": "C:\\Holyrics\\Holyrics\\files\\media\\image\\folder\\file.jpg",
  "is_dir": false,
  "extension": "jpg"
}
```
</details>

## FileInfo
| Name | Type  | Description |
| ---- | :---: | ------------|
| `file_name` | _String_ |  |
| `file_fullname` | _String_ |  |
| `file_relative_path` | _String_ |  |
| `file_path` | _String_ |  |
| `is_dir` | _Boolean_ |  |
| `extension` | _String_ |  |
| `properties` | _Object_ |  |
<details>
  <summary>See example</summary>

```json
{
  "file_name": "file.txt",
  "file_fullname": "folder\\file.txt",
  "file_relative_path": "file\\folder\\file.txt",
  "file_path": "C:\\Holyrics\\Holyrics\\files\\media\\file\\folder\\file.txt",
  "is_dir": false,
  "extension": "txt"
}
```
</details>

## AutomaticPresentationInfo
| Name | Type  | Description |
| ---- | :---: | ------------|
| `name` | _String_ |  |
<details>
  <summary>See example</summary>

```json
{
  "name": "name"
}
```
</details>

## AnnouncementInfo
| Name | Type  | Description |
| ---- | :---: | ------------|
| `id` | _Number_ |  |
| `name` | _String_ |  |
<details>
  <summary>See example</summary>

```json
{
  "id": 0,
  "name": "name"
}
```
</details>

## SongSlideInfo
| Name | Type  | Description |
| ---- | :---: | ------------|
| `id` | _String_ | Song ID |
| `slide_index` | _Number_ |  |
| `slide_total` | _Number_ |  |
| `slide_description` | _String_ |  |
| `slide_show_index` | _Number_ |  |
| `slide_show_total` | _Number_ |  |
| `title` | _String_ | Song title |
| `artist` | _String_ | Music artist |
| `author` | _String_ | Music author |
| `note` | _String_ | Music annotation |
| `copyright` | _String_ | Music copyright |
| `key` | _String_ | Tone of music.<br>Can be: `C` `C#` `Db` `D` `D#` `Eb` `E` `F` `F#` `Gb` `G` `G#` `Ab` `A` `A#` `Bb` `B` `Cm` `C#m` `Dbm` `Dm` `D#m` `Ebm` `Em` `Fm` `F#m` `Gbm` `Gm` `G#m` `Abm` `Am` `A#m` `Bbm` `Bm` |
| `bpm` | _Number_ | BPM of the song |
| `time_sig` | _String_ | Music time.<br>Can be: `2/2` `2/4` `3/4` `4/4` `5/4` `6/4` `3/8` `6/8` `7/8` `9/8` `12/8` |
| `text` | _String_ |  |
| `comment` | _String_ |  |
| `extra` | _String_ |  |
<details>
  <summary>See example</summary>

```json
{
  "id": "0",
  "slide_index": 2,
  "slide_total": 8,
  "slide_description": "",
  "slide_show_index": 2,
  "slide_show_total": 12,
  "title": "",
  "artist": "",
  "author": "",
  "note": "",
  "copyright": "",
  "key": "",
  "bpm": 0.0,
  "time_sig": "",
  "text": "",
  "comment": "",
  "extra": ""
}
```
</details>

## TextSlideInfo
| Name | Type  | Description |
| ---- | :---: | ------------|
| `id` | _String_ | Text ID |
| `title` | _String_ | Text title |
| `text` | _String_ |  |
| `comment` | _String_ |  |
| `folder` | _String_ |  |
| `slide_index` | _String_ |  |
| `slide_total` | _String_ |  |
<details>
  <summary>See example</summary>

```json
{
  "id": "",
  "title": "",
  "text": "",
  "comment": "",
  "folder": "",
  "slide_index": 2,
  "slide_total": 8
}
```
</details>

## PPTInfo
| Name | Type  | Description |
| ---- | :---: | ------------|
| `file_name` | _String_ |  |
| `file_fullname` | _String_ |  |
| `file_relative_path` | _String_ |  |
| `file_path` | _String_ |  |
| `is_dir` | _Boolean_ |  |
| `extension` | _String_ |  |
| `properties` | _Object_ |  |
<details>
  <summary>See example</summary>

```json
{
  "file_name": "file.txt",
  "file_fullname": "folder\\file.txt",
  "file_relative_path": "file\\folder\\file.txt",
  "file_path": "C:\\Holyrics\\Holyrics\\files\\media\\file\\folder\\file.txt",
  "is_dir": false,
  "extension": "txt"
}
```
</details>

## PPTSlideInfo
| Name | Type  | Description |
| ---- | :---: | ------------|
| `file_name` | _String_ |  |
| `slide_number` | _Number_ |  |
| `slide_total` | _Number_ |  |
<details>
  <summary>See example</summary>

```json
{
  "file_name": "name",
  "slide_number": 1,
  "slide_total": 10
}
```
</details>

## ThemeInfo
| Name | Type  | Description |
| ---- | :---: | ------------|
| `id` | _Number_ |  |
| `name` | _String_ |  |
| `from_user_list` | _Boolean_ |  |
| `tags` | _String_ |  |
| `bpm` | _String_ |  |
<details>
  <summary>See example</summary>

```json
{
  "id": 0,
  "name": "name",
  "from_user_list": true,
  "tags": "",
  "bpm": "0"
}
```
</details>

## BackgroundInfo
| Name | Type  | Description |
| ---- | :---: | ------------|
| `type` | _String_ | `THEME` `MY_VIDEO` `MY_IMAGE` `VIDEO` `IMAGE` |
| `id` | _Number_ |  |
| `name` | _String_ |  |
| `from_user_list` | _Boolean_ |  |
| `tags` | _String_ |  |
| `bpm` | _String_ |  |
| `color_map` | _Array&lt;Object&gt;_ |  |
| `color_map.*.hex` | _String_ | Color in hexadecimal format |
| `color_map.*.red` | _Number_ | Red  `0 ~ 255` |
| `color_map.*.green` | _Number_ | Green  `0 ~ 255` |
| `color_map.*.blue` | _Number_ | Blue  `0 ~ 255` |
<details>
  <summary>See example</summary>

```json
{
  "type": "MY_VIDEO",
  "id": 0,
  "name": "name",
  "from_user_list": true,
  "tags": "",
  "bpm": "0",
  "color_map": [
    {
      "hex": "000000",
      "red": 0,
      "green": 0,
      "blue": 0
    },
    {
      "hex": "000000",
      "red": 0,
      "green": 0,
      "blue": 0
    },
    {
      "hex": "000000",
      "red": 0,
      "green": 0,
      "blue": 0
    },
    {
      "hex": "000000",
      "red": 0,
      "green": 0,
      "blue": 0
    },
    {
      "hex": "000000",
      "red": 0,
      "green": 0,
      "blue": 0
    },
    {
      "hex": "000000",
      "red": 0,
      "green": 0,
      "blue": 0
    },
    {
      "hex": "000000",
      "red": 0,
      "green": 0,
      "blue": 0
    },
    {
      "hex": "000000",
      "red": 0,
      "green": 0,
      "blue": 0
    }
  ]
}
```
</details>

## TitleInfo
| Name | Type  | Description |
| ---- | :---: | ------------|
| `title` | _String_ |  |
| `subitem` | _Object_ |  |
| `subitem_index` | _Number_ |  |
| `playlist_index` | _Number_ |  |
<details>
  <summary>See example</summary>

```json
{
  "title": "",
  "subitem_index": -1,
  "playlist_index": -1
}
```
</details>

## WebcamInfo
| Name | Type  | Description |
| ---- | :---: | ------------|
| `name` | _String_ |  |
| `fps` | _Number_ |  |
| `width` | _Number_ |  |
| `height` | _Number_ |  |
| `mute` | _Boolean_ |  |
| `aspect_ratio` | _String_ |  |
<details>
  <summary>See example</summary>

```json
{
  "name": "name",
  "fps": 30.0,
  "width": 1280,
  "height": 720,
  "mute": false,
  "aspect_ratio": "1:1"
}
```
</details>

## CountdownInfo
| Name | Type  | Description |
| ---- | :---: | ------------|
| `time` | _String_ |  |
| `exact_time` | _Boolean_ |  |
| `text_before` | _String_ |  |
| `text_after` | _String_ |  |
| `zero_fill` | _Boolean_ |  |
| `countdown_relative_size` | _Number_ |  |
| `hide_zero_minute` | _Boolean_ |  |
<details>
  <summary>See example</summary>

```json
{
  "time": "",
  "exact_time": false,
  "text_before": "",
  "text_after": "",
  "zero_fill": false,
  "countdown_relative_size": 0,
  "hide_zero_minute": false
}
```
</details>

## AutomaticPresentationSlideInfo
| Name | Type  | Description |
| ---- | :---: | ------------|
| `id` | _String_ |  |
| `slide_index` | _Number_ |  |
<details>
  <summary>See example</summary>

```json
{
  "id": "filename.ap",
  "slide_index": 2
}
```
</details>

## PresentationModifierInfoInfo
| Name | Type  | Description |
| ---- | :---: | ------------|
| `type` | _String_ | `WALLPAPER` `BLANK_SCREEN` `BLACK_SCREEN` |
| `name` | _String_ | Item name |
| `shortcut` | _String_ | `F8` `F9` `F10` |
| `presentation_type` | _String_ | `SONG` `TEXT` `VERSE` `ANY_ITEM` |
| `state` | _Boolean_ |  |
<details>
  <summary>See example</summary>

```json
{
  "type": "BLANK_SCREEN",
  "name": "Blank Screen",
  "shortcut": "F9",
  "presentation_type": "SONG",
  "state": false
}
```
</details>

## NewChatMessageInfo
| Name | Type  | Description |
| ---- | :---: | ------------|
| `id` | _String_ |  |
| `datetime` | _Number_ |  |
| `user_id` | _String_ |  |
| `name` | _String_ |  |
| `message` | _String_ |  |
<details>
  <summary>See example</summary>

```json
{
  "id": "1742142790725",
  "datetime": 1742142790725,
  "user_id": "-1qfe9t8wtrsb6p5",
  "name": "example",
  "message": "example"
}
```
</details>

## VersePresentationChangedInfo
| Name | Type  | Description |
| ---- | :---: | ------------|
| `id` | _String_ |  |
| `book` | _Number_ |  |
| `chapter` | _Number_ |  |
| `verse` | _Number_ |  |
| `reference` | _String_ |  |
<details>
  <summary>See example</summary>

```json
{
  "id": "01001001",
  "book": 1,
  "chapter": 1,
  "verse": 1,
  "reference": "Gn 1:1"
}
```
</details>

## PlaylistChangedInfo
<details>
  <summary>See example</summary>

```json
{}
```
</details>

## FileModifiedInfo
| Name | Type  | Description |
| ---- | :---: | ------------|
| `media_type` | _String_ |  |
| `action` | _String_ |  |
| `name` | _String_ |  |
| `is_dir` | _Boolean_ |  |
<details>
  <summary>See example</summary>

```json
{
  "media_type": "image",
  "action": "created",
  "name": "image.jpg",
  "is_dir": false
}
```
</details>

## PlayerProgressInfo
| Name | Type  | Description |
| ---- | :---: | ------------|
| `time` | _Number_ |  |
| `total` | _Number_ |  |
<details>
  <summary>See example</summary>

```json
{
  "time": 0,
  "total": 60000
}
```
</details>

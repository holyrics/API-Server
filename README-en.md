# API-Server
**EN** | [PT](README.md)

The Holyrics program provides a communication interface to receive requests both over the local network and over the internet.
<br/>
When perform a request, the default implementation of this documentation will be used.
<br/>
But it is also possible to implement a JavaScript method in the Holyrics program itself so that requests are redirected to this method and perform custom actions as needed.
<br/>
Use the `POST` method and the `Content-Type: application/json` to perform the requests.
<br/>
It is necessary to add the `token` parameter to the request URL, which is created in the API Server settings, option 'manage permissions'.
<br/>
You can create multiple tokens and set permissions for each token.
<br/>

To access API Server settings:
<br/>
File menu > Settings > API Server

# Example of a local network request

You can add the token in the URL to perform the request.
<br/>
However, because it is a local connection without SSL, if you want greater security, use the "hash" method to send requests without having to inform the access token in the request.

Default URL - Using token

```
http://[IP]:[PORT]/api/{action}?token=abcdef
```

Default URL - Using hash

```
http://[IP]:[PORT]/api/{action}?dtoken=xyz123&sid=456&rid=3

dtoken
hash generated for each request from a 'nonce' (temporary token) obtained by the server

sid
Session ID, obtained along with the nonce

rid: request id
Sent by the client, a positive number that must always be greater than the one used in the previous request
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
data  = '{}';      <- requisition content

dtoken is the sha256 hash of the concatenation of the request information as shown in the example
dtoken = sha256(nonce + ':' + rid + ':' + token + ':' + data);
result: f3391f69cbe03940bd0d4a63ee191092aab2f3573f56b410a9cf94da05d4cdb5

curl -X 'POST' \
  'http://ip:port/api/GetCPInfo?sid=abc&rid=3&dtoken=f3391f69cbe03940bd0d4a63ee191092aab2f3573f56b410a9cf94da05d4cdb5' \
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
  'http://ip:port/api/SetTextCP?sid=abc&rid=7&dtoken=02a7789759694c535cd032489bf101110837c972d76cec51c7ad7e797696749d' \
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

Use the 'Auth' action again to authenticate, adding the parameters in the URL

```
nonce = 'b58ba4f605bed27c40a20be53ee3cf3d';
rid   = 0;       <- must be zero for authentication
token = '1234';  <- access token
data  = 'auth';  <- must be 'auth' for authentication

dtoken = sha256(nonce + ':' + rid + ':' + token + ':' + data);
result: 5d632009dfde5e9771b4f98f1b28c88ac2f73ae1f9d81b62a9af241a304c4d7a

http://ip:port/api/Auth?sid=u80fbjbcknir&rid=0&dtoken=5d632009dfde5e9771b4f98f1b28c88ac2f73ae1f9d81b62a9af241a304c4d7a

Response
{ "status": "ok" }
```

# Example of internet request

Use the **API_KEY** value available in the API Server settings to make requests to the Holyrics server endpoint along with the access token.

### Request - send
**Just send the request without waiting for a response**

Default URL
```
https://api.holyrics.com.br/send/{action}?api_key=API_KEY&token=abcdef
```

Request
```
curl -X 'POST' \
  'https://api.holyrics.com.br/send/SetTextCP?api_key=API_KEY&token=abcdef' \
  -H 'Content-Type: application/json' \
  -d '{"text": "Example", "show": true, "display_ahead": true}'
```

Response

```
{ "status": "ok" }
```

The status of **send** requests only informs if the request was sent to Holyrics opened on the computer.

---

### Request - request
Send the request and wait for the response

Default URL
```
https://api.holyrics.com.br/request/{action}?api_key=API_KEY&token=abcdef
```

Request
```
curl -X 'POST' \
  'https://api.holyrics.com.br/request/GetCPInfo?api_key=API_KEY&token=abcdef' \
  -H 'Content-Type: application/json' \
  -d '{}'
```

Response

```
{
  "status": "ok",           <-  status of sending the request to the Holyrics
  "response_status": "ok",  <-  response status of sent request
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
        "key": "invalid_api_key",
        "message": "Invalid API Key"
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

# Available actions 
  - [GetLyrics](#getlyrics)
  - [SearchLyrics](#searchlyrics)
  - [ShowLyrics](#showlyrics)
  - [ShowText](#showtext)
  - [ShowVerse](#showverse)
  - [GetAudios](#getaudios)
  - [PlayAudio](#playaudio)
  - [PlayVideo](#playvideo)
  - [ShowImage](#showimage)
  - [ShowAnnouncement](#showannouncement)
  - [GetCustomMessages](#getcustommessages)
  - [ShowCustomMessage](#showcustommessage)
  - [ShowQuickPresentation](#showquickpresentation)
  - [PlayAutomaticPresentation](#playautomaticpresentation)
  - [GetMediaPlayerInfo](#getmediaplayerinfo)
  - [MediaPlayerAction](#mediaplayeraction)
  - [GetLyricsPlaylist](#getlyricsplaylist)
  - [AddLyricsToPlaylist](#addlyricstoplaylist)
  - [RemoveFromLyricsPlaylist](#removefromlyricsplaylist)
  - [GetMediaPlaylist](#getmediaplaylist)
  - [MediaPlaylistAction](#mediaplaylistaction)
  - [RemoveFromMediaPlaylist](#removefrommediaplaylist)
  - [GetFavorites](#getfavorites)
  - [FavoriteAction](#favoriteaction)
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
  - [GetBackgrounds](#getbackgrounds)
  - [SetCurrentBackground](#setcurrentbackground)
  - [SetAlert](#setalert)
  - [GetCurrentSchedule](#getcurrentschedule)
  - [GetSchedules](#getschedules)
  - [GetSavedPlaylists](#getsavedplaylists)
  - [LoadSavedPlaylist](#loadsavedplaylist)
  - [GetHistory](#gethistory)
  - [GetHistories](#gethistories)
  - [GetMembers](#getmembers)
  - [GetRoles](#getroles)
  - [GetCommunicationPanelInfo](#getcommunicationpanelinfo)
  - [StartCountdownCommunicationPanel](#startcountdowncommunicationpanel)
  - [StopCountdownCommunicationPanel](#stopcountdowncommunicationpanel)
  - [SetTextCommunicationPanel](#settextcommunicationpanel)
  - [SetAlertCommunicationPanel](#setalertcommunicationpanel)
  - [GetWallpaperSettings](#getwallpapersettings)
  - [SetWallpaperSettings](#setwallpapersettings)
  - [GetDisplaySettings](#getdisplaysettings)
  - [SetDisplaySettings](#setdisplaysettings)
  - [GetBpm](#getbpm)
  - [SetBpm](#setbpm)
  - [GetHue](#gethue)
  - [SetHue](#sethue)
  - [GetRuntimeEnvironment](#getruntimeenvironment)
  - [SetRuntimeEnvironment](#setruntimeenvironment)
  - [SetLogo](#setlogo)
  - [GetSyncStatus](#getsyncstatus)


---

### GetLyrics
### GetSong
- v2.19.0

Returns a song.

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `id` | _String_ | Song ID |


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

### SearchLyrics
### SearchSong
- v2.19.0

Performs a search in the user's lyrics list

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `input` | _String_ | Filter |
| `text` | _String_ | Text to be searched |
| `title` | _Boolean (optional)_ |  _(Default=*true*)_ |
| `artist` | _Boolean (optional)_ |  _(Default=*true*)_ |
| `note` | _Boolean (optional)_ |  _(Default=*true*)_ |
| `lyrics` | _Boolean (optional)_ |  _(Default=*false*)_ |
| `group` | _String (optional)_ |  |


**Response:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `data` | _Array&lt;[Lyrics](#lyrics)&gt;_ |  |


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


_Method does not return value_

**Example:**
```
Request
{
  "id": "123"
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

Starts a Bible verse presentation.

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `input` | _Object_ | **id**, **ids** or **references** |
| `id` | _String (optional)_ | To display a verse. Item ID in BBCCCVVV format. Example: '19023001' (i.e. book 19, chapter 023, verse 001) |
| `ids` | _Array&lt;String&gt; (optional)_ | To display a list of verses. List with the ID of each verse. Example: ['19023001', '43003016', '45012002'] |
| `references` | _String (optional)_ | References. Example: **John 3:16** or **Rm 12:2** or **Gn 1:1-3 Sl 23.1** |


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
- v2.19.0

Returns the file list of the respective tab: audio, video, image

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `folder` | _String (optional)_ | Subfolder name to list files |
| `filter` | _String (optional)_ | Filter files by name |


**Response:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `data` | _Array&lt;Object&gt;_ |  |
| `data.*.name` | _String_ | Item name |
| `data.*.isDir` | _Boolean_ | Return **true** if it's a folder or **false** if it's a file. |


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
      "isDir": true
    },
    {
      "name": "abcd.jpg",
      "isDir": false
    }
  ]
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
| `automatic` | _[Automatic](#automatic-presentation) (optional)_ | If informed, the presentation of the items will be automatic |


_Method does not return value_

**Example:**
```
Request
{
  "file": "image.jpg"
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
| `automatic` | _[Automatic](#automatic-presentation) (optional)_ | If informed, the presentation of the items will be automatic |


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

| Name | Type  | Description |
| ---- | :---: | ------------|
| `data` | _Array&lt;[CustomMessage](#custom-message)&gt;_ |  |


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
| `position_?` | _Object_ | Variable added at the specified position according to the value returned in **variables.*.position** of the CustomMessage class. |
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
| `text` | _String_ | Text to be displayed. [Styled Text](#styled-text) from v2.19.0 |
| `theme` | _Object (optional)_ | Filter selected theme for display |
| `theme.id` | _String (optional)_ | Theme ID |
| `theme.name` | _String (optional)_ | Theme name |
| `automatic` | _[Automatic](#automatic-presentation) (optional)_ | If informed, the presentation of the items will be automatic |


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


_Method does not return value_

**Example:**
```
Request
{
  "file": "filename"
}
```


---

### GetMediaPlayerInfo
- v2.19.0

Returns player information



**Response:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `name` | _String_ | Name of current media in player |
| `path` | _String_ | Full path of media in player |
| `playing` | _Boolean_ | Checks if the player is running |
| `duration_ms` | _Number_ | Total time in milliseconds |
| `time_ms` | _Number_ | Current media time in milliseconds |
| `time_elapsed` | _String_ | Elapsed time in format HH:MM:SS |
| `time_remaining` | _String_ | Remaining time in format HH:MM:SS |
| `volume` | _Number_ | Current player volume. Minimum=0, Maximum=100 |
| `mute` | _Boolean_ | Checks if the **mute** option is enabled |
| `repeat` | _Boolean_ | Checks if the **repeat** option is enabled |
| `execute_single` | _Boolean_ | Checks if the player is set to play only the current list item |
| `shuffle` | _Boolean_ | Checks if the **random** option is enabled |
| `fullscreen` | _Boolean_ | Checks if the **full screen** option is enabled |


**Example:**
```
Response
{
  "status": "ok",
  "data": {
    "name": "video.mp4",
    "path": "C:\\Holyrics\\Holyrics\\files\\media\\video\\video.mp4",
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

| Name | Type  | Description |
| ---- | :---: | ------------|
| `data` | _Array&lt;[Lyrics](#lyrics)&gt;_ |  |


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
| `index` | _Number (optional)_ | Position in the list where the item will be added (starts at zero). Items are added to the end of the list by default. _(Default=*-1*)_ |
| `media_playlist` | _Boolean (optional)_ | Add the lyrics to the media playlist _(Default=*false*)_ |


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

### GetMediaPlaylist
- v2.19.0

Media playlist



**Response:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `data` | _Array&lt;[Item](#item)&gt;_ |  |


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

### GetFavorites
- v2.19.0

Favorites bar items



**Response:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `data` | _Array&lt;[Favorite Item](#favorite-item)&gt;_ |  |


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

Executes a request to the associated receiver and returns the receiver's response.

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



**Response:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `data` | _[Item](#item)_ |  |


**Example:**
```
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

### GetBackgrounds
- v2.19.0

List of themes and backgrounds

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `input` | _Object (optional)_ | Filter |
| `type` | _String (optional)_ | It might be: theme, my_video, my_image, video, image |
| `tag` | _String (optional)_ |  |
| `tags` | _Array&lt;String&gt; (optional)_ |  |
| `intersection` | _Boolean (optional)_ | If the **tags** field is filled with multiple items, the **intersection** option defines the type of join. If **true**, the filter will only return items that contain **all** the informed tags, if **false**, the filter will return items that have at least one tag of the informed tags _(Default=*false*)_ |


**Response:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `data` | _Array&lt;[Background](#background)&gt;_ |  |


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
| `type` | _String (optional)_ | It might be: theme, my_video, my_image, video, image |
| `tag` | _String (optional)_ |  |
| `tags` | _Array&lt;String&gt; (optional)_ |  |
| `intersection` | _Boolean (optional)_ | If the **tags** field is filled with multiple items, the **intersection** option defines the type of join. If **true**, the filter will only return items that contain **all** the informed tags, if **false**, the filter will return items that have at least one tag of the informed tags _(Default=*false*)_ |


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



**Response:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `data` | _[Schedule](#schedule)_ |  |


**Example:**
```
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

| Name | Type  | Description |
| ---- | :---: | ------------|
| `data` | _Array&lt;[Schedule](#schedule)&gt;_ |  |


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



**Response:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `data` | _Array&lt;Object&gt;_ |  |
| `data.*.music_id` | _String_ | Song ID |
| `data.*.history` | _Array&lt;String&gt;_ | Date and time in format: YYYY-MM-DD HH:MM |


**Example:**
```
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

### GetMembers
- v2.19.0

List of members



**Response:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `data` | _Array&lt;[Member](#member)&gt;_ |  |


**Example:**
```
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



**Response:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `data` | _Array&lt;[Role](#role)&gt;_ |  |


**Example:**
```
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

### GetCommunicationPanelInfo
### GetCPInfo
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
| `stop_at_zero` | _Boolean (optional)_ | Stop the countdown when it reaches zero _(Default=*false*)_ |


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

### SetTextCommunicationPanel
### SetTextCP
- v2.19.0

Change communication panel text

**Parameters:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `text` | _String (optional)_ | Change the text of the communication panel. [Styled Text](#styled-text) from v2.19.0 |
| `show` | _Boolean (optional)_ | Show/hide the text |
| `display_ahead` | _Boolean (optional)_ | Change the *'display in front of all'* option |


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

### GetWallpaperSettings
- v2.19.0

Wallpaper settings



**Response:**

| Name | Type  | Description |
| ---- | :---: | ------------|
| `data.image_base64` | _String_ | Wallpaper image in base 64 |
| `data.enabled` | _Boolean_ | Show wallpaper |
| `data.fill_color` | _String_ | Color in hexadecimal defined in the **fill** option. |
| `data.extend` | _Boolean_ | Extend wallpaper |
| `data.show_clock` | _Boolean_ | Show clock |


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
| `extend` | _Boolean (optional)_ | Extend wallpaper |
| `show_clock` | _Boolean (optional)_ | Show clock |


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

| Name | Type  | Description |
| ---- | :---: | ------------|
| `data` | _Array&lt;[DisplaySettings](#display-settings)&gt;_ |  |


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



# JavaScript implementation example

Use the methods of the [JSLIB](https://github.com/holyrics/jslib) class to create your own implementation.
<br/>
If you create your own implementation, you need to return 'true' if you want the program to perform the request using the default implementation.
<br/>
Any returned value that is different from 'true', the program will consider that the request has already been consumed and will respond with the returned value.

```javascript
function request(action, headers, content, info) {
    switch (action) {
        case 'my_custom_action':
            //perform your own action and response
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
`info.client_address` Request source address
<br/>
`info.token` access token used in the request
<br/>
`info.local` **true** if the source of the request is from the local network
<br/>
`info.web` **true** if the origin of the request is from the internet
<br/>

# Classes

## Lyrics
| Name | Type  | Description |
| ---- | :---: | ------------|
| `id` | _String_ | Song ID |
| `title` | _String_ | Song title |
| `artist` | _String_ | Music artist |
| `author` | _String_ | Music author |
| `note` | _String_ | Music annotation |
| `key` | _String_ | Tone of music |
| `time_sig` | _String_ | Music time |
| `groups` | _Array&lt;[Group](#group)&gt;_ | Groups where music is added |
| `midi` | _[Midi](#midi)_ | Item MIDI shortcut |
| `archived` | _Boolean_ | If the song is archived |
## Background
| Name | Type  | Description |
| ---- | :---: | ------------|
| `id` | _String_ | Item ID |
| `type` | _String_ | Item type. It might be:theme, my_video, my_image, video, image |
| `name` | _String_ | Item name |
| `tags` | _Array&lt;String&gt;_ | Item tag list |
| `bpm` | _Number_ | BPM value of item |
| `midi` | _[Midi](#midi) (optional)_ | Item MIDI shortcut |
## Item
| Name | Type  | Description |
| ---- | :---: | ------------|
| `id` | _String_ | Item ID |
| `type` | _String_ | Item type. It might be: song, verse, text, audio, video, image, file, custom-text, announcement, countdown, countdown-cp, automatic_presentation, api, script |
| `name` | _String_ | Item name |
## Group
| Name | Type  | Description |
| ---- | :---: | ------------|
| `name` | _String_ | Item name |
## MIDI
| Name | Type  | Description |
| ---- | :---: | ------------|
| `code` | _Number_ | MIDI code |
| `velocity` | _Number_ | Speed/intensity midi |
## Favorite Item
| Name | Type  | Description |
| ---- | :---: | ------------|
| `id` | _String_ | Item ID |
| `name` | _String_ | Item name |
## Schedule
| Name | Type  | Description |
| ---- | :---: | ------------|
| `type` | _String_ | Playlist type. Can be: temporary, service, event |
| `name` | _String_ |  |
| `datetime` | _String_ | Date and time in format: YYYY-MM-DD HH:MM |
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
## Member
| Name | Type  | Description |
| ---- | :---: | ------------|
| `id` | _String_ | Item ID |
| `name` | _String_ | Item name |
## Role
| Name | Type  | Description |
| ---- | :---: | ------------|
| `id` | _String_ | Item ID |
| `name` | _String_ | Item name |
## Automatic Presentation
| Name | Type  | Description |
| ---- | :---: | ------------|
| `seconds` | _Number_ | Time each item will be displayed |
| `repeat` | _Boolean_ | **true** to keep repeating the presentation (go back to the first item after the last one) |
## Input Param
| Name | Type  | Description |
| ---- | :---: | ------------|
| `key` | _String_ | Key of item |
| `type` | _String_ | Input type. Can be: text, textarea, number, password, title, separator |
| `label` | _String_ | Item name |
| `default_value` | _Object (optional)_ | Item default value |
| `allowed_values` | _Array&lt;String&gt; (optional)_ | Available if type is **text**. Defines a list of allowed values, to be selected as a combobox |
| `suggested_values` | _Array&lt;String&gt; (optional)_ | Available if type is **text**. Defines a list of suggested values, however the user can enter any value in the text field |
| `min` | _Number (optional)_ | Available if type is **number**. Sets the minimum allowed value _(Default=*0*)_ |
| `max` | _Number (optional)_ | Available if type is **number**. Sets the maximum allowed value _(Default=*100*)_ |
| `show_as_combobox` | _Boolean (optional)_ | Available if type is **number**. Display the list of values as a combobox and not as a spinner _(Default=*false*)_ |
## Display Settings
| Name | Type  | Description |
| ---- | :---: | ------------|
| `id` | _String_ | Item ID |
| `name` | _String_ | Item name |
| `stage_view` | _[StageView](#stage-view)_ | Stage view settings. (Unavailable for public screen) |
| `slide_info` | _[SlideAdditionalInfo](#slide-additional-info)_ | Additional slide info |
| `slide_translation` | _String_ | translation name |
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
## Stage View
| Name | Type  | Description |
| ---- | :---: | ------------|
| `enabled` | _Boolean_ | Stage view enabled |
| `preview_mode` | _String_ | Lyrics display mode. Available options:<br/>CURRENT_SLIDE<br/>FIRST_LINE_OF_THE_NEXT_SLIDE_WITH_SEPARATOR<br/>FIRST_LINE_OF_THE_NEXT_SLIDE_WITHOUT_SEPARATOR<br/>NEXT_SLIDE<br/>CURRENT_AND_NEXT_SLIDE<br/>ALL_SLIDES |
| `uppercase` | _Boolean_ | Show in capital letters |
| `remove_line_break` | _Boolean_ | remove line break |
| `show_comment` | _Boolean_ | Show comments |
| `show_advanced_editor` | _Boolean_ | Show advanced editor |
| `show_communication_panel` | _Boolean_ | Show communication panel content |
| `custom_theme` | _Number_ | Custom Theme ID used in presentations |
| `apply_custom_theme_to_bible` | _Boolean_ | Use custom theme in verses |
| `apply_custom_theme_to_text` | _Boolean_ | Use custom theme in texts |
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
| `info_2.layout_row_1` | _String_ | First row information layout [Slide Additional Info Layout](#slide-additional-info-layout) |
| `info_2.layout_row_2` | _String (optional)_ | Second line information layout [Slide Additional Info Layout](#slide-additional-info-layout) |
| `info_2.horizontal_align` | _String_ | Horizontal alignment of information on the slide. left, center, right |
| `info_2.vertical_align` | _String_ | Vertical alignment of information on the slide. top, bottom |
| `font` | _Object_ |  |
| `font.name` | _String_ | Font name. If it is **null**, it uses the theme's default font. |
| `font.bold` | _Boolean_ | Bold. If it is **null**, use the default theme setting |
| `font.italic` | _Boolean_ | Italid. If it is **null**, use the default theme setting |
| `font.color` | _String_ | Font color in hexadecimal. If it is **null**, use the theme's default font color |
| `height` | _Number_ | Height in percent of slide height |
| `paint_theme_effect` | _String_ | Render text with theme outline, glow, and shadow effects, if available |
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
## Custom Message Param
| Name | Type  | Description |
| ---- | :---: | ------------|
| `position` | _Number_ | Parameter position in the message (in number of characters) |
| `name` | _String_ | Item name |
| `only_number` | _Boolean_ | Parameter accepts only numbers |
| `uppercase` | _Boolean_ | Parameter always displayed in uppercase |
| `suggestions` | _Array&lt;String&gt; (optional)_ | List with default values for the parameter |

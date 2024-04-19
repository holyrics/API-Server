# API-Server
**PT** | [EN](README-en.md)

O programa Holyrics disponibiliza uma interface de comunicação para receber requisições tanto pela rede local quanto pela internet.
<br/>
Ao realizar uma requisição, a implementação padrão desta documentação será utilizada.
<br/>
Mas também é possível implementar um método JavaScript no próprio programa Holyrics para que as requisições sejam redirecionadas para esse método e executar ações customizadas conforme necessidade.
<br/>
Utilize o método `POST` e o `Content-Type: application/json` para realizar as requisições.
<br/>
É necessário adicionar na URL da requisição o parâmetro `token`, que é criado nas configurações API Server, opção 'gerenciar permissões'.
<br/>
Você pode criar múltiplos tokens e definir as permissões de cada token.
<br/>

Para acessar as configurações API Server:
<br/>
Menu arquivo > Configurações > API Server

# Exemplo de requisição pela rede local

Você pode passar o token na URL para realizar a requisição.
<br/>
Porém por se tratar de conexão local sem SSL, se você quiser uma maior segurança, utilize o método "hash" para enviar as requisições sem precisar informar o token de acesso na requisição.

URL padrão - Utilizando token

```
http://[IP]:[PORT]/api/{action}?token=abcdef
```

URL padrão - Utilizando hash

```
http://[IP]:[PORT]/api/{action}?dtoken=xyz123&sid=456&rid=3

dtoken
hash gerado para cada requisição a partir de um 'nonce' (token temporário) obtido pelo servidor

sid
ID da sessão, obtido junto com o nonce

rid: request id
Enviado pelo cliente, um número positivo que deve ser sempre maior do que o utilizado na requisição anterior
```

Requisição

```
Utilizando token
curl -X 'POST' \
  'http://ip:port/api/GetCPInfo?token=abcdef' \
  -H 'Content-Type: application/json' \
  -d '{}'
```

```
Utilizando hash
nonce = '1a2b3c';  <- token temporário
rid   = 4;         <- id da requisição
token = 'abcdef';  <- token de acesso
data  = '{}';      <- conteúdo da requisição

dtoken é o resumo sha256 da concatenação das informações da requisição conforme exemplo
dtoken = sha256(nonce + ':' + rid + ':' + token + ':' + data);
resultado: f3391f69cbe03940bd0d4a63ee191092aab2f3573f56b410a9cf94da05d4cdb5

curl -X 'POST' \
  'http://ip:port/api/GetCPInfo?sid=abc&rid=4&dtoken=f3391f69cbe03940bd0d4a63ee191092aab2f3573f56b410a9cf94da05d4cdb5' \
  -H 'Content-Type: application/json' \
  -d '{}'
```

Resposta

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

Requisição

```
Utilizando token
curl -X 'POST' \
  'http://ip:port/api/SetTextCP?token=abcdef' \
  -H 'Content-Type: application/json' \
  -d '{"text": "Example", "show": true, "display_ahead": true}'
```

```
Utilizando hash
nonce = '1a2b3c';
rid   = 5;
token = 'abcdef';
data  = '{"text": "Example", "show": true, "display_ahead": true}';

dtoken = sha256(nonce + ':' + rid + ':' + token + ':' + data);
resultado: 02a7789759694c535cd032489bf101110837c972d76cec51c7ad7e797696749d

curl -X 'POST' \
  'http://ip:port/api/SetTextCP?sid=abc&rid=5&dtoken=02a7789759694c535cd032489bf101110837c972d76cec51c7ad7e797696749d' \
  -H 'Content-Type: application/json' \
  -d '{"text": "Example", "show": true, "display_ahead": true}'
```

Resposta

```
{ "status": "ok" }
```

---

No caso de erro, **status** irá retornar **error**, exemplo:

```
{
    "status": "error",
    "error": "invalid token"
}
```

### Como obter um nonce

Utilize a ação 'Auth' sem incluir parâmetros na URL para obter um nonce
```
Requisição
http://ip:port/api/Auth

Resposta
{
    "status": "ok",
    "data": {
        "sid": "u80fbjbcknir",
        "nonce":"b58ba4f605bed27c40a20be53ee3cf3d"
    }
}
```

Utilize a ação 'Auth' novamente para se autenticar, passando os parâmetros na URL

```
nonce = 'b58ba4f605bed27c40a20be53ee3cf3d';
rid   = 0;       <- deve ser zero para autenticação
token = '1234';  <- token de acesso
data  = 'auth';  <- deve ser 'auth' para autenticação

dtoken = sha256(nonce + ':' + rid + ':' + token + ':' + data);
resultado: 5d632009dfde5e9771b4f98f1b28c88ac2f73ae1f9d81b62a9af241a304c4d7a

http://ip:port/api/Auth?sid=u80fbjbcknir&rid=0&dtoken=5d632009dfde5e9771b4f98f1b28c88ac2f73ae1f9d81b62a9af241a304c4d7a

Resposta
{ "status": "ok" }
```

# Exemplo de requisição pela rede internet

Utilize o valor **API_KEY** disponível nas configurações do API Server para realizar as requisições no endpoint do servidor do Holyrics juntamente com o token de acesso.

### Requisição - send
**Apenas envia a requisição sem aguardar retorno**

URL padrão
```
https://api.holyrics.com.br/send/{action}
```

Requisição
```
curl -X 'POST' \
  'https://api.holyrics.com.br/send/SetTextCP' \
  -H 'Content-Type: application/json' \
  -H 'api_key: API_KEY' \
  -H 'token: abcdef' \
  -d '{"text": "Example", "show": true, "display_ahead": true}'
```

Resposta

```
{ "status": "ok" }
```

O status das requisições **send** informa apenas se a requisição foi enviada ao Holyrics aberto no computador.

---

### Requisição - request
Envia a requisição e aguarda a resposta

URL padrão
```
https://api.holyrics.com.br/request/{action}
```

Requisição
```
curl -X 'POST' \
  'https://api.holyrics.com.br/request/GetCPInfo' \
  -H 'Content-Type: application/json' \
  -H 'api_key: API_KEY' \
  -H 'token: abcdef' \
  -d '{}'
```

Resposta

```
{
  "status": "ok",           <-  status do envio da requisição ao Holyrics
  "response_status": "ok",  <-  status da resposta da requisição enviada
  "response": {             <-  resposta da requisição
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

### Exemplos de erro no envio da requisição:

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

### Exemplos de erro na resposta da requisição:

```
{
  "status": "ok",           <-  a requisição foi enviada ao computador
  "response_status": "ok",  <-  a resposta foi recebida
  "response": {             <-  resposta recebida do computador
      "status": "error",
      "error": "invalid token"
    }
  }
}
```

```
{
    "status": "ok",               <-  a requisição foi enviada ao computador
    "response_status": "timeout"  <-  o tempo aguardando a resposta foi esgotado
}
```

# Ações disponíveis 
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
  - [GetBibleSettings](#getbiblesettings)
  - [SetBibleSettings](#setbiblesettings)
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
  - [OpenDrawLots](#opendrawlots)
  - [GetMediaDuration](#getmediaduration)
  - [GetVersion](#getversion)


---

### GetLyrics
### GetSong
- v2.19.0

Retorna uma música.

**Parâmetros:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `id` | _String_ | ID da música |


**Resposta:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `data` | _[Lyrics](#lyrics)_ | Música ou NULL se não for encontrado |


**Exemplo:**
```
Requisição
{
  "id": "123"
}

Resposta
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

Retorna a lista de músicas



**Resposta:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `data` | _Array&lt;[Lyrics](#lyrics)&gt;_ |  |


**Exemplo:**
```
Resposta
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

Realiza uma busca na lista de letras do usuário

**Parâmetros:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `input` | _String_ | Filtro |
| `text` | _String_ | Texto a ser pesquisado |
| `title` | _Boolean (opcional)_ |  `Padrão: true` |
| `artist` | _Boolean (opcional)_ |  `Padrão: true` |
| `note` | _Boolean (opcional)_ |  `Padrão: true` |
| `lyrics` | _Boolean (opcional)_ |  `Padrão: false` |
| `group` | _String (opcional)_ |  |


**Resposta:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `data` | _Array&lt;[Lyrics](#lyrics)&gt;_ |  |


**Exemplo:**
```
Requisição
{
  "text": "abc"
}

Resposta
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

Inicia uma apresentação de letra de música.

**Parâmetros:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `id` | _String_ | ID do item |


_Método sem retorno_

**Exemplo:**
```
Requisição
{
  "id": "123"
}
```


---

### GetText
- v2.21.0

Retorna um texto.

**Parâmetros:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `id` | _String_ | ID do texto |


**Resposta:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `data` | _[Text](#text)_ | Texto ou NULL se não for encontrado |


**Exemplo:**
```
Requisição
{
  "id": "xyz"
}

Resposta
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
    ]
  }
}
```


---

### GetTexts
- v2.21.0

Retorna a lista de textos



**Resposta:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `data` | _Array&lt;[Text](#text)&gt;_ |  |


**Exemplo:**
```
Resposta
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

Realiza uma busca na lista de textos do usuário

**Parâmetros:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `input` | _String_ | Filtro |
| `text` | _String_ | Texto a ser pesquisado |


**Resposta:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `data` | _Array&lt;[Lyrics](#lyrics)&gt;_ |  |


**Exemplo:**
```
Requisição
{
  "text": "example"
}

Resposta
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

Inicia uma apresentação de um item da aba texto.

**Parâmetros:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `id` | _String_ | ID do item |


_Método sem retorno_

**Exemplo:**
```
Requisição
{
  "id": "abc"
}
```


---

### ShowVerse
- v2.19.0

Inicia uma apresentação de versículo da Bíblia.

**Parâmetros:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `input` | _Object_ | **id**, **ids** ou **references** |
| `id` | _String (opcional)_ | Para exibir um versículo. ID do item no formato LLCCCVVV.<br/>Exemplo: '19023001' (livro 19, capítulo 023, versículo 001) |
| `ids` | _Array&lt;String&gt; (opcional)_ | Para exibir uma lista de versículos. Lista com o ID de cada versículo.<br/>Exemplo: ['19023001', '43003016', '45012002'] |
| `references` | _String (opcional)_ | Referências. Exemplo: **João 3:16** ou **Rm 12:2** ou **Gn 1:1-3 Sl 23.1** |
| `version` | _String (opcional)_ | Nome ou abreviação da tradução utilizada `v2.21.0+` |


_Método sem retorno_

**Exemplo:**
```
Requisição
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

Retorna a lista de arquivos da respectiva aba: áudio, vídeo, imagem, arquivo

**Parâmetros:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `folder` | _String (opcional)_ | Nome da subpasta para listar os arquivos |
| `filter` | _String (opcional)_ | Filtrar arquivos pelo nome |
| `include_metadata` | _Boolean (opcional)_ | Adicionar metadados na resposta `Padrão: false` `v2.22.0+` |
| `include_thumbnail` | _Boolean (opcional)_ | Adicionar thumbnail na resposta (80x45) `Padrão: false` `v2.22.0+` |


**Resposta:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `data` | _Array&lt;Object&gt;_ |  |
| `data.*.name` | _String_ | Nome do item |
| `data.*.isDir` | _Boolean_ | Retorna **true** se for uma pasta ou **false** se for arquivo. |
| <br>Disponível se **include_metadata=true** |  |  |
| `data.*.length` | _Number_ | Tamanho do arquivo (bytes). Disponível se **isDir=false** `v2.22.0+` |
| `data.*.modified_time` | _String_ | Data de modificação do arquivo. Data e hora no formato: YYYY-MM-DD HH:MM `v2.22.0+` |
| `data.*.duration_ms` | _Number_ | Duração do arquivo. Disponível se o arquivo for: audio ou vídeo `v2.22.0+` |
| `data.*.width` | _Number_ | Largura. Disponível se o arquivo for: imagem ou vídeo `v2.22.0+` |
| `data.*.height` | _Number_ | Altura. Disponível se o arquivo for: imagem ou vídeo `v2.22.0+` |
| `data.*.position` | _String_ | Ajuste da imagem. Disponível para imagens. Pode ser: `adjust` `extend` `fill` `v2.22.0+` |
| `data.*.blur` | _Boolean_ | Aplicar efeito blur `v2.22.0+` |
| `data.*.transparent` | _Boolean_ | Exibir imagens com transparência `v2.22.0+` |
| <br>Disponível se **include_thumbnail=true** |  |  |
| `data.*.thumbnail` | _String_ | Imagem no formato base64 `v2.22.0+` |


**Exemplo:**
```
Requisição
{
  "folder": "folder_name",
  "filter": "abc"
}

Resposta
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

Executa um áudio ou uma lista de áudios (pasta)

**Parâmetros:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `file` | _String_ | Nome do arquivo ou da pasta. Exemplo: **arquivo.mp3** ou **pasta** ou **pasta/arquivo.mp3** |
| `settings` | _[PlayMediaSettings](#play-media-settings) (opcional)_ | Configurações para execução da mídia `v2.21.0+` |


_Método sem retorno_

**Exemplo:**
```
Requisição
{
  "file": "audio.mp3"
}
```


---

### PlayVideo
- v2.19.0

Executa um vídeo ou uma lista de vídeos (pasta)

**Parâmetros:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `file` | _String_ | Nome do arquivo ou da pasta. Exemplo: **arquivo.mp4** ou **pasta** ou **pasta/arquivo.mp4** |
| `settings` | _[PlayMediaSettings](#play-media-settings) (opcional)_ | Configurações para execução da mídia `v2.21.0+` |


_Método sem retorno_

**Exemplo:**
```
Requisição
{
  "file": "video.mp4"
}
```


---

### ShowImage
- v2.19.0

Apresenta uma imagem ou uma lista de imagens (pasta)

**Parâmetros:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `file` | _String_ | Nome do arquivo ou da pasta. Exemplo: **arquivo.jpg** ou **pasta** ou **pasta/arquivo.jpg** |
| `automatic` | _[Automatic](#automatic-presentation) (opcional)_ | Se informado, a apresentação dos itens será automática |


_Método sem retorno_

**Exemplo:**
```
Requisição
{
  "file": "image.jpg"
}
```


---

### ExecuteFile
- v2.21.0

Executa um arquivo. Disponível apenas para extensões seguras, como áudio, vídeo, imagem, documentos, etc.

**Parâmetros:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `file` | _String_ | Nome do arquivo |


_Método sem retorno_

**Exemplo:**
```
Requisição
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

Verifica se existe o arquivo com o nome informado

**Parâmetros:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `file` | _String_ | Nome do arquivo |


**Resposta:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `data` | _Boolean_ |  |


**Exemplo:**
```
Requisição
{
  "file": "file.mp3"
}

Resposta
{
  "status": "ok",
  "data": true
}
```


---

### ShowAnnouncement
- v2.19.0

Apresenta um anúncio ou uma lista de anúncios

**Parâmetros:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `id` | _String (opcional)_ | ID do anúncio. Pode ser **all** para exibir todos |
| `ids` | _Array&lt;String&gt; (opcional)_ | Lista com o ID de cada anúncio |
| `name` | _String (opcional)_ | Nome do anúncio |
| `names` | _Array&lt;String&gt; (opcional)_ | Lista com o nome de cada anúncio |
| `automatic` | _[Automatic](#automatic-presentation) (opcional)_ | Se informado, a apresentação dos itens será automática |


_Método sem retorno_

**Exemplo:**
```
Requisição
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

Lista das mensagens personalizadas



**Resposta:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `data` | _Array&lt;[CustomMessage](#custom-message)&gt;_ |  |


**Exemplo:**
```
Resposta
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

Exibir uma mensagem personalizada. Obs.: Uma mensagem personalizada não é exibida diretamente na tela. é criada uma notificação no canto da tela para o operador aceitar e exibir.

**Parâmetros:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `name` | _String_ | Nome da mensagem personalizada |
| `position_?` | _Object (opcional)_ | Variável adicionada na posição especificada conforme valor retornado em **variables.*.position** da classe CustomMessage. |
| `params` | _Object (opcional)_ | Método alternativo. Mapa chave/valor onde a chave é o nome do campo **variables.*.name** da classe CustomMessage. Se necessário adicionar o mesmo parâmetro, adicione `*_n` no final do nome, começando em 2<br>Exemplo: **params.name, params.name_2, params.name_3** `v2.21.0+` |
| `note` | _String_ | Informação extra exibida na janela popup para o operador |


_Método sem retorno_

**Exemplo:**
```
Requisição
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

Apresentação rápida de um texto

**Parâmetros:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `text` | _String_ | Texto que será exibido. [Styled Text](#styled-text) a partir da v2.19.0 |
| `theme` | _Object (opcional)_ | Filtrar tema selecionado para exibição |
| `theme.id` | _String (opcional)_ | ID do tema |
| `theme.name` | _String (opcional)_ | Nome do tema |
| `theme.edit` | _[Theme](#theme) (opcional)_ | Configurações para modificar o Tema selecionado para exibição `v2.21.0+` |
| `custom_theme` | _[Theme](#theme) (opcional)_ | Tema personalizado utilizado para exibir o texto `v2.21.0+` |
| `automatic` | _[Automatic](#automatic-presentation) (opcional)_ | Se informado, a apresentação dos itens será automática |


_Método sem retorno_

**Exemplo:**
```
Requisição
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

Exibir uma contagem regressiva na tela público

**Parâmetros:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `time` | _String_ | HH:MM ou MM:SS |
| `exact_time` | _Boolean (opcional)_ | Se **true**, `time` deve ser HH:MM (hora e minuto exato). Se **false**, `time` deve ser MM:SS (quantidade de minutos e segundos) `Padrão: false` |
| `text_before` | _String (opcional)_ | Texto exibido na parte superior da contagem regressiva |
| `text_after` | _String (opcional)_ | Texto exibido na parte inferior da contagem regressiva |
| `zero_fill` | _Boolean (opcional)_ | Preencher o campo 'minuto' com zero à esquerda `Padrão: false` |
| `countdown_relative_size` | _Number (opcional)_ | Tamanho relativo da contagem regressiva `Padrão: 250` |
| `theme` | _String (opcional)_ | ID do Tema `v2.21.0+` |
| `countdown_style` | _[FontSettings](#font-settings) (opcional)_ | Fonte personalizada para a contagem regressiva `v2.21.0+` |
| `custom_theme` | _[Theme](#theme) (opcional)_ | Tema personalizado `v2.21.0+` |


_Método sem retorno_

**Exemplo:**
```
Requisição
{
  "time": "05:00",
  "zero_fill": true
}
```


---

### ShowQuiz
- v2.20.0

Iniciar uma apresentação no formato múltipla escolha

**Parâmetros:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `questions` | _Array&lt;[QuizQuestion](#quiz-question)&gt;_ | Questões para exibir |
| `settings` | _[QuizSettings](#quiz-settings)_ | Configurações |


_Método sem retorno_

**Exemplo:**
```
Requisição
{
  "questions": [
    {
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

Executar uma ação em uma apresentação de múltipla escolha

**Parâmetros:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `action` | _String (opcional)_ | Um dos seguintes valores: `previous_slide`  `next_slide`  `previous_question`  `next_question`  `show_result`  `close` |
| `hide_alternative` | _Number (opcional)_ | Ocultar uma alternativa. Começa em 1 |
| `select_alternative` | _Number (opcional)_ | Selecionar uma alternativa. Começa em 1 |
| `countdown` | _Number (opcional)_ | Iniciar uma contagem regressiva. [1-120] |


_Método sem retorno_

**Exemplo:**
```
Requisição
{
  "action": "next_slide"
}
```


---

### GetAutomaticPresentations
### GetAPs
- v2.21.0

Retorna a lista de apresentações automáticas



**Resposta:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `data` | _Array&lt;Object&gt;_ |  |
| `data.*.name` | _String_ | Nome do arquivo. Exemplo: **arquivo.ap** |


**Exemplo:**
```
Resposta
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

### PlayAutomaticPresentation
### PlayAP
- v2.19.0

Executa um item apresentação automática

**Parâmetros:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `file` | _String_ | Nome do arquivo. Exemplo: **arquivo.ap** |
| `theme` | _Object (opcional)_ | Filtrar tema selecionado para exibição |
| `theme.id` | _String (opcional)_ | ID do tema |
| `theme.name` | _String (opcional)_ | Nome do tema |
| `theme.edit` | _[Theme](#theme) (opcional)_ | Configurações para modificar o Tema selecionado para exibição `v2.21.0+` |
| `custom_theme` | _[Theme](#theme) (opcional)_ | Tema personalizado utilizado para exibir a apresentação automática `v2.21.0+` |


_Método sem retorno_

**Exemplo:**
```
Requisição
{
  "file": "filename"
}
```


---

### GetAutomaticPresentationPlayerInfo
### GetAPPlayerInfo
- v2.20.0

Retorna as informações da apresentação automática em exibição



**Resposta:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `data.name` | _String_ | Nome do item |
| `data.playing` | _Boolean_ | Verifica se o player está em execução |
| `data.time_ms` | _Number_ | Tempo atual da mídia em milissegundos |
| `data.volume` | _Number_ | Volume atual do player. Mínimo=0, Máximo=100 |
| `data.mute` | _Boolean_ | Verifica se a opção **mudo** está ativada |
| `data.duration_ms` | _Number_ | Tempo total em milissegundos `v2.21.0+` |


**Exemplo:**
```
Resposta
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

Executa ações no player

**Parâmetros:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `action` | _String (opcional)_ | Nome da ação que será executada no player. play, pause, stop |
| `volume` | _Number (opcional)_ | Altera o volume do player. Mínimo=0, Máximo=100 |
| `mute` | _Boolean (opcional)_ | Altera a opção **mudo** |
| `time_ms` | _Boolean (opcional)_ | Alterar o tempo atual da mídia em milissegundos |


_Método sem retorno_

**Exemplo:**
```
Requisição
{
  "action": "play",
  "volume": 80
}
```


---

### GetMediaPlayerInfo
- v2.19.0

Retorna as informações do player



**Resposta:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `data.name` | _String_ | Nome da mídia atual no player |
| `data.path` | _String_ | Caminho completo da mídia no player |
| `data.playing` | _Boolean_ | Verifica se o player está em execução |
| `data.duration_ms` | _Number_ | Tempo total em milissegundos |
| `data.time_ms` | _Number_ | Tempo atual da mídia em milissegundos |
| `data.time_elapsed` | _String_ | Tempo decorrido no formato HH:MM:SS |
| `data.time_remaining` | _String_ | Tempo restante no formato HH:MM:SS |
| `data.volume` | _Number_ | Volume atual do player. Mínimo=0, Máximo=100 |
| `data.mute` | _Boolean_ | Verifica se a opção **mudo** está ativada |
| `data.repeat` | _Boolean_ | Verifica se a opção **repetir** está ativada |
| `data.execute_single` | _Boolean_ | Verifica se o player está definido para executar somente o item atual da lista |
| `data.shuffle` | _Boolean_ | Verifica se a opção **aleatório** está ativada |
| `data.fullscreen` | _Boolean_ | Verifica se a opção **tela cheia** está ativada |


**Exemplo:**
```
Resposta
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

Executa ações no player

**Parâmetros:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `action` | _String (opcional)_ | Nome da ação que será executada no player. play, pause, stop, next, previous |
| `volume` | _Number (opcional)_ | Altera o volume do player. Mínimo=0, Máximo=100 |
| `mute` | _Boolean (opcional)_ | Altera a opção **mudo** |
| `repeat` | _Boolean (opcional)_ | Altera a opção **repetir** |
| `shuffle` | _Boolean (opcional)_ | Altera a opção **aleatório** |
| `execute_single` | _Boolean (opcional)_ | Altera a configuração do player para executar somente o item atual da lista |
| `fullscreen` | _Boolean (opcional)_ | Altera a opção **tela cheia** do player |
| `time_ms` | _Boolean (opcional)_ | Alterar o tempo atual da mídia em milissegundos `v2.20.0+` |


_Método sem retorno_

**Exemplo:**
```
Requisição
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

Lista de reprodução de letras



**Resposta:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `data` | _Array&lt;[Lyrics](#lyrics)&gt;_ |  |


**Exemplo:**
```
Resposta
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

Adicionar letra de música na lista de reprodução

**Parâmetros:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `id` | _String (opcional)_ | ID da letra |
| `ids` | _Array&lt;String&gt; (opcional)_ | Lista com id de cada letra |
| `index` | _Number (opcional)_ | Posição na lista onde o item será adicionado (inicia em zero). Os itens são adicionados no final da lista por padrão. `Padrão: -1` |
| `media_playlist` | _Boolean (opcional)_ | Adicionar as letras na lista de reprodução de mídia `Padrão: false` |


_Método sem retorno_

**Exemplo:**
```
Requisição
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

Remover letra de música na lista de reprodução

**Parâmetros:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `id` | _String (opcional)_ | ID da letra |
| `ids` | _Array&lt;String&gt; (opcional)_ | Lista com id de cada letra |
| `index` | _Number (opcional)_ | Posição do item na lista que será removido (inicia em zero). |
| `indexes` | _Array&lt;Number&gt; (opcional)_ | Lista com a posição de cada item na lista que será removido. (Inicia em zero) |


_Método sem retorno_

**Exemplo:**
```
Requisição
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

Alterar um item da lista de reprodução de letra de música

**Parâmetros:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `index` | _Number_ | Índice do item na lista |
| `song_id` | _String_ | Novo item |


_Método sem retorno_

**Exemplo:**
```
Requisição
{
  "index": 2,
  "song_id": "123"
}
```


---

### GetMediaPlaylist
- v2.19.0

Lista de reprodução de mídia



**Resposta:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `data` | _Array&lt;[Item](#item)&gt;_ |  |


**Exemplo:**
```
Resposta
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

Alterar um item da lista de reprodução de mídia

**Parâmetros:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `index` | _Number_ | Índice do item na lista |
| `item` | _[AddItem](#additem)_ | Novo item |


_Método sem retorno_

**Exemplo:**
```
Requisição
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

Executa um item da lista de reprodução de mídia

**Parâmetros:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `id` | _String_ | ID do item |


_Método sem retorno_

**Exemplo:**
```
Requisição
{
  "id": "abc"
}
```


---

### GetNextSongPlaylist
- v2.22.0

Retorna a próxima música da lista de reprodução. Pode ser null



**Resposta:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `data` | _[Lyrics](#lyrics)_ |  |


**Exemplo:**
```
Resposta
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

Retorna o próximo item executável da lista de reprodução de mídia. Pode ser null



**Resposta:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `data` | _[Item](#item)_ |  |


**Exemplo:**
```
Resposta
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

Executa a próxima música da lista de reprodução



_Método sem retorno_



---

### ShowNextMediaPlaylist
- v2.22.0

Executa o próximo item da lista de reprodução de mídia



_Método sem retorno_



---

### GetPreviousSongPlaylist
- v2.22.0

Retorna a música anterior da lista de reprodução. Pode ser null



**Resposta:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `data` | _[Lyrics](#lyrics)_ |  |


**Exemplo:**
```
Resposta
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

Retorna o item anterior executável da lista de reprodução de mídia. Pode ser null



**Resposta:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `data` | _[Item](#item)_ |  |


**Exemplo:**
```
Resposta
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

Executa a música anterior da lista de reprodução



_Método sem retorno_



---

### ShowPreviousMediaPlaylist
- v2.22.0

Executa o item anterior da lista de reprodução de mídia



_Método sem retorno_



---

### AddToPlaylist
- v2.20.0

Adicionar itens à lista de reprodução de mídias

**Parâmetros:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `items` | _Array&lt;[AddItem](#additem)&gt;_ | Lista com os itens que serão adicionados |
| `index` | _Number (opcional)_ | Posição na lista onde o item será adicionado (inicia em zero). Os itens são adicionados no final da lista por padrão. `Padrão: -1` |
| `ignore_duplicates` | _Boolean (opcional)_ | Não duplicar itens ao adicionar novos itens, ou seja, não adiciona um item se ele já estiver na lista. `Padrão: false` |


_Método sem retorno_

**Exemplo:**
```
Requisição
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

Remover itens da lista de reprodução de mídia

**Parâmetros:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `id` | _String (opcional)_ | ID do item |
| `ids` | _Array&lt;String&gt; (opcional)_ | Lista com id de cada item |
| `index` | _Number (opcional)_ | Posição do item na lista que será removido (inicia em zero). |
| `indexes` | _Array&lt;Number&gt; (opcional)_ | Lista com a posição de cada item na lista que será removido. (Inicia em zero) |


_Método sem retorno_

**Exemplo:**
```
Requisição
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

Alterar duração de um item da lista de reprodução de mídia.

**Parâmetros:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `id` | _String (opcional)_ | ID do item |
| `index` | _Number (opcional)_ | Posição do item na lista (inicia em zero). |
| `duration` | _Number (opcional)_ | Duração do item (em segundos) |


_Método sem retorno_

**Exemplo:**
```
Requisição
{
  "id": "xyz",
  "duration": 300
}
```


---

### GetSlideDescriptions
- v2.21.0

Lista das descrições do slide disponíveis



**Resposta:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `data` | _Array&lt;[SlideDescription](#slide-description)&gt;_ |  |


**Exemplo:**
```
Resposta
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

Itens da barra de favoritos



**Resposta:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `data` | _Array&lt;[FavoriteItem](#favorite-item)&gt;_ |  |


**Exemplo:**
```
Resposta
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

Executa um item da barra de favoritos

**Parâmetros:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `id` | _String_ | ID do item |


_Método sem retorno_

**Exemplo:**
```
Requisição
{
  "id": "abcxyz"
}
```


---

### GetApis
- v2.21.0

Retorna a lista de APIs



**Resposta:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `data` | _Array&lt;Object&gt;_ |  |
| `data.*.id` | _String_ | ID do item |
| `data.*.name` | _String_ | Nome do item |


**Exemplo:**
```
Resposta
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

Retorna a lista de scripts



**Resposta:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `data` | _Array&lt;Object&gt;_ |  |
| `data.*.id` | _String_ | ID do item |
| `data.*.name` | _String_ | Nome do item |


**Exemplo:**
```
Resposta
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

Executa a ação de um item API existente no programa

**Parâmetros:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `id` | _String_ | ID do item |


_Método sem retorno_

**Exemplo:**
```
Requisição
{
  "id": "abcxyz"
}
```


---

### ScriptAction
- v2.19.0

Executa a ação de um item **Script** existente no programa

**Parâmetros:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `id` | _String_ | ID do item |


_Método sem retorno_

**Exemplo:**
```
Requisição
{
  "id": "abc"
}
```


---

### ApiRequest
- v2.19.0

Executa uma requisição para o receptor associado e retorna a resposta do receptor.

**Parâmetros:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `id` | _String_ | id do receptor |
| `raw` | _Object_ | dados da requisição |


**Resposta:**

| Tipo  | Descrição |
| :---: | ------------|
| _Object_ | Retorno da requisição ou NULL para erro/timeout |


**Exemplo:**
```
Requisição
{
  "id": "abcxyz",
  "raw": {
    "request-type": "GetSourceActive",
    "sourceName": "example"
  }
}

Resposta
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

Item sendo apresentado no momento ou **null** se não tiver apresentação sendo exibida

**Parâmetros:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `include_slides` | _Boolean (opcional)_ | Retornar a lista de slides da apresentação atual. Indisponível para apresentação de versículos. `Padrão: false` `v2.21.0+` |
| `include_slide_comment` | _Boolean (opcional)_ | Incluir comentários (se houver) no texto dos slides. Disponível se **include_slides=true**. `Padrão: false` `v2.21.0+` |
| `include_slide_preview` | _Boolean (opcional)_ | Incluir imagem preview do slide. Disponível se **include_slides=true**. `Padrão: false` `v2.21.0+` |
| `lide_preview_size` | _String (opcional)_ | Tamanho do preview no formato WxH (ex. 320x180). (max 640x360)<br>Disponível se **include_slide_preview=true** `Padrão: false` `v2.21.0+` |


**Resposta:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `data.id` | _String_ | ID do item |
| `data.type` | _String_ | Tipo do item. Pode ser: `song` `verse` `text` `audio` `image` `announcement` `automatic_presentation` `quick_presentation` |
| `data.name` | _String_ | Nome do item |
| `data.slide_number` | _Number_ | Começa em 1 `v2.20.0+` |
| `data.total_slides` | _Number_ | Total de slides `v2.20.0+` |
| `data.slide_type` | _String_ | Um dos seguintes valores: `default`  `wallpaper`  `blank`  `black`  `final_slide` `v2.20.0+` |
| `data.slides` | _Array&lt;[PresentationSlideInfo](#presentation-slide-info)&gt;_ | Lista com os slides da apresentação atual. Disponível se **include_slides=true** `v2.21.0+` |


**Exemplo:**
```
Requisição
{}

Resposta
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

Encerra a apresentação atual



_Método sem retorno_



---

### GetF8
### GetF9
### GetF10
- v2.19.0

Retorna o estado atual da respectiva opção **F8 (papel de parede), F9 (tela vazia) ou F10 (tela preta)**



**Resposta:**

| Tipo  | Descrição |
| :---: | ------------|
| _Boolean_ | **true** ou **false** |


**Exemplo:**
```
Resposta
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

Altera o estado atual da respectiva opção **F8 (papel de parede), F9 (tela vazia) ou F10 (tela preta)**

**Parâmetros:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `enable` | _Boolean_ | **true** ou **false** |


_Método sem retorno_

**Exemplo:**
```
Requisição
{
  "enable": true
}
```


---

### ToggleF8
### ToggleF9
### ToggleF10
- v2.19.0

Troca o estado atual da respectiva opção **F8 (papel de parede), F9 (tela vazia) ou F10 (tela preta)**



_Método sem retorno_



---

### ActionNext
- v2.19.0

Executa um comando de **avançar** na apresentação atual



_Método sem retorno_



---

### ActionPrevious
- v2.19.0

Executa um comando de **voltar** na apresentação atual



_Método sem retorno_



---

### ActionGoToIndex
- v2.19.0

Altera o slide em exibição a partir do índice do slide

**Parâmetros:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `index` | _Number_ | Índice do slide na apresentação (começa em zero) |


_Método sem retorno_

**Exemplo:**
```
Requisição
{
  "index": 3
}
```


---

### ActionGoToSlideDescription
- v2.19.0

Altera o slide em exibição a partir do nome da descrição do slide

**Parâmetros:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `name` | _String_ | Nome da descrição do slide |


_Método sem retorno_

**Exemplo:**
```
Requisição
{
  "name": "Verse 1"
}
```


---

### GetCurrentBackground
- v2.19.0

Retorna o plano de fundo da apresentação em exibição.



**Resposta:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `data` | _[Background](#background)_ | Plano de fundo atual ou NULL se não houver apresentação em exibição |


**Exemplo:**
```
Resposta
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

Retorna o tema da apresentação em exibição.



**Resposta:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `data` | _[Background](#background)_ | Tema atual ou NULL se não houver apresentação em exibição |


**Exemplo:**
```
Resposta
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

Lista dos temas e planos de fundo

**Parâmetros:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `input` | _Object (opcional)_ | Filtro |
| `type` | _String (opcional)_ | Pode ser: `theme` `my_video` `my_image` `video` `image` |
| `tag` | _String (opcional)_ |  |
| `tags` | _Array&lt;String&gt; (opcional)_ |  |
| `intersection` | _Boolean (opcional)_ | Se o campo **tags** estiver preenchido com múltiplos itens, a opção **intersection** define o tipo de junção. Se **true**, o filtro retornará apenas itens que contém **todas** as tags informadas, se **false**, o filtro retornará os itens que têm pelo menos uma tag das tags informadas `Padrão: false` |


**Resposta:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `data` | _Array&lt;[Background](#background)&gt;_ |  |


**Exemplo:**
```
Requisição
{
  "type": "my_video",
  "tag": "circle",
  "tags": [],
  "intersection": true
}

Resposta
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

Altera o plano de fundo (ou tema) da apresentação atual. Se mais de um item for encontrado de acordo com os filtros, será escolhido um item da lista de forma aleatória

**Parâmetros:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `input` | _Object (opcional)_ | Filtro |
| `id` | _String (opcional)_ | ID do tema ou plano de fundo |
| `name` | _String (opcional)_ | Nome do tema ou plano de fundo |
| `type` | _String (opcional)_ | Pode ser: `theme` `my_video` `my_image` `video` `image` |
| `tag` | _String (opcional)_ |  |
| `tags` | _Array&lt;String&gt; (opcional)_ |  |
| `intersection` | _Boolean (opcional)_ | Se o campo **tags** estiver preenchido com múltiplos itens, a opção **intersection** define o tipo de junção. Se **true**, o filtro retornará apenas itens que contém **todas** as tags informadas, se **false**, o filtro retornará os itens que têm pelo menos uma tag das tags informadas `Padrão: false` |
| `edit` | _[Theme](#theme) (opcional)_ | Configurações para modificar o Tema selecionado para exibição `v2.21.0+` |
| `custom_theme` | _[Theme](#theme) (opcional)_ | Tema personalizado `v2.21.0+` |


_Método sem retorno_

**Exemplo:**
```
Requisição
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

Retorna a imagem miniatura de um item no programa

**Parâmetros:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `id` | _String (opcional)_ | ID do item |
| `ids` | _Array&lt;String&gt; (opcional)_ | ID dos itens |
| `type` | _String_ | Tipo do item. Pode ser: `video` `image` `announcement` `theme` `background` `api` `script` |


**Resposta:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `data` | _Array&lt;Object&gt;_ |  |
| `data.*.type` | _String_ | Tipo do item |
| `data.*.id` | _String_ | ID do item |
| `data.*.image` | _String_ | Imagem no formato base64 |


**Exemplo:**
```
Requisição
{
  "id": "image.jpg",
  "type": "image"
}

Resposta
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

Retorna as informações de cor predominante de um respectivo tipo de item<br/>O array retornado contém 8 índices, e cada índice corresponde ao trecho conforme imagem de exemplo a seguir.<br/> <br/>![Color Map Example](https://holyrics.com.br/images/color_map_item_example.png)<br/>

**Parâmetros:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `type` | _String_ | Um dos seguintes valores:<br/>**background** - um item de tema ou plano de fundo<br/>**presentation** - apresentação atual em exibição<br/>**image** - uma imagem da aba 'imagens'<br/>**video** - um vídeo da aba 'vídeos'<br/>**printscreen** - um printscreen atual de uma tela do sistema<br/> |
| `source` | _Object (opcional)_ | O item de acordo com o tipo informado:<br/>**background** - ID do tema ou plano de fundo<br/>**presentation** - não é necessário informar um valor, a apresentação da tela público será retornada<br/>**image** - o nome do arquivo da aba 'imagens'<br/>**video** - o nome do arquivo da aba 'vídeos'<br/>**printscreen** `opcional` -  o nome da tela (public, screen_2, screen_3, ...); o padrão é `public`<br/> |


**Resposta:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `data` | _Array&lt;Object&gt;_ |  |
| `data.*.hex` | _String_ | Cor no formato hexadecimal |
| `data.*.red` | _Number_ | Vermelho  0-255 |
| `data.*.green` | _Number_ | Verde  0-255 |
| `data.*.blue` | _Number_ | Azul  0-255 |


**Exemplo:**
```
Requisição
{
  "type": "background",
  "source": 12345678
}

Resposta
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

Retorna as configurações da mensagem de alerta



**Resposta:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `data.text` | _String_ | Texto atual do alerta |
| `data.show` | _Boolean_ | Se a exibição do alerta está ativada |


**Exemplo:**
```
Resposta
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

Altera as configurações da mensagem de alerta

**Parâmetros:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `text` | _String (opcional)_ | Alterar o texto de alerta |
| `show` | _Boolean (opcional)_ | Exibir/ocultar o alerta |


_Método sem retorno_

**Exemplo:**
```
Requisição
{
  "text": "",
  "show": false
}
```


---

### GetCurrentSchedule
- v2.19.0

Programação atual (selecionada na janela principal do programa)



**Resposta:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `data` | _Array&lt;[Schedule](#schedule)&gt;_ |  |


**Exemplo:**
```
Resposta
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

Retorna a lista de programação de um mês específico

**Parâmetros:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `month` | _Number_ | Mês (1-12) |
| `year` | _Number_ | Ano |


**Resposta:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `data` | _Array&lt;[Schedule](#schedule)&gt;_ |  |


**Exemplo:**
```
Requisição
{
  "month": 3,
  "year": 2022
}

Resposta
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

Retorna as listas de reprodução salvas



**Resposta:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `data` | _Array&lt;Object&gt;_ |  |
| `data.*.id` | _String_ | ID do item |
| `data.*.name` | _String_ | Nome do item |
| `data.*.items` | _Array&lt;[Item](#item)&gt;_ | Itens salvos na lista |


**Exemplo:**
```
Resposta
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

Preenche a lista de mídias da lista de reprodução selecionada atualmente no programa com a lista informada

**Parâmetros:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `name` | _String_ | Nome da lista de reprodução salva |


_Método sem retorno_

**Exemplo:**
```
Requisição
{
  "name": ""
}
```


---

### GetHistory
- v2.19.0

Histórico de "Música tocada"

**Parâmetros:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `id` | _String_ | ID da letra da música |


**Resposta:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `data` | _Array&lt;String&gt;_ | Data e hora no formato: YYYY-MM-DD HH:MM |


**Exemplo:**
```
Requisição
{
  "id": "123"
}

Resposta
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

Histórico de todas as marcações de "Música tocada"



**Resposta:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `data` | _Array&lt;Object&gt;_ |  |
| `data.*.music_id` | _String_ | ID da música |
| `data.*.history` | _Array&lt;String&gt;_ | Data e hora no formato: YYYY-MM-DD HH:MM |


**Exemplo:**
```
Resposta
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

### GetTeams
- v2.22.0

Lista de times



**Resposta:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `data` | _Array&lt;[Team](#team)&gt;_ |  |


**Exemplo:**
```
Resposta
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

Lista de integrantes



**Resposta:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `data` | _Array&lt;[Member](#member)&gt;_ |  |


**Exemplo:**
```
Resposta
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

Lista de funções



**Resposta:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `data` | _Array&lt;[Role](#role)&gt;_ |  |


**Exemplo:**
```
Resposta
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

Lista de cultos



**Resposta:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `data` | _Array&lt;[Service](#service)&gt;_ |  |


**Exemplo:**
```
Resposta
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

Lista de eventos

**Parâmetros:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `month` | _Number_ | Mês (1-12) |
| `year` | _Number_ | Ano |


**Resposta:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `data` | _Array&lt;[Event](#event)&gt;_ |  |


**Exemplo:**
```
Requisição
{
  "month": 8,
  "year": 2022
}

Resposta
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

Anúncio

**Parâmetros:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `id` | _String (opcional)_ | ID do anúncio |
| `name` | _String (opcional)_ | Nome do anúncio |


**Resposta:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `data` | _[Announcement](#announcement)_ |  |


**Exemplo:**
```
Requisição
{
  "id": "123"
}

Resposta
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

Lista de anúncios



**Resposta:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `data` | _Array&lt;[Announcement](#announcement)&gt;_ |  |


**Exemplo:**
```
Resposta
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

Configuração atual do painel de comunicação



**Resposta:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `data.text` | _String_ | Texto atual |
| `data.show` | _Boolean_ | Se o texto atual está em exibição |
| `data.display_ahead` | _Boolean_ | Se a opção *'exibir à frente de tudo'* está ativada |
| `data.alert_text` | _String_ | Texto atual do alerta |
| `data.alert_show` | _Boolean_ | Se a exibição do alerta está ativada |
| `data.countdown_show` | _Boolean_ | Se uma contagem regressiva está em exibição |
| `data.countdown_time` | _Number_ | O tempo atual da contagem regressiva em exibição (em segundos) |
| `data.stopwatch_show` | _Boolean_ | Se um cronômetro está em exibição `v2.20.0+` |
| `data.stopwatch_time` | _Number_ | O tempo atual do cronômetro em exibição (em segundos) `v2.20.0+` |
| `data.theme` | _Number_ | ID do tema `v2.20.0+` |
| `data.countdown_font_relative_size` | _Number_ | Tamanho relativo da contagem regressiva `v2.20.0+` |
| `data.countdown_font_color` | _String_ | Cor da fonte da contagem regressiva `v2.20.0+` |
| `data.stopwatch_font_color` | _String_ | Cor da fonte do cronômetro `v2.20.0+` |
| `data.time_font_color` | _String_ | Cor da fonte da hora `v2.20.0+` |
| `data.display_clock_as_background` | _Boolean_ | Exibir relógio como plano de fundo `v2.20.0+` |
| `data.display_clock_on_alert` | _Boolean_ | Exibir relógio no alerta `v2.20.0+` |
| `data.countdown_display_location` | _String_ | Local de exibição da contagem regressiva ou cronômetro. `FULLSCREEN`  `FULLSCREEN_OR_ALERT`  `ALERT` `v2.20.0+` |
| `data.display_clock_with_countdown_fullscreen` | _Boolean_ | Exibir relógio junto da contagem regressiva ou cronômetro quando exibido em tela cheia `v2.20.0+` |
| `data.display_vlc_player_remaining_time` | _Boolean_ | Exibir tempo restante da mídia em execução no VLC Player `v2.20.0+` |


**Exemplo:**
```
Resposta
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

Alterar configuração atual do painel de comunicação

**Parâmetros:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `text` | _String (opcional)_ | Texto atual |
| `show` | _Boolean (opcional)_ | Exibir o texto atual |
| `display_ahead` | _Boolean (opcional)_ | Opção *'exibir à frente de tudo'* |
| `theme` | _Object (opcional)_ | ID ou nome do tema padrão |
| `theme.id` | _String (opcional)_ |  |
| `theme.name` | _String (opcional)_ |  |
| `custom_theme` | _[Theme](#theme) (opcional)_ | Tema personalizado `v2.21.0+` |
| `alert_text` | _String (opcional)_ | Texto atual do alerta |
| `alert_show` | _Boolean (opcional)_ | Ativar a exibição do alerta |
| `countdown_font_relative_size` | _Number (opcional)_ | Tamanho relativo da contagem regressiva |
| `countdown_font_color` | _String (opcional)_ | Cor da fonte da contagem regressiva |
| `stopwatch_font_color` | _String (opcional)_ | Cor da fonte do cronômetro |
| `time_font_color` | _String (opcional)_ | Cor da fonte da hora |
| `display_clock_as_background` | _Boolean (opcional)_ | Exibir relógio como plano de fundo |
| `display_clock_on_alert` | _Boolean (opcional)_ | Exibir relógio no alerta |
| `countdown_display_location` | _String (opcional)_ | Local de exibição da contagem regressiva ou cronômetro. `FULLSCREEN`  `FULLSCREEN_OR_ALERT`  `ALERT` |
| `display_clock_with_countdown_fullscreen` | _Boolean (opcional)_ | Exibir relógio junto da contagem regressiva ou cronômetro quando exibido em tela cheia |
| `display_vlc_player_remaining_time` | _Boolean (opcional)_ | Exibir tempo restante da mídia em execução no VLC Player |


_Método sem retorno_

**Exemplo:**
```
Requisição
{
  "display_clock_as_background": false,
  "display_clock_on_alert": true
}
```


---

### StartCountdownCommunicationPanel
### StartCountdownCP
- v2.19.0

Inicia uma contagem regressiva no painel de comunicação

**Parâmetros:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `minutes` | _Number_ | Quantidade de minutos |
| `seconds` | _Number_ | Quantidade de segundos |
| `yellow_starts_at` | _Number (opcional)_ | Valor em segundos para definir a partir de quanto tempo a contagem regressiva ficará amarela |
| `stop_at_zero` | _Boolean (opcional)_ | Parar a contagem regressiva ao chegar em zero `Padrão: false` |


_Método sem retorno_

**Exemplo:**
```
Requisição
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

Encerra a contagem regressiva atual do painel de comunicação



_Método sem retorno_



---

### StartTimerCommunicationPanel
### StartTimerCP
- v2.20.0

Inicia um cronômetro no painel de comunicação



_Método sem retorno_



---

### StopTimerCommunicationPanel
### StopTimerCP
- v2.20.0

Encerra o cronômetro atual do painel de comunicação



_Método sem retorno_



---

### SetTextCommunicationPanel
### SetTextCP
- v2.19.0

Alterar o texto do painel de comunicação

**Parâmetros:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `text` | _String (opcional)_ | Alterar o texto do painel de comunicação. [Styled Text](#styled-text) a partir da v2.19.0 |
| `show` | _Boolean (opcional)_ | Exibir/ocultar o texto |
| `display_ahead` | _Boolean (opcional)_ | Alterar a opção *'exibir à frente de tudo'* |
| `theme` | _Object (opcional)_ | ID ou nome do Tema utilizado para exibir o texto `v2.21.0+` |
| `custom_theme` | _[Theme](#theme) (opcional)_ | Tema personalizado para exibir o texto `v2.21.0+` |


_Método sem retorno_

**Exemplo:**
```
Requisição
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

Alterar as configurações de alerta do painel de comunicação

**Parâmetros:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `text` | _String (opcional)_ | Alterar o texto de alerta |
| `show` | _Boolean (opcional)_ | Exibir/ocultar o alerta |


_Método sem retorno_

**Exemplo:**
```
Requisição
{
  "text": "",
  "show": false
}
```


---

### CommunicationPanelCallAttention
### CPCallAttention
- v2.20.0

Executa a opção 'chamar atenção' disponível no painel de comunicação



_Método sem retorno_



---

### GetWallpaperSettings
- v2.19.0

Configurações do papel de parede



**Resposta:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `data.image_base64` | _String_ | Imagem do papel de parede em base 64 |
| `data.enabled` | _Boolean_ | Exibir papel de parede |
| `data.fill_color` | _String_ | Cor em hexadecimal definida na opção **preencher**. |
| `data.extend` | _Boolean_ | `deprecated` Substituído por `adjust_type`<br>Estender papel de parede |
| `data.adjust_type` | _String_ | Ajuste da imagem: Pode ser: `ADJUST` `EXTEND` `FILL` `ADJUST_BLUR` `v2.22.0+` |
| `data.show_clock` | _Boolean_ | Exibir relógio |


**Exemplo:**
```
Resposta
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

Alterar as configurações do papel de parede

**Parâmetros:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `file` | _String (opcional)_ | Local do arquivo na aba **Imagens** |
| `enabled` | _Boolean (opcional)_ | Exibir papel de parede |
| `fill_color` | _String (opcional)_ | Cor em hexadecimal definida na opção **preencher**. **NULL** para desativar |
| `extend` | _Boolean (opcional)_ | `deprecated` Substituído por `adjust_type`<br>Estender papel de parede |
| `data.adjust_type` | _String_ | Ajuste da imagem: Pode ser: `ADJUST` `EXTEND` `FILL` `ADJUST_BLUR` `v2.22.0+` |
| `show_clock` | _Boolean (opcional)_ | Exibir relógio |


**Resposta:**

| Tipo  | Descrição |
| :---: | ------------|
| _Object_ | Retorna **true** ou uma lista com os erros ocorridos |


**Exemplo:**
```
Requisição
{
  "file": "wallpapers/image.jpg",
  "enabled": true,
  "fill_color": "#000000",
  "extend": true,
  "show_clock": false
}

Resposta
{
  "status": "ok",
  "data": true
}
```


---

### GetDisplaySettings
- v2.19.0

Lista das configurações de exibição de cada tela



**Resposta:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `data` | _Array&lt;[DisplaySettings](#display-settings)&gt;_ |  |


**Exemplo:**
```
Resposta
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

Alterar as configurações de exibição de uma tela

**Parâmetros:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `input` | _[DisplaySettings](#display-settings)_ | Novas configurações. As configurações são individualmente opcionais. Preencha apenas os campos que deseja alterar. |


**Resposta:**

| Tipo  | Descrição |
| :---: | ------------|
| _Object_ | Retorna **true** ou uma lista com os erros ocorridos |


**Exemplo:**
```
Requisição
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

Resposta
{
  "status": "ok",
  "data": true
}
```


---

### GetTransitionEffectSettings
- v2.21.0

Lista da configuração dos efeitos de transição



**Resposta:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `music` | _[TransitionEffectSettings](#transition-effect-settings)_ |  |
| `bible` | _[TransitionEffectSettings](#transition-effect-settings)_ |  |
| `image` | _[TransitionEffectSettings](#transition-effect-settings)_ |  |
| `announcement` | _[TransitionEffectSettings](#transition-effect-settings)_ |  |


**Exemplo:**
```
Resposta
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

Alterar as configurações de um efeito de transição

**Parâmetros:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `id` | _Object_ | ID do item |
| `settings` | _[TransitionEffectSettings](#transition-effect-settings)_ | Novas configurações. As configurações são individualmente opcionais. Preencha apenas os campos que deseja alterar. |


**Resposta:**

| Tipo  | Descrição |
| :---: | ------------|
| _Object_ | Retorna **true** ou uma lista com os erros ocorridos |


**Exemplo:**
```
Requisição
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

Resposta
{
  "status": "ok",
  "data": true
}
```


---

### GetBibleVersions
- v2.21.0

Retorna a lista de versões disponíveis da Bíblia, e também dos atalhos associados



**Resposta:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `data` | _Object_ |  |
| `data.*.key` | _String_ | Abreviação da versão ou o nome do atalho, se começar com '#shortcut ' |
| `data.*.title` | _String_ | Nome da versão |
| `data.*.version` | _String (opcional)_ | Abreviação da versão. Disponível se o item for um atalho, ou seja se 'key' começar com '#shortcut ' |


**Exemplo:**
```
Resposta
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

### GetBibleSettings
- v2.21.0

Configurações do módulo Bíblia



**Resposta:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `data` | _[BibleSettings](#bible-settings)_ |  |


**Exemplo:**
```
Resposta
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

Alterar as configurações do módulo Bíblia

**Parâmetros:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `input` | _[BibleSettings](#bible-settings)_ | Novas configurações. As configurações são individualmente opcionais. Preencha apenas os campos que deseja alterar. |


**Resposta:**

| Tipo  | Descrição |
| :---: | ------------|
| _Object_ | Retorna **true** ou uma lista com os erros ocorridos |


**Exemplo:**
```
Requisição
{
  "tab_version_1": "pt_acf",
  "show_x_verses": 1,
  "theme": {
    "public": "123"
  }
}

Resposta
{
  "status": "ok",
  "data": true
}
```


---

### GetBpm
- v2.19.0

Retorna o valor BPM atual definido no programa



**Resposta:**

| Tipo  | Descrição |
| :---: | ------------|
| _Number_ | Valor BPM atual |


**Exemplo:**
```
Resposta
{
  "status": "ok",
  "data": 80
}
```


---

### SetBpm
- v2.19.0

Altera o valor BPM atual do programa

**Parâmetros:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `bpm` | _Number_ | Valor BPM |


_Método sem retorno_

**Exemplo:**
```
Requisição
{
  "bpm": 80
}
```


---

### GetHue
- v2.19.0

Retorna o valor matiz atual definido no programa



**Resposta:**

| Tipo  | Descrição |
| :---: | ------------|
| _Number_ | Valor matiz atual. Mínimo=0, Máximo=360. Retorna **null** se desativado. |


**Exemplo:**
```
Resposta
{
  "status": "ok",
  "data": 120
}
```


---

### SetHue
- v2.19.0

Altera o valor matiz atual do programa

**Parâmetros:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `hue` | _Number_ | Valor matiz. Mínimo=0, Máximo=360 ou **null** para desativar. |


_Método sem retorno_

**Exemplo:**
```
Requisição
{
  "hue": null
}
```


---

### GetRuntimeEnvironment
### GetRE
- v2.19.0

Retorna o nome do ambiente de execução definido atualmente nas configurações do programa.



**Resposta:**

| Tipo  | Descrição |
| :---: | ------------|
| _String_ | Nome do ambiente de execução |


**Exemplo:**
```
Resposta
{
  "status": "ok",
  "data": "runtime environment name"
}
```


---

### SetRuntimeEnvironment
### SetRE
- v2.19.0

Altera o ambiente de execução atual.

**Parâmetros:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `name` | _String_ | Nome do ambiente de execução |


_Método sem retorno_

**Exemplo:**
```
Requisição
{
  "name": "runtime environment name"
}
```


---

### SetLogo
- v2.19.0

Alterar as configurações da funcionalidade *Logo* do programa (menu ferramentas)

**Parâmetros:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `enable` | _Boolean (opcional)_ | Ativar/desativar a funcionalidade |
| `hide` | _Boolean (opcional)_ | Exibir/ocultar a funcionalidade |


_Método sem retorno_

**Exemplo:**
```
Requisição
{
  "enable": true,
  "hide": true
}
```


---

### GetSyncStatus
- v2.19.0

Retorna o estado atual da sincronização online via Google Drive™



**Resposta:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `data.enabled` | _Boolean_ | Se a sincronização está ativada |
| `data.started` | _Boolean_ | Se a sincronização foi iniciada (internet disponível, por exemplo) |
| `data.progress` | _Number_ | Progresso da sincronização de 0 a 100 |


**Exemplo:**
```
Resposta
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

Retorna o valor de um campo da interface do programa

**Parâmetros:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `id` | _String_ | ID do item. Pode ser: <br>`main_lyrics_tab_search`<br>`main_text_tab_search`<br>`main_audio_tab_search`<br>`main_video_tab_search`<br>`main_image_tab_search`<br>`main_file_tab_search`<br>`main_automatic_presentation_tab_search`<br>`main_selected_theme` |


**Resposta:**

| Tipo  | Descrição |
| :---: | ------------|
| _String_ | Conteúdo do item |


**Exemplo:**
```
Requisição
{
  "id": "main_lyrics_tab_search"
}

Resposta
{
  "status": "ok",
  "data": "input value"
}
```


---

### SetInterfaceInput
- v2.21.0

Altera o valor de um campo da interface do programa

**Parâmetros:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `id` | _String_ | ID do item |
| `value` | _String_ | Novo valor |
| `focus` | _Boolean (opcional)_ | Fazer o componente receber o foco do sistema |


_Método sem retorno_

**Exemplo:**
```
Requisição
{
  "id": "main_lyrics_tab_search",
  "value": "new input value",
  "focus": true
}
```


---

### OpenDrawLots
- v2.21.0

Abre a janela de sorteio a partir de uma lista de itens

**Parâmetros:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `items` | _Array&lt;String&gt;_ | Lista com os itens para serem sorteados |


_Método sem retorno_

**Exemplo:**
```
Requisição
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

Retorna a duração da mídia

**Parâmetros:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `type` | _String_ | Tipo do item. Pode ser: `video`, `audio`, `automatic_presentation` |
| `name` | _String_ | Nome do item |


**Resposta:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `data.type` | _String_ |  |
| `data.name` | _String_ |  |
| `data.duration` | _Number_ | Duração em segundos |
| `data.duration_ms` | _Number_ | Duração em milissegundos |


**Exemplo:**
```
Requisição
{
  "type": "audio",
  "name": "file.mp3"
}

Resposta
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

Retorna informações da versão do programa em execução



**Resposta:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `data.version` | _String_ | Versão do programa |
| `data.platform` | _String_ | Sistema operacional. Pode ser: `win` `uni` `osx` |
| `data.platformDescription` | _String_ | Nome detalhado do sistema operacional |


**Exemplo:**
```
Resposta
{
  "status": "ok",
  "data": {
    "data": {
      "version": "2.22.0",
      "platform": "win",
      "platformDescription": "Windows 10"
    }
  }
}
```


---



# Exemplo de implementação JavaScript

Utilize os métodos da classe [JSLIB](https://github.com/holyrics/jslib) para criar sua própria implementação.
<br/>
Se você criar sua própria implementação, é necessário retornar 'true' se quiser que o programa consuma a requisição utilizando a implementação padrão.
<br/>
Qualquer valor retornado que seja diferente de 'true', o programa irá considerar que a requisição já foi consumida e vai respondê-la com o valor retornado.

```javascript
function request(action, headers, content, info) {
    switch (action) {
        case 'my_custom_action':
            //executar sua própria ação e resposta
            return {'status': 'ok'}; //  <-  Resposta da requisição
    }
    //Retornando 'true' para o programa consumir esta requisição com a implementação padrão
    return true;
}

```

### Parâmetros do método

`action` - Nome da ação

`headers` - Contém os cabeçalhos da requisição. Exemplo: `headers.Authorization`

`content` - Objeto com o conteúdo extraído da requisição. Exemplo: `content.theme.id`

`info` - Informações da requisição.
<br/>
`info.client_address` Endereço da origem da requisição
<br/>
`info.token` token de acesso utilizado na requisição
<br/>
`info.local` **true** se a origem da requisição for da rede local
<br/>
`info.web` **true** se a origem da requisição for da internet
<br/>

# Classes

## Lyrics
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `id` | _String_ | ID da música |
| `title` | _String_ | Título da música |
| `artist` | _String_ | Artista da música |
| `author` | _String_ | Autor da música |
| `note` | _String_ | Anotação da música |
| `copyright` | _String_ | Copyright da música |
| `slides` | _Array&lt;Object&gt;_ |  `v2.21.0+` |
| `slides.*.text` | _String_ | Texto do slide `v2.21.0+` |
| `slides.*.slide_description` | _Number_ | Descrição do slide `v2.21.1+` |
| `slides.*.background_id` | _Number_ | ID do tema ou plano de fundo salvo para o slide `v2.21.0+` |
| `order` | _String_ | Ordem dos slides (índice a partir do 1), separado por vírgula `v2.21.0+` |
| `key` | _String_ | Tom da música.<br>Pode ser: `C` `C#` `Db` `D` `D#` `Eb` `E` `F` `F#` `Gb` `G` `G#` `Ab` `A` `A#` `Bb` `B` `Cm` `C#m` `Dbm` `Dm` `D#m` `Ebm` `Em` `Fm` `F#m` `Gbm` `Gm` `G#m` `Abm` `Am` `A#m` `Bbm` `Bm` |
| `bpm` | _Number_ | BPM da música |
| `time_sig` | _String_ | Tempo da música.<br>Pode ser: `2/2` `2/4` `3/4` `4/4` `5/4` `6/4` `3/8` `6/8` `7/8` `9/8` `12/8` |
| `groups` | _Array&lt;[Group](#group)&gt;_ | Grupos onde a música está adicionada |
| `linked_audio_file` | _String_ | Caminho do arquivo de áudio linkado com a música `v2.22.0+` |
| `linked_backing_track_file` | _String_ | Caminho do arquivo de áudio (playback) linkado com a música `v2.22.0+` |
| `streaming` | _Object_ | URI ou ID dos streamings `v2.22.0+` |
| `streaming.audio` | _Object_ | Áudio `v2.22.0+` |
| `streaming.audio.spotify` | _String_ |  `v2.22.0+` |
| `streaming.audio.youtube` | _String_ |  `v2.22.0+` |
| `streaming.audio.deezer` | _String_ |  `v2.22.0+` |
| `streaming.backing_track` | _Object_ | Playback `v2.22.0+` |
| `streaming.backing_track.spotify` | _String_ |  `v2.22.0+` |
| `streaming.backing_track.youtube` | _String_ |  `v2.22.0+` |
| `streaming.backing_track.deezer` | _String_ |  `v2.22.0+` |
| `midi` | _[Midi](#midi)_ | Atalho MIDI do item |
| `extras` | _Object_ | Mapa de objetos extras (adicionados pelo usuário) `v2.21.0+` |
| `archived` | _Boolean_ | Se a música está arquivada |
<details>
  <summary>Ver exemplo</summary>

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
      "slide_description": "Verse 1",
      "background_id": null
    },
    {
      "text": "Slide 2 line 1\nSlide 2 line 2",
      "slide_description": "Chorus",
      "background_id": null
    },
    {
      "text": "Slide 3 line 1\nSlide 3 line 2",
      "slide_description": "Verse 3",
      "background_id": null
    }
  ],
  "order": "1,2,3,2,2",
  "key": "",
  "bpm": 0.0,
  "time_sig": "",
  "groups": [],
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
  "archived": false
}
```
</details>

## Text
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `id` | _String_ | ID do texto |
| `title` | _String_ | Título do texto |
| `folder` | _String_ | Caminho da pasta de localização |
| `theme` | _String_ | ID do tema salvo para o texto |
| `slides` | _Array&lt;Object&gt;_ |  |
| `slides.*.text` | _String_ | Texto do slide |
| `slides.*.background_id` | _Number_ | ID do tema ou plano de fundo salvo para o slide |
<details>
  <summary>Ver exemplo</summary>

```json
{
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
  ]
}
```
</details>

## Theme
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `copy_from_id` | _String (opcional)_ | ID de um Tema existente para utilizar como cópia inicial ao criar um novo item |
| `id` | _String_ | ID do item |
| `name` | _String_ | Nome do item |
| <br>**background** |  | <br>Plano de fundo |
| `background.type` | _String_ | Tipo do plano de fundo. Pode ser: `color`  `my_video`  `my_image`  `video`  `image`  `pattern`  `transparent`  `image_file`  `video_file` |
| `background.id` | _String_ | <table><tr><td><p align="right">**Tipo**</p></td><td>Valor</td></tr><tr><td><p align="right">color</p></td><td>Cor no formato hexadecimal</td></tr><tr><td><p align="right">my_video</p></td><td>ID do item</td></tr><tr><td><p align="right">my_image</p></td><td>ID do item</td></tr><tr><td><p align="right">video</p></td><td>ID do item</td></tr><tr><td><p align="right">image</p></td><td>ID do item</td></tr><tr><td><p align="right">pattern</p></td><td>ID do item</td></tr><tr><td><p align="right">transparent</p></td><td>"transparent"</td></tr><tr><td><p align="right">image_file</p></td><td>Nome do arquivo na biblioteca</td></tr><tr><td><p align="right">video_file</p></td><td>Nome do arquivo na biblioteca</td></tr></table> |
| `background.adjust_type` | _String_ | `fill` `extend` `adjust` `side_by_side` `center`<br>Disponível para: **type=my_image**, **type=image** |
| `background.opacity` | _Number_ | Opacidade. `0 ~ 100` |
| `background.velocity` | _Number_ | Disponível para: **type=my_video**, **type=video**<br>`0.25 ~ 4.0` |
| `base_color` | _String_ | Cor no formato hexadecimal. Cor base do plano de fundo ao diminuir a opacidade. |
| <br>**font** |  | <br>Fonte |
| `font.name` | _String_ | Nome da fonte |
| `font.bold` | _Boolean_ | Negrito |
| `font.italic` | _Boolean_ | Itálico |
| `font.size` | _Number_ | Tamanho `0.0 ~ 0.4`<br>Valor em porcentagem, baseado na altura do slide. |
| `font.color` | _String_ | Cor no formato hexadecimal |
| `font.line_spacing` | _Number_ | Espaçamento entre linhas. `-0.5 ~ 1.0`<br>Valor em porcentagem baseado na altura da linha. |
| `font.char_spacing` | _Number_ | Espaçamento entre caracteres. `-40 ~ 120` |
| <br>**align** |  | <br>Alinhamento |
| `align.horizontal` | _String_ | `left`  `center`  `right`  `justify` |
| `align.vertical` | _String_ | `top`  `middle`  `bottom` |
| `align.margin.top` | _Number_ | `0 ~ 90` |
| `align.margin.right` | _Number_ | `0 ~ 90` |
| `align.margin.bottom` | _Number_ | `0 ~ 90` |
| `align.margin.left` | _Number_ | `0 ~ 90` |
| <br>**effect** |  | <br>Efeitos da fonte |
| `effect.outline_color` | _String_ | Cor no formato hexadecimal |
| `effect.outline_weight` | _Number_ | `0.0 ~ 100.0` |
| `effect.brightness_color` | _String_ | Cor no formato hexadecimal |
| `effect.brightness_weight` | _Number_ | `0.0 ~ 100.0` |
| `effect.shadow_color` | _String_ | Cor no formato hexadecimal |
| `effect.shadow_x_weight` | _Number_ | `-100.0 ~ 100.0` |
| `effect.shadow_y_weight` | _Number_ | `-100.0 ~ 100.0` |
| `effect.blur` | _Boolean_ | Aplicar efeito 'blur' no brilho e sombra |
| <br>**shape_fill** |  | <br>Cor de fundo |
| `shape_fill.type` | _String_ | `box`  `line`  `line_fill`  `theme_margin` |
| `shape_fill.enabled` | _Boolean_ |  |
| `shape_fill.color` | _String_ | Cor no formato hexadecimal (RGBA) |
| `shape_fill.margin.top` | _Number_ | `0 ~ 100` |
| `shape_fill.margin.right` | _Number_ | `0 ~ 100` |
| `shape_fill.margin.bottom` | _Number_ | `0 ~ 100` |
| `shape_fill.margin.left` | _Number_ | `0 ~ 100` |
| `shape_fill.corner` | _Number_ | `0 ~ 100` |
| <br>**shape_outline** |  | <br>Contorno |
| `shape_outline.type` | _String_ | `box`  `line`  `line_fill`  `theme_margin` |
| `shape_outline.enabled` | _Boolean_ |  |
| `shape_outline.color` | _String_ | Cor no formato hexadecimal (RGBA) |
| `shape_outline.outline_thickness` | _Number_ | `0 ~ 25` |
| `shape_outline.margin.top` | _Number_ | `0 ~ 100` |
| `shape_outline.margin.right` | _Number_ | `0 ~ 100` |
| `shape_outline.margin.bottom` | _Number_ | `0 ~ 100` |
| `shape_outline.margin.left` | _Number_ | `0 ~ 100` |
| `shape_outline.corner` | _Number_ | `0 ~ 100` |
| <br>**comment** |  | <br>Comentário |
| `comment.font_name` | _String_ | Nome da fonte |
| `comment.bold` | _Boolean_ | Negrito |
| `comment.italic` | _Boolean_ | Itálico |
| `comment.relative_size` | _Number_ | tamanho relativo da fonte. `40 ~ 100` |
| `comment.color` | _String_ | Cor no formato hexadecimal |
| <br>**settings** |  | <br>Configurações |
| `settings.uppercase` | _Boolean_ | Exibir o texto em maiúsculo |
| `settings.line_break` | _String_ | Aplicar quebra de linha. `system`  `true`  `false`<br> `Padrão: system` |
<details>
  <summary>Ver exemplo</summary>

```json
{
  "id": "123",
  "name": "",
  "background": {
    "type": "color", "id": "212121", "opacity": 100
  },
  "base_color": "FFFFFF",
  "font": {
    "name": "CMG Sans", "bold": true,
    "italic": false,
    "size": 10.0,
    "color": "F5F5F5", "line_spacing": 0.3,
    "char_spacing": 0
  },
  "align": {
    "horizontal": "center", "vertical": "middle", "margin": {
      "top": 3.0,
      "right": 3.0,
      "bottom": 3.0,
      "left": 3.0
    }
  },
  "effect": {
    "outline_color": "404040", "outline_weight": 0.0,
    "brightness_color": "C0C0C0", "brightness_weight": 0.0,
    "shadow_color": "404040", "shadow_x_weight": 0.0,
    "shadow_y_weight": 0.0,
    "blur": true
  },
  "shape_fill": {
    "type": "box", "enabled": false,
    "color": "000000", "margin": {
      "top": 5.0,
      "right": 30.0,
      "bottom": 10.0,
      "left": 30.0
    },
    "corner": 0
  },
  "shape_outline": {
    "type": "box", "enabled": false,
    "color": "000000", "outline_thickness": 10,
    "margin": {
      "top": 5.0,
      "right": 30.0,
      "bottom": 10.0,
      "left": 30.0
    },
    "corner": 0
  },
  "comment": {
    "font_name": "Arial", "bold": false,
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
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `id` | _String_ | ID do item |
| `type` | _String_ | Tipo do item. Pode ser: `theme` `my_video` `my_image` `video` `image` |
| `name` | _String_ | Nome do item |
| `tags` | _Array&lt;String&gt;_ | Lista de tags do item |
| `bpm` | _Number_ | Valor BPM do item |
| `midi` | _[Midi](#midi) (opcional)_ | Atalho MIDI do item |
<details>
  <summary>Ver exemplo</summary>

```json
{
  "id": "10",
  "type": "video",
  "name": "Hexagons",
  "tags": [],
  "bpm": 0.0
}
```
</details>

## Slide Description
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `name` | _String_ | Nome do item |
| `tag` | _String_ | Nome curto do item |
| `aliases` | _Array&lt;String&gt;_ | Lista com os nomes alternativos |
| `font_color` | _String_ | Cor da fonte no formato hexadecimal |
| `bg_color` | _String_ | Cor de fundo no formato hexadecimal |
| `background` | _Number_ | ID do plano de fundo personalizado |
| `midi` | _[Midi](#midi) (opcional)_ | Atalho MIDI do item |
<details>
  <summary>Ver exemplo</summary>

```json
{
  "name": "Chorus",
  "tag": "C",
  "aliases": [],
  "font_color": "FFFFFF",
  "bg_color": "000080",
  "background": null,
  "midi": null
}
```
</details>

## Item
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `id` | _String_ | ID do item |
| `type` | _String_ | Tipo do item. Pode ser: `title`  `song`  `verse`  `text`  `audio`  `video`  `image`  `file`  `announcement`  `automatic_presentation`  `countdown`  `countdown_cp`  `cp_text`  `plain_text`  `uri`  `global_action`  `api`  `script` |
| `name` | _String_ | Nome do item |

## Group
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `name` | _String_ | Nome do item |
| `songs` | _Array&lt;Number&gt;_ | Lista dos IDs das músicas |

## Announcement
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `id` | _String_ | ID do item |
| `name` | _String_ | Nome do item |
| `text` | _String_ | Texto do anúncio |
| `archived` | _Boolean_ | Se o item está arquivado |

## Midi
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `code` | _Number_ | Código midi |
| `velocity` | _Number_ | Velocidade/intensidade midi |
<details>
  <summary>Ver exemplo</summary>

```json
{
  "code": 80,
  "velocity": 20
}
```
</details>

## Favorite Item
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `id` | _String_ | ID do item |
| `name` | _String_ | Nome do item |

## Service
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `name` | _String_ | Nome do item |
| `week` | _String_ | Semana. Pode ser: `all` `first` `second` `third` `fourth` `last` |
| `day` | _String_ | Dia da semana. Pode ser: `sun` `mon` `tue` `wed` `thu` `fri` `sat` |
| `hour` | _Number_ | Hora [0-23] |
| `minute` | _Number_ | Minuto [0-59] |
| `type` | _String_ | Tipo do item. Pode ser: `service` `event` |
| `hide_week` | _Array&lt;String&gt;_ | Lista com as semanas ocultadas. Disponível se `week=all` |

## Event
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `name` | _String_ | Nome do evento |
| `datetime` | _String_ | Data e hora no formato: YYYY-MM-DD HH:MM |
| `wallpaper` | _String_ | Caminho relativo do arquivo utilizado como papel de parede do evento |

## Schedule
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `type` | _String_ | Tipo da lista de reprodução. Pode ser: temporary, service, event |
| `name` | _String_ |  |
| `datetime` | _String_ | Data e hora no formato: YYYY-MM-DD HH:MM |
| `lyrics_playlist` | _Array&lt;[Lyrics](#lyrics)&gt;_ | Lista de letras |
| `media_playlist` | _Array&lt;[Item](#item)&gt;_ | Lista de mídias |
| `responsible` | _[Member](#member)_ | Integrante definido como responsável pelo evento |
| `members` | _Array&lt;Object&gt;_ | Lista de integrantes |
| `members.*.id` | _String_ | ID do integrante |
| `members.*.name` | _String_ | Nome do integrante escalado |
| `members.*.scheduled` | _Boolean_ | Se o integrande está escalado ou definido para uma função |
| `roles` | _Array&lt;Object&gt;_ | Lista das funções na escala |
| `roles.*.id` | _String_ | ID da função |
| `roles.*.name` | _String_ | Nome da função |
| `roles.*.member` | _[Member](#member)_ | Integrante escalado para a função |
| `notes` | _String_ | Anotações `v2.21.0+` |
<details>
  <summary>Ver exemplo</summary>

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
  "members": [],
  "roles": [],
  "notes": ""
}
```
</details>

## Team
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `id` | _String_ | ID do item |
| `name` | _String_ | Nome do item |
| `description` | _String_ | Descrição do item |

## Member
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `id` | _String_ | ID do item |
| `name` | _String_ | Nome do item |
| `skills` | _String_ | Habilidades |
| `roles` | _Array&lt;[Role](#role)&gt;_ | Funções |

## Role
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `id` | _String_ | ID do item |
| `name` | _String_ | Nome do item |
| `team` | _[Team](#team)_ | Time |

## Automatic Presentation
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `seconds` | _Number_ | Tempo que cada item ficará sendo apresentado |
| `repeat` | _Boolean_ | **true** para ficar repetindo a apresentação (voltar para o primeiro item após o último) |

## Presentation Slide Info
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `number` | _Number_ | Número do slide (começa em 1) |
| `text` | _String_ | Texto do slide |
| `theme_id` | _String_ | ID do tema do slide |
| `slide_description` | _String (opcional)_ | Nome da descrição do slide. Disponível se for uma apresentação de música. |
| `preview` | _String (opcional)_ | Imagem no formato base64 |

## Input Param
Utiliza a mesma estrutura/sintaxe da funcionalidade FunctionInput [documentação](https://github.com/holyrics/Scripts/blob/main/FunctionInput.md#syntax)


## Trigger Item
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `id` | _String (opcional)_ | ID do item |
| `when` | _String_ | `displaying` `closing` `change` |
| `item` | _String_ | Tipo do item. Pode ser:<br>**when=displaying**: `any_song` `any_text` `any_verse` `any_announcement` `any_audio` `any_video` `any_image` `any_automatic_presentation` `any_song_slide` `any_text_slide` `any_ppt_slide` `any_theme` `any_background` `any_title_subitem` `any_webcam` `any_countdown` `any_automatic_presentation_slide` `f8` `f9` `f10`<br><br>**when=closing**: `any_song` `any_text` `any_verse` `any_announcement` `any_audio` `any_video` `any_image` `any_automatic_presentation` `any_webcam` `f8` `f9` `f10`<br><br>**when=change**: `countdown_seconds_public` `countdown_seconds_communication_panel` `timer_seconds_communication_panel` `wallpaper` `wallpaper_service` `stage` `playlist` `bpm` `hue` |
| `action` | _Function_ | Ação que será executada |
<details>
  <summary>Ver exemplo</summary>

```javascript
{
  "id": "",
  "when": "displaying",
  "item": "any_song",
  "action": function(obj) { /* TODO */ }
}
```
</details>

## Play Media Settings
Configurações para execução da mídia

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `volume` | _Number_ | Altera o volume do player |
| `repeat` | _Boolean_ | Altera a opção **repetir** |
| `shuffle` | _Boolean_ | Altera a opção **aleatório** |
| `start_time` | _String_ | Tempo inicial para execução no formato SS, MM:SS ou HH:MM:SS |
| `stop_time` | _String_ | Tempo final para execução no formato SS, MM:SS ou HH:MM:SS |
<details>
  <summary>Ver exemplo</summary>

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
Configurações de exibição

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `id` | _String_ | ID do item. `public` `screen_2` `screen_3` `screen_?` `stream_image` `stream_html_1` `stream_html_2` `stream_html_3` |
| `name` | _String_ | Nome do item |
| `stage_view` | _[StageView](#stage-view)_ | Configurações da visão do palco. (Indisponível para tela público) |
| `slide_info` | _[SlideAdditionalInfo](#slide-additional-info)_ | Informações adicionais do slide |
| `slide_translation` | _String_ | Nome da tradução |
| `bible_version_tab` | _Number_ | Número da aba (1, 2 ou 3) da tradução da Bíblia exibida na tela, conforme traduções carregadas na janela da Bíblia |
| `margin` | _Object_ | Margens definidas na opção **Editar posição da tela**. margin.top, margin.right, margin.bottom, margin.left |
| `area` | _[Rectangle](#rectangle)_ | Área da tela com as margens aplicadas (se disponível) |
| `total_area` | _[Rectangle](#rectangle)_ | Área total da tela no sistema |
| `hide` | _Boolean_ | Ocultar a tela |
| `show_items` | _Object_ | Define os tipos de apresentação que serão exibidos (disponível apenas para telas de transmissão - imagem e html) |
| `show_items.lyrics` | _Boolean_ | Letra de música |
| `show_items.text` | _Boolean_ | Texto |
| `show_items.verse` | _Boolean_ | Versículo |
| `show_items.image` | _Boolean_ | Imagem |
| `show_items.alert` | _Boolean_ | Alerta |
| `show_items.announcement` | _Boolean_ | Anúncio |
| `media_player.show` | _Boolean_ | Exibir VLC Player `v2.20.0+` |
| `media_player.margin` | _[Rectangle](#rectangle)_ | Margem para exibição dos vídeos pelo VLC Player `v2.20.0+` |
<details>
  <summary>Ver exemplo</summary>

```json
{
  "id": "public",
  "name": "Público",
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
  "slide_translation": null,
  "margin": {
    "top": 0.0,
    "right": 0.0,
    "bottom": 0.0,
    "left": 0.0
  },
  "area": {
    "x": 0,
    "y": 0,
    "width": 0,
    "height": 0
  },
  "total_area": {
    "x": 0,
    "y": 0,
    "width": 0,
    "height": 0
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
      "x": 0,
      "y": 0,
      "width": 0,
      "height": 0
    }
  }
}
```
</details>

## Transition Effect Settings
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `type` | _String_ | Tipo de efeito. Pode ser: `random` `fade` `slide` `accordion` `linear_fade` `zoom` `curtain` |
| `enabled` | _Boolean_ | Se está ativado ou desativado |
| `duration` | _Number_ | Duração total da transição (em milissegundos) `200 ~ 2400` |
| `only_area_within_margin` | _Number_ | Realiza o efeito de transição apenas dentro da margem definida no Tema. (Disponível somente para transição de texto) |
| <br>**type=fade** |  |  |
| `merge` | _Object_ | Valores aceitos: true,&nbsp;false |
| `division_point` | _Object_ | Valores aceitos: min:&nbsp;10,&nbsp;max:&nbsp;100 |
| `increase_duration_blank_slides` | _Object_ | Valores aceitos: true,&nbsp;false |
| <br>**type=slide** |  |  |
| `direction` | _Object_ | Valores aceitos: random,&nbsp;left,&nbsp;up |
| `slide_move_type` | _Object_ | Valores aceitos: random,&nbsp;move_new,&nbsp;move_old,&nbsp;move_both |
| <br>**type=accordion** |  |  |
| `direction` | _Object_ | Valores aceitos: random,&nbsp;horizontal,&nbsp;vertical |
| <br>**type=linear_fade** |  |  |
| `direction` | _Object_ | Valores aceitos: random,&nbsp;horizontal,&nbsp;vertical,&nbsp;up,&nbsp;down,&nbsp;left,&nbsp;right |
| `distance` | _Object_ | Valores aceitos: min:&nbsp;5,&nbsp;max:&nbsp;90 |
| `fade` | _Object_ | Valores aceitos: min:&nbsp;2,&nbsp;max:&nbsp;90 |
| <br>**type=zoom** |  |  |
| `zoom_type` | _Object_ | Valores aceitos: random,&nbsp;increase,&nbsp;decrease |
| `directions` | _Object_ | Valores aceitos: {<br>&nbsp;&nbsp;top_left:&nbsp;boolean,<br>&nbsp;&nbsp;top_center:&nbsp;boolean,<br>&nbsp;&nbsp;top_right:&nbsp;boolean,<br>&nbsp;&nbsp;middle_left:&nbsp;boolean,<br>&nbsp;&nbsp;middle_center:&nbsp;boolean,<br>&nbsp;&nbsp;middle_right:&nbsp;boolean,<br>&nbsp;&nbsp;bottom_left:&nbsp;boolean,<br>&nbsp;&nbsp;bottom_center:&nbsp;boolean,<br>&nbsp;&nbsp;bottom_right:&nbsp;boolean<br>} |
| <br>**type=curtain** |  |  |
| `direction` | _Object_ | Valores aceitos: random,&nbsp;horizontal,&nbsp;vertical |
| `direction_lines` | _Object_ | Valores aceitos: random,&nbsp;down_right,&nbsp;up_left,&nbsp;alternate |
| `slide_move_type` | _Object_ | Valores aceitos: random,&nbsp;new,&nbsp;old,&nbsp;both |
| <br>**type=random** |  |  |
| `random_enabled_types` | _Object_ | Valores aceitos: {<br>&nbsp;&nbsp;fade:&nbsp;boolean,<br>&nbsp;&nbsp;slide:&nbsp;boolean,<br>&nbsp;&nbsp;accordion:&nbsp;boolean,<br>&nbsp;&nbsp;linear_fade:&nbsp;boolean,<br>&nbsp;&nbsp;zoom:&nbsp;boolean,<br>&nbsp;&nbsp;curtain:&nbsp;boolean<br>} |
<details>
  <summary>Ver exemplo</summary>

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
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `tab_version_1` | _String_ | Versão da Bíblia definida na primeira aba |
| `tab_version_2` | _String_ | Versão da Bíblia definida na segunda aba |
| `tab_version_3` | _String_ | Versão da Bíblia definida na terceira aba |
| `show_x_verses` | _Number_ | Quantidade de versículos exibidos na projeção |
| `uppercase` | _Boolean_ | Exibir o texto do versículo em maiúsculo |
| `show_only_reference` | _Boolean_ | Exibir somente a referência do versículo |
| `show_two_versions` | _Boolean_ | `deprecated` Substituído por: `show_second_version` `show_third_version`<br>Exibir duas versões. |
| `show_second_version` | _Boolean_ | Exibir segunda versão `v2.22.0+` |
| `show_third_version` | _Boolean_ | Exibir terceira versão `v2.22.0+` |
| `book_panel_type` | _String_ | Tipo de visualização dos livros da Bíblia `grid` `list` |
| `book_panel_order` | _String_ | Tipo de ordenação dos livros da Bíblia |
| `book_panel_order_available_items` | _Array&lt;String&gt;_ |  |
| `multiple_verses_separator_type` | _String_ | Tipo de separação na exibição de múltiplos versículos. Pode ser: no_line_break, single_line_break, double_line_break |
| `multiple_versions_separator_type` | _String_ | Tipo de separação na exibição de múltiplas versões. Pode ser: no_line_break, single_line_break, double_line_break `v2.22.0+` |
| `versification` | _Boolean_ | Aplicar mapeamento de versículos |
| `theme` | _Object_ | ID do Tema de exibição para as diferentes telas do sistema |
| `theme.public` | _String_ |  |
| `theme.screen_n` | _String_ | n >= 2 |
<details>
  <summary>Ver exemplo</summary>

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
    "automatic", "standard", "ru", "tyv"
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
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `font_name` | _String (opcional)_ | Nome da fonte `Padrão: null` |
| `bold` | _Boolean (opcional)_ | Negrito `Padrão: null` |
| `italic` | _Boolean (opcional)_ | Itálico `Padrão: null` |
| `color` | _String (opcional)_ | Cor em hexadecimal `Padrão: null` |

## Stage View
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `enabled` | _Boolean_ | Visão do palco ativada |
| `preview_mode` | _String_ | Modo de visualização das letras. Opções disponíveis:<br/>CURRENT_SLIDE<br/>FIRST_LINE_OF_THE_NEXT_SLIDE_WITH_SEPARATOR<br/>FIRST_LINE_OF_THE_NEXT_SLIDE_WITHOUT_SEPARATOR<br/>NEXT_SLIDE<br/>CURRENT_AND_NEXT_SLIDE<br/>ALL_SLIDES |
| `uppercase` | _Boolean_ | Exibir em maiúsculo |
| `remove_line_break` | _Boolean_ | Remover quebra de linha |
| `show_comment` | _Boolean_ | Exibir comentários |
| `show_advanced_editor` | _Boolean_ | Exibir edições avançadas |
| `show_communication_panel` | _Boolean_ | Exibir conteúdo do painel de comunicação |
| `show_next_image` | _Boolean_ | Exibir imagem seguinte `v2.21.0+` |
| `custom_theme` | _Number_ | ID do tema personalizado utilizado nas apresentações |
| `apply_custom_theme_to_bible` | _Boolean_ | Utilizar o tema personalizado nos versículos |
| `apply_custom_theme_to_text` | _Boolean_ | Utilizar o tema personalizado nos textos |
| `apply_custom_theme_to_quick_presentation` | _Boolean_ | Utilizar o tema personalizado na opção **Apresentação Rápida** `v2.21.0+` |
<details>
  <summary>Ver exemplo</summary>

```json
{
  "enabled": false,
  "preview_mode": "FIRST_LINE_OF_THE_NEXT_SLIDE_WITH_SEPARATOR",
  "uppercase": false,
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
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `info_1` | _Object_ |  |
| `info_1.show_page_count` | _Boolean_ | Exibir contador de slides |
| `info_1.show_slide_description` | _Boolean_ | Exibir descrição do slide (coro, por exemplo) |
| `info_1.horizontal_align` | _String_ | Alinhamento horizontal da informação no slide. left, center, right |
| `info_1.vertical_align` | _String_ | Alinhamento vertical da informação no slide. top, bottom |
| `info_2` | _Object_ |  |
| `info_2.show` | _Boolean_ |  |
| `info_2.layout_row_1` | _String_ | Layout da informação da primeira linha [Slide Additional Info Layout](#slide-additional-info-layout) |
| `info_2.layout_row_2` | _String (opcional)_ | Layout da informação da segunda linha [Slide Additional Info Layout](#slide-additional-info-layout) |
| `info_2.horizontal_align` | _String_ | Alinhamento horizontal da informação no slide. left, center, right |
| `info_2.vertical_align` | _String_ | Alinhamento vertical da informação no slide. top, bottom |
| `font` | _Object_ |  |
| `font.name` | _String_ | Nome da fonte. Se for **null**, utiliza a fonte padrão do tema. |
| `font.bold` | _Boolean_ | Negrito. Se for **null**, utiliza a configuração padrão do tema |
| `font.italic` | _Boolean_ | Itálido. Se for **null**, utiliza a configuração padrão do tema |
| `font.color` | _String_ | Cor da fonte em hexadecimal. Se for **null**, utiliza a cor da fonte padrão do tema |
| `height` | _Number_ | Altura em porcentagem em relação à altura do slide `4 ~ 15` |
| `paint_theme_effect` | _String_ | Renderizar o texto com os efeitos contorno, brilho e sombra do tema, se disponível |
<details>
  <summary>Ver exemplo</summary>

```json
{
  "info_1": {
    "show_page_count": false,
    "show_slide_description": false,
    "horizontal_align": "right", "vertical_align": "bottom"
  },
  "info_2": {
    "show": false,
    "layout_row_1": "<title>< (%author_or_artist%)>", "horizontal_align": "right", "vertical_align": "bottom"
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
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `x` | _Number_ |  |
| `y` | _Number_ |  |
| `width` | _Number_ |  |
| `height` | _Number_ |  |

## Custom Message
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `id` | _String_ | ID do item |
| `name` | _String_ | Nome do item |
| `message_model` | _String_ | Mensagem sem preenchimento |
| `message_example` | _String_ | Mensagem de exemplo com o nome dos parâmetros preenchidos |
| `variables` | _Array&lt;[CustomMessageParam](#custom-message-param)&gt;_ | Parâmetros da mensagem |
<details>
  <summary>Ver exemplo</summary>

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
      "uppercase": false,
      "suggestions": []
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
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `position` | _Number_ | Posição do parâmetro na mensagem (em número de caracteres) |
| `name` | _String_ | Nome do item |
| `only_number` | _Boolean_ | Parâmetro aceita somente números |
| `uppercase` | _Boolean_ | Parâmetro exibido sempre em maiúsculo |
| `suggestions` | _Array&lt;String&gt; (opcional)_ | Lista com valores padrões para o parâmetro |
<details>
  <summary>Ver exemplo</summary>

```json
{
  "position": 0,
  "name": "",
  "only_number": false,
  "uppercase": false,
  "suggestions": []
}
```
</details>

## Quiz Question
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `title` | _String_ | Pergunta |
| `alternatives` | _Array&lt;String&gt;_ | Alternativas |
| `correct_alternative_number` | _Number (opcional)_ | Número da alternativa correta. Começa em 1 `Padrão: 1` |
| `source` | _String (opcional)_ | Fonte da resposta |
<details>
  <summary>Ver exemplo</summary>

```json
{
  "title": "...",
  "alternatives": [
    "Item 1", "Item 2", "Item 3"
  ],
  "correct_alternative_number": 2,
  "source": ""
}
```
</details>

## Quiz Settings
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `correct_answer_color_font` | _String (opcional)_ | Cor da fonte para a resposta correta |
| `correct_answer_color_background` | _String (opcional)_ | Cor de fundo para a resposta correta |
| `incorrect_answer_color_font` | _String (opcional)_ | Cor da fonte para a resposta incorreta |
| `incorrect_answer_color_background` | _String (opcional)_ | Cor de fundo para a resposta incorreta |
| `question_and_alternatives_different_slides` | _Boolean (opcional)_ | Exibir a pergunta e as alternativas em slides separados `Padrão: false` |
| `display_alternatives_one_by_one` | _Boolean (opcional)_ | Exibir as alternativas uma a uma `Padrão: true` |
| `alternative_char_type` | _String (opcional)_ | Tipo de caractere para listar as alternativas `number (1, 2, 3...)`  `alpha (A, B, C...)` `Padrão: 'alpha'` |
| `alternative_separator_char` | _String (opcional)_ | Caractere separador. Valores permitidos:  ` `  `.`  `)`  `-`  `:` `Padrão: '.'` |
<details>
  <summary>Ver exemplo</summary>

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

## AddItem
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `type` | _String_ | Tipo do item. Pode ser: `title`  `song`  `verse`  `text`  `audio`  `video`  `image`  `file`  `announcement`  `automatic_presentation`  `countdown`  `countdown_cp`  `cp_text`  `plain_text`  `uri`  `global_action`  `api`  `script` |

## AddItemTitle
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `type` | _String_ | title |
| `name` | _String_ | Nome do item |
| `background_color` | _String (opcional)_ | Cor de fundo em hexadecimal, exemplo: 000080 |
| `collapsed` | _Boolean (opcional)_ |  `Padrão: false` |
<details>
  <summary>Ver exemplo</summary>

```json
{
  "type": "title",
  "name": "Exemplo",
  "background_color": "0000FF",
  "collapsed": false
}
```
</details>

## AddItemSong
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `type` | _String_ | song |
| `id` | _String_ | ID do item |
<details>
  <summary>Ver exemplo</summary>

```json
{
  "type": "song",
  "id": "123"
}
```
</details>

## AddItemVerse
**id**, **ids** ou **references**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `type` | _String_ | verse |
| `id` | _String (opcional)_ | Para exibir um versículo. ID do item no formato LLCCCVVV.<br/>Exemplo: '19023001' (livro 19, capítulo 023, versículo 001) |
| `ids` | _Array&lt;String&gt; (opcional)_ | Para exibir uma lista de versículos. Lista com o ID de cada versículo.<br/>Exemplo: ['19023001', '43003016', '45012002'] |
| `references` | _String (opcional)_ | Referências. Exemplo: **João 3:16** ou **Rm 12:2** ou **Gn 1:1-3 Sl 23.1** |
| `version` | _String (opcional)_ | Nome ou abreviação da tradução utilizada `v2.21.0+` |
<details>
  <summary>Ver exemplo</summary>

```json
{
  "type": "verse",
  "references": "Ps 23.1-6 Rm 12.2",
  "version": "en_kjv"
}
```
</details>

## AddItemText
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `type` | _String_ | text |
| `id` | _String_ | ID do item |
<details>
  <summary>Ver exemplo</summary>

```json
{
  "type": "text",
  "id": "xyz"
}
```
</details>

## AddItemAudio
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `type` | _String_ | audio |
| `name` | _String_ | Nome do arquivo |
| `settings` | _[PlayMediaSettings](#play-media-settings) (opcional)_ | Configurações para execução da mídia `v2.21.0+` |
<details>
  <summary>Ver exemplo</summary>

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
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `type` | _String_ | video |
| `name` | _String_ | Nome do arquivo |
| `settings` | _[PlayMediaSettings](#play-media-settings) (opcional)_ | Configurações para execução da mídia `v2.21.0+` |
<details>
  <summary>Ver exemplo</summary>

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
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `type` | _String_ | image |
| `name` | _String_ | Nome do arquivo |
<details>
  <summary>Ver exemplo</summary>

```json
{
  "type": "image",
  "name": "file.ext"
}
```
</details>

## AddItemAutomaticPresentation
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `type` | _String_ | automatic_presentation |
| `name` | _String_ | Nome do arquivo |
<details>
  <summary>Ver exemplo</summary>

```json
{
  "type": "automatic_presentation",
  "name": "filename.ap"
}
```
</details>

## AddItemAnnouncement
**id**, **ids**, **name** ou **names**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `type` | _String_ | announcement |
| `id` | _String (opcional)_ | ID do anúncio. Pode ser **all** para exibir todos |
| `ids` | _Array&lt;String&gt; (opcional)_ | Lista com o ID de cada anúncio |
| `name` | _String (opcional)_ | Nome do anúncio |
| `names` | _Array&lt;String&gt; (opcional)_ | Lista com o nome de cada anúncio |
| `automatic` | _[Automatic](#automatic-presentation) (opcional)_ | Se informado, a apresentação dos itens será automática |
<details>
  <summary>Ver exemplo</summary>

```json
{
  "type": "announcement",
  "names": [
    "Anúncio 1", "Anúncio 2", "Anúncio 3"
  ],
  "automatic": {
    "seconds": 10,
    "repeat": true
  }
}
```
</details>

## AddItemCountdown
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `type` | _String_ | countdown |
| `time` | _String_ | HH:MM ou MM:SS |
| `exact_time` | _Boolean (opcional)_ | Se **true**, `time` deve ser HH:MM (hora e minuto exato). Se **false**, `time` deve ser MM:SS (quantidade de minutos e segundos) `Padrão: false` |
| `text_before` | _String (opcional)_ | Texto exibido na parte superior da contagem regressiva |
| `text_after` | _String (opcional)_ | Texto exibido na parte inferior da contagem regressiva |
| `zero_fill` | _Boolean (opcional)_ | Preencher o campo 'minuto' com zero à esquerda `Padrão: false` |
| `countdown_relative_size` | _Number (opcional)_ | Tamanho relativo da contagem regressiva `Padrão: 250` |
| `theme` | _String (opcional)_ | ID do Tema `v2.21.0+` |
| `countdown_style` | _[FontSettings](#font-settings) (opcional)_ | Fonte personalizada para a contagem regressiva `v2.21.0+` |
<details>
  <summary>Ver exemplo</summary>

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
  }
}
```
</details>

## AddItemCountdownCommunicationPanel
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `type` | _String_ | countdown_cp |
| `minutes` | _Number_ | Quantidade de minutos |
| `seconds` | _Number_ | Quantidade de segundos |
| `stop_at_zero` | _Boolean (opcional)_ | Parar a contagem regressiva ao chegar em zero `Padrão: false` |
| `description` | _String_ | Descrição do item |
<details>
  <summary>Ver exemplo</summary>

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

## AddItemTextCommunicationPanel
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `type` | _String_ | cp_text |
| `name` | _String_ | Nome do item |
| `text` | _String_ | Texto |
| `display_ahead` | _Boolean (opcional)_ | Alterar a opção *'exibir à frente de tudo'* |
<details>
  <summary>Ver exemplo</summary>

```json
{
  "type": "cp_text",
  "name": "",
  "text": "Exemplo",
  "display_ahead": false
}
```
</details>

## AddItemScript
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `type` | _String_ | script |
| `id` | _String_ | ID do item |
| `description` | _String_ | Descrição do item |
| `inputs` | _Object (opcional)_ | Valor padrão para [Function Input](https://github.com/holyrics/Scripts/blob/main/FunctionInput.md) |
<details>
  <summary>Ver exemplo</summary>

```json
{
  "type": "script",
  "id": "xyz",
  "description": "",
  "inputs": {
    "message": "Exemplo", "duration": 30
  }
}
```
</details>

## AddItemAPI
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `type` | _String_ | api |
| `id` | _String_ | ID do item |
| `description` | _String_ | Descrição do item |
| `inputs` | _Object (opcional)_ | Valor padrão para [Function Input](https://github.com/holyrics/Scripts/blob/main/FunctionInput.md) |
<details>
  <summary>Ver exemplo</summary>

```json
{
  "type": "api",
  "id": "xyz",
  "description": "",
  "inputs": {
    "message": "Exemplo", "duration": 30
  }
}
```
</details>

## AddItemURI
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `type` | _String_ | uri |
| `title` | _String_ | Título do item |
| `uri_type` | _String_ | Pode ser: `spotify` `youtube` `deezer` |
| `value` | _String_ | URI |
<details>
  <summary>Ver exemplo</summary>

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
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `type` | _String_ | global_action |
| `action` | _String_ | Pode ser: `slide_exit` `vlc_stop` `vlc_stop_fade_out` |

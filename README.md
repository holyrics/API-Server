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
  - [ShowCountdown](#showcountdown)
  - [ShowQuiz](#showquiz)
  - [QuizAction](#quizaction)
  - [PlayAutomaticPresentation](#playautomaticpresentation)
  - [GetAutomaticPresentationPlayerInfo](#getautomaticpresentationplayerinfo)
  - [AutomaticPresentationPlayerAction](#automaticpresentationplayeraction)
  - [GetMediaPlayerInfo](#getmediaplayerinfo)
  - [MediaPlayerAction](#mediaplayeraction)
  - [GetLyricsPlaylist](#getlyricsplaylist)
  - [AddLyricsToPlaylist](#addlyricstoplaylist)
  - [RemoveFromLyricsPlaylist](#removefromlyricsplaylist)
  - [GetMediaPlaylist](#getmediaplaylist)
  - [MediaPlaylistAction](#mediaplaylistaction)
  - [AddToPlaylist](#addtoplaylist)
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
  - [GetColorMap](#getcolormap)
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
- v2.19.0

Retorna a lista de arquivos da respectiva aba: áudio, vídeo, imagem

**Parâmetros:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `folder` | _String (opcional)_ | Nome da subpasta para listar os arquivos |
| `filter` | _String (opcional)_ | Filtrar arquivos pelo nome |


**Resposta:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `data` | _Array&lt;Object&gt;_ |  |
| `data.*.name` | _String_ | Nome do item |
| `data.*.isDir` | _Boolean_ | Retorna **true** se for uma pasta ou **false** se for arquivo. |


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
| `position_?` | _Object_ | Variável adicionada na posição especificada conforme valor retornado em **variables.*.position** da classe CustomMessage. |
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

### GetFavorites
- v2.19.0

Itens da barra de favoritos



**Resposta:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `data` | _Array&lt;[Favorite Item](#favorite-item)&gt;_ |  |


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



**Resposta:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `data.id` | _String_ | ID do item |
| `data.type` | _String_ | Tipo do item. Pode ser: `title`  `song`  `verse`  `text`  `audio`  `video`  `image`  `announcement`  `automatic_presentation`  `countdown`  `countdown_cp`  `cp_text`  `api`  `script` |
| `data.name` | _String_ | Nome do item |
| `data.slide_number` | _Number_ | Começa em 1 `v2.20.0+` |
| `data.total_slides` | _Number_ | Total de slides `v2.20.0+` |
| `data.slide_type` | _String_ | Um dos seguintes valores: `default`  `wallpaper`  `blank`  `black`  `final_slide` `v2.20.0+` |


**Exemplo:**
```
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

### GetBackgrounds
- v2.19.0

Lista dos temas e planos de fundo

**Parâmetros:**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `input` | _Object (opcional)_ | Filtro |
| `type` | _String (opcional)_ | Pode ser: theme, my_video, my_image, video, image |
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
| `type` | _String (opcional)_ | Pode ser: theme, my_video, my_image, video, image |
| `tag` | _String (opcional)_ |  |
| `tags` | _Array&lt;String&gt; (opcional)_ |  |
| `intersection` | _Boolean (opcional)_ | Se o campo **tags** estiver preenchido com múltiplos itens, a opção **intersection** define o tipo de junção. Se **true**, o filtro retornará apenas itens que contém **todas** as tags informadas, se **false**, o filtro retornará os itens que têm pelo menos uma tag das tags informadas `Padrão: false` |


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
| `data` | _[Schedule](#schedule)_ |  |


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
| `theme` | _Boolean (opcional)_ | ID ou nome do tema padrão |
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
| `data.extend` | _Boolean_ | Estender papel de parede |
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
| `extend` | _Boolean (opcional)_ | Estender papel de parede |
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
| `key` | _String_ | Tom da música |
| `time_sig` | _String_ | Tempo da música |
| `groups` | _Array&lt;[Group](#group)&gt;_ | Grupos onde a música está adicionada |
| `midi` | _[Midi](#midi)_ | Atalho MIDI do item |
| `archived` | _Boolean_ | Se a música está arquivada |
## Background
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `id` | _String_ | ID do item |
| `type` | _String_ | Tipo do item. Pode ser: theme, my_video, my_image, video, image |
| `name` | _String_ | Nome do item |
| `tags` | _Array&lt;String&gt;_ | Lista de tags do item |
| `bpm` | _Number_ | Valor BPM do item |
| `midi` | _[Midi](#midi) (opcional)_ | Atalho MIDI do item |
## Item
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `id` | _String_ | ID do item |
| `type` | _String_ | Tipo do item. Pode ser: `title`  `song`  `verse`  `text`  `audio`  `video`  `image`  `announcement`  `automatic_presentation`  `countdown`  `countdown_cp`  `cp_text`  `api`  `script` |
| `name` | _String_ | Nome do item |
## Group
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `name` | _String_ | Nome do item |
## Midi
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `code` | _Number_ | Código midi |
| `velocity` | _Number_ | Velocidade/intensidade midi |
## Favorite Item
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `id` | _String_ | ID do item |
| `name` | _String_ | Nome do item |
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
## Member
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `id` | _String_ | ID do item |
| `name` | _String_ | Nome do item |
## Role
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `id` | _String_ | ID do item |
| `name` | _String_ | Nome do item |
## Automatic Presentation
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `seconds` | _Number_ | Tempo que cada item ficará sendo apresentado |
| `repeat` | _Boolean_ | **true** para ficar repetindo a apresentação (voltar para o primeiro item após o último) |
## Input Param
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `key` | _String_ | Chave/ID do item |
| `type` | _String_ | Tipo de entrada. Pode ser: text, textarea, number, password, title, separator |
| `label` | _String_ | Nome do item |
| `default_value` | _Object (opcional)_ | Valor padrão do item |
| `allowed_values` | _Array&lt;String&gt; (opcional)_ | Disponível se o tipo for **text**. Define uma lista de valores permitidos, para serem selecionados como um combobox |
| `suggested_values` | _Array&lt;String&gt; (opcional)_ | Disponível se o tipo for **text**. Define uma lista de valores sugeridos, porém o usuário pode inserir qualquer valor no campo de texto |
| `min` | _Number (opcional)_ | Disponível se o tipo for **number**. Define o valor mínimo permitido `Padrão: 0` |
| `max` | _Number (opcional)_ | Disponível se o tipo for **number**. Define o valor máximo permitido `Padrão: 100` |
| `show_as_combobox` | _Boolean (opcional)_ | Disponível se o tipo for **number**. Exibe a lista de valores como combobox e não como spinner `Padrão: false` |
## Display Settings
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `id` | _String_ | ID do item |
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
| `custom_theme` | _Number_ | ID do tema personalizado utilizado nas apresentações |
| `apply_custom_theme_to_bible` | _Boolean_ | Utilizar o tema personalizado nos versículos |
| `apply_custom_theme_to_text` | _Boolean_ | Utilizar o tema personalizado nos textos |
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
| `height` | _Number_ | Altura em porcentagem em relação à altura do slide |
| `paint_theme_effect` | _String_ | Renderizar o texto com os efeitos contorno, brilho e sombra do tema, se disponível |
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
## Custom Message Param
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `position` | _Number_ | Posição do parâmetro na mensagem (em número de caracteres) |
| `name` | _String_ | Nome do item |
| `only_number` | _Boolean_ | Parâmetro aceita somente números |
| `uppercase` | _Boolean_ | Parâmetro exibido sempre em maiúsculo |
| `suggestions` | _Array&lt;String&gt; (opcional)_ | Lista com valores padrões para o parâmetro |
## Quiz Question
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `title` | _String_ | Pergunta |
| `alternatives` | _Array&lt;String&gt;_ | Alternativas |
| `correct_alternative_number` | _Number (opcional)_ | Número da alternativa correta. Começa em 1 `Padrão: 1` |
| `source` | _String (opcional)_ | Fonte da resposta |
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
## AddItem
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `type` | _String_ | Tipo do item. Pode ser: `title`  `song`  `verse`  `text`  `audio`  `video`  `image`  `announcement`  `automatic_presentation`  `countdown`  `countdown_cp`  `cp_text`  `api`  `script` |
## AddItemTitle
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `type` | _String_ | title |
| `name` | _String_ | Nome do item |
| `background_color` | _String (opcional)_ | Cor de fundo em hexadecimal, exemplo: 000080 |
| `collapsed` | _Boolean (opcional)_ |  `Padrão: false` |
## AddItemSong
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `type` | _String_ | song |
| `id` | _String_ | ID do item |
## AddItemVerse
**id**, **ids** ou **references**

| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `type` | _String_ | verse |
| `id` | _String (opcional)_ | Para exibir um versículo. ID do item no formato LLCCCVVV.<br/>Exemplo: '19023001' (livro 19, capítulo 023, versículo 001) |
| `ids` | _Array&lt;String&gt; (opcional)_ | Para exibir uma lista de versículos. Lista com o ID de cada versículo.<br/>Exemplo: ['19023001', '43003016', '45012002'] |
| `references` | _String (opcional)_ | Referências. Exemplo: **João 3:16** ou **Rm 12:2** ou **Gn 1:1-3 Sl 23.1** |
## AddItemText
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `type` | _String_ | text |
| `id` | _String_ | ID do item |
## AddItemAudio
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `type` | _String_ | audio |
| `name` | _String_ | Nome do arquivo |
## AddItemVideo
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `type` | _String_ | video |
| `name` | _String_ | Nome do arquivo |
## AddItemImage
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `type` | _String_ | image |
| `name` | _String_ | Nome do arquivo |
## AddItemAutomaticPresentation
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `type` | _String_ | automatic_presentation |
| `name` | _String_ | Nome do arquivo |
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
## AddItemCountdownCommunicationPanel
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `type` | _String_ | countdown_cp |
| `minutes` | _Number_ | Quantidade de minutos |
| `seconds` | _Number_ | Quantidade de segundos |
| `stop_at_zero` | _Boolean (opcional)_ | Parar a contagem regressiva ao chegar em zero `Padrão: false` |
| `description` | _String_ | Descrição do item |
## AddItemTextCommunicationPanel
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `type` | _String_ | cp_text |
| `name` | _String_ | Nome do item |
| `text` | _String_ | Texto |
| `display_ahead` | _Boolean (opcional)_ | Alterar a opção *'exibir à frente de tudo'* |
## AddItemScript
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `type` | _String_ | script |
| `description` | _String_ | Descrição do item |
| `inputs` | _Object (opcional)_ | Valor padrão para [Function Input](https://github.com/holyrics/Scripts/blob/main/FunctionInput.md) |
## AddItemAPI
| Nome | Tipo  | Descrição |
| ---- | :---: | ------------|
| `type` | _String_ | api |
| `description` | _String_ | Descrição do item |
| `inputs` | _Object (opcional)_ | Valor padrão para [Function Input](https://github.com/holyrics/Scripts/blob/main/FunctionInput.md) |

# API-Server
**PT** | [EN](README-en.md)

O programa Holyrics disponibiliza uma interface de comunicação para receber requisições tanto pela rede local quanto pela internet.

As requisições recebidas serão redirecionadas para um método JavaScript no programa, que deverá ser implementado pelo próprio usuário.
<br/>
Ou seja, uma requisição por si só não realiza ações no programa. O que cada requisição fará depende da implementação definida pelo próprio usuário.
<br/>
A implementação do método JavaScript deverá utilizar a classe [JSLib](https://github.com/holyrics/jslib) para executar ações no programa.
<br/>
Também é responsabilidade da implementação do usuário definir, se necessário, controle de acesso para evitar requisições não autorizadas.
<br/>


Para acessar as configurações API-Server:
<br/>
Menu arquivo > Configurações > Avançado > JavaScript > API Server



# Exemplo de implementação JavaScript

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

`headers` - Contém os cabeçalhos da requisição.
<br/>
Exemplo: `headers.Authorization`

`content` - Objeto com o conteúdo extraído da requisição.
<br/>
Exemplo: `content.action`

`info` - Informações da requisição.
<br/>
`info.client_address` Endereço da origem da requisição
<br/>
`info.local` **true** se a origem da requisição for da rede local
<br/>
`info.web` **true** se a origem da requisição for da internet
<br/>
# Exemplo de requisição pela rede local
<br/>

```
URL padrão
http://[IP]:[PORT]/api
```

É possível realizar requisição GET ou POST.

Content-Type aceitos na requisição POST:
<br/>
application/x-www-form-urlencoded
<br/>
application/json
<br/>

---

Requisição

```
http://[IP]:[PORT]/api?pwd=password&action=show_alert&message=Hello
```

Resposta

```
{
    "status": "ok",             <-  status da requisição
    "data": { "status": "ok" }  <-  resposta obtida do método javascript
}
```

---

Requisição

```
http://[IP]:[PORT]/api?action=show_alert&message=Hello
```

Resposta

```
{
    "status": "ok",
    "data": { "status": "unauthorized" }
}
```

---

No caso de erro, **status** irá retornar **error**, exemplo:

```
{
    "status": "error",
    "error": "js unavailable"
}
```

# Exemplo de requisição pela rede internet

Utilize o valor **API_KEY** disponível nas configurações do API-Server para realizar as requisições no endpoint do servidor do Holyrics.
<br/>
Menu arquivo > Configurações > Avançado > JavaScript > API Server
<br/>
```
https://api.holyrics.com.br/send.php?api_key=API_KEY
Apenas envia a requisição sem aguardar retorno

https://api.holyrics.com.br/request.php?api_key=API_KEY
Envia a requisição e aguarda a resposta
```

É possível realizar requisição GET ou POST.

Content-Type aceitos na requisição POST:
<br/>
application/x-www-form-urlencoded
<br/>
application/json
<br/>
Tamanho máximo do corpo da solicitação: 20KB

O valor API_KEY deve ser passado na URL mesmo nas requisições POST


Requisição - send.php

```
https://api.holyrics.com.br/send.php?api_key=API_KEY&pwd=password&action=show_alert&message=Hello
```

Resposta

```
{ "status": "ok" }
```

O status das requisições **send.php** informa apenas se a requisição foi enviada ao Holyrics aberto no computador.

---

Requisição - request.php

```
https://api.holyrics.com.br/request.php?api_key=API_KEY&pwd=password&action=show_alert&message=Hello
```

Resposta

```
{
  "status": "ok",           <-  status do envio da requisição ao Holyrics
  "response_status": "ok",  <-  status da resposta da requisição enviada
  "response": {             <-  resposta da requisição
    "status": "ok",         <-  status da requisição
    "data": {               <-  resposta do método javascript
      "status": "ok"
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
        "key": "invalid_token",
        "message": "Invalid token"
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
      "error": "js unavailable"
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

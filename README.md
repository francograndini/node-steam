# Steam for Node.js

[![NPM version](https://img.shields.io/npm/v/steam.svg)](https://npmjs.org/package/steam "View this project on NPM")
[![Dependency Status](https://img.shields.io/david/seishun/node-steam.svg)](https://david-dm.org/seishun/node-steam)
[![PayPal donate button](https://img.shields.io/badge/paypal-donate-yellow.svg)](https://www.paypal.com/cgi-bin/webscr?cmd=_donations&business=Y83UZQZBJXEXQ&item_name=node%2dsteam&currency_code=EUR
 "Donate once-off to this project using PayPal")

Esta é uma porta Node.js do [SteamKit2](https://github.com/SteamRE/SteamKit). Ela permite você utilizar a interface Steam sem ter que rodar um Steam Client. Pode ser usada pra rodar um Bot de chat/troca autonomo.


# Instalação

```
npm install steam
```

Nota: quando instalar do git, voce tem que executar adicionalmente 'npm install' dentro de 'steam/node_modules/steam-resources' para rodar o `prepublish` script (veja [npm/npm#3055](https://github.com/npm/npm/issues/3055)). Pq puxa os recursos Steam (Protobufs and SteamLanguage) de SteamKit2 e requer `svn`.

**Nota: apenas Node.js v4.1.1 e acima são suportados.**

# Uso
Primeiro, `require` este módulo.
```js
var Steam = require('steam');
```
`Steam` é agora um 'namespace object' contendo:
* [SteamClient class](#steamclient)
* [Several handler classes](#handlers)
* [`servers` property](#servers)
* [Enums](#enums)

Enm seguida voce ira querer criar uma instancia do SteamClient e 'quaisquer manipuladores que vc precise'?(n entendi), chame [SteamClient#connect](#connect) 'e atribua ouvintes de eventos'.

```js
var steamClient = new Steam.SteamClient();
var steamUser = new Steam.SteamUser(steamClient);
steamClient.connect();
steamClient.on('connected', function() {
  steamUser.logOn({
    account_name: 'username',
    password: 'password'
  });
});
steamClient.on('logOnResponse', function() { /* ... */});
```

Veja example.js para a utilização de algumas API's disponíveis.

# Servidores

`Steam.servers` contem a lista de servidores CM a que o node-steam vai tentar se conectar. A lista de inicialização (veja [servers.js](https://github.com/seishun/node-steam/blob/master/lib/servers.js)) não está sempre atualizada e pode conter servers "mortos". Para evitar timeouts, substitua-a com sua propria lista antes de logar, caso vc tenha uma (veja ['servers' event](#servers-1)).

# SteamID

Como o tipo de numero do JavaScript nao tem precisao suficiente para armazenar inteiros de 64-bit, SteamIDs são representadas como cordas decimais. (Basta "enrolar" o nr entre aspas)

# Enums

Quando um metodo é aceito (ou surge um evento) um `ESomething`, é um numero que representa algum valor ENUM. Veja [enums.steamd](https://github.com/SteamRE/SteamKit/blob/master/Resources/SteamLanguage/enums.steamd) e [eresult.steamd](https://github.com/SteamRE/SteamKit/blob/master/Resources/SteamLanguage/eresult.steamd) para a lista completa deles. Para cada ENUM, existe uma propriedade de nome equivalente na `Steam`. A propriedade é um objeto; para cada um dos membros do ENUM, há uma propriedade de nome equivalente no objeto com um valor equivalente.

Note que vc pode facilmente pegar o valor correspondente para o numero, mas vc provavelmente nao precisará. Voce ainda pode usa-los em condiçoes (e.g. `if (type == Steam.EChatEntryType.Emote) ...`) ou mudar declarações.

# Protobufs

Quando um metodo é aceito (ou surge um evento) um `CMsgSomething`, é um objeto que representa uma mensagem protobuf. Ele tem uma propriedade de nome equivalente para cada campo na mensagem especificada com o tipo a seguir:

* `(u)int32` and `fixed32` fields: Number
* `uint64`, `fixed64` and `string` fields: String
* `bytes` fields: Buffer objects
* `bool` fields: Boolean

Veja [wiki](https://github.com/seishun/node-steam/wiki/Protobufs) para a descrição de campos de protobuf.

# Handlers (Manipuladores)

Maior parte do API é fornecido por claases de handler que enviam e recebem internamente mensagens de cliente de nivel baixo usando ['message'/send](#messagesend):

* [SteamUser](lib/handlers/user) - usuário da conta relacionadas com a funcionalidade, incluindo logon.
* [SteamFriends](lib/handlers/friends) - funcionalidade da comunidade, como chats e mensagens de amigos.
* [SteamTrading](lib/handlers/trading) - envio e recebimento de SOLICITAÇÕES de troca. Não confundir com oferta de troca.
* [SteamGameCoordinator](lib/handlers/game_coordinator) - enviar e receber mensagens do coordenador do jogo.
* [SteamUnifiedMessages](lib/handlers/unified_messages) - enviar e receber mensagens unificadas.
* [SteamRichPresence](lib/handlers/rich_presence) - enviar e receber mensagens de presença.

[If you think some unimplemented functionality belongs in one of the existing handlers, feel free to submit an issue to discuss it.]
Não necessario
# SteamClient

## Propriedades

### connected conectado

Um booleano(V/F xD) que indica se vc esta conectado e o cumprimento da encriptação está completo. ['connected'](#connected-1) é emitido quando ele muda para `true`, e ['error'](#error) é emitido quando muda para `false` a menos que vc tenha pedido [disconnect](#disconnect). Enviar qualquer mensagem é apenas permitido quando estiver marcado `true`.

### loggedOn

Um booleano que indica se vc esta logado. Chamando qualquer outro metodo de manipulação que nao seja
 [SteamUser#logOn](lib/handlers/user#logonlogondetails) só é permitido enquanto logado.

### sessionID

Sua session ID enquanto logado, de outra forma não especificada. (Nota: isso não tem nada a ver com o cookie "sessionid")

### steamID

Sua própria SteamID enquanto logada, de outra forma não especificada. Deve ser mudada para um valor de inicial válida antes de enviar uma mensagem de logon.
([SteamUser#logOn](lib/handlers/user#logonlogondetails) faz isso por vc).

## Metodos

### connect()

Conecta a Steam. Isso vai ficar tentando reconectar até que o cumprimento da encriptação esteja completo (veja ['connected'](#connected-1)), a menos que vc cancele com [disconnect](#disconnect).

Vc pode chamar esse metodo a qualquer momento. Se vc ja esta conectado, desconecta vc primeiro. Se houver uma tentativa de conexão acontecendo, cancela ela.

### disconnect()

Imediatamente termina a conexão e previne qualquer evento (incluindo ['error'](#error)) de ser emitido até que vc [connect](#connect) de novo. Se vc ja esta desconectado, nada acontece. Se há uma tentativa de conexão acontecendo, cancela ela.

## Events

### 'error'

Conexão fechada pelo servidor. Apenas emitido se o cumprimento de encriptação for completo, de outra forma irá reconectar automaticamente.
 [`loggedOn`](#loggedon) agora é `false`.

### 'connected'

Encryption handshake complete. From now on, it's your responsibility to handle disconnections and reconnect (see ['error'](#error)). You'll likely want to log on now (see [SteamUser#logOn](lib/handlers/user#logonlogondetails)).

### 'logOnResponse'
* [`CMsgClientLogonResponse`](https://github.com/SteamRE/SteamKit/blob/master/Resources/Protobufs/steamclient/steammessages_clientserver.proto)

Logon response received. If `eresult` is `EResult.OK`, [`loggedOn`](#loggedon) is now `true`.

### 'servers'
* an Array containing the up-to-date server list

node-steam will use this new list when reconnecting, but it will be lost when your application restarts. You might want to save it to a file or a database and assign it to [`Steam.servers`](#servers) before logging in next time.

Note that `Steam.servers` will be automatically updated _after_ this event is emitted. This will be useful if you want to compare the old list with the new one for some reason - otherwise it shouldn't matter.

### 'loggedOff'
* `EResult`

You were logged off from Steam. [`loggedOn`](#loggedon) is now `false`.


## 'message'/send

Sending and receiving client messages is designed to be symmetrical, so the event and the method are documented together. Both have the following arguments:

* `header` - an object representing the message header. It has the following properties:
  * `msg` - `EMsg` (no protomask).
  * `proto` - a [`CMsgProtoBufHeader`](https://github.com/SteamRE/SteamKit/blob/master/Resources/Protobufs/steamclient/steammessages_base.proto) object if this message is protobuf-backed, otherwise `header.proto` is falsy. The following fields are reserved for internal use and shall be ignored: `steamid`, `client_sessionid`, `jobid_source`, `jobid_target`. (Note: pass an empty object if you don't need to set any fields)
* `body` - a Buffer containing the rest of the message. (Note: in SteamKit2's terms, this is "Body" plus "Payload")
* `callback` (optional) - if not falsy, then this message is a request, and `callback` shall be called with any response to it instead of 'message'/send. `callback` has the same arguments as 'message'/send.

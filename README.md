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

Cumprimento de encriptação completo. De agora em diante, é sua responsabilidade manusear as desconexoes e reconexoes (veja ['error'](#error)). Vc provavelmente vai querer logar agora (veja [SteamUser#logOn](lib/handlers/user#logonlogondetails)).

### 'logOnResponse'
* [`CMsgClientLogonResponse`](https://github.com/SteamRE/SteamKit/blob/master/Resources/Protobufs/steamclient/steammessages_clientserver.proto)

Resposta de logon recebida. Se `eresult` é `EResult.OK`, [`loggedOn`](#loggedon) agora é `true`.

### 'servers'
* uma ordem contendo a lista de server atualizada

node-steam vai usar essa nova lista quando estiver reconectando, mas sera perdida quando a aplicação reiniciar. Vc provavelmente vai querer salvar isto num arquivo ou database para atribuir ela a [`Steam.servers`](#servers) antes de logar da proxima vez.

Note que `Steam.servers` sera atualizado automaticamente _after_ que este evento for emitido. Isso sera util se vc quiser comparar a antiga lista com a nova por algum motivo - do contrario não terá importancia.

### 'loggedOff'
* `EResult`

Vc foi desconectado da Steam. [`loggedOn`](#loggedon) agora é `false`.


## 'message'/send

Enviar e receber mensagens na steam foi desenvolvido para ser simetrico, entao o evento e o método são documentados juntos. Ambos tem os seguintes argumentos:

* `header` - um objeto representando o cabeçalho da mensagem. Tem as seguintes propriedades:
  * `msg` - `EMsg` (no protomask).
  * `proto` - a [`CMsgProtoBufHeader`](https://github.com/SteamRE/SteamKit/blob/master/Resources/Protobufs/steamclient/steammessages_base.proto) objeto caso essa mensagem é protobuf-backed, do contrario `header.proto` é falsa. Os campos a seguir sao reservados para uso interno e devem ser ignorados: `steamid`, `client_sessionid`, `jobid_source`, `jobid_target`. (Nota: passe um objeto vazio se vc nao precisa mudar nenhum campo)
* `body` - um Buffer contendo o resto da mensagem. (Nota: nos termos de SteamKit2, isso é "Body" mais "Payload")
* `callback` (opcional) - se nao sao falsas, entao essa mensagem é um pedido, e `callback` deve ser chamada com qualquer resposta a isto ao inves de 'message'/send. `callback` tem os mesmos argumentos de 'message'/send.

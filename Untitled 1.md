# Relatório Completo: Sistema `publish_realtime` do Frappe

  

## Resumo Executivo

  

O sistema `publish_realtime` do Frappe é uma arquitetura completa de comunicação em tempo real que permite enviar eventos do backend Python para o frontend JavaScript via WebSockets. O sistema utiliza **Redis como message broker** e **Socket.IO** para comunicação bidirecional, suportando multi-tenancy e targeting granular de mensagens.

  

## Arquitetura Geral

  

```mermaid

graph TD

A[Python Backend] -->|publish_realtime()| B[Redis Channel 'events']

B --> C[Node.js SocketIO Server]

C -->|WebSocket| D[Browser Client]

A1[frappe.publish_realtime] --> A2[frappe.realtime.publish_realtime]

A2 --> A3[emit_via_redis]

A3 --> B

C --> C1[Authentication Middleware]

C --> C2[Room Management]

C --> C3[Event Handlers]

D --> D1[frappe.realtime.on()]

D --> D2[socketio_client.js]

```

  

## 1. Ponto de Entrada: `frappe.publish_realtime()`

  

**Localização**: `/workspace/development/frappe-bench/apps/frappe/frappe/__init__.py`

  

```python

def publish_realtime(event=None, message=None, user=None, doctype=None, docname=None, **kwargs):

"""Publish a message to web-client through socket.io"""

from frappe.realtime import publish_realtime

publish_realtime(event=event, message=message, user=user, doctype=doctype, docname=docname, **kwargs)

```

  

Esta é apenas uma **função wrapper** que delega para o módulo core `frappe.realtime`.

  

## 2. Módulo Core: `frappe.realtime`

  

**Localização**: `/workspace/development/frappe-bench/apps/frappe/frappe/realtime.py`

  

### 2.1 Função Principal `publish_realtime()`

  

```python

def publish_realtime(

event=None,

message=None,

room=None,

user=None,

doctype=None,

docname=None,

task_id=None,

after_commit=False

):

```

  

**Parâmetros:**

- `event`: Nome do evento (obrigatório)

- `message`: Dados a serem enviados

- `room`: Sala específica (se não fornecida, é calculada automaticamente)

- `user`: Usuário específico para receber a mensagem

- `doctype`: Tipo de documento para targeting

- `docname`: Nome específico do documento

- `task_id`: ID de tarefa para eventos de progresso

- `after_commit`: Se True, agenda o envio para após commit da transação

  

### 2.2 Sistema de Rooms (Salas)

  

O sistema utiliza diferentes tipos de "rooms" para targeting de mensagens:

  

#### Tipos de Rooms:

1. **Site-wide**: `{site}` - Todos os usuários do site

2. **User-specific**: `{site}:user:{user}` - Usuário específico

3. **DocType-specific**: `{site}:doctype:{doctype}` - Todos os docs de um tipo

4. **Doc-specific**: `{site}:doc:{doctype}/{docname}` - Documento específico

5. **Task progress**: `{site}:task_progress:{task_id}` - Progresso de tarefa

  

#### Funções de Criação de Rooms:

```python

def get_user_room(user):

return f"{frappe.local.site}:user:{user}"

  

def get_doc_room(doctype, docname):

return f"{frappe.local.site}:doc:{doctype}/{docname}"

  

def get_site_room():

return frappe.local.site

```

  

### 2.3 Publicação via Redis

  

```python

def emit_via_redis(event, message, room):

"""Emit message via redis."""

r = get_redis_connection_without_auth()

frappe_message = json.dumps({

"event": event,

"message": message,

"room": room,

"namespace": frappe.local.site

})

r.publish("events", frappe_message)

```

  

**Características:**

- Canal Redis: `"events"`

- Formato da mensagem: JSON com event, message, room e namespace

- Namespace = site atual (multi-tenancy)

- Conexão Redis sem autenticação

  

### 2.4 Sistema de Permissões

  

```python

def can_subscribe_doc(doctype=None, docname=None, user=None):

"""Check if user can subscribe to updates for a document"""

def can_subscribe_doctype(doctype, user=None):

"""Check if user can subscribe to updates for a doctype"""

```

  

Sistema robusto de permissões que verifica:

- Permissões de leitura do documento

- Restrições por DocType

- Usuários específicos autorizados

  

## 3. Servidor Socket.IO (Node.js)

  

### 3.1 Entry Point

**Localização**: `/workspace/development/frappe-bench/apps/frappe/socketio.js`

```javascript

module.exports = require('./realtime');

```

  

### 3.2 Servidor Principal

**Localização**: `/workspace/development/frappe-bench/apps/frappe/realtime/index.js`

  

**Funcionalidades principais:**

- Servidor Socket.IO com suporte a namespaces por site

- Subscrição ao Redis canal "events"

- Distribuição de mensagens para clientes conectados

- Gerenciamento de rooms e conexões

- Sistema de heartbeat para conexões ativas

  

**Configuração:**

```javascript

const io = require('socket.io')(server, {

cors: {

origin: get_url(),

credentials: true

},

path: '/socket.io/'

});

```

  

### 3.3 Middleware de Autenticação

**Localização**: `/workspace/development/frappe-bench/apps/frappe/realtime/middlewares/authenticate.js`

  

**Métodos de autenticação:**

1. **Cookie-based**: Verifica cookies de sessão

2. **Authorization header**: Para APIs e integrações

3. **Token-based**: Suporte a tokens de API

  

**Fluxo:**

```javascript

const authenticate = (socket, next) => {

// 1. Extrai credenciais do cookie ou header

// 2. Faz chamada HTTP para /api/method/frappe.realtime.can_subscribe

// 3. Valida resposta e autoriza conexão

// 4. Atribui user ao socket para controle de acesso

};

```

  

### 3.4 Handlers de Eventos

**Localização**: `/workspace/development/frappe-bench/apps/frappe/realtime/handlers/frappe_handlers.js`

  

**Eventos suportados:**

- `doctype_subscribe`: Subscrição a atualizações de DocType

- `doc_subscribe`: Subscrição a documento específico

- `task_subscribe`: Subscrição a progresso de tarefa

- `doc_unsubscribe`: Cancelar subscrições

  

## 4. Cliente JavaScript (Browser)

  

**Localização**: `/workspace/development/frappe-bench/apps/frappe/frappe/public/js/frappe/socketio_client.js`

  

### 4.1 Inicialização

```javascript

frappe.socketio = {

init: function() {

if (frappe.boot.disable_async) return;

this.socket = io.connect(`/${frappe.boot.socketio_port}`, {

path: '/socket.io/'

});

}

};

```

  

### 4.2 Sistema de Eventos

```javascript

frappe.realtime = {

on: function(event, callback) {

// Registra callback para evento específico

},

off: function(event, callback) {

// Remove callback de evento

}

};

```

  

## 5. Fluxo de Dados Completo

  

### 5.1 Publicação de Evento (Python → Redis)

```python

# 1. Código Python chama

frappe.publish_realtime(

event='doc_update',

message={'doctype': 'Item', 'name': 'ITEM-001'},

doctype='Item',

docname='ITEM-001'

)

  

# 2. Sistema calcula room automaticamente

room = get_doc_room('Item', 'ITEM-001') # site:doc:Item/ITEM-001

  

# 3. Publica no Redis

emit_via_redis('doc_update', message, room)

  

# 4. Redis recebe JSON:

{

"event": "doc_update",

"message": {"doctype": "Item", "name": "ITEM-001"},

"room": "mysite:doc:Item/ITEM-001",

"namespace": "mysite"

}

```

  

### 5.2 Distribuição (Redis → Socket.IO → Browser)

```javascript

// 1. Servidor Node.js consome Redis

redis_client.on('message', (channel, message) => {

const data = JSON.parse(message);

const namespace = io.of(`/${data.namespace}`);

namespace.to(data.room).emit(data.event, data.message);

});

  

// 2. Browser recebe via Socket.IO

frappe.realtime.on('doc_update', (message) => {

console.log('Document updated:', message);

// Atualiza UI conforme necessário

});

```

  

## 6. Uso no Projeto NXLite

  

### 6.1 Casos de Uso Identificados

  

#### A. Notificações Toast

**Arquivo**: `/workspace/development/frappe-bench/apps/nxlite/nxlite/scripts/utils.py`

```python

def toast(message, title="Mensagem", indicator="blue"):

frappe.publish_realtime(

event="show_toast",

message={

"message": message,

"title": title,

"indicator": indicator

},

user=frappe.session.user

)

```

  

#### B. Status de Sensores IoT

**Arquivo**: `/workspace/development/frappe-bench/apps/nxlite/nxlite/commands/status_sensor.py`

```python

class StatusSensor:

def publish_status_change(self, sensor_id, status):

frappe.publish_realtime(

event="sensor_status_change",

message={

"sensor_id": sensor_id,

"status": status,

"timestamp": frappe.utils.now()

},

room=f"sensor:{sensor_id}"

)

```

  

#### C. Dashboards em Tempo Real

**Arquivos Vue.js**: Múltiplos dashboards que consomem eventos:

- Dashboard Comercial

- Dashboard de Produção

- Dashboard Financeiro

- Dashboard de Estoque

- Kanban de Produção

  

```javascript

// Exemplo típico em Vue.js

mounted() {

frappe.realtime.on('production_update', (data) => {

this.updateProductionData(data);

});

frappe.realtime.on('mqtt_message', (data) => {

this.handleMqttMessage(data);

});

}

```

  

### 6.2 Integração com MQTT/IoT

O sistema realtime é usado extensivamente para:

- Receber dados de sensores via MQTT

- Propagar status de equipamentos

- Atualizar dashboards de produção em tempo real

- Notificar falhas e alarmes

  

## 7. Características Técnicas Avançadas

  

### 7.1 Multi-tenancy

- Cada site tem seu próprio namespace no Socket.IO

- Isolamento completo entre sites diferentes

- Redis namespace garante segregação de dados

  

### 7.2 Sistema de Transações

```python

# Publicação após commit da transação

frappe.publish_realtime(

event='doc_saved',

message=message,

after_commit=True # Aguarda commit do banco

)

```

  

### 7.3 Controle de Sessão

- Heartbeat automático para detectar conexões mortas

- Cleanup automático de rooms vazias

- Reconexão automática em caso de queda

  

### 7.4 Performance e Escalabilidade

- Uso de Redis Pub/Sub para performance

- Suporte a múltiplos workers Socket.IO

- Balanceamento de carga via sticky sessions

  

## 8. Configuração e Deploy

  

### 8.1 Requisitos

- Redis server ativo

- Node.js com Socket.IO

- Processo separado para servidor realtime

  

### 8.2 Processo de Inicialização

```bash

# 1. Bench inicia servidor Socket.IO

node apps/frappe/socketio.js

  

# 2. Servidor conecta ao Redis

# 3. Subscreve ao canal "events"

# 4. Aguarda conexões de clientes

```

  

## 9. Troubleshooting e Monitoramento

  

### 9.1 Logs Importantes

- `logs/frappe.log`: Eventos Python

- `logs/socketio.log`: Eventos Socket.IO (se configurado)

- Redis logs: Pub/Sub atividade

  

### 9.2 Debugging

```javascript

// Cliente - Debug de conexão

console.log('SocketIO connected:', frappe.socketio.socket.connected);

  

// Servidor - Debug de rooms

console.log('Active rooms:', io.sockets.adapter.rooms);

```

  

### 9.3 Problemas Comuns

1. **Conexão recusada**: Verificar se processo socketio está ativo

2. **Eventos não recebidos**: Verificar permissões e rooms corretas

3. **Performance lenta**: Verificar Redis connection pool

4. **Memory leaks**: Cleanup adequado de event listeners

  

## 10. Conclusões e Recomendações

  

### 10.1 Pontos Fortes

- ✅ Sistema robusto e bem estruturado

- ✅ Suporte completo a multi-tenancy

- ✅ Controle granular de permissões

- ✅ Performance adequada via Redis

- ✅ Integração perfeita com framework Frappe

  

### 10.2 Áreas de Atenção

- ⚠️ Dependência crítica do Redis

- ⚠️ Processo Node.js adicional para manter

- ⚠️ Debugging pode ser complexo

- ⚠️ Necessita configuração adequada para produção

  

### 10.3 Uso Recomendado

O sistema `publish_realtime` é ideal para:

- Notificações em tempo real

- Dashboards com dados ao vivo

- Sistemas IoT e MQTT

- Atualizações de documentos colaborativos

- Progress tracking de tarefas longas

  

**Evitar para:**

- Alto volume de mensagens (>1000/seg)

- Dados críticos sem fallback

- Sistemas sem Redis disponível

  

---

  

## Referências Técnicas

  

**Arquivos Analisados:**

- `/workspace/development/frappe-bench/apps/frappe/frappe/__init__.py`

- `/workspace/development/frappe-bench/apps/frappe/frappe/realtime.py`

- `/workspace/development/frappe-bench/apps/frappe/socketio.js`

- `/workspace/development/frappe-bench/apps/frappe/realtime/index.js`

- `/workspace/development/frappe-bench/apps/frappe/realtime/middlewares/authenticate.js`

- `/workspace/development/frappe-bench/apps/frappe/realtime/handlers/frappe_handlers.js`

- `/workspace/development/frappe-bench/apps/frappe/frappe/public/js/frappe/socketio_client.js`

  

**Data da Análise:** 6 de Junho de 2025

**Versão do Frappe:** Detectada através do código fonte analisado
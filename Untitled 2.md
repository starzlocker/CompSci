# 🚀 **GUIA COMPLETO: COMO O FRAPPE FUNCIONA**

*Treinamento Técnico Detalhado - From Frontend to Database*

  

## 📋 **ÍNDICE**

1. [Visão Geral da Arquitetura](#arquitetura)

2. [Frontend: Form Scripts e Triggers](#frontend)

3. [Comunicação Frontend ↔ Backend](#comunicacao)

4. [Backend: Document Lifecycle](#backend)

5. [Database: Persistência e Transações](#database)

6. [Fluxo Completo de um Documento](#fluxo)

7. [Exemplos Práticos](#exemplos)

8. [Melhores Práticas](#praticas)

  

---

  

## 🏗️ **1. VISÃO GERAL DA ARQUITETURA** {#arquitetura}

  

### **Stack Tecnológico:**

```

┌─────────────────────────────────────────────────────────────┐

│ FRAPPE STACK │

├─────────────────────────────────────────────────────────────┤

│ Frontend │ JavaScript (ES6+) + jQuery + Bootstrap │

│ Backend │ Python 3.x + Flask/Werkzeug │

│ Database │ MariaDB/MySQL + SQLAlchemy-like ORM │

│ Cache │ Redis (sessões, cache, real-time) │

│ Queue │ Redis + RQ (background jobs) │

│ WebServer │ Nginx (proxy) + Gunicorn (WSGI) │

└─────────────────────────────────────────────────────────────┘

```

  

### **Arquitetura MVC Estendida:**

```

Frontend (View) Backend (Controller) Database (Model)

│ │ │

├─ form.js ├─ document.py ├─ tabDocType

├─ frappe.ui.form.on() ├─ hooks.py ├─ tabSingles

├─ triggers/events ├─ @frappe.whitelist() ├─ tab[CustomDocType]

└─ frappe.call() └─ DocType classes └─ Child Tables

```

  

---

  

## 🎭 **2. FRONTEND: FORM SCRIPTS E TRIGGERS** {#frontend}

  

### **2.1 Como `frappe.ui.form.on()` Funciona:**

  

```javascript

// Estrutura básica de um form script

frappe.ui.form.on('DocType Name', {

// Eventos do documento principal

refresh: function(frm) {

// Chamado sempre que o form é carregado/refrescado

console.log("Form refreshed:", frm.doc);

},

validate: function(frm) {

// Chamado antes de salvar (client-side validation)

if (!frm.doc.required_field) {

frappe.throw("Campo obrigatório não preenchido");

}

},

fieldname: function(frm) {

// Chamado quando 'fieldname' muda

frm.set_value('dependent_field', frm.doc.fieldname * 2);

}

});

  

// Eventos para tabelas filhas

frappe.ui.form.on('Child DocType', {

fieldname: function(frm, cdt, cdn) {

// cdt = Child DocType

// cdn = Child Document Name (unique ID)

let row = locals[cdt][cdn];

row.calculated_field = row.qty * row.rate;

frm.refresh_field('child_table');

}

});

```

  

### **2.2 Ordem de Execução dos Triggers:**

  

```javascript

// Sequência cronológica de eventos:

/*

1. setup() - Primeira vez que o form é criado

2. onload() - Toda vez que um documento é carregado

3. refresh() - Após onload, sempre que form atualiza

4. [field_change]() - Quando campo específico muda

5. validate() - Antes de salvar (client-side)

6. before_save() - Antes de enviar para servidor

7. after_save() - Após salvar com sucesso

*/

  

frappe.ui.form.on('Nota Fiscal - Servico', {

setup: function(frm) {

// Configurações iniciais do form

frm.set_query('customer', function() {

return { filters: { disabled: 0 } };

});

},

onload: function(frm) {

// Executado quando documento é carregado

if (frm.doc.__islocal) {

frm.set_value('date', frappe.datetime.now_date());

}

},

refresh: function(frm) {

// Sempre executado após onload

frm.toggle_display('section_break', !frm.doc.is_simple);

// Botões customizados

if (frm.doc.docstatus === 1) {

frm.add_custom_button('Custom Action', function() {

// Ação customizada

});

}

},

valor_servicos: function(frm) {

// Trigger específico do campo

if (frm.doc.valor_servicos) {

frm.set_value('base_calculo', frm.doc.valor_servicos);

// Chamar função Python

frappe.call({

method: 'nxlite.scripts.calculo_imposto.calculo_imposto_nfse',

args: { doc: frm.doc },

callback: function(r) {

if (r.message) {

frm.set_value(r.message);

}

}

});

}

},

validate: function(frm) {

// Validação client-side

if (frm.doc.valor_servicos <= 0) {

frappe.throw('Valor dos serviços deve ser maior que zero');

}

}

});

```

  

### **2.3 Gerenciamento de Estado no Frontend:**

  

```javascript

// Como o Frappe gerencia estado

/*

┌─────────────────────────────────────────────────────────────┐

│ FRONTEND STATE │

├─────────────────────────────────────────────────────────────┤

│ cur_frm.doc │ Documento atual (objeto principal) │

│ locals │ Cache de todos os documentos carregados │

│ frappe.model │ Funções para manipular documentos │

│ frm.dirty() │ Verifica se há mudanças não salvas │

│ frm.is_new() │ Documento é novo (__islocal = true) │

└─────────────────────────────────────────────────────────────┘

*/

  

// Exemplo de manipulação de estado

frappe.ui.form.on('Nota Fiscal - Servico', {

refresh: function(frm) {

console.log("Documento:", frm.doc);

console.log("É novo?", frm.is_new());

console.log("Está sujo?", frm.is_dirty());

console.log("Status:", frm.doc.docstatus);

// Acessar tabela filha

frm.doc.tab_servicos.forEach(function(row) {

console.log("Item:", row.item, "Valor:", row.valor_total);

});

}

});

```

  

---

  

## 🔄 **3. COMUNICAÇÃO FRONTEND ↔ BACKEND** {#comunicacao}

  

### **3.1 Métodos de Comunicação:**

  

```javascript

// 1. frappe.call() - Chamada direta para método Python

frappe.call({

method: 'nxlite.scripts.calculo_imposto.calculo_imposto_nfse',

args: {

doc: frm.doc,

extra_param: 'value'

},

callback: function(response) {

if (response.message) {

console.log("Resposta do servidor:", response.message);

frm.set_value(response.message); // Atualizar campos

}

},

error: function(error) {

console.error("Erro:", error);

}

});

  

// 2. frm.save() - Salvar documento

frm.save().then(function() {

frappe.msgprint("Documento salvo com sucesso!");

});

  

// 3. frappe.db.get_value() - Buscar valores do banco

frappe.db.get_value('Customer', frm.doc.customer, 'customer_name')

.then(function(response) {

console.log("Nome do cliente:", response.message.customer_name);

});

  

// 4. frappe.db.get_list() - Buscar lista de documentos

frappe.db.get_list('Item', {

filters: { disabled: 0 },

fields: ['name', 'item_name', 'item_group']

}).then(function(items) {

console.log("Itens encontrados:", items);

});

```

  

### **3.2 Protocolo de Comunicação:**

  

```

Frontend Backend Database

│ │ │

├─ frappe.call() │ │

│ └─ POST /api/method │ │

│ └─ JSON payload │ │

│ ├─ @frappe.whitelist() │

│ ├─ Autenticação │

│ ├─ Validação de params │

│ ├─ Execução do método │

│ │ └─ frappe.get_doc() │

│ │ └─ SQL SELECT ├─ MariaDB

│ │ └─ Business Logic │

│ │ └─ doc.save() │

│ │ └─ SQL INSERT/UPDATE ├─ MariaDB

│ ├─ Retorno JSON │

│ │ │

├─ response.message │ │

└─ Atualizar UI │ │

```

  

### **3.3 Exemplo Real de Comunicação:**

  

```javascript

// Frontend (nota_fiscal___servico.js)

frappe.ui.form.on('Nota Fiscal - Servico', {

valor_servicos: function(frm) {

// 1. Trigger no frontend

console.log("Campo valor_servicos alterado");

// 2. Chamada para o backend

frappe.call({

method: 'nxlite.scripts.calculo_imposto.calculo_imposto_nfse',

args: { doc: frm.doc },

callback: function(r) {

// 5. Resposta do backend processada

if (r.message) {

// 6. Atualizar campos na tela

Object.keys(r.message).forEach(function(field) {

frm.set_value(field, r.message[field]);

});

}

}

});

}

});

```

  

```python

# Backend (calculo_imposto.py)

@frappe.whitelist()

def calculo_imposto_nfse(doc):

"""

3. Método executado no backend

"""

# Validação de entrada

if not doc or doc.get('nota_fiscal_sem_calculo'):

return None

# Business logic

valor_servicos = Decimal(doc.get('valor_servicos', 0))

aliquota_iss = Decimal(doc.get('aliquota_iss', 0))

# Cálculos

valor_iss = (valor_servicos * aliquota_iss / 100).quantize(

Decimal('0.01'), ROUND_HALF_UP

)

# 4. Retorno para o frontend

return {

'valor_iss': float(valor_iss),

'valor_liquido': float(valor_servicos - valor_iss)

}

```

  

---

  

## 🐍 **4. BACKEND: DOCUMENT LIFECYCLE** {#backend}

  

### **4.1 Classe Document - Estrutura Base:**

  

```python

# /apps/frappe/frappe/model/document.py (simplificado)

class Document(BaseDocument):

def __init__(self, *args, **kwargs):

# Inicialização do documento

self.doctype = None

self.name = None

self.flags = frappe._dict()

def insert(self, ignore_permissions=None, ignore_links=None):

"""Inserir novo documento no banco"""

# 1. Validações iniciais

self.set_user_and_timestamp()

self.check_permission("create")

# 2. Hooks before_insert

self.run_method("before_insert")

# 3. Validações

self.validate()

# 4. Inserir no banco

self.db_insert()

# 5. Hooks after_insert

self.run_method("after_insert")

return self

def save(self):

"""Salvar documento existente"""

if self.get("__islocal"):

return self.insert()

# 1. Validações

self.check_permission("write")

# 2. Hooks before_save

self.run_method("before_save")

# 3. Validate

self.validate()

# 4. Atualizar banco

self.db_update()

# 5. Hooks after_save

self.run_method("after_save")

return self

```

  

### **4.2 Document Hooks - Ordem de Execução:**

  

```python

# Hooks disponíveis em ordem cronológica:

  

class NotaFiscalServico(Document):

def before_insert(self):

"""Executado ANTES de inserir no banco"""

# Configurações iniciais, numeração automática

pass

def validate(self):

"""Validação principal - SEMPRE executado antes de salvar"""

# Validações de negócio

if not self.valor_servicos:

frappe.throw("Valor dos serviços é obrigatório")

def before_save(self):

"""Executado ANTES de salvar (insert ou update)"""

# Cálculos automáticos

self.calcular_impostos()

def after_insert(self):

"""Executado APÓS inserir no banco"""

# Criar documentos relacionados

self.criar_titulos_financeiros()

def on_update(self):

"""Executado APÓS atualizar no banco"""

# Sincronizar com outros sistemas

pass

def before_submit(self):

"""Executado ANTES de submeter (docstatus = 1)"""

# Validações finais

pass

def on_submit(self):

"""Executado APÓS submeter"""

# Ações pós-submissão

pass

def before_cancel(self):

"""Executado ANTES de cancelar"""

# Verificar se pode cancelar

pass

def on_cancel(self):

"""Executado APÓS cancelar"""

# Reverter ações

pass

def on_trash(self):

"""Executado ANTES de deletar"""

# Limpar dados relacionados

pass

```

  

### **4.3 Exemplo Real de Document Class:**

  

```python

# nota_fiscal___servico.py

import frappe

from frappe.model.document import Document

from decimal import Decimal, ROUND_HALF_UP

  

class NotaFiscalServico(Document):

def validate(self):

"""Validação principal"""

# 1. Validações básicas

if not self.valor_servicos or self.valor_servicos <= 0:

frappe.throw("Valor dos serviços deve ser maior que zero")

# 2. Validar entidade tomador

if not self.entidade_tomador:

frappe.throw("Entidade tomador é obrigatória")

# 3. Calcular impostos automaticamente

self.calcular_impostos()

def before_save(self):

"""Antes de salvar"""

# Atualizar campos calculados

self.atualizar_totais()

def on_update(self):

"""Após atualizar"""

# Verificar mudança de status

old_doc = self.get_doc_before_save()

if old_doc and old_doc.status != 'autorizada' and self.status == 'autorizada':

# Status mudou para autorizada

self.processar_autorizacao()

def calcular_impostos(self):

"""Cálculo de impostos"""

from nxlite.scripts.calculo_imposto import calculo_imposto_nfse

calculo_imposto_nfse(self)

def processar_autorizacao(self):

"""Processar autorização da NF"""

# Incrementar numeração

self.incrementar_numero_nfse()

# Criar títulos financeiros

self.criar_titulos_financeiros()

# Publicar evento real-time

frappe.publish_realtime('nota_autorizada', {

'doctype': self.doctype,

'name': self.name,

'numero_nfse': self.numero_nfse

})

```

  

### **4.4 Frappe ORM - Operações de Banco:**

  

```python

# Como o Frappe interage com o banco

import frappe

  

# 1. CREATE - Criar novo documento

doc = frappe.new_doc('Nota Fiscal - Servico')

doc.update({

'empresa': 'Minha Empresa',

'entidade_tomador': 'CLIENTE001',

'valor_servicos': 1000.00

})

doc.insert()

  

# 2. READ - Buscar documento

doc = frappe.get_doc('Nota Fiscal - Servico', 'NF-001')

print(f"Valor: {doc.valor_servicos}")

  

# 3. UPDATE - Atualizar documento

doc.valor_servicos = 1500.00

doc.save()

  

# 4. DELETE - Deletar documento

doc.delete()

  

# 5. QUERIES - Consultas SQL via ORM

# Buscar lista

nfs = frappe.get_list('Nota Fiscal - Servico',

filters={'status': 'autorizada'},

fields=['name', 'numero_nfse', 'valor_servicos']

)

  

# SQL direto

result = frappe.db.sql("""

SELECT name, valor_servicos

FROM `tabNota Fiscal - Servico`

WHERE status = 'autorizada'

""", as_dict=True)

  

# Buscar valor único

numero = frappe.db.get_value('Nota Fiscal - Servico', 'NF-001', 'numero_nfse')

  

# Buscar múltiplos campos

empresa, valor = frappe.db.get_value('Nota Fiscal - Servico', 'NF-001',

['empresa', 'valor_servicos'])

  

# Contagem

count = frappe.db.count('Nota Fiscal - Servico', {'status': 'autorizada'})

  

# Verificar se existe

exists = frappe.db.exists('Nota Fiscal - Servico', 'NF-001')

```

  

---

  

## 💾 **5. DATABASE: PERSISTÊNCIA E TRANSAÇÕES** {#database}

  

### **5.1 Estrutura das Tabelas:**

  

```sql

-- Como o Frappe organiza as tabelas no MariaDB

  

-- 1. Tabela principal do DocType

CREATE TABLE `tabNota Fiscal - Servico` (

`name` varchar(140) NOT NULL PRIMARY KEY, -- ID único

`creation` datetime(6), -- Data criação

`modified` datetime(6), -- Data modificação

`modified_by` varchar(140), -- Usuário que modificou

`owner` varchar(140), -- Proprietário

`docstatus` int(1) DEFAULT 0, -- 0=Draft, 1=Submitted, 2=Cancelled

`idx` int(8), -- Índice

-- Campos específicos do DocType

`empresa` varchar(140),

`entidade_tomador` varchar(140),

`valor_servicos` decimal(18,6),

`valor_iss` decimal(18,6),

`numero_nfse` varchar(140),

`status` varchar(140),

-- Índices

KEY `modified` (`modified`),

KEY `creation` (`creation`)

);

  

-- 2. Tabela filha (Child Table)

CREATE TABLE `tabNota Fiscal Servico Item` (

`name` varchar(140) NOT NULL PRIMARY KEY,

`creation` datetime(6),

`modified` datetime(6),

-- Relacionamento com pai

`parent` varchar(140), -- Nome do documento pai

`parenttype` varchar(140), -- DocType do pai

`parentfield` varchar(140), -- Campo na tabela pai

`idx` int(8), -- Ordem na tabela

-- Campos específicos

`item` varchar(140),

`descricao` text,

`qtde` decimal(18,6),

`valor_unitario` decimal(18,6),

`valor_total` decimal(18,6),

KEY `parent` (`parent`)

);

  

-- 3. Tabela de Singles (DocTypes com uma única instância)

CREATE TABLE `tabSingles` (

`doctype` varchar(140),

`field` varchar(140),

`value` text,

PRIMARY KEY (`doctype`, `field`)

);

  

-- 4. Tabela de DocTypes (metadados)

CREATE TABLE `tabDocType` (

`name` varchar(140) PRIMARY KEY,

`module` varchar(140),

`is_submittable` int(1),

`issingle` int(1),

`istable` int(1),

-- ... outros metadados

);

```

  

### **5.2 Transações e Controle de Concorrência:**

  

```python

# Como o Frappe gerencia transações

  

# 1. Transação automática (padrão)

def salvar_documento():

doc = frappe.get_doc('Nota Fiscal - Servico', 'NF-001')

doc.valor_servicos = 1500.00

doc.save() # Auto-commit ao final

  

# 2. Transação manual

def operacao_complexa():

try:

# Iniciar transação

frappe.db.begin()

# Múltiplas operações

nf = frappe.get_doc('Nota Fiscal - Servico', 'NF-001')

nf.status = 'autorizada'

nf.save()

# Criar título

titulo = frappe.new_doc('Titulo')

titulo.update({

'documento_origem': nf.name,

'valor': nf.valor_servicos

})

titulo.insert()

# Confirmar transação

frappe.db.commit()

except Exception as e:

# Reverter em caso de erro

frappe.db.rollback()

frappe.throw(f"Erro na operação: {str(e)}")

  

# 3. Controle de versionamento

def salvar_com_versao():

doc = frappe.get_doc('Nota Fiscal - Servico', 'NF-001')

# Verificar se não foi modificado por outro usuário

if doc.modified != expected_modified:

frappe.throw("Documento foi modificado por outro usuário")

doc.valor_servicos = 1500.00

doc.save()

  

# 4. Locks de documento

def operacao_com_lock():

# Obter documento com lock (FOR UPDATE)

doc = frappe.get_doc('Nota Fiscal - Servico', 'NF-001', for_update=True)

# Operação longa...

time.sleep(5)

doc.save() # Outros usuários esperam até aqui

```

  

### **5.3 Cache e Performance:**

  

```python

# Sistema de cache do Frappe

  

# 1. Cache de documentos

# Primeiro acesso - busca no banco

doc1 = frappe.get_doc('Customer', 'CUST-001')

  

# Segundo acesso - busca no cache

doc2 = frappe.get_doc('Customer', 'CUST-001') # Mais rápido

  

# 2. Cache de valores

# Cache de valor específico

customer_name = frappe.get_cached_value('Customer', 'CUST-001', 'customer_name')

  

# 3. Cache de listas

# Cache de queries frequentes

items = frappe.get_all('Item',

filters={'disabled': 0},

cache=True, # Usar cache

cache_key='active_items' # Chave específica

)

  

# 4. Limpar cache

frappe.clear_cache(doctype='Customer') # Limpar cache específico

frappe.clear_cache() # Limpar todo cache

  

# 5. Cache Redis para dados de sessão

frappe.cache().set_value('user_preference', user_data, expire=3600)

user_data = frappe.cache().get_value('user_preference')

```

  

---

  

## 🔄 **6. FLUXO COMPLETO DE UM DOCUMENTO** {#fluxo}

  

### **6.1 Criação de Documento - Fluxo Detalhado:**

  

```

👤 USUÁRIO AÇÃO 🖥️ FRONTEND 🐍 BACKEND 💾 DATABASE

│ │ │ │

1. ├─ Clica "Novo" │ │ │

│ ├─ frappe.new_doc() │ │

│ ├─ Carrega form.js │ │

│ ├─ Executa setup() │ │

│ ├─ Executa onload() │ │

│ └─ Executa refresh() │ │

│ │ │

2. ├─ Preenche campos │ │ │

│ ├─ Triggers de campos │ │

│ ├─ valor_servicos() │ │

│ ├─ frappe.call() │ │

│ │ ├─ POST /api/method │ │

│ │ └─ JSON: {doc: {...}} │ │

│ ├─ @frappe.whitelist() │

│ ├─ calculo_imposto_nfse() │

│ ├─ Business logic │

│ └─ return {...} │

│ ├─ response.message │ │

│ └─ Atualiza campos UI │ │

│ │ │

3. ├─ Clica "Salvar" │ │ │

│ ├─ Executa validate() │ │

│ ├─ frm.save() │ │

│ ├─ POST /api/resource │ │

│ └─ JSON: documento completo │ │

│ ├─ frappe.get_doc() │

│ ├─ doc.insert() │

│ │ ├─ before_insert() │

│ │ ├─ validate() │

│ │ ├─ before_save() │

│ │ ├─ SQL INSERT ├─ INSERT tabNF...

│ │ ├─ after_insert() │

│ │ └─ on_update() │

│ └─ return doc.as_dict() │

│ ├─ Sucesso! │ │

│ ├─ Atualiza form │ │

│ └─ Executa after_save() │ │

│ │ │

4. ├─ Form atualizado │ │ │

└─ Documento salvo │ │ │

```

  

### **6.2 Atualização de Documento - Fluxo:**

  

```

👤 USUÁRIO 🖥️ FRONTEND 🐍 BACKEND 💾 DATABASE

│ │ │ │

1. ├─ Abre documento │ │ │

│ ├─ GET /api/resource │ │

│ │ ├─ frappe.get_doc() │

│ │ │ └─ SQL SELECT ├─ SELECT * FROM...

│ ├─ Carrega dados │ │

│ ├─ Executa onload() │ │

│ └─ Executa refresh() │ │

│ │ │

2. ├─ Modifica campo │ │ │

│ ├─ Trigger do campo │ │

│ └─ Lógica de negócio │ │

│ │ │

3. ├─ Salva alterações │ │ │

│ ├─ PUT /api/resource │ │

│ │ ├─ doc.save() │

│ │ │ ├─ before_save() │

│ │ │ ├─ validate() │

│ │ │ ├─ check_if_latest() │

│ │ │ ├─ SQL UPDATE ├─ UPDATE tabNF...

│ │ │ ├─ on_update() │

│ │ │ └─ after_save() │

│ └─ Form atualizado │ │

```

  

### **6.3 Submissão de Documento (Workflow):**

  

```

👤 USUÁRIO 🖥️ FRONTEND 🐍 BACKEND 💾 DATABASE

│ │ │ │

1. ├─ Clica "Submit" │ │ │

│ ├─ Validações client │ │

│ ├─ POST /api/method │ │

│ │ ├─ doc.submit() │

│ │ │ ├─ validate() │

│ │ │ ├─ before_submit() │

│ │ │ ├─ SET docstatus = 1 ├─ UPDATE...docstatus=1

│ │ │ ├─ on_submit() │

│ │ │ └─ after_submit() │

│ └─ Status atualizado │ │

│ │ │

2. ├─ Documento submetido │ │ │

│ (não pode mais editar) │ │ │

```

  

---

  

## 💡 **7. EXEMPLOS PRÁTICOS** {#exemplos}

  

### **7.1 Exemplo Completo: Sistema de Nota Fiscal**

  

#### **Frontend (nota_fiscal___servico.js):**

```javascript

frappe.ui.form.on('Nota Fiscal - Servico', {

// 1. Configuração inicial

setup: function(frm) {

// Filtrar apenas empresas ativas

frm.set_query('empresa', function() {

return {

filters: { disabled: 0 }

};

});

// Filtrar clientes

frm.set_query('entidade_tomador', function() {

return {

filters: {

disabled: 0,

is_supplier: 0

}

};

});

},

// 2. Carregamento do documento

onload: function(frm) {

if (frm.is_new()) {

// Documento novo - valores padrão

frm.set_value('data_emissao', frappe.datetime.now_date());

frm.set_value('empresa', frappe.defaults.get_user_default('Company'));

}

},

// 3. Refresh - sempre executado

refresh: function(frm) {

// Toggle de seções baseado no status

frm.toggle_display('section_impostos', frm.doc.valor_servicos > 0);

// Botões customizados

if (frm.doc.docstatus === 0 && frm.doc.status !== 'autorizada') {

frm.add_custom_button(__('Autorizar NFSe'), function() {

frm.call('autorizar_nfse').then(function() {

frm.refresh();

});

}, __('Actions'));

}

// Indicadores visuais

if (frm.doc.status === 'autorizada') {

frm.dashboard.add_indicator(__('Autorizada'), 'green');

}

},

// 4. Triggers de campos

valor_servicos: function(frm) {

if (frm.doc.valor_servicos) {

// Atualizar base de cálculo

frm.set_value('base_calculo', frm.doc.valor_servicos);

// Calcular impostos automaticamente

frm.trigger('calcular_impostos');

}

},

aliquota_iss: function(frm) {

frm.trigger('calcular_impostos');

},

// 5. Método customizado

calcular_impostos: function(frm) {

if (frm.doc.valor_servicos && frm.doc.aliquota_iss) {

frappe.call({

method: 'nxlite.scripts.calculo_imposto.calculo_imposto_nfse',

args: { doc: frm.doc },

callback: function(r) {

if (r.message) {

// Atualizar múltiplos campos

$.each(r.message, function(field, value) {

frm.set_value(field, value);

});

// Refresh da seção de impostos

frm.refresh_fields(['valor_iss', 'valor_liquido']);

}

}

});

}

},

// 6. Validação client-side

validate: function(frm) {

// Validações obrigatórias

if (!frm.doc.entidade_tomador) {

frappe.throw(__('Entidade Tomador é obrigatória'));

}

if (frm.doc.valor_servicos <= 0) {

frappe.throw(__('Valor dos serviços deve ser maior que zero'));

}

// Validar tabela de itens

if (frm.doc.tab_servicos && frm.doc.tab_servicos.length > 0) {

let total_itens = 0;

frm.doc.tab_servicos.forEach(function(item) {

if (!item.descricao) {

frappe.throw(__('Descrição é obrigatória para todos os itens'));

}

total_itens += item.valor_total || 0;

});

// Verificar se total dos itens bate com valor total

if (Math.abs(total_itens - frm.doc.valor_servicos) > 0.01) {

frappe.throw(__('Total dos itens não confere com valor dos serviços'));

}

}

}

});

  

// Eventos da tabela filha

frappe.ui.form.on('Nota Fiscal Servico Item', {

// Adicionar linha

tab_servicos_add: function(frm, cdt, cdn) {

frappe.msgprint(__('Item adicionado'));

frm.trigger('recalcular_total');

},

// Remover linha

tab_servicos_remove: function(frm, cdt, cdn) {

frm.trigger('recalcular_total');

},

// Campos da linha

qtde: function(frm, cdt, cdn) {

calcular_total_item(frm, cdt, cdn);

},

valor_unitario: function(frm, cdt, cdn) {

calcular_total_item(frm, cdt, cdn);

}

});

  

// Função auxiliar

function calcular_total_item(frm, cdt, cdn) {

let item = locals[cdt][cdn];

let total = (item.qtde || 0) * (item.valor_unitario || 0);

frappe.model.set_value(cdt, cdn, 'valor_total', total);

// Recalcular total geral

frm.trigger('recalcular_total');

}

```

  

#### **Backend (nota_fiscal___servico.py):**

```python

import frappe

from frappe.model.document import Document

from decimal import Decimal, ROUND_HALF_UP

from frappe.utils import now

  

class NotaFiscalServico(Document):

def validate(self):

"""Validações principais"""

self.validar_campos_obrigatorios()

self.validar_valores()

self.calcular_impostos()

self.validar_totais()

def before_save(self):

"""Antes de salvar"""

self.set_title()

self.atualizar_discriminacao()

def on_update(self):

"""Após atualizar"""

self.verificar_mudanca_status()

def before_submit(self):

"""Antes de submeter"""

if self.status != 'autorizada':

frappe.throw("Apenas NFSe autorizadas podem ser submetidas")

def on_submit(self):

"""Após submeter"""

self.criar_titulo_a_receber()

self.atualizar_numeracao()

def validar_campos_obrigatorios(self):

"""Validar campos obrigatórios"""

campos_obrigatorios = [

'empresa', 'entidade_tomador', 'valor_servicos'

]

for campo in campos_obrigatorios:

if not self.get(campo):

frappe.throw(f"Campo {frappe.get_meta(self.doctype).get_label(campo)} é obrigatório")

def validar_valores(self):

"""Validar valores numéricos"""

if self.valor_servicos <= 0:

frappe.throw("Valor dos serviços deve ser maior que zero")

# Validar alíquotas

if self.aliquota_iss and (self.aliquota_iss < 0 or self.aliquota_iss > 100):

frappe.throw("Alíquota ISS deve estar entre 0 e 100%")

def calcular_impostos(self):

"""Calcular impostos"""

# Importar função de cálculo

from nxlite.scripts.calculo_imposto import calculo_imposto_nfse

calculo_imposto_nfse(self)

def validar_totais(self):

"""Validar se totais conferem"""

if self.tab_servicos:

total_itens = sum(item.valor_total or 0 for item in self.tab_servicos)

if abs(total_itens - self.valor_servicos) > 0.01:

frappe.throw("Total dos itens não confere com valor dos serviços")

def set_title(self):

"""Definir título do documento"""

if self.entidade_tomador:

cliente = frappe.get_cached_value('Customer', self.entidade_tomador, 'customer_name')

self.title = f"{cliente} - {frappe.format(self.valor_servicos, {'fieldtype': 'Currency'})}"

def verificar_mudanca_status(self):

"""Verificar mudanças de status"""

if self.has_value_changed('status'):

if self.status == 'autorizada':

self.processar_autorizacao()

def processar_autorizacao(self):

"""Processar autorização da NFSe"""

# Validar se pode autorizar

if not self.numero_nfse:

self.numero_nfse = self.obter_proximo_numero()

# Log da autorização

frappe.logger().info(f"NFSe {self.name} autorizada com número {self.numero_nfse}")

# Evento real-time

frappe.publish_realtime('nfse_autorizada', {

'doctype': self.doctype,

'name': self.name,

'numero_nfse': self.numero_nfse

}, user=self.owner)

def obter_proximo_numero(self):

"""Obter próximo número de NFSe"""

# Buscar configuração da empresa

config = frappe.get_doc('NF Config', self.empresa)

proximo_numero = config.proximo_numero_nfse_producao_focus

# Incrementar

config.proximo_numero_nfse_producao_focus = proximo_numero + 1

config.save()

return proximo_numero

def criar_titulo_a_receber(self):

"""Criar título a receber"""

if self.previsao_de_parcelas:

from nxlite.nx_financeiro.doctype.titulo.titulo import criar_titulos_financeiros

criar_titulos_financeiros(

doctype=self.doctype,

docname=self.name,

empresa=self.empresa,

entidade=self.entidade_tomador,

parcelas=self.previsao_de_parcelas,

servicos=self.tab_servicos or []

)

@frappe.whitelist()

def autorizar_nfse(self):

"""Método público para autorizar NFSe"""

if self.status == 'autorizada':

frappe.throw("NFSe já está autorizada")

# Validações específicas

if not self.valor_servicos:

frappe.throw("Valor dos serviços é obrigatório para autorização")

# Autorizar

self.status = 'autorizada'

self.data_autorizacao = now()

self.save()

frappe.msgprint("NFSe autorizada com sucesso!", indicator='green')

return {

'status': 'success',

'numero_nfse': self.numero_nfse

}

  

# Funções auxiliares

@frappe.whitelist()

def get_dados_empresa(empresa):

"""Buscar dados da empresa"""

return frappe.get_cached_value('Company', empresa, [

'company_name', 'default_currency', 'tax_id'

], as_dict=True)

  

@frappe.whitelist()

def calcular_impostos_preview(doc):

"""Preview de cálculo de impostos"""

from nxlite.scripts.calculo_imposto import calculo_imposto_nfse

# Criar cópia temporária

temp_doc = frappe.new_doc('Nota Fiscal - Servico')

temp_doc.update(doc)

# Calcular

calculo_imposto_nfse(temp_doc)

return {

'valor_iss': temp_doc.valor_iss,

'valor_liquido': temp_doc.valor_liquido,

'percentual_total_tributos': temp_doc.percentual_total_tributos

}

```

  

---

  

## 🎯 **8. MELHORES PRÁTICAS** {#praticas}

  

### **8.1 Performance e Otimização:**

  

```javascript

// ❌ EVITAR - Múltiplas chamadas desnecessárias

frappe.ui.form.on('DocType', {

field1: function(frm) {

frappe.call({method: 'calculate'}); // Chamada 1

},

field2: function(frm) {

frappe.call({method: 'calculate'}); // Chamada 2

}

});

  

// ✅ MELHOR - Debounce de chamadas

frappe.ui.form.on('DocType', {

field1: function(frm) {

frm.trigger('recalculate');

},

field2: function(frm) {

frm.trigger('recalculate');

},

recalculate: frappe.utils.debounce(function(frm) {

frappe.call({method: 'calculate'}); // Uma única chamada

}, 300) // 300ms de delay

});

```

  

```python

# ❌ EVITAR - Queries N+1

def processar_documentos():

docs = frappe.get_all('DocType', fields=['name'])

for doc in docs:

# Query para cada documento

customer = frappe.get_value('Customer', doc.customer, 'customer_name')

  

# ✅ MELHOR - Batch queries

def processar_documentos():

docs = frappe.get_all('DocType', fields=['name', 'customer'])

customers = frappe.get_all('Customer',

filters={'name': ['in', [d.customer for d in docs]]},

fields=['name', 'customer_name']

)

customer_map = {c.name: c.customer_name for c in customers}

```

  

### **8.2 Tratamento de Erros:**

  

```javascript

// Frontend - Tratamento robusto

frappe.ui.form.on('DocType', {

custom_method: function(frm) {

frappe.call({

method: 'my_app.api.custom_method',

args: { doc: frm.doc },

freeze: true, // Mostrar loading

freeze_message: __('Processando...'),

callback: function(r) {

if (r.message) {

frappe.msgprint(__('Sucesso!'), __('Success'));

}

},

error: function(r) {

// Tratar erro específico

if (r.exc_type === 'ValidationError') {

frappe.msgprint(__('Dados inválidos'), __('Validation Error'));

} else {

frappe.msgprint(__('Erro inesperado'), __('Error'));

}

}

});

}

});

```

  

```python

# Backend - Logs e tratamento

import frappe

from frappe import _, ValidationError

  

def processar_documento(doc):

try:

# Validações

if not doc.get('required_field'):

frappe.throw(_("Campo obrigatório não preenchido"), ValidationError)

# Processamento

resultado = executar_logica_complexa(doc)

# Log de sucesso

frappe.logger().info(f"Documento {doc.name} processado com sucesso")

return resultado

except ValidationError:

# Re-raise validation errors

raise

except Exception as e:

# Log do erro

frappe.log_error(

title=f"Erro ao processar {doc.name}",

message=frappe.get_traceback()

)

# Erro amigável para usuário

frappe.throw(_("Ocorreu um erro no processamento. Tente novamente."))

```

  

### **8.3 Segurança:**

  

```python

# Validação de permissões

@frappe.whitelist()

def metodo_sensivel(docname):

# Verificar permissão explicitamente

doc = frappe.get_doc('DocType', docname)

if not doc.has_permission('write'):

frappe.throw(_("Sem permissão para esta operação"))

# Validar usuário

if frappe.session.user == 'Guest':

frappe.throw(_("Login necessário"))

# Continuar processamento...

```

  

### **8.4 Código Limpo e Manutenível:**

  

```javascript

// ✅ BOM - Código organizado e comentado

frappe.ui.form.on('Nota Fiscal - Servico', {

refresh: function(frm) {

setup_custom_buttons(frm);

update_status_indicators(frm);

toggle_field_visibility(frm);

},

valor_servicos: function(frm) {

if (should_calculate_taxes(frm)) {

calculate_taxes_async(frm);

}

}

});

  

// Funções auxiliares bem definidas

function setup_custom_buttons(frm) {

if (can_authorize(frm)) {

frm.add_custom_button(__('Autorizar'), () => authorize_nfse(frm));

}

}

  

function should_calculate_taxes(frm) {

return frm.doc.valor_servicos > 0 && frm.doc.empresa;

}

  

async function calculate_taxes_async(frm) {

try {

const result = await frappe.call({

method: 'calculate_taxes',

args: { doc: frm.doc }

});

update_tax_fields(frm, result.message);

} catch (error) {

handle_calculation_error(error);

}

}

```

  

---

  

## 🎓 **CONCLUSÃO**

  

Este guia aborda os principais aspectos do funcionamento do Frappe Framework:

  

1. **Frontend**: Como os `form.js` scripts funcionam e quando os triggers são executados

2. **Comunicação**: Como dados transitam entre frontend e backend via API

3. **Backend**: O ciclo de vida dos documentos e os hooks disponíveis

4. **Database**: Como a persistência é gerenciada e transações controladas

5. **Fluxo Completo**: A jornada de um documento do usuário ao banco de dados

  

### **Pontos-chave para lembrar:**

  

- **Triggers seguem ordem específica**: setup → onload → refresh → field_changes → validate → save

- **Comunicação é assíncrona**: Use `frappe.call()` e `callbacks` adequadamente

- **Document Lifecycle é bem definido**: before_insert → validate → after_insert → on_update

- **Performance importa**: Evite queries N+1 e use cache quando apropriado

- **Segurança é fundamental**: Sempre valide permissões em métodos whitelisted

  

Com este conhecimento, você pode desenvolver aplicações robustas e performáticas no Frappe Framework! 🚀

  

---

  

*Este guia foi criado baseado na análise do código real do workspace e nas melhores práticas da comunidade Frappe.*
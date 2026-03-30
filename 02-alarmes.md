# Tela 2 — Alarmes

## Objetivo

Listar, filtrar, detalhar e reconhecer alarmes do sistema de monitoramento da subestacao IEC 61850, conforme a taxonomia de alarmes GOOSE v1.2. Os alarmes sao classificados em tres tipos — PRESENCE (presenca/ausencia de stream), STATIC_INTEGRITY (divergencia entre PCAP e SCD) e DYNAMIC_INTEGRITY (anomalias de comportamento do stream) — e seguem uma maquina de estados (ACTIVE -> ACKNOWLEDGED -> CLEARED). Esta tela e o ponto central de operacao para que operadores identifiquem anomalias detectadas pelos agentes de monitoramento e executem o reconhecimento (ACK) dos alarmes conforme procedimento operacional.

A subestacao monitorada possui 4 IEDs — L90_DIG (rele de protecao de linha), MU320E_LAB (merging unit), MU_360_1 (merging unit) e T60_DIG (rele de protecao de transformador) — com protocolos GOOSE, SV, PTP, MMS, SNMP e SYSTEM.

---

## Wireframe

### Estado principal (com dados)

```
+----------------------------------------------------------------------------------------------------+
|  +--- Header Global (ver 00-navegacao-global.md) -------------------------------------------+      |
|  |  [Logo NMS]   Subestacao: MU_360_1     Mon: * RUNNING          operador01 [Logout]       |      |
|  +-------------------------------------------------------------------------------------------+     |
|                                                                                                    |
|  +--- Barra de Filtros --------------------------------------------------------------------+       |
|  |                                                                                         |       |
|  |  {Protocolo v}  {Severidade v}  {Estado v}  {Metodo v}  {Tipo v}                        |       |
|  |  [De: ____]  [Ate: ____]  [Limpar] [Refresh]                                            |       |
|  |                                                                                         |       |
|  |  Opcoes:        Opcoes:       Opcoes:         Opcoes:    Opcoes:                        |       |
|  |  o Todos        o Todos       o Todos         o Todos    o Todos                        |       |
|  |  o SV           o LOW         o ACTIVE        o pcap     o PRESENCE                     |       |
|  |  o GOOSE        o MEDIUM      o ACKNOWLEDGED  o mms      o STATIC_INTEGRITY             |       |
|  |  o PTP          o HIGH        o CLEARED                  o DYNAMIC_INTEGRITY            |       |
|  |  o MMS          o CRITICAL                                                              |       |
|  |  o SNMP                                                                                 |       |
|  |  o SYSTEM                                                                               |       |
|  |                                                                                         |       |
|  +-----------------------------------------------------------------------------------------+       |
|                                                                                                    |
|  +--- Tabela de Alarmes ---------------------------------------------------------------+           |
|  |                                                                                     |           |
|  |  Timestamp           | Tipo              | Sev.        | Resumo              |Oc|Est|           |
|  |  --------------------+-------------------+-------------+---------------------+--+---+           |
|  |  19/02/2026 14:32:01 | PRESENCE          | N CRITICAL  | Ausencia GOOSE rede | 3| A |           |
|  |  ####################|###################|#############| GoCB01 L90_DIG      |##|###|           |
|  |                      |                   |             |                     |  |   |           |
|  |  19/02/2026 14:30:15 | STATIC_INTEGRITY  | ! HIGH      | VLAN divergente     | 1| K |           |
|  |                      |                   |             | GoCB01 L90_DIG      |  |   |           |
|  |                      |                   |             |                     |  |   |           |
|  |  19/02/2026 14:25:00 | DYNAMIC_INTEGRITY | o MEDIUM    | Clock nao sincron.  | 2| A |           |
|  |                      |                   |             | GoCB01 L90_DIG      |  |   |           |
|  |                      |                   |             |                     |  |   |           |
|  |  19/02/2026 13:10:44 | DYNAMIC_INTEGRITY | ! HIGH      | Pacotes perdidos    |12| K |           |
|  |                      |                   |             | GoCB01 MU_360_1     |  |   |           |
|  |                      |                   |             |                     |  |   |           |
|  +-------------------------------------------------------------------------------------+           |
|                                                                                                    |
|  Legenda coluna Est.:   A = Ativo  |  K = Reconhecido  |  C = Encerrado                           | 
|  Legenda de severidade: N CRITICAL (vermelho) | ! HIGH (laranja) | o MEDIUM (amarelo)             | 
|                         o LOW (azul)                                                               |
|                                                                                                    |
|  Nota: Linhas com state=ACTIVE (A) possuem fundo destacado (ex: amarelo palido)                   | 
|  Nota: A linha ### indica destaque visual para alarmes CRITICAL com state=ACTIVE                   |
|                                                                                                    |
|  +--- Paginacao -----------------------------------------------------------------------+           |
|  |                                                                                     |           |
|  |  Mostrando 1-4 de 4 alarmes   [25|50|100]              < 1 2 3 ... N >              |           |
|  |                                                                                     |           |
|  +-------------------------------------------------------------------------------------+           |
|                                                                                                    |
+---------------------------------------------------------------------------------------------------+ 
```

**Comportamento da tabela:**

- Ordenacao padrao: `updated_at` descendente (mais recente primeiro)
- Colunas ordenaveis: Timestamp, Tipo, Severidade, Ocorrencias
- Clique na linha abre o drawer de detalhe (ver secao seguinte)
- Coluna "Oc." exibe o campo `occurrences` do alarme
- Coluna "Est." exibe estado abreviado: A (ACTIVE), K (ACKNOWLEDGED), C (CLEARED) com base no campo `state`
- Coluna "Resumo" e gerada no frontend a partir de `alarm_code` + campos de `details` (nao e um campo do payload)
- Badge de severidade com cor: CRITICAL=vermelho, HIGH=laranja, MEDIUM=amarelo, LOW=azul
- Linhas com `state=ACTIVE` possuem fundo levemente destacado para chamar atencao do operador
- Seletor de pageSize permite alternar entre 25, 50 e 100 itens por pagina

---

### Estado do drawer (detalhe do alarme)

Abre pela direita quando o usuario clica em uma linha da tabela. Ocupa aproximadamente 40% da largura da tela. A tabela principal permanece visivel ao fundo (com overlay escurecido ou nao, a criterio do designer).

```
+----------------------------------------------------+
|                                                    |
|  +--- Drawer: Detalhe do Alarme ----------[X]-- +  |
|  |                                              |  |
|  |  Ausencia de stream GOOSE   +-------------+  |  |
|  |  na rede                    | N CRITICAL  |  |  |
|  |                             +-------------+  |  |
|  |                                              |  |
|  |  ------------------------------------------  |  |
|  |                                              |  |
|  |  Alarm ID                                    |  |
|  |  a1b2c3d4-e5f6-7890-abcd-ef1234567890        |  |
|  |                                              |  |
|  |  Criado em                                   |  |
|  |  2026-02-19T14:32:01Z                        |  |
|  |                                              |  |
|  |  Atualizado em                               |  |
|  |  2026-02-19T14:32:45Z                        |  |
|  |                                              |  |
|  |  Tipo de Alarme                              |  |
|  |  PRESENCE                                    |  |
|  |                                              |  |
|  |  Codigo                                      |  |
|  |  GOOSE_PRESENCE_ABSENCE_NETWORK              |  |
|  |                                              |  |
|  |  Severidade                                  |  |
|  |  N CRITICAL                                  |  |
|  |                                              |  |
|  |  Estado                                      |  |
|  |  A ACTIVE                                    |  |
|  |                                              |  |
|  |  Ocorrencias                                 |  |
|  |  3                                           |  |
|  |                                              |  |
|  |  Metodo                                      |  |
|  |  pcap                                        |  |
|  |                                              |  |
|  |  Expira em                                   |  |
|  |  2026-02-19T14:38:01Z                        |  |
|  |                                              |  |
|  |  ------------------------------------------  |  |
|  |                                              |  |
|  |  Detalhes                                    |  |
|  |  +------------------+---------------------+  |  |
|  |  | Chave            | Valor               |  |  |
|  |  +------------------+---------------------+  |  |
|  |  | control_block    | L90_DIGMaster/LLN0  |  |  |
|  |  |                  | $GO$GoCB01          |  |  |
|  |  | app_id           | 0x0001              |  |  |
|  |  | dataset          | L90_DIGMaster/LLN0  |  |  |
|  |  |                  | $TT6DataSet1        |  |  |
|  |  | goose_id         | TxGOOSE1            |  |  |
|  |  | dest_mac         | 01:0c:cd:01:00:00   |  |  |
|  |  | vlan_id          | 0x0837              |  |  |
|  |  | conf_rev         | 1                   |  |  |
|  |  +------------------+---------------------+  |  |
|  |                                              |  |
|  |  Nota: a secao Detalhes e renderizada        |  |
|  |  dinamicamente como tabela chave-valor.      |  |
|  |  O conteudo varia conforme o alarm_type      |  |
|  |  e alarm_code (ver details do payload).      |  |
|  |                                              |  |
|  |  ------------------------------------------  |  |
|  |                                              |  |
|  |  Metricas (snapshot)                         |  |
|  |  +------------------+---------------------+  |  |
|  |  | Chave            | Valor               |  |  |
|  |  +------------------+---------------------+  |  |
|  |  | value            | 15                  |  |  |
|  |  | threshold        | 10                  |  |  |
|  |  | first_occurrence  | 2026-02-19T14:31:  |  |  |
|  |  |   _ts            |  58Z                |  |  |
|  |  +------------------+---------------------+  |  |
|  |                                              |  |
|  |  Nota: Secao Metricas visivel apenas para    |  |
|  |  DYNAMIC_INTEGRITY com snapshot != null.     |  |
|  |                                              |  |
|  |  ------------------------------------------  |  |
|  |                                              |  |
|  |  Captura de Pacotes                          |  |
|  |  < Ver captura de pacotes >                  |  |
|  |  (link visivel se pcap_id != null)           |  |
|  |  (se pcap_id == null: "Sem captura           |  |
|  |   associada" em texto cinza)                 |  |
|  |  (pcap_id e null para alarmes MMS-only)      |  |
|  |                                              |  |
|  |  ------------------------------------------  |  |
|  |                                              |  |
|  |  +----------------------------------------+  |  |
|  |  |         [Reconhecer Alarme]            |  |  |
|  |  +----------------------------------------+  |  |
|  |                                              |  |
|  |  (ADMIN/OPERATOR: botao visivel e ativo)     |  |
|  |  (VIEWER: botao oculto)                      |  |
|  |  (Se state != ACTIVE: botao desabilitado     |  |
|  |   com texto "Reconhecido por {user}          |  |
|  |   em {data}")                                |  |
|  |                                              |  |
|  +----------------------------------------------+  |
|                                                    |
+----------------------------------------------------+
```

**Apos reconhecimento bem-sucedido (ACK):**

```
+----------------------------------------------------+
|  +--- Drawer (pos-ACK) ------------------[X]--  +  |
|  |                                              |  |
|  |  ...campos do alarme...                      |  |
|  |                                              |  |
|  |  Estado                                      |  |
|  |  K ACKNOWLEDGED                              |  |
|  |    por: operador@empresa.com                 |  |
|  |    em: 19/02/2026 14:35:12 UTC               |  |
|  |                                              |  |
|  |  ------------------------------------------  |  |
|  |                                              |  |
|  |  +--------------------------------------+    |  |
|  |  | [Reconhecer Alarme (desabilitado)]   |    |  |
|  |  +--------------------------------------+    |  |
|  |                                              |  |
|  |  +--------------------------------------+    |  |
|  |  | * Alarme reconhecido com sucesso     |    |  |
|  |  +--------------------------------------+    |  |
|  |  (feedback inline - badge verde,             |  |
|  |   exibido por 5 segundos)                    |  |
|  |                                              |  |
|  +----------------------------------------------+  |
+----------------------------------------------------+
```

**Apos conflito de ACK (409):**

```
+----------------------------------------------------+
|  +--- Drawer (conflito ACK) -------------[X]--  +  |
|  |                                              |  |
|  |  ...campos do alarme...                      |  |
|  |                                              |  |
|  |  +--------------------------------------+    |  |
|  |  | [Reconhecer Alarme (desabilitado)]   |    |  |
|  |  +--------------------------------------+    |  |
|  |                                              |  |
|  |  +--------------------------------------+    |  |
|  |  | ! Alarme ja reconhecido por          |    |  |
|  |  |   outro operador                     |    |  |
|  |  +--------------------------------------+    |  |
|  |  (mensagem inline - badge amarelo)           |  |
|  |                                              |  |
|  +----------------------------------------------+  |
+----------------------------------------------------+
```

---

### Estado vazio

```
+---------------------------------------------------------------------------------------------------+ 
|  +--- Header Global --------------------------------------------------------------------+         | 
|  |  [Logo NMS]   Dashboard | Alarmes | Topologia | PCAPs | Config    [User v] [Sair]    |         | 
|  +-------------------------------------------------------------------------------------------+    | 
|                                                                                                    |
|  +--- Barra de Filtros --------------------------------------------------------------------+       |
|  |  {Protocolo v}  {Severidade v}  {Estado v}  {Metodo v}  {Tipo v}                        |       |
|  |  [De: ____]  [Ate: ____]  [Limpar] [Refresh]                                            |       |
|  +-----------------------------------------------------------------------------------------+       |
|                                                                                                    |
|                                                                                                    |
|                          +--------------------------+                                              |
|                          |                          |                                              |
|                          |      (icone: sino        |                                              |
|                          |       com "Z" ou         |                                              |
|                          |       check verde)       |                                              |
|                          |                          |                                              |
|                          |  Nenhum alarme           |                                              |
|                          |  registrado              |                                              |
|                          |                          |                                              |
|                          +--------------------------+                                              |
|                                                                                                    |
|                                                                                                    |
+---------------------------------------------------------------------------------------------------+ 
```

**Estado de filtro sem resultados:**

```
+---------------------------------------------------------------------------------------------------+ 
|  +--- Barra de Filtros (filtros ativos indicados por badge/cor) ----------------------------+      |
|  |  {GOOSE v}  {CRITICAL v}  {ACTIVE v}  {pcap v}  {PRESENCE v}                            |      | 
|  |  [De: ____]  [Ate: ____]  [Limpar] [Refresh]                                            |      | 
|  +-----------------------------------------------------------------------------------------+      | 
|                                                                                                    |
|                          +----------------------------------+                                      |
|                          |                                  |                                      |
|                          |      (icone: lupa vazia)         |                                      |
|                          |                                  |                                      |
|                          |  Nenhum alarme para os           |                                      |
|                          |  filtros selecionados            |                                      |
|                          |                                  |                                      |
|                          |  [Limpar filtros]                |                                      |
|                          |                                  |                                      |
|                          +----------------------------------+                                      |
|                                                                                                    |
+---------------------------------------------------------------------------------------------------+ 
```

---

## Componentes

| Componente | Fonte de dados | Formato de exibicao |
|---|---|---|
| **Barra de filtros** | Valores estaticos (enums) + inputs do usuario | Dropdowns com opcoes pre-definidas; campos de data com date-picker |
| **Dropdown Protocolo** | `message_type`: GOOSE, SV, PTP + legado: MMS, SNMP, SYSTEM | Select com opcao "Todos" como padrao |
| **Dropdown Severidade** | `severity`: LOW, MEDIUM, HIGH, CRITICAL | Select com opcao "Todos" como padrao; cada opcao com cor indicativa |
| **Dropdown Estado** | `state`: ACTIVE, ACKNOWLEDGED, CLEARED | Select com opcao "Todos" como padrao |
| **Dropdown Metodo** | `method`: pcap, mms | Select com opcao "Todos" como padrao |
| **Dropdown Tipo** | `alarm_type`: PRESENCE, STATIC_INTEGRITY, DYNAMIC_INTEGRITY | Select com opcao "Todos" como padrao |
| **Campo De/Ate** | Input do usuario | Date-time picker; envia como `from`/`to` ISO 8601 |
| **Botao Limpar filtros** | -- | Reseta todos os filtros ao estado padrao; re-executa busca sem filtros |
| **Botao Refresh** | -- | Re-executa a busca com os filtros atuais; util para atualizar a lista manualmente |
| **Tabela de alarmes** | `GET /api/v1/alarms` -> `AlarmListResponse.items[]` | Tabela com linhas clicaveis |
| **Coluna Timestamp** | `alarm.updated_at` | Formato local: `DD/MM/AAAA HH:mm:ss` (convertido de UTC) |
| **Coluna Tipo** | `alarm.alarm_type` | Texto monoespacado (ex: `PRESENCE`) |
| **Coluna Severidade** | `alarm.severity` | Badge colorido: N CRITICAL (vermelho), ! HIGH (laranja), o MEDIUM (amarelo), o LOW (azul) |
| **Coluna Resumo** | Gerado no frontend a partir de `alarm_code` + `details` | Texto corrido; truncado com ellipsis se muito longo |
| **Coluna Ocorrencias** | `alarm.occurrences` | Numero inteiro |
| **Coluna Estado** | `alarm.state` | Letra abreviada: A (ACTIVE), K (ACKNOWLEDGED), C (CLEARED) |
| **Seletor de pageSize** | Valores fixos: 25, 50, 100 | Botoes inline ao lado da paginacao |
| **Paginacao** | `AlarmListResponse.meta` | "Mostrando X-Y de Z alarmes" + navegacao de paginas |
| **Drawer de detalhe** | `GET /api/v1/alarms/{alarm_id}` -> `GooseAlarm` | Painel lateral direito (~40% largura) |
| **Secao Detalhes no drawer** | `alarm.details` (campos especificos por alarm_type) | Tabela chave-valor renderizada dinamicamente |
| **Secao Metricas no drawer** | `alarm.snapshot` (objeto com value, threshold, first_occurrence_ts) | Tabela chave-valor; visivel apenas para DYNAMIC_INTEGRITY com snapshot != null |
| **Link PCAP** | `alarm.pcap_id` | Link "Ver captura de pacotes" se `pcap_id != null`; texto cinza "Sem captura associada" caso contrario. `pcap_id` e null para alarmes MMS-only |
| **Botao Reconhecer** | `POST /api/v1/alarms/{alarm_id}/ack` | Botao primario; estados: ativo (state=ACTIVE), desabilitado (state != ACTIVE), oculto (role VIEWER) |
| **Feedback ACK sucesso** | `AckResponse` | Badge verde inline com dados de quem reconheceu e quando |
| **Feedback ACK conflito** | HTTP 409 `ErrorResponse` | Badge amarelo inline: "Alarme ja reconhecido por outro operador" |
| **Banner de erro de API** | HTTP 5xx / erro de rede | Banner vermelho no topo da tabela |
| **Estado vazio** | `AlarmListResponse.meta.total == 0` sem filtros | Ilustracao + "Nenhum alarme registrado" |
| **Estado filtro vazio** | `AlarmListResponse.meta.total == 0` com filtros ativos | Ilustracao + "Nenhum alarme para os filtros selecionados" + botao [Limpar filtros] |

---

## Dados e Endpoints

| Endpoint | Metodo | Uso na tela | Campos utilizados | Exemplo de chamada |
|---|---|---|---|---|
| `/api/v1/alarms` | GET | Carregar lista de alarmes (paginada, filtrada, ordenada) | Query: `message_type`, `alarm_type`, `severity`, `state`, `method`, `from`, `to`, `page`, `pageSize`, `sort`, `order` | `GET /api/v1/alarms?page=1&pageSize=50&order=desc&sort=updated_at` |
| `/api/v1/alarms` | GET | Filtrar por protocolo | Query: `message_type=GOOSE` | `GET /api/v1/alarms?message_type=GOOSE&page=1&pageSize=50` |
| `/api/v1/alarms` | GET | Filtrar por severidade | Query: `severity=CRITICAL` | `GET /api/v1/alarms?severity=CRITICAL&page=1&pageSize=50` |
| `/api/v1/alarms` | GET | Filtrar por estado | Query: `state=ACTIVE` | `GET /api/v1/alarms?state=ACTIVE&page=1&pageSize=50` |
| `/api/v1/alarms` | GET | Filtrar por metodo | Query: `method=pcap` | `GET /api/v1/alarms?method=pcap&page=1&pageSize=50` |
| `/api/v1/alarms` | GET | Filtrar por tipo de alarme | Query: `alarm_type=PRESENCE` | `GET /api/v1/alarms?alarm_type=PRESENCE&page=1&pageSize=50` |
| `/api/v1/alarms` | GET | Filtrar por periodo | Query: `from`, `to` | `GET /api/v1/alarms?from=2026-02-19T00:00:00Z&to=2026-02-19T23:59:59Z` |
| `/api/v1/alarms/{alarm_id}` | GET | Carregar detalhe completo do alarme no drawer | Response: `GooseAlarm` (estrutura flat com todos os campos) | `GET /api/v1/alarms/a1b2c3d4-e5f6-7890-abcd-ef1234567890` |
| `/api/v1/alarms/{alarm_id}/ack` | POST | Reconhecer alarme | Response 200: `AckResponse` (alarm_id, state, acknowledged_at, acknowledged_by); Response 409: `ErrorResponse` | `POST /api/v1/alarms/a1b2c3d4-e5f6-7890-abcd-ef1234567890/ack` |

**Schemas utilizados:**

| Schema | Campos | Observacoes |
|---|---|---|
| `GooseAlarm` | `alarm_id` (uuid), `alarm_type` (PRESENCE/STATIC_INTEGRITY/DYNAMIC_INTEGRITY), `alarm_code` (string), `severity` (LOW/MEDIUM/HIGH/CRITICAL), `state` (ACTIVE/ACKNOWLEDGED/CLEARED), `occurrences` (integer >= 1), `created_at` (ISO 8601), `updated_at` (ISO 8601), `expires_at` (ISO 8601), `method` (pcap/mms), `message_type` (GOOSE/SV/PTP), `pcap_id` (string ou null), `scd_id` (string), `details` (object), `snapshot` (object ou null), `dedup_key` (string) | Estrutura flat (sem wrapper). `details` varia por alarm_type. `snapshot` presente apenas em DYNAMIC_INTEGRITY. `expires_at` confirmado pelo backend (VER-619), pendente de inclusao formal na taxonomia |
| `AlarmListResponse` | `items` (GooseAlarm[]), `meta` ({ page, pageSize, total }) | Resposta paginada |
| `AckResponse` | `alarm_id` (uuid), `state` (string), `acknowledged_at` (ISO 8601), `acknowledged_by` (string) | Retornado apos ACK bem-sucedido |
| `AlarmSeverity` | LOW, MEDIUM, HIGH, CRITICAL | Enum para filtro e badge |
| `AlarmType` | PRESENCE, STATIC_INTEGRITY, DYNAMIC_INTEGRITY | Enum para filtro de tipo de alarme |
| `AlarmState` | ACTIVE, ACKNOWLEDGED, CLEARED | Enum para filtro de estado |
| `MessageType` | GOOSE, SV, PTP | Enum para filtro de protocolo (param `message_type`) |

---

## Fluxos de Interacao

### 1. Carregamento inicial da pagina

```
> Usuario acessa /alarmes
> Frontend executa GET /api/v1/alarms?page=1&pageSize=50&order=desc&sort=updated_at
> Resposta com AlarmListResponse
> Renderiza tabela com dados
> Coluna "Resumo" e gerada no frontend a partir de alarm_code + details
> Renderiza paginacao com base em meta.total
> Se meta.total == 0: exibe estado vazio "Nenhum alarme registrado"
```

### 2. Aplicar filtro

```
> Usuario seleciona valor em qualquer dropdown ou preenche campo de data
> Frontend monta query string com todos os filtros ativos
   Exemplo: message_type=GOOSE&severity=CRITICAL&state=ACTIVE&method=pcap&alarm_type=PRESENCE
> Reseta page=1 (volta para primeira pagina)
> GET /api/v1/alarms?{filtros}&page=1&pageSize=50&order=desc&sort=updated_at
> Tabela atualiza com resultados filtrados
> Se meta.total == 0: exibe "Nenhum alarme para os filtros selecionados" + [Limpar filtros]
```

### 3. Limpar filtros

```
> Usuario clica [Limpar filtros]
> Todos os dropdowns voltam para "Todos", campos de data limpos
> GET /api/v1/alarms?page=1&pageSize=50&order=desc&sort=updated_at
> Tabela atualiza com lista completa
```

### 4. Refresh manual

```
> Usuario clica [Refresh]
> Re-executa a busca com os filtros e paginacao atuais
> GET /api/v1/alarms?{filtros_atuais}&page={pagina_atual}&pageSize={tamanho}&sort={sort}&order={order}
> Tabela atualiza com dados mais recentes
```

### 5. Abrir detalhe do alarme

```
> Usuario clica em uma linha da tabela
> Frontend executa GET /api/v1/alarms/{alarm_id}
> Drawer abre pela direita com animacao de slide-in
> Exibe todos os campos do alarme:
   - Cabecalho: resumo traduzido do alarm_code + badge de severidade
   - Campos: alarm_id, created_at, updated_at, alarm_type, alarm_code,
     severity, state, occurrences, method, expires_at
   - Secao Detalhes: tabela chave-valor renderizada de alarm.details
   - Secao Metricas: tabela chave-valor de alarm.snapshot
     (visivel apenas para DYNAMIC_INTEGRITY com snapshot != null)
   - Link PCAP: visivel se pcap_id != null (null para MMS-only)
   - Botao [Reconhecer]: visivel para ADMIN/OPERATOR, oculto para VIEWER
> Se state != ACTIVE: botao desabilitado, exibe dados de quem reconheceu
```

### 6. Reconhecer alarme (ACK)

```
> Usuario (ADMIN ou OPERATOR) clica [Reconhecer Alarme] no drawer
> Botao entra em estado de loading (spinner)
> Frontend executa POST /api/v1/alarms/{alarm_id}/ack
> Resposta 200 (sucesso):
   > Exibe feedback inline: "Alarme reconhecido com sucesso" (badge verde, 5 segundos)
   > Campo "Estado" atualiza para: K ACKNOWLEDGED
   > Exibe dados: "por {acknowledged_by} em {acknowledged_at}"
   > Botao fica desabilitado
   > Linha correspondente na tabela atualiza: coluna Est. muda de A para K
   > Linha perde o destaque visual de fundo
> Resposta 409 (conflito):
   > Exibe mensagem inline: "Alarme ja reconhecido por outro operador" (badge amarelo)
   > Botao fica desabilitado
   > Re-executa GET /api/v1/alarms/{alarm_id} para atualizar dados no drawer
```

### 7. Navegar para PCAP

```
> Usuario clica em "Ver captura de pacotes" no drawer
> Navega para a tela de PCAP com o pcap_id pre-selecionado
> (Escopo futuro - link placeholder na primeira versao)
```

### 8. Ordenar tabela

```
> Usuario clica no cabecalho de uma coluna ordenavel (Timestamp, Tipo, Severidade, Ocorrencias)
> Frontend atualiza parametros sort e order
> GET /api/v1/alarms?sort={coluna}&order={asc|desc}&page=1&pageSize={tamanho}&{filtros}
> Tabela atualiza com nova ordenacao
> Indicador visual na coluna (seta ^ ou v)
```

### 9. Navegar entre paginas

```
> Usuario clica em numero de pagina ou seta de navegacao
> GET /api/v1/alarms?page={N}&pageSize={tamanho}&{filtros}&sort={sort}&order={order}
> Tabela atualiza com pagina solicitada
> Paginacao atualiza indicador de pagina atual
```

### 10. Alterar tamanho da pagina

```
> Usuario clica em [25], [50] ou [100] no seletor de pageSize
> Reseta page=1 (volta para primeira pagina)
> GET /api/v1/alarms?page=1&pageSize={novo_tamanho}&{filtros}&sort={sort}&order={order}
> Tabela atualiza com nova quantidade de itens
> Paginacao atualiza total de paginas
```

---

## Maquina de Estados do Alarme

```
                        +--------+
                        | ACTIVE |
                        +--------+
                       /          \
                (ACK) /            \ (TTL ~1 ano)
                     v              v
             +--------------+    +---------+
             | ACKNOWLEDGED |    | CLEARED |
             +--------------+    +---------+
                     |                |
            (TTL ~6 meses)        (30 dias)
                     v                v
             +---------+        +-----------+
             | CLEARED |        | [deletado]|
             +---------+        +-----------+
                     |
                (30 dias)
                     v
             +-----------+
             | [deletado]|
             +-----------+
```

**Descricao dos estados:**

| Estado | Descricao |
|---|---|
| **ACTIVE** | Alarme ativo, aguardando reconhecimento do operador. Possui PCAP associado (se method=pcap). Fundo destacado na tabela. Botao [Reconhecer] habilitado no drawer. |
| **ACKNOWLEDGED** | Alarme reconhecido pelo operador via ACK. Ainda possui PCAP associado. Botao [Reconhecer] desabilitado no drawer. Exibe dados de quem reconheceu. |
| **CLEARED** | Alarme encerrado cujo TTL de retencao expirou. PCAP nao mais disponivel. Mantido por 30 dias para historico antes de ser deletado. |

**Transicoes:**

- `ACTIVE -> ACKNOWLEDGED`: operador executa ACK (POST /api/v1/alarms/{alarm_id}/ack)
- `ACKNOWLEDGED -> CLEARED`: TTL de retencao do PCAP expira (~6 meses apos ACK)
- `ACTIVE -> CLEARED`: TTL do alarme expira sem ACK (~1 ano)
- `CLEARED -> [deletado]`: alarme removido do sistema apos 30 dias em estado CLEARED

---

## Estados

| Estado | Descricao | Visual |
|---|---|---|
| **Com dados** | Lista de alarmes com dados retornados pela API | Tabela completa com linhas preenchidas, paginacao visivel |
| **Vazio** | Nenhum alarme no sistema (`meta.total == 0` sem filtros ativos) | Ilustracao centralizada (icone de sino) + mensagem "Nenhum alarme registrado" |
| **Filtro sem resultados** | Filtros aplicados mas nenhum alarme corresponde (`meta.total == 0` com filtros) | Area de tabela vazia + "Nenhum alarme para os filtros selecionados" + botao [Limpar filtros] |
| **Alarme ACTIVE** | `state=ACTIVE` | Linha na tabela com fundo levemente destacado (ex: amarelo palido); "A" na coluna Est.; no drawer, botao [Reconhecer] ativo |
| **Alarme ACKNOWLEDGED** | `state=ACKNOWLEDGED` | Linha na tabela com fundo neutro/atenuado; "K" na coluna Est.; no drawer, botao [Reconhecer] desabilitado com info de quem reconheceu |
| **Alarme CLEARED** | `state=CLEARED` | Linha na tabela com fundo neutro; "C" na coluna Est.; no drawer, botao [Reconhecer] desabilitado; link PCAP indisponivel |
| **Carregando** | Requisicao em andamento (lista ou detalhe) | Skeleton/shimmer na tabela ou spinner no drawer |
| **Erro de API** | HTTP 500 ou erro de rede na listagem | Banner de erro vermelho inline no topo da area de conteudo: "Erro ao carregar alarmes. Tente novamente." + botao [Tentar novamente] |
| **Erro de API no drawer** | HTTP 500 ou erro de rede ao carregar detalhe | Mensagem de erro dentro do drawer + botao [Tentar novamente] |
| **ACK em progresso** | POST de reconhecimento em andamento | Botao [Reconhecer] com spinner/loading; desabilitado para evitar duplo clique |
| **ACK sucesso** | Reconhecimento concluido (HTTP 200) | Feedback inline verde: "Alarme reconhecido com sucesso"; botao desabilitado; dados de quem reconheceu exibidos; state muda para ACKNOWLEDGED |
| **ACK conflito (409)** | Outro operador ja reconheceu o alarme | Mensagem inline amarela no drawer: "Alarme ja reconhecido por outro operador"; botao desabilitado; drawer recarrega dados atualizados |

---

## Permissoes por Role

| Elemento | ADMIN | OPERATOR | VIEWER |
|---|---|---|---|
| Visualizar lista de alarmes | ok | ok | ok |
| Filtrar alarmes | ok | ok | ok |
| Ordenar tabela | ok | ok | ok |
| Navegar entre paginas | ok | ok | ok |
| Ver detalhe do alarme (drawer) | ok | ok | ok |
| Ver secao Detalhes (chave-valor) | ok | ok | ok |
| Ver secao Metricas (snapshot) | ok | ok | ok |
| Link PCAP no drawer | ok | ok | ok |
| Botao [Reconhecer] no drawer | Visivel e ativo | Visivel e ativo | **Oculto** |

**Nota:** O botao [Reconhecer] e o unico elemento com restricao por role nesta tela. Para roles ADMIN e OPERATOR, o botao e visivel quando `state=ACTIVE` e desabilitado quando `state != ACTIVE`. Para o role VIEWER, o botao nao e renderizado no DOM.

---

## Referencias

- `00-navegacao-global.md` — Layout master, padroes de tabela, drawer, paginacao, barra de filtros, convencoes visuais globais
- `01-tela-inicial.md` — Badges de alarmes nos IEDs da topologia (os contadores de alarme da tela inicial linkam para esta tela com filtros pre-aplicados)
- `docs/extras/2026-03-27/1. Alarmes GOOSE v1.2.md` — Taxonomia de alarmes GOOSE v1.2 (fonte da verdade: campos, alarm_codes, exemplos JSON, state machine)
- `docs/extras/2026-03-26/1. Alarmes GOOSE v1.1.md` — Taxonomia v1.1 (historico, substituida pela v1.2)

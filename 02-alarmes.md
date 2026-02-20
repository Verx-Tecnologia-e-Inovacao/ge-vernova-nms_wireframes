# Tela 2 — Alarmes

## Objetivo

Listar, filtrar, detalhar e reconhecer alarmes do sistema de monitoramento da subestacao IEC 61850. Esta tela e o ponto central de operacao para que operadores identifiquem anomalias detectadas pelos agentes de monitoramento (perda de sinal, jitter excessivo, falha de sincronismo PTP, perda de pacotes, etc.) e executem o reconhecimento (ACK) dos alarmes conforme procedimento operacional.

A subestacao monitorada possui 4 IEDs — L90_DIG (rele de protecao de linha), MU320E_LAB (merging unit), MU_360_1 (merging unit) e T60_DIG (rele de protecao de transformador) — com protocolos GOOSE, SV, PTP, MMS, SNMP e SYSTEM.

---

## Wireframe

### Estado principal (com dados)

```
+----------------------------------------------------------------------------------------------+
|  +--- Header Global (ver 00-navegacao-global.md) ---------------------------------------+    |
|  |  [Logo NMS]   Subestacao: MU_360_1     Mon: * RUNNING           operador01 [Logout]  |    |
|  +--------------------------------------------------------------------------------------+    |
|                                                                                              |
|  +--- Barra de Filtros ---------------------------------------------------------------  +    |
|  |                                                                                      |    |
|  |  {Protocolo v}  {Severidade v}  {Reconhecimento v}  [De: ____] [Ate: ____] [Limpar]  |    |
|  |                                                                                      |    |
|  |  Opcoes:         Opcoes:         Opcoes:             Data/hora  Data/hora   filtros  |    |
|  |  o Todos         o Todos         o Todos             ISO 8601   ISO 8601             |    |
|  |  o SV            o LOW           o Nao reconhecidos                                  |    |
|  |  o GOOSE         o MEDIUM        o Reconhecidos                                      |    |
|  |  o PTP           o MAJOR                                                             |    |
|  |  o MMS           o CRITICAL                                                          |    |
|  |  o SNMP                                                                              |    |
|  |  o SYSTEM                                                                            |    |
|  |                                                                                      |    |
|  +--------------------------------------------------------------------------------------+    |
|                                                                                              |
|  +--- Tabela de Alarmes --------------------------------------------------------------  +    |
|  |                                                                                      |    |
|  |  Timestamp           | Tipo            | Severidade  | Resumo                |Oc.|ACK|    |
|  |  --------------------+-----------------+-------------+-----------------------+---+---+    |
|  |  19/02/2026 14:32:01 | LOSS_OF_SIGNAL  | N CRITICAL  | Perda sinal SV stream | 3 | o |    |
|  |  ####################|#################|#############| MU_360_1 LDTM1        |###|###|    |
|  |                      |                 |             |                       |   |   |    |
|  |  19/02/2026 14:30:15 | JITTER_EXCEEDED | ! MAJOR     | Jitter SV acima limiar| 1 | * |    |
|  |                      |                 |             | MU320E_LAB            |   |   |    |
|  |                      |                 |             |                       |   |   |    |
|  |  19/02/2026 14:25:00 | PTP_SYNC_LOSS   | ! MAJOR     | Perda sincronismo PTP | 2 | o |    |
|  |                      |                 |             | T60_DIG               |   |   |    |
|  |                      |                 |             |                       |   |   |    |
|  |  19/02/2026 13:10:44 | PACKET_LOSS     | o MEDIUM    | Perda pacotes GOOSE   |12 | * |    |
|  |                      |                 |             | GCB01 MU_360_1        |   |   |    |
|  |                      |                 |             |                       |   |   |    |
|  +--------------------------------------------------------------------------------------+    |
|                                                                                              |
|  Legenda da coluna ACK:  * = Reconhecido  |  o = Nao reconhecido                             |
|  Legenda de severidade:  N CRITICAL (vermelho) | ! MAJOR (laranja) | o MEDIUM (amarelo)      |
|                          o LOW (azul)                                                        |
|                                                                                              |
|  Nota: Linhas com ack=false (o) possuem fundo levemente destacado (ex: amarelo palido)       |
|  Nota: A linha ### indica destaque visual para alarmes CRITICAL nao reconhecidos             |
|                                                                                              |
|  +--- Paginacao ------------------------------------------------------------------      +    |
|  |                                                                                      |    |
|  |  Mostrando 1-4 de 4 alarmes                           < 1 2 3 ... N >                |    |
|  |                                                                                      |    |
|  +--------------------------------------------------------------------------------------+    |
|                                                                                              |
+----------------------------------------------------------------------------------------------+
```

**Comportamento da tabela:**

- Ordenacao padrao: `timestampUtc` descendente (mais recente primeiro)
- Colunas ordenaveis: Timestamp, Tipo, Severidade, Ocorrencias
- Clique na linha abre o drawer de detalhe (ver secao seguinte)
- Coluna "Oc." exibe o campo `occurrences` do alarme
- Coluna "ACK" exibe icone ● (reconhecido) ou ○ (nao reconhecido) com base no campo `ack`
- Badge de severidade com cor: CRITICAL=vermelho, MAJOR=laranja, MEDIUM=amarelo, LOW=azul
- Linhas com `ack=false` possuem fundo levemente destacado para chamar atencao do operador

---

### Estado do drawer (detalhe do alarme)

Abre pela direita quando o usuario clica em uma linha da tabela. Ocupa aproximadamente 40% da largura da tela. A tabela principal permanece visivel ao fundo (com overlay escurecido ou nao, a criterio do designer).

```
+----------------------------------------------------+
|                                                    |
|  +--- Drawer: Detalhe do Alarme ----------[X]-- +  |
|  |                                              |  |
|  |  LOSS_OF_SIGNAL          +----------------+  |  |
|  |                          | N CRITICAL     |  |  |
|  |                          +----------------+  |  |
|  |                                              |  |
|  |  ------------------------------------------  |  |
|  |                                              |  |
|  |  Alarm ID                                    |  |
|  |  a1b2c3d4-e5f6-7890-abcd-ef1234567890        |  |
|  |                                              |  |
|  |  Timestamp (UTC)                             |  |
|  |  2026-02-19T14:32:01Z                        |  |
|  |                                              |  |
|  |  Tipo                                        |  |
|  |  LOSS_OF_SIGNAL                              |  |
|  |                                              |  |
|  |  Severidade                                  |  |
|  |  N CRITICAL                                  |  |
|  |                                              |  |
|  |  Resumo                                      |  |
|  |  Perda de sinal SV stream MU_360_1 LDTM1     |  |
|  |                                              |  |
|  |  Ocorrencias                                 |  |
|  |  3                                           |  |
|  |                                              |  |
|  |  Status de Reconhecimento                    |  |
|  |  o Nao reconhecido                           |  |
|  |                                              |  |
|  |  ------------------------------------------  |  |
|  |                                              |  |
|  |  Detalhes                                    |  |
|  |  +------------------+---------------------+  |  |
|  |  | Chave            | Valor               |  |  |
|  |  +------------------+---------------------+  |  |
|  |  | streamId         | LDTM1/F4800S2I4U4_1 |  |  |
|  |  | publisher        | MU_360_1            |  |  |
|  |  | subscribers      | L90_DIG, T60_DIG    |  |  |
|  |  | lastSeenUtc      | 2026-02-19T14:31:58Z|  |  |
|  |  | packetsMissed    | 240                 |  |  |
|  |  +------------------+---------------------+  |  |
|  |                                              |  |
|  |  Nota: a secao Detalhes e renderizada        |  |
|  |  dinamicamente como tabela chave-valor.      |  |
|  |  O conteudo varia conforme o tipo do         |  |
|  |  alarme (Alarm.details).                     |  |
|  |                                              |  |
|  |  ------------------------------------------  |  |
|  |                                              |  |
|  |  Captura de Pacotes                          |  |
|  |  < Ver captura de pacotes >                  |  |
|  |  (link visivel se pcapId != null)            |  |
|  |  (se pcapId == null: "Sem captura            |  |
|  |   associada" em texto cinza)                 |  |
|  |                                              |  |
|  |  ------------------------------------------  |  |
|  |                                              |  |
|  |  +----------------------------------------+  |  |
|  |  |         [Reconhecer Alarme]            |  |  |
|  |  +----------------------------------------+  |  |
|  |                                              |  |
|  |  (ADMIN/OPERATOR: botao visivel e ativo)     |  |
|  |  (VIEWER: botao oculto)                      |  |
|  |  (Se ack=true: botao desabilitado com        |  |
|  |   texto "Reconhecido por {user} em {data}")  |  |
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
|  |  Status de Reconhecimento                    |  |
|  |  * Reconhecido                               |  |
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
+----------------------------------------------------------------------------------------------+
|  +--- Header Global ---------------------------------------------------------------    +     |
|  |  [Logo NMS]   Dashboard | Alarmes | Topologia | PCAPs | Config   [User v] [Sair]    |     |
|  +-------------------------------------------------------------------------------------+     |
|                                                                                              |
|  +--- Barra de Filtros --------------------------------------------------------------- +     |
|  |  {Protocolo v}  {Severidade v}  {Reconhecimento v}  [De: ____] [Ate: ____] [Limpar] |     |
|  +-------------------------------------------------------------------------------------+     |
|                                                                                              |
|                                                                                              |
|                                                                                              |
|                            +--------------------------+                                      |
|                            |                          |                                      |
|                            |      (icone: sino        |                                      |
|                            |       com "Z" ou         |                                      |
|                            |       check verde)       |                                      |
|                            |                          |                                      |
|                            |  Nenhum alarme           |                                      |
|                            |  registrado              |                                      |
|                            |                          |                                      |
|                            +--------------------------+                                      |
|                                                                                              |
|                                                                                              |
|                                                                                              |
+----------------------------------------------------------------------------------------------+
```

**Estado de filtro sem resultados:**

```
+----------------------------------------------------------------------------------------------+
|  +--- Barra de Filtros (filtros ativos indicados por badge/cor) ----------------------+      |
|  |  {SV v}  {CRITICAL v}  {Nao reconhecidos v}  [De: ____] [Ate: ____] [Limpar]       |      |
|  +------------------------------------------------------------------------------------+      |
|                                                                                              |
|                            +----------------------------------+                              |
|                            |                                  |                              |
|                            |      (icone: lupa vazia)         |                              |
|                            |                                  |                              |
|                            |  Nenhum alarme para os           |                              |
|                            |  filtros selecionados            |                              |
|                            |                                  |                              |
|                            |  [Limpar filtros]                |                              |
|                            |                                  |                              |
|                            +----------------------------------+                              |
|                                                                                              |
+----------------------------------------------------------------------------------------------+
```

---

## Componentes

| Componente | Fonte de dados | Formato de exibicao |
|---|---|---|
| **Barra de filtros** | Valores estaticos (enums) + inputs do usuario | Dropdowns com opcoes pre-definidas; campos de data com date-picker |
| **Dropdown Protocolo** | `ProtocolEnum`: SV, GOOSE, PTP, MMS, SNMP, SYSTEM | Select com opcao "Todos" como padrao |
| **Dropdown Severidade** | `AlarmSeverity`: LOW, MEDIUM, MAJOR, CRITICAL | Select com opcao "Todos" como padrao; cada opcao com cor indicativa |
| **Dropdown Reconhecimento** | Valores fixos: Todos, Nao reconhecidos, Reconhecidos | Select; mapeia para query param `ack` (null, false, true) |
| **Campo De/Ate** | Input do usuario | Date-time picker; envia como `fromUtc`/`toUtc` ISO 8601 |
| **Botao Limpar filtros** | — | Reseta todos os filtros ao estado padrao; re-executa busca sem filtros |
| **Tabela de alarmes** | `GET /api/v1/alarms` → `AlarmListResponse.items[]` | Tabela com linhas clicaveis |
| **Coluna Timestamp** | `alarm.timestampUtc` | Formato local: `DD/MM/AAAA HH:mm:ss` (convertido de UTC) |
| **Coluna Tipo** | `alarm.type` | Texto monoespacado (ex: `LOSS_OF_SIGNAL`) |
| **Coluna Severidade** | `alarm.severity` | Badge colorido: ✕ CRITICAL (vermelho), ⚠ MAJOR (laranja), ○ MEDIUM (amarelo), ○ LOW (azul) |
| **Coluna Resumo** | `alarm.summary` | Texto corrido; truncado com ellipsis se muito longo |
| **Coluna Ocorrencias** | `alarm.occurrences` | Numero inteiro |
| **Coluna ACK** | `alarm.ack` | Icone: ● (true) ou ○ (false) |
| **Paginacao** | `AlarmListResponse.meta` | "Mostrando X-Y de Z alarmes" + navegacao de paginas |
| **Drawer de detalhe** | `GET /api/v1/alarms/{alarmId}` → `AlarmRecord` | Painel lateral direito (~40% largura) |
| **Secao Detalhes no drawer** | `alarm.details` (additionalProperties) | Tabela chave-valor renderizada dinamicamente |
| **Link PCAP** | `alarm.pcapId` | Link "Ver captura de pacotes" se `pcapId != null`; texto cinza "Sem captura associada" caso contrario |
| **Botao Reconhecer** | `POST /api/v1/alarms/{alarmId}/ack` | Botao primario; estados: ativo, desabilitado (ja reconhecido), oculto (role VIEWER) |
| **Feedback ACK sucesso** | `AcknowledgementResponse` | Badge verde inline com dados de quem reconheceu e quando |
| **Feedback ACK conflito** | HTTP 409 `ErrorResponse` | Badge amarelo inline: "Alarme ja reconhecido por outro operador" |
| **Banner de erro de API** | HTTP 5xx / erro de rede | Banner vermelho no topo da tabela |
| **Estado vazio** | `AlarmListResponse.meta.total == 0` sem filtros | Ilustracao + "Nenhum alarme registrado" |
| **Estado filtro vazio** | `AlarmListResponse.meta.total == 0` com filtros ativos | Ilustracao + "Nenhum alarme para os filtros selecionados" + botao [Limpar filtros] |

---

## Dados e Endpoints

| Endpoint | Metodo | Uso na tela | Campos utilizados | Exemplo de chamada |
|---|---|---|---|---|
| `/api/v1/alarms` | GET | Carregar lista de alarmes (paginada, filtrada, ordenada) | Query: `source`, `type`, `severity`, `ack`, `fromUtc`, `toUtc`, `page`, `pageSize`, `sort`, `order` | `GET /api/v1/alarms?page=1&pageSize=50&order=desc&sort=timestampUtc` |
| `/api/v1/alarms` | GET | Filtrar por protocolo | Query: `source=SV` | `GET /api/v1/alarms?source=SV&page=1&pageSize=50` |
| `/api/v1/alarms` | GET | Filtrar por severidade | Query: `severity=CRITICAL` | `GET /api/v1/alarms?severity=CRITICAL&page=1&pageSize=50` |
| `/api/v1/alarms` | GET | Filtrar por status de ACK | Query: `ack=false` | `GET /api/v1/alarms?ack=false&page=1&pageSize=50` |
| `/api/v1/alarms` | GET | Filtrar por periodo | Query: `fromUtc`, `toUtc` | `GET /api/v1/alarms?fromUtc=2026-02-19T00:00:00Z&toUtc=2026-02-19T23:59:59Z` |
| `/api/v1/alarms/{alarmId}` | GET | Carregar detalhe completo do alarme no drawer | Response: `AlarmRecord` (scdId, alarm com todos os campos incluindo details) | `GET /api/v1/alarms/a1b2c3d4-e5f6-7890-abcd-ef1234567890` |
| `/api/v1/alarms/{alarmId}/ack` | POST | Reconhecer alarme | Response 200: `AcknowledgementResponse` (alarmId, acknowledgedAtUtc, acknowledgedBy); Response 409: `ErrorResponse` | `POST /api/v1/alarms/a1b2c3d4-e5f6-7890-abcd-ef1234567890/ack` |

**Schemas utilizados:**

| Schema | Campos | Observacoes |
|---|---|---|
| `Alarm` | `alarmId` (uuid), `timestampUtc` (ISO 8601), `type` (string), `severity` (LOW/MEDIUM/MAJOR/CRITICAL), `summary` (string), `details` (object, additionalProperties:true), `occurrences` (integer >= 1), `ack` (boolean), `pcapId` (uuid ou null) | `details` e dinamico — renderizar como tabela chave-valor |
| `AlarmRecord` | `scdId` (uuid), `alarm` (Alarm) | Wrapper que associa o alarme ao SCD ativo |
| `AlarmListResponse` | `items` (AlarmRecord[]), `meta` ({ page, pageSize, total }) | Resposta paginada |
| `AcknowledgementResponse` | `alarmId` (uuid), `acknowledgedAtUtc` (ISO 8601), `acknowledgedBy` (string) | Retornado apos ACK bem-sucedido |
| `AlarmSeverity` | LOW, MEDIUM, MAJOR, CRITICAL | Enum para filtro e badge |
| `ProtocolEnum` | SV, GOOSE, PTP, MMS, SNMP, SYSTEM | Enum para filtro de protocolo (param `source`) |

---

## Fluxos de Interacao

### 1. Carregamento inicial da pagina

```
> Usuario acessa /alarmes
> Frontend executa GET /api/v1/alarms?page=1&pageSize=50&order=desc&sort=timestampUtc
> Resposta com AlarmListResponse
> Renderiza tabela com dados
> Renderiza paginacao com base em meta.total
> Se meta.total == 0: exibe estado vazio "Nenhum alarme registrado"
```

### 2. Aplicar filtro

```
> Usuario seleciona valor em qualquer dropdown ou preenche campo de data
> Frontend monta query string com todos os filtros ativos
   Exemplo: source=SV&severity=CRITICAL&ack=false&fromUtc=2026-02-19T00:00:00Z
> Reseta page=1 (volta para primeira pagina)
> GET /api/v1/alarms?{filtros}&page=1&pageSize=50&order=desc&sort=timestampUtc
> Tabela atualiza com resultados filtrados
> Se meta.total == 0: exibe "Nenhum alarme para os filtros selecionados" + [Limpar filtros]
```

### 3. Limpar filtros

```
> Usuario clica [Limpar filtros]
> Todos os dropdowns voltam para "Todos", campos de data limpos
> GET /api/v1/alarms?page=1&pageSize=50&order=desc&sort=timestampUtc
> Tabela atualiza com lista completa
```

### 4. Abrir detalhe do alarme

```
> Usuario clica em uma linha da tabela
> Frontend executa GET /api/v1/alarms/{alarmId}
> Drawer abre pela direita com animacao de slide-in
> Exibe todos os campos do alarme:
   - Cabecalho: type + badge de severidade
   - Campos: alarmId, timestampUtc, type, severity, summary, occurrences, ack
   - Secao Detalhes: tabela chave-valor renderizada de alarm.details
   - Link PCAP: visivel se pcapId != null
   - Botao [Reconhecer]: visivel para ADMIN/OPERATOR, oculto para VIEWER
> Se ack=true: botao desabilitado, exibe dados de quem reconheceu
```

### 5. Reconhecer alarme (ACK)

```
> Usuario (ADMIN ou OPERATOR) clica [Reconhecer Alarme] no drawer
> Botao entra em estado de loading (spinner)
> Frontend executa POST /api/v1/alarms/{alarmId}/ack
> Resposta 200 (sucesso):
   > Exibe feedback inline: "Alarme reconhecido com sucesso" (badge verde, 5 segundos)
   > Atualiza campo "Status de Reconhecimento" para:
     * Reconhecido por {acknowledgedBy} em {acknowledgedAtUtc}
   > Botao fica desabilitado
   > Linha correspondente na tabela atualiza: icone ACK muda de o para *
   > Linha perde o destaque visual de fundo
> Resposta 409 (conflito):
   > Exibe mensagem inline: "Alarme ja reconhecido por outro operador" (badge amarelo)
   > Botao fica desabilitado
   > Re-executa GET /api/v1/alarms/{alarmId} para atualizar dados no drawer
```

### 6. Navegar para PCAP

```
> Usuario clica em "Ver captura de pacotes" no drawer
> Navega para a tela de PCAP com o pcapId pre-selecionado
> (Escopo futuro - link placeholder na primeira versao)
```

### 7. Ordenar tabela

```
> Usuario clica no cabecalho de uma coluna ordenavel (Timestamp, Tipo, Severidade, Ocorrencias)
> Frontend atualiza parametros sort e order
> GET /api/v1/alarms?sort={coluna}&order={asc|desc}&page=1&pageSize=50&{filtros}
> Tabela atualiza com nova ordenacao
> Indicador visual na coluna (seta ^ ou v)
```

### 8. Navegar entre paginas

```
> Usuario clica em numero de pagina ou seta de navegacao
> GET /api/v1/alarms?page={N}&pageSize=50&{filtros}&sort={sort}&order={order}
> Tabela atualiza com pagina solicitada
> Paginacao atualiza indicador de pagina atual
```

---

## Estados

| Estado | Descricao | Visual |
|---|---|---|
| **Com dados** | Lista de alarmes com dados retornados pela API | Tabela completa com linhas preenchidas, paginacao visivel |
| **Vazio** | Nenhum alarme no sistema (`meta.total == 0` sem filtros ativos) | Ilustracao centralizada (icone de sino) + mensagem "Nenhum alarme registrado" |
| **Filtro sem resultados** | Filtros aplicados mas nenhum alarme corresponde (`meta.total == 0` com filtros) | Area de tabela vazia + "Nenhum alarme para os filtros selecionados" + botao [Limpar filtros] |
| **Alarme nao reconhecido** | `ack=false` | Linha na tabela com fundo levemente destacado (ex: amarelo palido); icone ○ na coluna ACK; no drawer, botao [Reconhecer] ativo |
| **Alarme reconhecido** | `ack=true` | Linha na tabela com fundo neutro/atenuado; icone ● na coluna ACK; no drawer, botao [Reconhecer] desabilitado com info de quem reconheceu |
| **Carregando** | Requisicao em andamento (lista ou detalhe) | Skeleton/shimmer na tabela ou spinner no drawer |
| **Erro de API** | HTTP 500 ou erro de rede na listagem | Banner de erro vermelho inline no topo da area de conteudo: "Erro ao carregar alarmes. Tente novamente." + botao [Tentar novamente] |
| **Erro de API no drawer** | HTTP 500 ou erro de rede ao carregar detalhe | Mensagem de erro dentro do drawer + botao [Tentar novamente] |
| **ACK em progresso** | POST de reconhecimento em andamento | Botao [Reconhecer] com spinner/loading; desabilitado para evitar duplo clique |
| **ACK sucesso** | Reconhecimento concluido (HTTP 200) | Feedback inline verde: "Alarme reconhecido com sucesso"; botao desabilitado; dados de quem reconheceu exibidos |
| **ACK conflito (409)** | Outro operador ja reconheceu o alarme | Mensagem inline amarela no drawer: "Alarme ja reconhecido por outro operador"; botao desabilitado; drawer recarrega dados atualizados |

---

## Permissoes por Role

| Elemento | ADMIN | OPERATOR | VIEWER |
|---|---|---|---|
| Visualizar lista de alarmes | ✓ | ✓ | ✓ |
| Filtrar alarmes | ✓ | ✓ | ✓ |
| Ordenar tabela | ✓ | ✓ | ✓ |
| Navegar entre paginas | ✓ | ✓ | ✓ |
| Ver detalhe do alarme (drawer) | ✓ | ✓ | ✓ |
| Ver secao Detalhes (chave-valor) | ✓ | ✓ | ✓ |
| Link PCAP no drawer | ✓ | ✓ | ✓ |
| Botao [Reconhecer] no drawer | Visivel e ativo | Visivel e ativo | **Oculto** |

**Nota:** O botao [Reconhecer] e o unico elemento com restricao por role nesta tela. Para roles ADMIN e OPERATOR, o botao e visivel quando `ack=false` e desabilitado quando `ack=true`. Para o role VIEWER, o botao nao e renderizado no DOM.

---

## Referencias

- `00-navegacao-global.md` — Layout master, padroes de tabela, drawer, paginacao, barra de filtros, convencoes visuais globais
- `01-tela-inicial.md` — Badges de alarmes nos IEDs da topologia (os contadores de alarme da tela inicial linkam para esta tela com filtros pre-aplicados)

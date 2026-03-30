# Tela 6 — SNMP (Simple Network Management Protocol)

## Objetivo

Detalhar informacoes de rede coletadas via SNMP dos switches e equipamentos de infraestrutura da subestacao IEC 61850. Os OIDs coletados pelos agentes de monitoramento sao traduzidos para nomes legiveis utilizando os MIBs cadastrados na tela de Configuracao (secao 7.3).

Diferente das demais telas de protocolo (GOOSE, SV, MMS), os dados SNMP **nao sao derivados do SCD** — eles vem inteiramente do polling dos agentes de monitoramento sobre switches e equipamentos de rede da subestacao. Os dispositivos monitorados sao tipicamente switches industriais do barramento de processo, barramento de estacao e rede administrativa.

---

## Wireframe

### Estado principal (com dados)

```
+-----------------------------------------------------------------------------------------------+
|  +--- Header Global (ver 00-navegacao-global.md) --------------------------------------+      |
|  |  [Logo NMS]   Subestacao: SE Exemplo   Mon: * RUNNING                    {user} [Sair]  |  |
|  +-------------------------------------------------------------------------------------+      |
|                                                                                               |
|  +--- Barra de Filtros ----------------------------------------------------------------+      |
|  |                                                                                     |      |
|  |  [Buscar dispositivo: ____________________________]       {Status v}                |      |
|  |                                                                                     |      |
|  |  Opcoes Status:                                                                     |      |
|  |    o Todos                                                                          |      |
|  |    o Online                                                                         |      |
|  |    o Degraded                                                                       |      |
|  |    o Offline                                                                        |      |
|  |                                                                                     |      |
|  +-------------------------------------------------------------------------------------+      |
|                                                                                               |
|  +--- Banner de aviso (exibido se nenhum MIB cadastrado) ------------------------------+      |
|  |  ! Nenhum MIB cadastrado. Os OIDs serao exibidos sem traducao para nomes legiveis.  |      |
|  |    < Cadastrar MIBs na Configuracao >                                         [X]   |      |
|  +-------------------------------------------------------------------------------------+      |
|                                                                                               |
|  +--- Tabela de Dispositivos SNMP -----------------------------------------------------+      |
|  |                                                                                     |      |
|  |  Dispositivo ^                       | Status     | OIDs Coletados | Ult. Atualiz.  |      |
|  |  ------------------------------------+------------+----------------+----------------+      |
|  |  SW-PROC-01 (Process Bus Switch)     | * Online   | 24             | 27/03/26 14:32 |      |
|  |                                      |            |                |                |      |
|  |  ------------------------------------+------------+----------------+----------------+      |
|  |  SW-STAT-01 (Station Bus Switch)     | * Online   | 18             | 27/03/26 14:32 |      |
|  |                                      |            |                |                |      |
|  |  ------------------------------------+------------+----------------+----------------+      |
|  |  SW-ADMIN-01 (Admin Switch)          | ! Degraded | 12             | 27/03/26 14:28 |      |
|  |                                      |            |                |                |      |
|  |  ------------------------------------+------------+----------------+----------------+      |
|  |  SW-PROC-02 (Process Bus Switch #2)  | N Offline  |  0             | 27/03/26 13:45 |      |
|  |                                      |            |                |                |      |
|  +-------------------------------------------------------------------------------------+      |
|                                                                                               |
|  Legenda: * Online (verde) | ! Degraded (amarelo) | N Offline (vermelho)                      |
|                                                                                               |
|  +--- Paginacao -----------------------------------------------------------------------+      |
|  |                                                                                     |      |
|  |  < Anterior  Pagina 1 de 1  Proxima >    Exibindo 4 de 4                            |      |
|  |                                                                                     |      |
|  +-------------------------------------------------------------------------------------+      |
|                                                                                               |
+-----------------------------------------------------------------------------------------------+
```

**Comportamento da tabela:**

- Ordenacao padrao: `timestampUtc` descendente (ultima atualizacao mais recente primeiro)
- Colunas ordenaveis: Dispositivo, Status, OIDs Coletados, Ultima Atualizacao
- Clique na linha abre o drawer de detalhe do dispositivo (ver secao seguinte)
- Coluna "Dispositivo" exibe o `deviceId` seguido de descricao amigavel entre parenteses (quando disponivel)
- Coluna "Status" exibe badge colorido conforme legenda
- Coluna "OIDs Coletados" exibe a contagem de OIDs distintos recebidos do dispositivo
- Coluna "Ultima Atualizacao" exibe o `timestampUtc` mais recente entre os dados SNMP do dispositivo
- Linhas com status ⚠ Degraded possuem fundo levemente destacado (amarelo palido)
- Linhas com status ✕ Offline possuem fundo levemente destacado (vermelho palido)

**Determinacao de status do dispositivo:**

| Status | Condicao | Indicador |
|---|---|---|
| Online | Todos os OIDs com valores normais, ultima coleta recente | ● (verde) |
| Degraded | Alguns OIDs com valores anormais (ex: erros de porta > 0, status de porta down) | ⚠ (amarelo) |
| Offline | Nenhuma resposta SNMP dentro do periodo esperado de polling | ✕ (vermelho) |

---

### Drawer (detalhe do dispositivo)

Abre pela direita quando o usuario clica em uma linha da tabela. Ocupa aproximadamente 40% da largura da tela. A tabela principal permanece visivel ao fundo.

```
+-------------------------------------------------------------------------------+
|                                                                               |
|  +--- Drawer: Detalhe do Dispositivo SNMP ----------------------------[X]-+   |
|  |                                                                        |   |
|  |  SW-PROC-01                                                            |   |
|  |  Process Bus Switch                                                    |   |
|  |                                        +------------------+            |   |
|  |                                        |   * Online       |            |   |
|  |                                        +------------------+            |   |
|  |                                                                        |   |
|  |  Ultima coleta: 27/03/2026 14:32                                       |   |
|  |  OIDs coletados: 24                                                    |   |
|  |                                                                        |   |
|  |  -------------------------------------------------------------------   |   |
|  |                                                                        |   |
|  |  +--- Tabela de OIDs ----------------------------------------------+   |   |
|  |  |                                                                 |   |   |
|  |  |  OID                    | Nome (MIB)         | Tipo    | Valor  |   |   |
|  |  |  -----------------------+--------------------+---------+--------+   |   |
|  |  |  1.3.6.1.2.1.1.3.0      | sysUpTime          | integer |1234567 |   |   |
|  |  |  1.3.6.1.2.1.1.5.0      | sysName            | string  |SW-PR-01|   |   |
|  |  |  1.3.6.1.2.1.2.2.1.8.1  | ifOperStatus (Pt1) | integer |1 (up)  |   |   |
|  |  |  1.3.6.1.2.1.2.2.1.8.2  | ifOperStatus (Pt2) | integer |1 (up)  |   |   |
|  |  |  1.3.6.1.2.1.2.2.1.10.1 | ifInOctets (Pt1)   | integer |45893021|   |   |
|  |  |  1.3.6.1.2.1.2.2.1.16.1 | ifOutOctets (Pt1)  | integer |32105678|   |   |
|  |  |  1.3.6.1.2.1.2.2.1.14.1 | ifInErrors (Pt1)   | integer |0       |   |   |
|  |  |  1.3.6.1.2.1.2.2.1.20.1 | ifOutErrors (Pt1)  | integer |3       |   |   |
|  |  |                                                                 |   |   |
|  |  +-----------------------------------------------------------------+   |   |
|  |                                                                        |   |
|  |  Nota: Quando nenhum MIB esta cadastrado, a coluna "Nome (MIB)"        |   |
|  |  exibe o proprio OID (sem traducao).                                   |   |
|  |                                                                        |   |
|  |  -------------------------------------------------------------------   |   |
|  |                                                                        |   |
|  |  +--- Metricas SNMP - SW-PROC-01 ----------------------------------+   |   |
|  |  |                                                                 |   |   |
|  |  |  {OID v: ifInOctets (Port1)}     [De: ____]  [Ate: ____]        |   |   |
|  |  |                                                                 |   |   |
|  |  |  bytes                                                          |   |   |
|  |  |  50M +                               /--                        |   |   |
|  |  |  40M +                          /---/                           |   |   |
|  |  |  30M +                    /----/                                |   |   |
|  |  |  20M +              /----/                                      |   |   |
|  |  |  10M +        /----/                                            |   |   |
|  |  |    0 +-------/                                                  |   |   |
|  |  |      +----------------------------------------------            |   |   |
|  |  |       10:00   11:00   12:00   13:00   14:00                     |   |   |
|  |  |                                                                 |   |   |
|  |  +-----------------------------------------------------------------+   |   |
|  |                                                                        |   |
|  +------------------------------------------------------------------------+   |
|                                                                               |
+-------------------------------------------------------------------------------+
```

**Tabela de OIDs completa (referencia para o designer):**

```
+-------------------------+----------------------+---------+---------------+
| OID                     | Nome (MIB)           | Tipo    | Valor         |
+-------------------------+----------------------+---------+---------------+
| 1.3.6.1.2.1.1.3.0       | sysUpTime            | integer | 1234567       |
| 1.3.6.1.2.1.1.5.0       | sysName              | string  | SW-PROC-01    |
| 1.3.6.1.2.1.2.2.1.8.1   | ifOperStatus (Port1) | integer | 1 (up)        |
| 1.3.6.1.2.1.2.2.1.8.2   | ifOperStatus (Port2) | integer | 1 (up)        |
| 1.3.6.1.2.1.2.2.1.10.1  | ifInOctets (Port1)   | integer | 45893021      |
| 1.3.6.1.2.1.2.2.1.16.1  | ifOutOctets (Port1)  | integer | 32105678      |
| 1.3.6.1.2.1.2.2.1.14.1  | ifInErrors (Port1)   | integer | 0             |
| 1.3.6.1.2.1.2.2.1.20.1  | ifOutErrors (Port1)  | integer | 3             |
+-------------------------+----------------------+---------+---------------+
```

**Comportamento do drawer:**

- O dropdown `{OID ▼}` lista todos os OIDs numericos do dispositivo, exibindo o nome traduzido via MIB quando disponivel
- Os campos `[De]` e `[Ate]` sao date-time pickers que definem o intervalo da serie temporal (mapeados para `fromUtc` e `toUtc` na API)
- O grafico de metricas e renderizado com os dados retornados por `GET /protocols/SNMP/metrics`
- Ao selecionar um OID diferente no dropdown, o grafico recarrega automaticamente com os dados do novo OID
- A tabela de OIDs exibe os dados da ultima coleta SNMP para o dispositivo selecionado
- OIDs com valores anormais (ex: `ifOutErrors > 0`) podem ter destaque visual (texto em amarelo ou badge de atencao)

**Comportamento da coluna "Nome (MIB)":**

| Cenario | Exibicao na coluna "Nome (MIB)" |
|---|---|
| MIBs carregados e OID encontrado | Nome traduzido (ex: `sysUpTime`, `ifInOctets (Port1)`) |
| MIBs carregados mas OID nao encontrado | OID numerico original (ex: `1.3.6.1.4.1.9999.1.1.0`) |
| Nenhum MIB cadastrado | OID numerico original para todos os registros |

---

### Estado sem MIBs

Quando nenhum MIB esta cadastrado no sistema, a tela ainda exibe a tabela de dispositivos normalmente, porem com um banner de aviso no topo e a coluna "Nome (MIB)" no drawer exibindo os OIDs sem traducao.

```
+-----------------------------------------------------------------------------------------------+
|  +--- Header Global (ver 00-navegacao-global.md) --------------------------------------+      |
|  |  [Logo NMS]   Subestacao: SE Exemplo   Mon: * RUNNING                    {user} [Sair]  |  |
|  +-------------------------------------------------------------------------------------+      |
|                                                                                               |
|  +--- Barra de Filtros ----------------------------------------------------------------+      |
|  |  [Buscar dispositivo: ____________________________]       {Status v}                |      |
|  +-------------------------------------------------------------------------------------+      |
|                                                                                               |
|  +--- ! ------------------------------------------------------------------------------ +      |
|  |  Nenhum MIB cadastrado. Os OIDs serao exibidos sem traducao para nomes legiveis.    |      |
|  |  < Cadastrar MIBs na Configuracao >                                           [X]   |      |
|  +-------------------------------------------------------------------------------------+      |
|                                                                                               |
|  +--- Tabela de Dispositivos SNMP -----------------------------------------------------+      |
|  |                                                                                     |      |
|  |  Dispositivo ^                       | Status     | OIDs Coletados | Ult. Atualiz.  |      |
|  |  ------------------------------------+------------+----------------+----------------+      |
|  |  SW-PROC-01 (Process Bus Switch)     | * Online   | 24             | 27/03/26 14:32 |      |
|  |  SW-STAT-01 (Station Bus Switch)     | * Online   | 18             | 27/03/26 14:32 |      |
|  |  SW-ADMIN-01 (Admin Switch)          | ! Degraded | 12             | 27/03/26 14:28 |      |
|  |                                                                                     |      |
|  +-------------------------------------------------------------------------------------+      |
|                                                                                               |
|  Nota: O link < Cadastrar MIBs na Configuracao > e visivel apenas para ADMIN.                 |
|  Para OPERATOR e VIEWER, exibe apenas a mensagem de aviso sem o link.                         |
|                                                                                               |
+-----------------------------------------------------------------------------------------------+
```

**Drawer sem MIBs (coluna Nome exibe OID numerico):**

```
+--- Tabela de OIDs (sem MIBs) -------------------------------------------------+
|                                                                               |
|  OID                    | Nome (MIB)             | Tipo    | Valor            |
|  -----------------------+------------------------+---------+------------------+
|  1.3.6.1.2.1.1.3.0      | 1.3.6.1.2.1.1.3.0      | integer | 1234567          |
|  1.3.6.1.2.1.1.5.0      | 1.3.6.1.2.1.1.5.0      | string  | SW-PROC-01       |
|  1.3.6.1.2.1.2.2.1.8.1  | 1.3.6.1.2.1.2.2.1.8.1  | integer | 1                |
|  1.3.6.1.2.1.2.2.1.10.1 | 1.3.6.1.2.1.2.2.1.10.1 | integer | 45893021         |
|                                                                               |
+-------------------------------------------------------------------------------+
```

---

### Estado sem dispositivos

Exibido quando o sistema ainda nao coletou nenhum dado SNMP — ou seja, nenhum dispositivo respondeu ao polling dos agentes de monitoramento.

```
+-----------------------------------------------------------------------------------------------+
|  +--- Header Global (ver 00-navegacao-global.md) ---------------------------------------+     |
|  |  [Logo NMS]   Subestacao: SE Exemplo   Mon: * RUNNING                    {user} [Sair]  |  |
|  +--------------------------------------------------------------------------------------+     |
|                                                                                               |
|  +--- Barra de Filtros (desabilitada quando sem dados) -------------------------------- +     |
|  |  [Buscar dispositivo: ____ (desabilitado)]       {Status v (desabilitado)}           |     |
|  +--------------------------------------------------------------------------------------+     |
|                                                                                               |
|                                                                                               |
|                                                                                               |
|                            +----------------------------------+                               |
|                            |                                  |                               |
|                            |    (icone: rede/switch)          |                               |
|                            |                                  |                               |
|                            |  Nenhum dispositivo SNMP         |                               |
|                            |  monitorado.                     |                               |
|                            |                                  |                               |
|                            |  O monitoramento precisa estar   |                               |
|                            |  ativo para coletar dados SNMP.  |                               |
|                            |                                  |                               |
|                            |  < Ir para Configuracao >        |                               |
|                            |                                  |                               |
|                            +----------------------------------+                               |
|                                                                                               |
|                                                                                               |
|                                                                                               |
+-----------------------------------------------------------------------------------------------+
```

**Estado filtro sem resultados:**

```
+----------------------------------------------------------------------------------------------+
|                                                                                              |
|  +--- Barra de Filtros (filtros ativos indicados por badge/cor) ----------------------- +    |
|  |  [Buscar dispositivo: SW-XYZ___________]       {Offline v}                           |    |
|  +--------------------------------------------------------------------------------------+    |
|                                                                                              |
|                            +----------------------------------+                              |
|                            |                                  |                              |
|                            |      (icone: lupa vazia)         |                              |
|                            |                                  |                              |
|                            |  Nenhum dispositivo para os      |                              |
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
| **Barra de filtros** | Input do usuario + valores estaticos | Campo de busca por texto livre + dropdown de status |
| **Campo de busca** | Input do usuario → filtro client-side sobre `deviceId` | Input de texto com placeholder "Buscar dispositivo" |
| **Dropdown Status** | Valores fixos: Todos, Online, Degraded, Offline | Select com opcao "Todos" como padrao |
| **Banner de aviso (sem MIBs)** | `GET /api/v1/mibs` → `meta.total == 0` | Banner amarelo com icone ⚠, mensagem descritiva e link para Configuracao (ADMIN) |
| **Tabela de dispositivos** | `GET /api/v1/protocols/SNMP/data` → `ProtocolDataListResponse.items[]` agrupados por `deviceId` | Tabela com linhas clicaveis |
| **Coluna Dispositivo** | `data.deviceId` dos registros SNMP | Texto com ID do dispositivo + descricao amigavel entre parenteses |
| **Coluna Status** | Derivado dos valores dos OIDs e timestamp da ultima coleta | Badge colorido: ● Online (verde), ⚠ Degraded (amarelo), ✕ Offline (vermelho) |
| **Coluna OIDs Coletados** | Contagem de OIDs distintos por `deviceId` | Numero inteiro |
| **Coluna Ultima Atualizacao** | `timestampUtc` mais recente por `deviceId` | Formato local: `DD/MM/AAAA HH:mm` (convertido de UTC) |
| **Paginacao** | `ProtocolDataListResponse.meta` | Padrao conforme `00-navegacao-global.md` secao 6.1 |
| **Drawer de detalhe** | `GET /api/v1/protocols/SNMP/data` filtrado por `deviceId` + `GET /api/v1/mibs` | Painel lateral direito (~40% largura) |
| **Tabela de OIDs (drawer)** | Dados SNMP do dispositivo + MIBs para traducao | Colunas: OID, Nome (MIB), Tipo, Valor |
| **Coluna Nome (MIB)** | Traducao OID→nome via MIBs carregados | Nome legivel se MIB disponivel; OID numerico caso contrario |
| **Dropdown OID (grafico)** | Lista de OIDs distintos do dispositivo | Select exibindo nome traduzido (MIB) ou OID numerico |
| **Campos De/Ate (grafico)** | Input do usuario | Date-time picker; envia como `fromUtc`/`toUtc` ISO 8601 |
| **Grafico de metricas** | `GET /api/v1/protocols/SNMP/metrics?fromUtc=...&toUtc=...` | Grafico de linha (time series) com eixo X = tempo, eixo Y = valor do OID selecionado |
| **Estado vazio** | `ProtocolDataListResponse.meta.total == 0` sem filtros | Ilustracao centralizada + mensagem + link para Configuracao |
| **Estado filtro vazio** | `ProtocolDataListResponse.meta.total == 0` com filtros ativos | Ilustracao + "Nenhum dispositivo para os filtros selecionados" + [Limpar filtros] |
| **Banner de erro de API** | HTTP 5xx / erro de rede | Banner vermelho inline no topo da area de conteudo |

---

## Dados e Endpoints

| Endpoint | Metodo | Uso na tela | Campos utilizados | Exemplo de chamada |
|---|---|---|---|---|
| `/api/v1/mibs` | GET | Carregar MIBs para traducao de OIDs (cacheado apos primeiro carregamento) | Response: `MibListResponse` → `items[]` com `name`, `content` (para parsing de OID→nome) | `GET /api/v1/mibs?page=1&pageSize=250` |
| `/api/v1/protocols/SNMP/data` | GET | Carregar dados SNMP persistidos (dispositivos e seus OIDs) | Query: `type`, `scdId`, `page`, `pageSize`, `sort`, `order`. Response: `ProtocolDataListResponse` → `items[]` com `id`, `protocol`, `type`, `data` (contendo `deviceId`, `oids[]`) | `GET /api/v1/protocols/SNMP/data?page=1&pageSize=50&sort=createdAtUtc&order=desc` |
| `/api/v1/protocols/SNMP/metrics` | GET | Carregar serie temporal de metricas SNMP para o grafico no drawer | Query: `fromUtc`, `toUtc`, `page`, `pageSize`, `sort`, `order`. Response: `MetricPointListResponse` → `items[]` com `timestampUtc`, `hostId`, `metrics` | `GET /api/v1/protocols/SNMP/metrics?fromUtc=2026-03-27T10:00:00Z&toUtc=2026-03-27T14:30:00Z&page=1&pageSize=200` |

**Schemas utilizados:**

| Schema | Campos | Observacoes |
|---|---|---|
| `SnmpIngestRequest` | `timestampUtc` (datetime), `deviceId` (string), `oids` (array de `{ oid, type, value }`) | Define a estrutura dos dados SNMP coletados pelo agente. Cada item em `oids` tem: `oid` (string, ex: "1.3.6.1.2.1.1.3.0"), `type` (string, ex: "integer", "string", "boolean"), `value` (number, integer, string ou boolean) |
| `MetricPoint` | `timestampUtc` (datetime), `hostId` (string), `metrics` (object, additionalProperties:true) | Ponto individual da serie temporal. `metrics` contem pares OID→valor para o instante |
| `MetricPointListResponse` | `items` (MetricPoint[]), `meta` (PageMeta) | Resposta paginada da serie temporal |
| `ProtocolDataRecord` | `id` (uuid), `protocol` (string), `type` (string), `scdId` (uuid), `createdAtUtc` (datetime), `data` (object, additionalProperties:true) | Registro generico de dados de protocolo. Para SNMP, `data` contem `deviceId` e `oids[]` |
| `ProtocolDataListResponse` | `items` (ProtocolDataRecord[]), `meta` (PageMeta) | Resposta paginada de dados SNMP |
| `Mib` | `mibId` (uuid), `name` (string), `version` (string\|null), `description` (string\|null), `content` (string), `createdAtUtc` (datetime), `updatedAtUtc` (datetime) | Conteudo textual da MIB, usado para traduzir OIDs numericos em nomes legiveis |
| `MibListResponse` | `items` (Mib[]), `meta` (PageMeta) | Lista paginada de MIBs |
| `PageMeta` | `page` (integer), `pageSize` (integer), `total` (integer) | Metadados de paginacao |

**Estrategia de carregamento de dados:**

```
> Pagina carrega
     |
     +--> GET /api/v1/mibs (paralelo)
     |     > Cacheia MIBs no client-side
     |     > Se meta.total == 0: exibe banner de aviso "Nenhum MIB cadastrado"
     |     > Usado para traduzir OIDs em nomes legiveis
     |
     +--> GET /api/v1/protocols/SNMP/data (paralelo)
           > Agrupa registros por deviceId
           > Renderiza tabela de dispositivos
           > Se meta.total == 0: exibe estado vazio
           |
           +--> (Apos usuario clicar em dispositivo e selecionar OID no drawer)
                 > GET /api/v1/protocols/SNMP/metrics?fromUtc=...&toUtc=...
                 > Renderiza grafico de metricas
```

**Polling periodico:**

- Os dados SNMP devem ser atualizados periodicamente via polling (intervalo sugerido: 30 segundos)
- Ao receber novos dados, a tabela de dispositivos atualiza os campos "OIDs Coletados", "Status" e "Ultima Atualizacao"
- Se o drawer estiver aberto, a tabela de OIDs e o grafico de metricas tambem sao atualizados

---

## Fluxos de Interacao

### 1. Carregamento inicial da pagina

```
> Usuario acessa /snmp
> Frontend executa em paralelo:
   > GET /api/v1/mibs?page=1&pageSize=250 (cache de MIBs)
   > GET /api/v1/protocols/SNMP/data?page=1&pageSize=50&sort=createdAtUtc&order=desc
> Resposta dos MIBs:
   > Se meta.total == 0: exibe banner de aviso amarelo no topo da tabela
   > Se meta.total > 0: armazena MIBs em cache local para traducao de OIDs
> Resposta dos dados SNMP:
   > Agrupa registros por deviceId
   > Para cada dispositivo: calcula status, contagem de OIDs e ultima atualizacao
   > Renderiza tabela de dispositivos
   > Renderiza paginacao com base em meta.total
   > Se meta.total == 0: exibe estado vazio "Nenhum dispositivo SNMP monitorado"
> Inicia polling periodico (a cada 30s) para atualizar dados SNMP
```

### 2. Clicar em dispositivo (abrir drawer)

```
> Usuario clica em uma linha da tabela de dispositivos (ex: SW-PROC-01)
> Drawer abre pela direita com animacao de slide-in
> Exibe cabecalho: nome do dispositivo + status
> Exibe tabela de OIDs do dispositivo:
   > Para cada OID: busca nome legivel nos MIBs cacheados
   > Se MIB encontrado: exibe nome traduzido (ex: "sysUpTime")
   > Se MIB nao encontrado: exibe OID numerico (ex: "1.3.6.1.2.1.1.3.0")
> Exibe area de grafico de metricas:
   > Dropdown OID pre-selecionado com o primeiro OID numerico da lista
   > Campos De/Ate pre-preenchidos com as ultimas 4 horas
   > Carrega serie temporal automaticamente (ver fluxo 3)
```

### 3. Selecionar OID no grafico de metricas

```
> Usuario seleciona um OID no dropdown {OID v} (ou altera periodo De/Ate)
> Frontend executa GET /api/v1/protocols/SNMP/metrics?fromUtc={de}&toUtc={ate}
> Filtra os MetricPoints pelo hostId (deviceId) do dispositivo selecionado
> Extrai os valores do OID selecionado de cada MetricPoint.metrics
> Renderiza grafico de linha com eixo X = timestampUtc, eixo Y = valor
> Se nenhum dado no periodo: exibe mensagem inline no grafico "Sem dados para o periodo selecionado"
```

### 4. Filtrar e buscar dispositivos

```
> Usuario digita no campo de busca ou seleciona status no dropdown
> Filtro de busca: filtra client-side por deviceId (contem texto digitado)
> Filtro de status: filtra client-side pelo status calculado do dispositivo
> Tabela atualiza com dispositivos que correspondem aos filtros
> Se nenhum dispositivo corresponde: exibe "Nenhum dispositivo para os filtros selecionados" + [Limpar filtros]
> Se drawer estava aberto e o dispositivo selecionado nao esta mais visivel: fecha o drawer
```

### 5. Refresh periodico de dados SNMP

```
> A cada 30 segundos (intervalo configuravel):
> Frontend executa GET /api/v1/protocols/SNMP/data?page={pagina_atual}&pageSize=50&sort=createdAtUtc&order=desc
> Compara novos dados com dados atuais:
   > Se houve mudanca: atualiza tabela de dispositivos (status, OIDs, timestamp)
   > Se drawer aberto: atualiza tabela de OIDs e recarrega grafico de metricas
> Se houver erro de rede: nao exibe erro intrusivo, tenta novamente no proximo ciclo
> Se houver erro 500 persistente (3+ tentativas): exibe banner de erro discreto
```

---

## Estados

| Estado | Condicao | Comportamento Visual |
|---|---|---|
| **Com dados + MIBs** | Dispositivos SNMP respondendo e MIBs cadastrados | Tabela de dispositivos com OIDs traduzidos para nomes legiveis. Funcionamento normal da tela |
| **Com dados, sem MIBs** | Dispositivos SNMP respondendo mas `GET /mibs` retorna `meta.total == 0` | Banner amarelo ⚠ no topo: "Nenhum MIB cadastrado. Os OIDs serao exibidos sem traducao para nomes legiveis." + link para Configuracao (ADMIN). Tabela de dispositivos exibida normalmente. Drawer exibe OIDs numericos na coluna "Nome (MIB)" |
| **Sem dispositivos** | `GET /protocols/SNMP/data` retorna `meta.total == 0` sem filtros ativos | Ilustracao centralizada (icone de rede/switch) + "Nenhum dispositivo SNMP monitorado." + "O monitoramento precisa estar ativo para coletar dados SNMP." + link « Ir para Configuracao » |
| **Monitoramento inativo** | Dados SNMP vazios e monitoramento em estado STOPPED | Mesmo visual do estado "Sem dispositivos" — o link « Ir para Configuracao » direciona para a tab Monitoramento na tela de Configuracao |
| **Filtro sem resultados** | Filtros aplicados mas nenhum dispositivo corresponde (`meta.total == 0` com filtros) | Area de tabela vazia + "Nenhum dispositivo para os filtros selecionados" + botao [Limpar filtros] |
| **Dispositivo Online** | Todos os OIDs com valores normais, ultima coleta recente | Linha com badge ● Online (verde), fundo neutro |
| **Dispositivo Degraded** | Alguns OIDs com valores anormais (erros de porta, interfaces down) | Linha com badge ⚠ Degraded (amarelo), fundo levemente amarelo palido |
| **Dispositivo Offline** | Nenhuma resposta SNMP dentro do periodo esperado de polling | Linha com badge ✕ Offline (vermelho), fundo levemente vermelho palido, contagem de OIDs = 0 |
| **Drawer aberto** | Usuario clicou em um dispositivo | Drawer com tabela de OIDs + grafico de metricas. Tabela principal visivel ao fundo |
| **Grafico sem dados** | OID selecionado sem metricas no periodo informado | Area do grafico com mensagem "Sem dados para o periodo selecionado" |
| **Carregando** | Requisicao em andamento (lista, detalhe ou metricas) | Skeleton/shimmer na tabela ou spinner no drawer/grafico |
| **Erro de API** | HTTP 500 ou erro de rede na listagem | Banner de erro vermelho inline no topo da area de conteudo: "Erro ao carregar dados SNMP. Tente novamente." + botao [Tentar novamente] |
| **Erro de API no drawer** | HTTP 500 ou erro de rede ao carregar metricas | Mensagem de erro dentro do drawer + botao [Tentar novamente] |

---

## Permissoes por Role

Esta tela e **somente leitura** — nao possui acoes de escrita. Todos os roles possuem acesso completo a visualizacao dos dados SNMP. A unica diferenca de role refere-se ao link de navegacao para o cadastro de MIBs.

| Elemento | ADMIN | OPERATOR | VIEWER |
|---|---|---|---|
| Visualizar tabela de dispositivos | ✓ | ✓ | ✓ |
| Ver detalhe do dispositivo (drawer) | ✓ | ✓ | ✓ |
| Ver tabela de OIDs no drawer | ✓ | ✓ | ✓ |
| Ver grafico de metricas no drawer | ✓ | ✓ | ✓ |
| Selecionar OID e periodo no grafico | ✓ | ✓ | ✓ |
| Filtrar/buscar dispositivos | ✓ | ✓ | ✓ |
| Ordenar tabela | ✓ | ✓ | ✓ |
| Navegar entre paginas | ✓ | ✓ | ✓ |
| Link « Cadastrar MIBs na Configuracao » (banner sem MIBs) | Visivel | **Oculto** | **Oculto** |
| Link « Ir para Configuracao » (estado vazio) | ✓ | ✓ | **Oculto** |

**Nota:** O link « Cadastrar MIBs na Configuracao » no banner de aviso e o unico elemento com restricao por role nesta tela. Para OPERATOR e VIEWER, o banner ainda e exibido (pois a informacao sobre OIDs sem traducao e relevante para todos), porem sem o link de acao, ja que apenas ADMIN pode gerenciar MIBs.

---

## Referencias

- `00-navegacao-global.md` — Layout master, padroes de tabela, drawer, paginacao, barra de filtros, convencoes visuais globais, RBAC e regras de timestamp
- `07-configuracao.md` — Gerenciamento de MIBs (secao 7.3). Os MIBs cadastrados nessa tela sao consumidos aqui para traduzir OIDs em nomes legiveis

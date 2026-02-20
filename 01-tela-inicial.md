# Tela 1 — Tela Inicial (Topologia de Rede)

## Objetivo

Visualizar a topologia de rede da subestacao com overlay de status em tempo real. Esta e a tela inicial da aplicacao apos login — o usuario e redirecionado automaticamente para ca apos autenticacao bem-sucedida (ver `08-autenticacao.md`).

A tela apresenta os **IEDs** (Intelligent Electronic Devices) da subestacao organizados pelas **sub-redes** (Station Bus, Process Bus) definidas no SCD ativo, com indicadores visuais de:

- Estado de cada IED (normal, falha, ausente, inesperado)
- Contagem de alarmes ativos por IED (badges)
- Estado global do monitoramento (RUNNING, STOPPED, etc.)
- Conexoes e fluxos de dados entre IEDs (GOOSE, SV)

A subestacao monitorada possui 4 IEDs — L90_DIG (rele de protecao de linha), MU320E_LAB (merging unit), MU_360_1 (merging unit) e T60_DIG (rele de protecao de transformador) — distribuidos em 6 sub-redes (W1, W2, W4, W01, W02, W03).

---

## Wireframe

### Estado principal (com SCD ativo e monitoramento)

```
+------------------------------------------------------------------------------------------------+
|  +- Header Global (ver 00-navegacao-global.md) ----------------------------------------+       |
|  |  [Logo NMS]   Subestacao: MU_360_1    Mon: * RUNNING           operador@empresa.com |       |
|  +-------------------------------------------------------------------------------------+       |
+----------+-------------------------------------------------------------------------------------+
| SIDEBAR  |                                                                                     |
|          |  +- Indicador Global -------------------------------------------------------+       |
| > Topol. |  |  * Monitoramento: RUNNING desde 19/02/2026 10:00   4 IEDs      |  6 Redes|       |
|   Alm (2)|  +------------------------------------------------------------------------- +       |
|   Sinc   |                                                                                     |
|   Red    |  +- Area de Topologia (Grafo de Rede) --------------------------------------+       |
|   Com    |  |                                                                          |       |
|   SNMP   |  |  +- Station Bus (W1, W2) -- 8-MMS -------------------------------- +     |       |
|   Cfg    |  |  |                                                                 |     |       |
|          |  |  |  +-------------------+                  +-------------------+   |     |       |
|          |  |  |  | * L90_DIG         |                  | * T60_DIG         |   |     |       |
|          |  |  |  | GE Multilin       |                  | GE Vernova        |   |     |       |
|          |  |  |  | L90 v8.60         |                  | T60 v8.70         |   |     |       |
|          |  |  |  | ! 2 alarmes       |                  | 0 alarmes         |   |     |       |
|          |  |  |  +---------+---------+                  +---------+---------+   |     |       |
|          |  |  |            | S1, S2                               | S1, S2      |     |       |
|          |  |  +------------+--------------------------------------+-------------+     |       |
|          |  |               |                                      |                   |       |
|          |  |  +- Process Bus (W4) -- Layer 2 -----------------------------------------+       |
|          |  |  |            |                                      |                   |       |
|          |  |  |            | P1           GOOSE/SV                | P1                |       |
|          |  |  |  +---------+---------+  <-------------  +---------+---------+         |       |
|          |  |  |  | * MU_360_1        |  ---- SV ---->   | (L90_DIG)         |         |       |
|          |  |  |  | GE Vernova        |  ---- SV ---->   | (T60_DIG)         |         |       |
|          |  |  |  | MU360 v1.0        |  <-- GOOSE --    | (ref. acima)      |         |       |
|          |  |  |  | 0 alarmes         |                  +-------------------+         |       |
|          |  |  |  +---------+---------+                                                |       |
|          |  |  +------------+----------------------------------------------------------+       |
|          |  |               |                                                          |       |
|          |  |  +- Process Bus Principal (W01) ----------------------------------------+  |     |
|          |  |  |            |                                                         |  |     |
|          |  |  |  +---------+---------+            +-------------------+              |  |     |
|          |  |  |  | (MU_360_1)        |            | o MU320E_LAB      |              |  |     |
|          |  |  |  | (ref. acima)      |            | GE                |              |  |     |
|          |  |  |  |                   |            | Meas. v6.1        |              |  |     |
|          |  |  |  |                   |            | AUSENTE           |              |  |     |
|          |  |  |  +-------------------+            +---------+---------+              |  |     |
|          |  |  +--------------------------------------------+------------------------+  |      |
|          |  |                                               |                          |       |
|          |  |  +- Redes Auxiliares ---------------------------------------------------+  |     |
|          |  |  |                                                                     |  |      |
|          |  |  |  +- W02 (Backup) --------+      +- W03 (Admin IP) --------+        |  |       |
|          |  |  |  |  (MU320E_LAB)         |      |  (MU_360_1)             |        |  |       |
|          |  |  |  |  Ethernet_2           |      |  ADMIN_AP               |        |  |       |
|          |  |  |  +-----------------------+      +--------------------------+        |  |      |
|          |  |  |                                                                     |  |      |
|          |  |  +--------------------------------------------------------------------- +  |     |
|          |  |                                                                          |       |
|          |  |  Nota: Este diagrama e CONCEITUAL. O grafo real sera renderizado por     |       |
|          |  |  uma biblioteca de grafos (Canvas/WebGL). Os IEDs que aparecem em        |       |
|          |  |  multiplas redes sao representados como um unico no do grafo com         |       |
|          |  |  conexoes para cada sub-rede. As zonas (Station Bus, Process Bus, etc.)  |       |
|          |  |  agrupam visualmente os nos conectados a cada sub-rede.                  |       |
|          |  |                                                                          |       |
|          |  +--------------------------------------------------------------------------+       |
|          |                                                                                     |
|          |  +- Legenda --------------------------------------------------------------- +       |
|          |  |  * Normal (verde, borda solida)      o Ausente (cinza, borda tracejada)  |       |
|          |  |  N Falha (vermelho, borda solida)    ! Inesperado (laranja, pontilhada)  |       |
|          |  |                                                                          |       |
|          |  |  --- SV -->  Fluxo Sampled Values    --- GOOSE -->  Fluxo GOOSE          |       |
|          |  +--------------------------------------------------------------------------+       |
|          |                                                                                     |
| [Logout] |                                                                                     |
+----------+-------------------------------------------------------------------------------------+
```

**Detalhamento da area de topologia:**

- **Zonas de rede**: Cada sub-rede e representada como uma regiao visual (container com borda e label). As zonas agrupam os IEDs conectados aquela sub-rede.
- **Nos de IED**: Cada IED e representado como um card/caixa contendo nome, vendor, tipo, firmware e badge de alarmes. Um IED que aparece em multiplas redes (ex: L90_DIG em W1, W2, W4, W01) e renderizado como um unico no do grafo com arestas para cada zona.
- **Conexoes (arestas)**: Linhas entre IEDs indicando fluxos de dados. Setas indicam direcao (publisher -> subscriber). Podem ser rotuladas com o protocolo (SV, GOOSE).
- **Layout automatico**: O grafo utiliza layout automatico (force-directed ou hierarquico). O designer deve prever que a disposicao exata dos nos pode variar — o importante e a organizacao por zonas e a visibilidade das conexoes.
- **Interacao**: Zoom e pan na area do grafo. Clique em um no de IED abre o drawer de detalhe.

**Indicador global de monitoramento:**

- Posicionado acima da area de topologia, em uma barra informativa.
- Exibe: estado do monitoramento (icone + texto colorido), data/hora de inicio, contagem total de IEDs e redes.
- Cores do estado conforme `00-navegacao-global.md` secao 3.

---

### Estado do drawer (detalhe do IED)

Abre pela direita quando o usuario clica em um no de IED na topologia. Ocupa aproximadamente 40% da largura da area de conteudo. A topologia permanece visivel ao fundo (com overlay escurecido opcional).

**Exemplo com dados reais do L90_DIG:**

```
                              +--------------------------------------------+ 
                              | [X]                                        | 
                              |                                            | 
                              |  * L90_DIG                                 | 
                              |  -----------------------------------       | 
                              |                                            | 
                              |  -- Informacoes do IED --                  | 
                              |                                            | 
                              |  Vendor:         GE Multilin               | 
                              |  Tipo:           L90                       | 
                              |  Firmware:       8.60                      | 
                              |  Logical Devices: 3                        | 
                              |  Data Attributes: 75                       | 
                              |  Data Points                               | 
                              |  monitorados:    42                        | 
                              |                                            | 
                              |  -- Access Points / Redes --               | 
                              |                                            | 
                              |  +-----------+-------+------------------+  | 
                              |  | AP        | Rede  | IP               |  | 
                              |  +-----------+-------+------------------+  | 
                              |  | S1        | W1    | 192.168.110.104  |  | 
                              |  | S2        | W2    | 192.168.110.104  |  | 
                              |  | P1        | W4    | (Layer 2)        |  | 
                              |  | P1        | W01   | (via W4)         |  | 
                              |  +-----------+-------+------------------+  | 
                              |                                            | 
                              |  -- Alarmes Ativos (2) --                  | 
                              |                                            | 
                              |  +-----------+--------------------------+  | 
                              |  | Sev.      | Resumo                   |  | 
                              |  +-----------+--------------------------+  | 
                              |  | N CRITICAL| Perda de sinal SV        |  | 
                              |  |           | stream LDTM1             |  | 
                              |  | ! MAJOR   | Jitter SV acima do       |  | 
                              |  |           | limiar                   |  | 
                              |  +-----------+--------------------------+  | 
                              |                                            | 
                              |  < Ver todos os alarmes deste IED >        | 
                              |  (navega para Tela 2 filtrado por          | 
                              |   L90_DIG)                                 | 
                              |                                            | 
                              |  -- Monitoramento --                       | 
                              |                                            | 
                              |  +-----------+----------+----------+       | 
                              |  | Protocolo | Target   | Status   |       | 
                              |  +-----------+----------+----------+       | 
                              |  | SV        | LDTM1    | * running|       | 
                              |  | GOOSE     | GoCB01   | * running|       | 
                              |  | PTP       | L90_DIG  | * running|       | 
                              |  +-----------+----------+----------+       | 
                              |                                            | 
                              +--------------------------------------------+ 
```

**Detalhamento do drawer:**

- **Cabecalho**: Indicador de estado (icone colorido) + nome do IED. Botao `[X]` para fechar.
- **Secao Informacoes**: Campos estaticos extraidos do SCD — vendor, tipo, firmware, contagem de Logical Devices, Data Attributes e Data Points monitorados. Fonte: `GET /scds/{scdId}/IEDS`.
- **Secao Access Points / Redes**: Tabela com os access points do IED e suas respectivas redes e enderecos IP. Fonte: `GET /scds/{scdId}/NETWORKS` cruzado com dados do IED.
- **Secao Alarmes Ativos**: Mini-tabela com os alarmes nao reconhecidos deste IED. Exibe severidade (badge colorido) e resumo. Link `« Ver todos os alarmes deste IED »` navega para `02-alarmes.md` com filtro pre-aplicado pelo IED. Fonte: `GET /alarms?ack=false` filtrado client-side pelo IED.
- **Secao Monitoramento**: Tabela com os itens de monitoramento ativos para este IED. Exibe protocolo, target e status. Fonte: `GET /monitoring` -> campo `monitorings[]` filtrado pelo IED.

---

### Estado sem SCD ativo

Exibido quando a chamada `GET /scds?latest=true` nao retorna nenhum SCD com status `ACTIVE`, ou quando nao ha nenhum SCD cadastrado no sistema. A area de topologia e substituida por um estado vazio centralizado.

```
+----------------------------------------------------------------------------------------------+
|  +- Header Global (ver 00-navegacao-global.md) -------------------------------------------+  |
|  |  [Logo NMS]   Subestacao: -              Mon: o STOPPED           admin@empresa.com    |  |
|  +----------------------------------------------------------------------------------------+  |
+----------+-----------------------------------------------------------------------------------+
| SIDEBAR  |                                                                                   |
|          |                                                                                   |
| > Topol. |                                                                                   |
|   Alm    |                                                                                   |
|   Sinc   |                                                                                   |
|   Red    |                                                                                   |
|   Com    |         +-----------------------------------------------------+                   |
|   SNMP   |         |                                                     |                   |
|   Cfg    |         |              (icone de rede/topologia               |                   |
|          |         |               com nos desconectados)                |                   |
|          |         |                                                     |                   |
|          |         |      Nenhum SCD ativo encontrado.                   |                   |
|          |         |                                                     |                   |
|          |         |      A topologia sera exibida apos a ativacao       |                   |
|          |         |      de um arquivo SCD na tela de Configuracao.     |                   |
|          |         |                                                     |                   |
|          |         |      < Ir para Configuracao >                       |                   |
|          |         |                                                     |                   |
|          |         +-----------------------------------------------------+                   |
|          |                                                                                   |
|          |                                                                                   |
| [Logout] |                                                                                   |
+----------+-----------------------------------------------------------------------------------+
```

**Detalhamento:**

- O icone ilustrativo deve representar uma rede vazia ou topologia desconectada.
- A mensagem explica que o SCD e necessario para gerar a topologia.
- O link `« Ir para Configuracao »` navega para `/configuracao` (Tela 7), onde o usuario pode fazer upload e ativar um SCD.
- O header exibe `Subestacao: —` (traco) e o monitoramento como `○ STOPPED` quando nao ha SCD ativo.
- A visibilidade do link `« Ir para Configuracao »` depende do role do usuario (ver secao Permissoes).

---

## Componentes

| Componente | Descricao | Fonte de Dados |
|---|---|---|
| **Indicador global de monitoramento** | Barra informativa acima da topologia exibindo estado do monitoramento (icone + texto colorido), data/hora de inicio (`sinceUtc`), contagem de IEDs e redes | `GET /monitoring` → `state`, `sinceUtc`; `GET /scds/{scdId}/summary` → contagens |
| **Area de topologia (grafo)** | Area principal renderizada por biblioteca de grafos (Canvas/WebGL). Suporta zoom, pan e clique em nos. Organizada por zonas de rede | `GET /scds/{scdId}/summary`, `GET /scds/{scdId}/IEDS`, `GET /scds/{scdId}/NETWORKS` |
| **No de IED** | Card dentro do grafo representando um IED. Exibe: indicador de estado (cor), nome, vendor, tipo/firmware, badge de alarmes ativos | SCD (estatico) + `GET /alarms?ack=false` (dinamico) + `GET /monitoring` (dinamico) |
| **Zona de rede** | Container visual agrupando IEDs pertencentes a uma mesma sub-rede. Exibe label com nome e tipo da rede (ex: "Station Bus (W1) — 8-MMS") | `GET /scds/{scdId}/NETWORKS` |
| **Conexoes (arestas)** | Linhas entre nos de IED representando fluxos de dados. Setas indicam direcao (publisher → subscriber). Podem ser rotuladas com protocolo (SV, GOOSE) | `GET /scds/{scdId}/NETWORKS` → trafego por sub-rede |
| **Legenda de estados** | Barra inferior explicando os icones e cores dos estados de IED: normal (verde), falha (vermelho), ausente (cinza), inesperado (laranja) | Estatico |
| **Drawer de detalhe do IED** | Painel lateral direito (~40% largura) com informacoes detalhadas do IED selecionado. Contem secoes: informacoes, redes, alarmes ativos, monitoramento | Multiplos endpoints (ver secao Dados) |
| **Secao Alarmes no drawer** | Mini-tabela de alarmes ativos do IED selecionado + link para Tela 2 filtrada | `GET /alarms?ack=false` filtrado por IED |
| **Secao Monitoramento no drawer** | Tabela de itens de monitoramento ativos para o IED selecionado (protocolo, target, status) | `GET /monitoring` → `monitorings[]` filtrado por IED |
| **Estado vazio** | Tela centralizada quando nao ha SCD ativo. Icone ilustrativo + mensagem + link para Configuracao | `GET /scds?latest=true` retorna vazio ou sem status ACTIVE |

---

## Dados e Endpoints

| # | Metodo | Endpoint | Uso na tela | Campos utilizados | Exemplo |
|---|---|---|---|---|---|
| 1 | `GET` | `/api/v1/scds?latest=true` | Obter o SCD ativo | `scdId`, `status` | Retorna o SCD mais recente; se `status != ACTIVE`, exibir estado vazio |
| 2 | `GET` | `/api/v1/scds/{scdId}/summary` | Nome da subestacao, contagens de IEDs e redes | `substationName`, `ieds[]` (name, vendor, monitoredDataPointsCount), `networks[]` (name, type, connectedDevices) | `GET /api/v1/scds/181b236a/summary` → `substationName: "MU_360_1"`, `ieds: [{iedName: "L90_DIG", vendor: "GE Multilin", monitoredDataPointsCount: 42}, ...]`, `networks: [{name: "W1", type: "8-MMS", connectedDevices: ["L90_DIG", "T60_DIG"]}, ...]` |
| 3 | `GET` | `/api/v1/scds/{scdId}/IEDS` | Detalhes completos de cada IED (para o drawer e nos do grafo) | `data` (object) — contem vendor, type, firmware, logical devices, data attributes, access points por IED | `GET /api/v1/scds/181b236a/IEDS` → Dados detalhados de L90_DIG, MU320E_LAB, MU_360_1, T60_DIG |
| 4 | `GET` | `/api/v1/scds/{scdId}/NETWORKS` | Topologia de rede (sub-redes, access points, trafego) | `data` (object) — contem nome, tipo, IEDs conectados, enderecos IP, trafego GOOSE/SV por rede | `GET /api/v1/scds/181b236a/NETWORKS` → 6 sub-redes: W1, W2, W4, W01, W02, W03 com dispositivos conectados |
| 5 | `GET` | `/api/v1/monitoring` | Estado global do monitoramento + lista de itens monitorados | `enabled`, `state` (STOPPED/STARTING/RUNNING/STOPPING/ERROR), `sinceUtc`, `monitorings[]` (id, protocol, target, status, updatedAtUtc) | `GET /api/v1/monitoring` → `state: "RUNNING"`, `sinceUtc: "2026-02-19T13:00:00Z"`, `monitorings: [{protocol: "SV", target: "LDTM1", status: "running"}, ...]` |
| 6 | `GET` | `/api/v1/alarms?ack=false` | Alarmes ativos nao reconhecidos (para badges nos IEDs e secao do drawer) | `items[]` (alarmId, timestampUtc, type, severity, summary, details, occurrences, ack, pcapId) | `GET /api/v1/alarms?ack=false` → `items: [{type: "LOSS_OF_SIGNAL", severity: "CRITICAL", summary: "Perda de sinal SV stream LDTM1"}, ...]` |

**Schemas relevantes:**

| Schema | Campos | Observacoes |
|---|---|---|
| `ScdSummaryResponse` | `scdId` (uuid), `status` (ScdStatus), `substationName` (string), `parsedAtUtc` (datetime), `sourceFiles` (string[]), `ieds` (ScdSummaryIed[]), `networks` (ScdSummaryNetwork[]) | Resposta leve para renderizacao inicial |
| `ScdSummaryIed` | `iedName` (string), `vendor` (string), `monitoredDataPointsCount` (integer) | Dados minimos de cada IED |
| `ScdSummaryNetwork` | `name` (string), `type` (string), `connectedDevices` (string[]) | Dados minimos de cada sub-rede |
| `ScdProtocolDetailsResponse` | `scdId`, `status`, `substationName`, `parsedAtUtc`, `sourceFiles`, `data` (object) | Views: SV, GOOSE, IEDS, NETWORKS, VALIDATIONS |
| `MonitoringStatusResponse` | `enabled` (boolean), `state` (MonitoringState), `sinceUtc` (datetime), `monitorings` (MonitoringItem[]), `stoppedAtUtc` (datetime/null), `stoppedBy` (string/null) | Singleton — estado global |
| `MonitoringItem` | `id` (uuid), `protocol` (ProtocolEnum), `target` (string), `status` (string), `updatedAtUtc` (datetime) | Item individual monitorado |
| `Alarm` | `alarmId` (uuid), `timestampUtc` (datetime), `type` (string), `severity` (LOW/MEDIUM/MAJOR/CRITICAL), `summary` (string), `details` (object), `occurrences` (integer), `ack` (boolean), `pcapId` (uuid/null) | `details` e dinamico (additionalProperties) |
| `AlarmRecord` | `scdId` (uuid), `alarm` (Alarm) | Wrapper com referencia ao SCD |

---

### Estrategia de carregamento de dados

O carregamento e feito em etapas para garantir uma experiencia progressiva:

```
> Etapa 1 - Dados essenciais (bloqueante)
   GET /scds?latest=true
   +- Se nao ha SCD ativo > exibir estado vazio (fim do fluxo)
   +- Se ha SCD ativo > obter scdId

> Etapa 2 - Estrutura da topologia (bloqueante)
   GET /scds/{scdId}/summary
   +- Renderizar skeleton da topologia: zonas de rede + nos de IED com dados basicos
   +- Exibir nome da subestacao no header

> Etapa 3 - Detalhes (paralelo, lazy load)
   GET /scds/{scdId}/IEDS       > enriquecer nos com dados detalhados
   GET /scds/{scdId}/NETWORKS   > renderizar conexoes e trafego entre zonas

> Etapa 4 - Overlay em tempo real (paralelo, polling periodico)
   GET /monitoring               > aplicar estado de monitoramento nos nos
   GET /alarms?ack=false         > aplicar badges de alarme nos nos

> Polling continuo (a cada 15-30 segundos):
   GET /monitoring               > atualizar overlay de monitoramento
   GET /alarms?ack=false         > atualizar badges de alarme
   (Nao re-renderizar a topologia inteira - apenas atualizar os atributos visuais dos nos)
```

**Cache no client:**

- Os dados do SCD (etapas 1-3) nao mudam apos ativacao — podem ser cacheados com TTL longo ou ate invalidacao manual.
- Os dados de monitoramento e alarmes (etapa 4) sao dinamicos e devem ser atualizados via polling periodico.

---

## Fluxos de Interacao

### 1. Carregamento da pagina

```
> Usuario acessa / (tela inicial) apos login
> Frontend executa GET /api/v1/scds?latest=true
> Se nao ha SCD ativo:
     > Exibir estado vazio com link < Ir para Configuracao >
     > Fim do fluxo
> Se ha SCD ativo:
     > GET /api/v1/scds/{scdId}/summary
     > Renderizar skeleton: zonas de rede + nos de IED com nome e vendor
     > Exibir substationName no header
     > Paralelo: GET /scds/{scdId}/IEDS + GET /scds/{scdId}/NETWORKS
     > Renderizar grafo completo com conexoes e detalhes
     > Paralelo: GET /monitoring + GET /alarms?ack=false
     > Aplicar overlay: estados dos nos + badges de alarme
     > Iniciar polling periodico (etapa 4)
```

### 2. Clique em um no de IED (abrir drawer)

```
> Usuario clica em um no de IED na topologia (ex: L90_DIG)
> Frontend coleta dados do IED ja carregados (cache das etapas 2-3)
> Frontend filtra alarmes ativos para este IED (cache da etapa 4)
> Frontend filtra itens de monitoramento para este IED (cache da etapa 4)
> Drawer abre pela direita com animacao de slide-in
> Exibe:
     - Informacoes estaticas: vendor, tipo, firmware, LDs, DAs, data points
     - Access points / redes conectadas
     - Alarmes ativos com severidade e resumo
     - Itens de monitoramento com protocolo, target e status
```

### 3. Navegar para alarmes do IED (a partir do drawer)

```
> Usuario clica < Ver todos os alarmes deste IED > no drawer
> Frontend navega para /alarmes com filtro pre-aplicado (rota interna da aplicacao):
     /alarmes?ied={iedName}
> Tela 2 (Alarmes) abre e filtra client-side pelos alarmes do IED selecionado
  Nota: A API nao possui parametro de filtro por IED. O frontend deve carregar
  os alarmes e filtrar localmente pelo campo `summary` ou `details` que contem
  o nome do IED, ou implementar logica de correlacao propria.
```

### 4. Navegar para Configuracao (a partir do estado vazio)

```
> Usuario clica < Ir para Configuracao > no estado vazio
> Frontend navega para /configuracao (Tela 7)
> Tela 7 exibe opcoes de upload e ativacao de SCD
```

### 5. Atualizacao periodica (polling)

```
> A cada 15-30 segundos:
     > GET /api/v1/monitoring
     > GET /api/v1/alarms?ack=false
> Frontend compara dados novos com dados anteriores
> Se houve mudanca:
     > Atualizar icones de estado nos nos de IED (sem re-renderizar o grafo)
     > Atualizar badges de contagem de alarmes nos nos
     > Se o drawer esta aberto: atualizar secoes de alarmes e monitoramento
> Se o estado global de monitoramento mudou:
     > Atualizar indicador global no header e na barra informativa
```

### 6. Zoom e navegacao na topologia

```
> Usuario usa scroll/pinch para zoom in/out na area de topologia
> Zoom semantico:
     - Zoom baixo: exibe apenas zonas de rede com labels e nos resumidos
     - Zoom alto: exibe detalhes dos nos (vendor, firmware, badges)
> Usuario arrasta para pan (mover a area visivel do grafo)
> Double-click em zona de rede: zoom para enquadrar a zona
```

---

## Estados

### Estados do IED

| Estado | Condicao | Visual |
|---|---|---|
| **Normal** | IED definido no SCD + presente no monitoramento + sem alarmes ativos | ● Verde, borda solida |
| **Falha** | IED com alarmes ativos (`ack=false`) | ✕ Vermelho, borda solida, badge com contagem de alarmes |
| **Ausente** | IED definido no SCD mas nao encontrado nos itens de monitoramento | ○ Cinza, borda tracejada |
| **Inesperado** | Dispositivo presente nos itens de monitoramento mas nao definido no SCD | ⚠ Laranja, borda pontilhada |

**Prioridade de estados (quando multiplas condicoes se aplicam):**
1. Falha (maior prioridade — se ha alarmes, exibir como falha independente de outros fatores)
2. Inesperado
3. Ausente
4. Normal (menor prioridade)

### Estados da tela

| Estado | Condicao | Visual |
|---|---|---|
| **Topologia ativa** | SCD ativo + dados carregados | Grafo completo com nos, zonas, conexoes e overlays |
| **Carregando** | Requisicoes em andamento (etapas 1-3) | Skeleton/shimmer na area de topologia |
| **Sem SCD ativo** | `GET /scds?latest=true` nao retorna SCD ativo | Estado vazio com icone, mensagem e link para Configuracao |
| **Erro de API** | Falha na comunicacao com o backend | Banner de erro inline no topo da area de conteudo: "Erro ao carregar topologia. Tente novamente." + botao [Tentar novamente] |
| **Monitoramento parado** | `state == STOPPED` | Indicador global `○ STOPPED` (cinza). Nos de IED sem overlay de status (apenas dados estaticos do SCD) |
| **Monitoramento em erro** | `state == ERROR` | Indicador global `⚠ ERROR` (vermelho). Banner de alerta na barra informativa |

### Estados do drawer

| Estado | Condicao | Visual |
|---|---|---|
| **Aberto com dados** | IED selecionado, dados carregados | Drawer completo com todas as secoes preenchidas |
| **Sem alarmes** | IED sem alarmes ativos | Secao "Alarmes Ativos" exibe: "Nenhum alarme ativo" (texto cinza). Link para Tela 2 ainda visivel |
| **Sem monitoramento** | IED nao presente nos itens de monitoramento | Secao "Monitoramento" exibe: "IED nao monitorado" (texto cinza, estado Ausente) |
| **Carregando** | Dados ainda sendo processados | Skeleton/shimmer dentro do drawer |

---

## Permissoes por Role

| Elemento | ADMIN | OPERATOR | VIEWER |
|---|---|---|---|
| Visualizar topologia | ✓ | ✓ | ✓ |
| Zoom e pan no grafo | ✓ | ✓ | ✓ |
| Ver detalhe do IED (drawer) | ✓ | ✓ | ✓ |
| Ver alarmes no drawer | ✓ | ✓ | ✓ |
| Ver monitoramento no drawer | ✓ | ✓ | ✓ |
| Link « Ver todos os alarmes » | ✓ | ✓ | ✓ |
| Link « Ir para Configuracao » (estado vazio) | ✓ Visivel e ativo | Link visivel (navega para Tela 7, onde botoes de acao serao desabilitados) | **Oculto** (VIEWER nao configura — remover o link, manter apenas a mensagem informativa) |

**Notas:**

- Esta tela e exclusivamente de **visualizacao** — nao ha acoes de escrita (nao ha botoes de start/stop, ACK, upload, etc.).
- A unica diferenca por role e a visibilidade do link para Configuracao no estado vazio.
- O botao de start/stop do monitoramento **nao esta presente nesta tela** — ele pertence a Tela 7 (Configuracao).

---

## Referencias

- `00-navegacao-global.md` — Layout master, header, sidebar, drawer, convencoes visuais e RBAC
- `02-alarmes.md` — Tela de alarmes (destino do link « Ver todos os alarmes deste IED »)
- `07-configuracao.md` — Tela de configuracao (destino do link « Ir para Configuracao » no estado vazio)
- `08-autenticacao.md` — Tela de login (origem do redirect para esta tela apos autenticacao)
- `docs/parsed-scd/scd.md` — Dados reais da subestacao (4 IEDs, 6 sub-redes, fluxos GOOSE/SV)
- `docs/structure-proposal/frontend-recommendations.md` — Estrategia de renderizacao de topologia (carregamento progressivo, virtualizacao, cache, zoom semantico)

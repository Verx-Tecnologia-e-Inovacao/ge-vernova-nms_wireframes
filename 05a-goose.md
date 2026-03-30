# Tela 5a — GOOSE

## Objetivo

Visualizar os control blocks GOOSE da subestacao, combinando dados estaticos extraidos do SCD (publishers, subscribers, datasets, parametros de rede) com metricas em tempo real coletadas pelo motor de monitoramento (contadores, timing, qualidade, redundancia). Acessivel via rota `/goose` na sidebar de navegacao (submenu Protocols).

**Escala real:** Em subestacoes de producao, a pagina pode conter 237 control blocks GOOSE distribuidos em 9 sub-redes. A interface usa paginacao obrigatoria (pageSize=50) e filtros para navegacao eficiente.

**Exemplo do wireframe:** A subestacao de exemplo possui 4 IEDs reais com 11 control blocks GOOSE em 4 sub-redes (W01, W1, W2, W4).

---

## Wireframe

### Estado principal (com SCD ativo)

```
+--------------------------------------------------------------------------------------------------+
|  +- Header Global (ver 00-navegacao-global.md) ------------------------------------------+       |
|  |  [Logo NMS]   Subestacao: MU_360_1    Mon: * RUNNING           operador@empresa.com   |       |
|  +----------------------------------------------------------------------------------------+       |
+----------+---------------------------------------------------------------------------------------+
| SIDEBAR  |                                                                                       |
|          |  +- Barra de Filtros ---------------------------------------------------------+       |
| > Topol. |  |                                                                           |       |
|   Alm (2)|  |  {Publisher v}  {Subscriber v}  {Subnet v}  {Status v}        [Limpar]    |       |
|   Sinc   |  |                                                                           |       |
|   Red    |  +----------------------------------------------------------------------------+       |
| > Proto  |                                                                                       |
|   GOOSE  |  +- Tabela de Control Blocks GOOSE (237 registros) --------------------------+       |
|   SV     |  |                                                                           |       |
|   MMS    |  |  Publisher ^| CB Name  | GOOSE ID    | Dataset | Dest MAC          | ...  |       |
|   SNMP   |  |  ----------+----------+-------------+---------+-------------------+------|       |
|   Cfg    |  |  L90_DIG   | GoCB01   | TxGOOSE1    | 36 ent. | 01:0c:cd:01:00:00 | ...  |       |
|          |  |  L90_DIG   | GoCB02   | TxGOOSE2    |  2 ent. | -                 | ...  |       |
|          |  |  MU_360_1  | GCB01    | MU_360_1..  | 64 ent. | 01:0c:cd:01:00:00 | ...  |       |
|          |  |  MU_360_1  | GCB03    | MU_360_1..  | 18 ent. | 01:0c:cd:01:00:02 | ...  |       |
|          |  |  MU_360_1  | GCB04    | MU_360_1..  | 16 ent. | 01:0c:cd:01:00:03 | ...  |       |
|          |  |  ----------+----------+-------------+---------+-------------------+------|       |
|          |  |  MU320E_LAB| FastGO.. | MU320E_LA.. | 63 ent. | -                 | ...  |       |
|          |  |  MU320E_LAB| GOCB1    | GOID_BER    |  8 ent. | -                 | ...  |       |
|          |  |  MU320E_LAB| GOCB2    | GOID_Fixed  |  8 ent. | -                 | ...  |       |
|          |  |  T60_DIG   | GoCB01   | T60_DIG_G01 |  2 ent. | -                 | ...  |       |
|          |  |  T60_DIG   | GoCB02   | T60_DIG_G02 |  2 ent. | -                 | ...  |       |
|          |  |                                                                           |       |
|          |  |  --- colunas continuam (scroll horizontal ou layout responsivo) ---        |       |
|          |  |                                                                           |       |
|          |  |  ... | APP ID | VLAN   | Conf Rev | Subnet | Subscriber(s)    | Status   |       |
|          |  |  ... +--------+--------+----------+--------+------------------+----------|       |
|          |  |  ... | 0x0000 | 0x0837 | 1        | W4     | MU_360_1         | * Normal |       |
|          |  |  ... | 0x0001 | -      | 1        | W2     | -                | o Sem sub|       |
|          |  |  ... | 0x0000 | 0x0000 | 1        | W01    | L90_DIG          | * Normal |       |
|          |  |  ... | 0x0000 | 0x0000 | 1        | W01    | L90_DIG          | * Normal |       |
|          |  |  ... | 0x0000 | 0x0000 | 1        | W01    | L90_DIG          | * Normal |       |
|          |  |  ... +--------+--------+----------+--------+------------------+----------|       |
|          |  |  ... | -      | -      | -        | W01    | -                | o Sem sub|       |
|          |  |  ... | -      | -      | -        | W01    | -                | o Sem sub|       |
|          |  |  ... | -      | -      | -        | W01    | -                | o Sem sub|       |
|          |  |  ... | -      | -      | -        | W1     | -                | o Sem sub|       |
|          |  |  ... | -      | -      | -        | W4     | -                | o Sem sub|       |
|          |  |                                                                           |       |
|          |  +----------------------------------------------------------------------------+       |
|          |                                                                                       |
|          |  < Anterior  Pagina 1 de 5  Proxima >    Exibindo 50 de 237                           |
|          |                                                                                       |
| [Logout] |                                                                                       |
+----------+---------------------------------------------------------------------------------------+
```

**Detalhamento da tabela GOOSE:**

- **Colunas:** Publisher, CB Name (`cb_name`), GOOSE ID (`goose_id`/`stream_id`), Dataset (contagem de entradas), Dest MAC (`dest_mac`), APP ID (`app_id`), VLAN (`vlan_id`), Conf Rev (`conf_rev`), Subnet (`subnet`), Subscriber(s) (`subscribers[]`), Status
- **Ordenacao padrao:** Publisher (ascendente)
- **Colunas ordenaveis:** Publisher, CB Name, Dataset, Subnet
- **Agrupamento visual:** Linhas COM subscribers aparecem primeiro, separadas das linhas SEM subscriber por divisor visual mais espesso
- **Linhas sem subscriber:** Exibem `—` na coluna Subscriber. MAC, VLAN, APP ID e Conf Rev podem exibir `—` se nao definidos no SCD. Linha com opacidade reduzida (muted)
- **Truncamento:** GOOSE ID pode ser longo — truncar com ellipsis e tooltip com valor completo
- **Clique na linha:** Abre o drawer de detalhe do control block
- **Paginacao:** Obrigatoria para 237 registros, pageSize=50 (conforme `00-navegacao-global.md` secao 6.1)

**Barra de filtros:**

- **Publisher:** Dropdown com todos os IEDs publishers (`Todos`, `L90_DIG`, `MU_360_1`, `MU320E_LAB`, `T60_DIG`, ...)
- **Subscriber:** Dropdown com todos os IEDs subscribers (`Todos`, `L90_DIG`, `MU_360_1`, ..., `Sem subscriber`)
- **Subnet:** Dropdown com todas as sub-redes (`Todas`, `W01`, `W1`, `W2`, `W4`, ...)
- **Status:** Dropdown com estados (`Todos`, `Normal`, `Sem subscriber`)
- **Botao Limpar:** Reseta todos os filtros
- Filtros aplicados client-side sobre dados ja carregados

---

### Drawer de detalhe do control block GOOSE

Abre pela direita quando o usuario clica em uma linha da tabela. Ocupa aproximadamente 40% da largura da area de conteudo. Combina dados estaticos do SCD com metricas runtime do parser GOOSE.

**Exemplo com dados do GoCB01 (L90_DIG → MU_360_1):**

```
                              +----------------------------------------------+
                              | [X]                                          |
                              |                                              |
                              |  * GoCB01 — L90_DIG                          |
                              |  ----------------------------------------    |
                              |                                              |
                              |  -- Time Window --                           |
                              |                                              |
                              |  Inicio:  15/01/2024 10:30:00                |
                              |  Fim:     15/01/2024 10:30:05                |
                              |                                              |
                              |  -- Identificacao --                         |
                              |                                              |
                              |  Publisher:      L90_DIG                     |
                              |  Control Block:  GoCB01                      |
                              |  GOOSE ID:       TxGOOSE1                    |
                              |  Dataset:        36 entradas                 |
                              |  Dest MAC:       01:0c:cd:01:00:00           |
                              |  APP ID:         0x0000                      |
                              |  VLAN ID:        0x0837                      |
                              |  VLAN Priority:  4                           |
                              |  Conf Rev:       1                           |
                              |  TTL:            2000 ms                     |
                              |  Subnet:         W4                          |
                              |                                              |
                              |  -- Subscribers --                           |
                              |                                              |
                              |  +------------+---------------------+        |
                              |  | IED        | Status              |        |
                              |  +------------+---------------------+        |
                              |  | MU_360_1   | * Recebendo         |        |
                              |  +------------+---------------------+        |
                              |                                              |
                              |  -- Contadores --                            |
                              |                                              |
                              |  +---------------------+--------+--------+  |
                              |  | Metrica             | eth0   | eth1   |  |
                              |  +---------------------+--------+--------+  |
                              |  | total_packets       | 500    | 498    |  |
                              |  | number_of_events    | 3      | 3      |  |
                              |  | lost_packets        | 0      | N 2    |  |
                              |  |   (1a ocorrencia)   | -      | 15/01  |  |
                              |  |                     |        | 10:30  |  |
                              |  | out_of_order        | 0      | 0      |  |
                              |  | duplicates          | 0      | 0      |  |
                              |  | ied_restarts        | 0      | 0      |  |
                              |  | missed_events       | 0      | 0      |  |
                              |  +---------------------+--------+--------+  |
                              |                                              |
                              |  -- Timing --                                |
                              |                                              |
                              |  +---------------------+--------+--------+  |
                              |  | Metrica             | eth0   | eth1   |  |
                              |  +---------------------+--------+--------+  |
                              |  | min_time (s)        | 0.001  | 0.001  |  |
                              |  | max_time (s)        | 1.002  | 1.003  |  |
                              |  | transfer_time (s)   | 0.002  | 0.002  |  |
                              |  +---------------------+--------+--------+  |
                              |                                              |
                              |  -- Qualidade --                             |
                              |                                              |
                              |  quality_bits:           [good]              |
                              |  simulated:              [false]             |
                              |  clock_failure:          [false]             |
                              |  clock_not_synchronized: [false]             |
                              |                                              |
                              |  -- Redundancia --                           |
                              |                                              |
                              |  +---------------------+--------+--------+  |
                              |  | Metrica             | eth0   | eth1   |  |
                              |  +---------------------+--------+--------+  |
                              |  | PRP LANs            | [A]    | [B]    |  |
                              |  | HSR Paths           | []     | []     |  |
                              |  +---------------------+--------+--------+  |
                              |                                              |
                              |  Diagnostico PRP: Normal                     |
                              |  (LAN-A em eth0, LAN-B em eth1)             |
                              |                                              |
                              +----------------------------------------------+
```

**Detalhamento do drawer GOOSE:**

- **Cabecalho:** Indicador de estado (icone colorido) + CB Name + Publisher. Botao `[X]` para fechar.
- **Secao Time Window:** Inicio (`start`) e fim (`end`) da captura PCAP. Fonte: `GET /protocols/GOOSE/metrics` → `time_window`.
- **Secao Identificacao:** Dados estaticos do SCD — publisher, control_block, goose_id, dataset (entries), dest_mac, app_id, vlan_id, vlan_priority, conf_rev, ttl, subnet. Fonte: `GET /scds/{scdId}/GOOSE`.
- **Secao Subscribers:** Tabela com IEDs assinantes e status de recepcao. Fonte: SCD (subscribers[]) + metricas (status).
- **Secao Contadores:** Tabela com colunas **eth0 | eth1** para comparacao por interface. Exibe total_packets, number_of_events, lost_packets (+ts), out_of_order (+ts), duplicates (+ts), ied_restarts (+ts), missed_events (+ts). Valores com anomalia (> 0 para perdas/erros) destacados em vermelho. Timestamps `_ts` exibidos abaixo do valor em fonte menor. Se `_ts == null`, exibir `—`. Fonte: `GET /protocols/GOOSE/metrics` → `stats.{interface}.counters`.
- **Secao Timing:** Tabela com colunas **eth0 | eth1**. Exibe min_time, max_time, transfer_time (em segundos). Fonte: `GET /protocols/GOOSE/metrics` → `stats.{interface}.timing`.
- **Secao Qualidade:** Lista de arrays observados — quality_bits[], simulated[], clock_failure[], clock_not_synchronized[]. Se contem valores anormais (ex: `[true]` em clock_failure), destacar em vermelho. Fonte: `GET /protocols/GOOSE/metrics` → campos raiz da metrica.
- **Secao Redundancia:** Tabela com colunas **eth0 | eth1**. Exibe prp_lans[] e hsr_paths[]. Abaixo da tabela, texto de diagnostico PRP/HSR baseado na combinacao de valores (ver documentacao GOOSE). Fonte: `GET /protocols/GOOSE/metrics` → `stats.{interface}.prp_lans`, `stats.{interface}.hsr_paths`.

---

### Drawer de control block SEM subscriber

Exemplo com GoCB02 (L90_DIG, sem subscriber):

```
                              +----------------------------------------------+
                              | [X]                                          |
                              |                                              |
                              |  o GoCB02 — L90_DIG                          |
                              |  ----------------------------------------    |
                              |                                              |
                              |  -- Identificacao --                         |
                              |                                              |
                              |  Publisher:      L90_DIG                     |
                              |  Control Block:  GoCB02                      |
                              |  GOOSE ID:       TxGOOSE2                    |
                              |  Dataset:        2 entradas                  |
                              |  Dest MAC:       -                           |
                              |  APP ID:         0x0001                      |
                              |  VLAN ID:        -                           |
                              |  Conf Rev:       1                           |
                              |  Subnet:         W2                          |
                              |                                              |
                              |  -- Subscribers --                           |
                              |                                              |
                              |  +--------------------------------------+    |
                              |  |                                      |    |
                              |  |  Nenhum subscriber configurado       |    |
                              |  |  no SCD para este control block.     |    |
                              |  |                                      |    |
                              |  +--------------------------------------+    |
                              |                                              |
                              |  -- Metricas --                              |
                              |                                              |
                              |  Sem metricas disponiveis.                   |
                              |  (Control block sem subscriber nao e         |
                              |   monitorado pelo motor de captura)          |
                              |                                              |
                              +----------------------------------------------+
```

---

### Drawer sem metricas (monitoramento inativo)

Quando o monitoramento esta parado (`state == STOPPED`) ou nao ha dados de metricas para o control block:

```
                              +----------------------------------------------+
                              | [X]                                          |
                              |                                              |
                              |  * GoCB01 — L90_DIG                          |
                              |  ----------------------------------------    |
                              |                                              |
                              |  -- Identificacao --                         |
                              |  (campos estaticos do SCD - identicos ao     |
                              |   drawer com metricas)                       |
                              |                                              |
                              |  -- Subscribers --                           |
                              |  (tabela de subscribers com status)          |
                              |                                              |
                              |  -- Metricas --                              |
                              |                                              |
                              |  +--------------------------------------+    |
                              |  |                                      |    |
                              |  |  Metricas indisponiveis.             |    |
                              |  |  Monitoramento inativo ou dados      |    |
                              |  |  ainda nao coletados para este       |    |
                              |  |  control block.                      |    |
                              |  |                                      |    |
                              |  +--------------------------------------+    |
                              |                                              |
                              +----------------------------------------------+
```

---

### Estado sem SCD ativo

Exibido quando a chamada `GET /scds?latest=true` nao retorna nenhum SCD com status `ACTIVE`.

```
+--------------------------------------------------------------------------------------------------+
|  +- Header Global (ver 00-navegacao-global.md) ------------------------------------------+       |
|  |  [Logo NMS]   Subestacao: -              Mon: o STOPPED           admin@empresa.com   |       |
|  +----------------------------------------------------------------------------------------+       |
+----------+---------------------------------------------------------------------------------------+
| SIDEBAR  |                                                                                       |
|          |                                                                                       |
| > Topol. |                                                                                       |
|   Alm    |                                                                                       |
|   Sinc   |                                                                                       |
|   Red    |                                                                                       |
| > Proto  |          +-----------------------------------------------------+                      |
|   GOOSE  |          |                                                     |                      |
|   SV     |          |          (icone: setas bidirecionais /              |                      |
|   MMS    |          |           comunicacao GOOSE)                        |                      |
|   SNMP   |          |                                                     |                      |
|   Cfg    |          |      Nenhum SCD ativo.                              |                      |
|          |          |                                                     |                      |
|          |          |      Os control blocks GOOSE serao exibidos          |                      |
|          |          |      apos a ativacao de um arquivo SCD              |                      |
|          |          |      em Configuracao.                               |                      |
|          |          |                                                     |                      |
|          |          |      < Ir para Configuracao >                       |                      |
|          |          |                                                     |                      |
|          |          +-----------------------------------------------------+                      |
|          |                                                                                       |
| [Logout] |                                                                                       |
+----------+---------------------------------------------------------------------------------------+
```

**Detalhamento:**

- O link `« Ir para Configuracao »` navega para `/settings/scd`.
- A visibilidade do link depende do role do usuario (ver secao Permissoes).

---

## Componentes

| Componente | Descricao | Fonte de Dados |
|---|---|---|
| **Barra de filtros** | Dropdowns: Publisher, Subscriber, Subnet, Status. Botao Limpar. Filtros aplicados client-side | Valores extraidos do SCD (IEDs, sub-redes) |
| **Tabela de control blocks GOOSE** | Tabela paginada (pageSize=50) com dados do SCD. Colunas ordenaveis. Clique abre drawer | `GET /scds/{scdId}/GOOSE` |
| **Drawer de detalhe GOOSE** | Painel lateral ~40% largura. 7 secoes estruturadas: Time Window, Identificacao, Subscribers, Contadores (eth0/eth1), Timing (eth0/eth1), Qualidade, Redundancia (eth0/eth1) | SCD (estatico) + `GET /protocols/GOOSE/metrics` (runtime) |
| **Tabela por interface (eth0/eth1)** | Padrao reutilizado nas secoes Contadores, Timing e Redundancia. Colunas por interface para comparacao visual rapida. Valores anomalos destacados em vermelho | `GET /protocols/GOOSE/metrics` → `stats` |
| **Diagnostico PRP/HSR** | Texto abaixo da tabela de redundancia interpretando a combinacao de prp_lans/hsr_paths entre interfaces | Logica client-side sobre dados de redundancia |
| **Linha muted (sem subscriber)** | Linha com opacidade reduzida e `—` na coluna Subscriber para CBs sem assinantes | Dados do SCD |
| **Estado vazio (sem SCD)** | Tela centralizada com icone, mensagem e link para `/settings/scd` | `GET /scds?latest=true` retorna vazio ou sem ACTIVE |
| **Paginacao** | Padrao conforme `00-navegacao-global.md` secao 6.1. PageSize=50 | `meta` da resposta de API |
| **Banner de erro** | Erro inline no topo da area de conteudo | HTTP 4xx/5xx ou erro de rede |

---

## Dados e Endpoints

### Dados estaticos (SCD)

| # | Metodo | Endpoint | Uso na tela | Campos utilizados |
|---|---|---|---|---|
| 1 | `GET` | `/api/v1/scds?latest=true` | Verificar se ha SCD ativo | `scdId`, `status` |
| 2 | `GET` | `/api/v1/scds/{scdId}/GOOSE` | Tabela principal: lista de control blocks com dados estaticos | `data[]` — cada CB contem: `publisher_ied`, `cb_name`, `goose_id`, `dataset`, `dataset_entries`, `dest_mac`, `app_id`, `vlan_id`, `vlan_priority`, `conf_rev`, `retransmission_time`, `subscribers[]`, `subnetwork`, `source_file` |

### Metricas runtime (PCAP)

| # | Metodo | Endpoint | Uso na tela | Campos utilizados |
|---|---|---|---|---|
| 3 | `GET` | `/api/v1/protocols/GOOSE/metrics` | Drawer: metricas estruturadas do parser GOOSE | `time_window` (start, end), `interfaces[]`, `invalid_packets`, `metrics[]` — cada metrica contem: `control_block`, `app_id`, `goose_id`, `dest_mac`, `vlan_id`, `vlan_priority`, `dataset`, `dataset_entries`, `conf_rev`, `ttl`, `quality_bits[]`, `simulated[]`, `clock_failure[]`, `clock_not_synchronized[]`, `interfaces[]`, `stats.{iface}` (counters, timing, prp_lans, hsr_paths) |

### Dados persistidos

| # | Metodo | Endpoint | Uso na tela | Campos utilizados |
|---|---|---|---|---|
| 4 | `GET` | `/api/v1/protocols/GOOSE/data` | Alternativa ao endpoint SCD para dados GOOSE persistidos | `items[]` (ProtocolDataRecord) com `data` (object) |

### Estrutura detalhada das metricas GOOSE (por interface)

**Counters (`stats.{iface}.counters`):**

| Campo | Tipo | Descricao |
|---|---|---|
| `total_packets` | int | Total de pacotes recebidos nesta interface |
| `number_of_events` | int | Total de eventos (mudancas de stNum) |
| `lost_packets` | int | Pacotes perdidos (gaps reais no sqNum) |
| `lost_packets_ts` | datetime/null | Timestamp da primeira perda |
| `out_of_order` | int | Pacotes fora de sequencia |
| `out_of_order_ts` | datetime/null | Timestamp da primeira ocorrencia |
| `duplicates` | int | Pacotes duplicados (mesmo stNum + sqNum) |
| `duplicates_ts` | datetime/null | Timestamp da primeira duplicata |
| `ied_restarts` | int | Restarts do IED (stNum retrocedeu) |
| `ied_restarts_ts` | datetime/null | Timestamp do primeiro restart |
| `missed_events` | int | Eventos perdidos (gap no stNum > 1) |
| `missed_events_ts` | datetime/null | Timestamp do primeiro evento perdido |

**Timing (`stats.{iface}.timing`):**

| Campo | Tipo | Descricao |
|---|---|---|
| `min_time` | float | Menor intervalo entre pacotes (s) |
| `max_time` | float | Maior intervalo entre pacotes (s) |
| `transfer_time` | float | Maximo transfer time em eventos (s) |

**Redundancia (`stats.{iface}`):**

| Campo | Tipo | Descricao |
|---|---|---|
| `prp_lans` | string[] | LANs PRP observadas: `["A"]`, `["B"]`, `["A","B"]`, `[]` |
| `hsr_paths` | string[] | Paths HSR observados: `["A"]`, `["B"]`, `["A","B"]`, `[]` |

---

### Estrategia de carregamento de dados

```
> Etapa 1 - Verificacao de SCD (bloqueante)
   GET /scds?latest=true
   +- Se nao ha SCD ativo > exibir estado vazio (fim do fluxo)
   +- Se ha SCD ativo > obter scdId

> Etapa 2 - Dados do SCD (bloqueante, para tabela)
   GET /scds/{scdId}/GOOSE
   +- Renderizar tabela com dados estaticos (237 CBs paginados)
   +- Popular opcoes dos filtros (publishers, subscribers, subnets)

> Etapa 3 - Metricas runtime (sob demanda, para drawer)
   GET /protocols/GOOSE/metrics
   +- Dados carregados quando usuario abre o drawer (clique na linha)
   +- Correlacao por control_block entre SCD e metricas

> Polling continuo (a cada 15-30 segundos):
   GET /protocols/GOOSE/metrics
   +- Se drawer aberto: atualizar secoes de contadores, timing, qualidade, redundancia
   +- Nao atualizar tabela principal (metricas ficam no drawer)
```

**Cache no client:**

- Dados do SCD (etapas 1-2) nao mudam apos ativacao — cachear com TTL longo
- Metricas (etapa 3) sao dinamicas — atualizar via polling quando drawer aberto
- Filtros aplicados client-side sobre dados ja carregados — nao disparam novas requisicoes

---

### Exemplos de JSON (para designs Figma)

Os exemplos abaixo representam dados realistas para popular os componentes visuais.

#### Exemplo: Linha da tabela (dados SCD — 1 control block)

```json
{
  "publisher_ied": "L90_DIG",
  "ld_inst": "Master",
  "cb_name": "GoCB01",
  "control_block": "L90_DIGMaster/LLN0$GO$GoCB01",
  "goose_id": "TxGOOSE1",
  "dataset": "L90_DIGMaster/LLN0$TT6DataSet1",
  "dataset_entries": 36,
  "conf_rev": 1,
  "app_id": "0x0000",
  "dest_mac": "01:0c:cd:01:00:00",
  "vlan_id": "0x0837",
  "vlan_priority": 4,
  "retransmission_time": 1,
  "subscribers": ["MU_360_1"],
  "subnetwork": "W4",
  "source_file": "L90_DIG-Lab_20260129.CID"
}
```

#### Exemplo: Linha da tabela — sem subscriber

```json
{
  "publisher_ied": "L90_DIG",
  "ld_inst": "Master",
  "cb_name": "GoCB02",
  "control_block": "L90_DIGMaster/LLN0$GO$GoCB02",
  "goose_id": "TxGOOSE2",
  "dataset": "L90_DIGMaster/LLN0$TT6DataSet2",
  "dataset_entries": 2,
  "conf_rev": 1,
  "app_id": "0x0001",
  "dest_mac": "01:0c:cd:01:00:01",
  "vlan_id": "0x0BBB",
  "vlan_priority": 4,
  "retransmission_time": 1,
  "subscribers": [],
  "subnetwork": "W2",
  "source_file": "L90_DIG-Lab_20260129.CID"
}
```

#### Exemplo: Drawer completo — metricas GOOSE (1 control block com 2 interfaces)

```json
{
  "time_window": {
    "start": "2024-01-15T10:30:00.000000Z",
    "end": "2024-01-15T10:30:05.000000Z"
  },
  "interfaces": ["eth0", "eth1"],
  "invalid_packets": 0,
  "metrics": [
    {
      "control_block": "L90_DIGMaster/LLN0$GO$GoCB01",
      "app_id": "0x0000",
      "dataset": "L90_DIGMaster/LLN0$TT6DataSet1",
      "dataset_entries": 36,
      "goose_id": "TxGOOSE1",
      "dest_mac": "01:0c:cd:01:00:00",
      "vlan_id": "0x0837",
      "vlan_priority": 4,
      "conf_rev": 1,
      "ttl": 2000,
      "quality_bits": ["good"],
      "simulated": [false],
      "clock_failure": [false],
      "clock_not_synchronized": [false],
      "interfaces": ["eth0", "eth1"],
      "stats": {
        "eth0": {
          "prp_lans": ["A"],
          "hsr_paths": [],
          "counters": {
            "total_packets": 500,
            "number_of_events": 3,
            "lost_packets": 0,
            "lost_packets_ts": null,
            "out_of_order": 0,
            "out_of_order_ts": null,
            "duplicates": 0,
            "duplicates_ts": null,
            "ied_restarts": 0,
            "ied_restarts_ts": null,
            "missed_events": 0,
            "missed_events_ts": null
          },
          "timing": {
            "min_time": 0.001,
            "max_time": 1.002,
            "transfer_time": 0.002345
          }
        },
        "eth1": {
          "prp_lans": ["B"],
          "hsr_paths": [],
          "counters": {
            "total_packets": 498,
            "number_of_events": 3,
            "lost_packets": 2,
            "lost_packets_ts": "2024-01-15T10:30:00.123456Z",
            "out_of_order": 0,
            "out_of_order_ts": null,
            "duplicates": 0,
            "duplicates_ts": null,
            "ied_restarts": 0,
            "ied_restarts_ts": null,
            "missed_events": 0,
            "missed_events_ts": null
          },
          "timing": {
            "min_time": 0.001,
            "max_time": 1.003,
            "transfer_time": 0.002456
          }
        }
      }
    }
  ]
}
```

#### Exemplo: Drawer — control block com anomalias (perdas + restart)

```json
{
  "control_block": "MU_360_1LDSUIED/LLN0$GO$GCB01",
  "app_id": "0x0000",
  "dataset": "MU_360_1LDSUIED/LLN0$DSGoCB01",
  "dataset_entries": 64,
  "goose_id": "MU_360_1LDSUIED/LLN0$GO$GCB01",
  "dest_mac": "01:0c:cd:01:00:00",
  "vlan_id": "0x0000",
  "vlan_priority": 4,
  "conf_rev": 1,
  "ttl": 2000,
  "quality_bits": ["good", "questionable"],
  "simulated": [false],
  "clock_failure": [false, true],
  "clock_not_synchronized": [false],
  "interfaces": ["eth0", "eth1"],
  "stats": {
    "eth0": {
      "prp_lans": ["A"],
      "hsr_paths": [],
      "counters": {
        "total_packets": 12450,
        "number_of_events": 47,
        "lost_packets": 15,
        "lost_packets_ts": "2024-01-15T10:31:22.456789Z",
        "out_of_order": 3,
        "out_of_order_ts": "2024-01-15T10:32:01.234567Z",
        "duplicates": 0,
        "duplicates_ts": null,
        "ied_restarts": 1,
        "ied_restarts_ts": "2024-01-15T10:33:45.678901Z",
        "missed_events": 2,
        "missed_events_ts": "2024-01-15T10:31:22.456789Z"
      },
      "timing": {
        "min_time": 0.0008,
        "max_time": 2.105,
        "transfer_time": 0.015678
      }
    },
    "eth1": {
      "prp_lans": ["B"],
      "hsr_paths": [],
      "counters": {
        "total_packets": 12448,
        "number_of_events": 47,
        "lost_packets": 17,
        "lost_packets_ts": "2024-01-15T10:31:22.456812Z",
        "out_of_order": 1,
        "out_of_order_ts": "2024-01-15T10:32:01.234590Z",
        "duplicates": 2,
        "duplicates_ts": "2024-01-15T10:34:12.345678Z",
        "ied_restarts": 1,
        "ied_restarts_ts": "2024-01-15T10:33:45.679012Z",
        "missed_events": 2,
        "missed_events_ts": "2024-01-15T10:31:22.456812Z"
      },
      "timing": {
        "min_time": 0.0009,
        "max_time": 2.108,
        "transfer_time": 0.016234
      }
    }
  }
}
```

#### Exemplo: Diagnostico de redundancia — cenarios

**PRP Normal:**
```json
{ "eth0": { "prp_lans": ["A"] }, "eth1": { "prp_lans": ["B"] } }
```
Diagnostico: ✅ Normal — LAN-A em eth0, LAN-B em eth1

**PRP LANs invertidas:**
```json
{ "eth0": { "prp_lans": ["B"] }, "eth1": { "prp_lans": ["A"] } }
```
Diagnostico: ⚠ LANs invertidas — cabos trocados?

**PRP Ambas LANs em ambas interfaces:**
```json
{ "eth0": { "prp_lans": ["A", "B"] }, "eth1": { "prp_lans": ["A", "B"] } }
```
Diagnostico: ❌ Problema de topologia — ambas LANs em ambas interfaces

**HSR Normal:**
```json
{ "eth0": { "hsr_paths": ["A", "B"] } }
```
Diagnostico: ✅ Normal — ambos paths do anel visiveis

**HSR Ruptura parcial:**
```json
{ "eth0": { "hsr_paths": ["A"] } }
```
Diagnostico: ⚠ Apenas um path — possivel ruptura no anel

**Sem redundancia:**
```json
{ "eth0": { "prp_lans": [], "hsr_paths": [] }, "eth1": { "prp_lans": [], "hsr_paths": [] } }
```
Diagnostico: Sem PRP/HSR nesta captura

---

## Fluxos de Interacao

### 1. Carregamento da pagina

```
> Usuario acessa /goose via sidebar
> Frontend executa GET /api/v1/scds?latest=true
> Se nao ha SCD ativo:
     > Exibir estado vazio com link < Ir para Configuracao >
     > Fim do fluxo
> Se ha SCD ativo:
     > GET /api/v1/scds/{scdId}/GOOSE
     > Renderizar tabela com 237 CBs (pagina 1, 50 registros)
     > Popular filtros com valores extraidos do SCD
```

### 2. Clique em linha da tabela (abrir drawer)

```
> Usuario clica em uma linha (ex: GoCB01 - L90_DIG)
> Frontend coleta dados estaticos do CB do cache SCD
> Frontend executa GET /api/v1/protocols/GOOSE/metrics
> Correlacionar metrica pelo campo control_block
> Drawer abre pela direita com animacao slide-in
> Exibe secoes estruturadas:
     - Time Window (start/end da captura)
     - Identificacao (dados SCD)
     - Subscribers (IEDs + status)
     - Contadores (tabela eth0/eth1)
     - Timing (tabela eth0/eth1)
     - Qualidade (arrays observados)
     - Redundancia (tabela eth0/eth1 + diagnostico PRP/HSR)
> Iniciar polling de metricas para manter drawer atualizado
```

### 3. Atualizacao periodica (polling)

```
> A cada 15-30 segundos (apenas se drawer aberto):
     > GET /api/v1/protocols/GOOSE/metrics
> Frontend compara dados novos com dados anteriores
> Se houve mudanca:
     > Atualizar secoes de contadores, timing, qualidade, redundancia no drawer
```

### 4. Aplicar filtros na tabela

```
> Usuario seleciona filtro (ex: Publisher = "MU_360_1")
> Tabela filtra client-side (dados ja carregados do SCD)
> Linhas que nao correspondem ao filtro ficam ocultas
> Paginacao recalcula total de resultados
> Combinar multiplos filtros (publisher + subnet + status)
> Botao [Limpar] reseta todos os filtros
```

### 5. Navegar para Configuracao (a partir do estado vazio)

```
> Usuario clica < Ir para Configuracao > no estado vazio
> Frontend navega para /settings/scd
```

---

## Estados

### Estados gerais da tela

| Estado | Condicao | Visual |
|---|---|---|
| **Com SCD + metricas** | SCD ativo + monitoramento RUNNING + metricas disponiveis | Tabela com dados do SCD. Drawer exibe metricas estruturadas |
| **Com SCD, sem metricas** | SCD ativo + monitoramento STOPPED ou sem dados | Tabela com dados estaticos do SCD. Drawer: "Metricas indisponiveis — monitoramento inativo" |
| **Sem SCD ativo** | `GET /scds?latest=true` nao retorna SCD ativo | Estado vazio centralizado com icone, mensagem e link para `/settings/scd` |
| **Carregando** | Requisicoes em andamento | Skeleton/shimmer na tabela |
| **Erro de API** | HTTP 500 ou erro de rede | Banner de erro inline: "Erro ao carregar dados GOOSE. Tente novamente." + botao [Tentar novamente] |

### Estados por linha da tabela

| Estado | Condicao | Visual |
|---|---|---|
| **Normal** | CB com subscribers configurados no SCD | * Verde na coluna Status. Linha com aparencia padrao |
| **Sem subscriber** | Lista de subscribers vazia no SCD | o Cinza na coluna Status. Texto `—` na coluna Subscriber. Linha muted |

### Estados do drawer

| Estado | Condicao | Visual |
|---|---|---|
| **Aberto com dados completos** | CB selecionado + metricas disponiveis | Todas as 7 secoes preenchidas |
| **Aberto sem metricas** | CB selecionado + monitoramento inativo | Secoes de identificacao e subscribers preenchidas. Secoes de metricas: "Metricas indisponiveis" |
| **Aberto sem subscriber** | CB sem subscribers no SCD | Secao Subscribers: "Nenhum subscriber configurado". Secao de metricas: "Sem metricas" |
| **Carregando** | Metricas sendo buscadas | Skeleton/shimmer nas secoes de metricas |
| **Com anomalia** | Contadores com perdas/erros > 0 | Valores anomalos destacados em vermelho nas tabelas eth0/eth1 |

---

## Permissoes por Role

| Elemento | ADMIN | OPERATOR | VIEWER |
|---|---|---|---|
| Visualizar tabela GOOSE | ✓ | ✓ | ✓ |
| Aplicar filtros | ✓ | ✓ | ✓ |
| Abrir drawer de detalhe | ✓ | ✓ | ✓ |
| Visualizar metricas runtime | ✓ | ✓ | ✓ |
| Visualizar redundancia PRP/HSR | ✓ | ✓ | ✓ |
| Link « Ir para Configuracao » (estado vazio) | ✓ Visivel e ativo | Link visivel (navega para `/settings/scd`, onde botoes de acao serao desabilitados) | **Oculto** (VIEWER nao configura — remover o link, manter apenas a mensagem informativa) |

**Notas:**

- Esta pagina e exclusivamente de **leitura** — nao ha acoes de escrita.
- A unica diferenca por role e a visibilidade do link para Configuracao no estado vazio.

---

## Referencias

- `00-navegacao-global.md` — Layout master, header, sidebar, drawer, convencoes visuais, paginacao e RBAC
- `01-tela-inicial.md` — Topologia de rede (drawer de edge referencia `/goose` para detalhes do stream)
- `05b-sampled-values.md` — Pagina Sampled Values (estrutura similar, mesma familia de protocolos)
- `05c-mms.md` — Pagina MMS (mesma familia de protocolos)
- `07-configuracao.md` — Tela de configuracao em `/settings/scd` (destino do link no estado vazio)
- `docs/extras-2026-03-16/3.2. Documentacao GOOSE.md` — Parser GOOSE: estrutura JSON, metricas por interface, contadores, timing, qualidade, redundancia PRP/HSR
- `docs/extras-2026-03-16/3. Exemplo GOOSE.json` — Exemplo real de saida do parser GOOSE

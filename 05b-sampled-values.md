# Tela 5b — Sampled Values

## Objetivo

Visualizar os streams de Sampled Values (SV) da subestacao, combinando dados estaticos extraidos do SCD (publishers, subscribers, parametros de rede) com metricas em tempo real coletadas pelo motor de monitoramento (contadores, packet interval/jitter, processing time, clock drift, qualidade, redundancia). Acessivel via rota `/sampled-values` na sidebar de navegacao (submenu Protocols).

**Escala real:** Em subestacoes de producao, a pagina pode conter 80 streams SV. A interface usa paginacao (pageSize=50, 2 paginas) e filtros para navegacao eficiente.

**Exemplo do wireframe:** A subestacao de exemplo possui 42 IEDs com 80 SV streams. O wireframe mostra uma amostra representativa de 8 streams na sub-rede W01 (4800 samples/s, 60 Hz).

---

## Wireframe

### Estado principal (com SCD ativo)

```
+--------------------------------------------------------------------------------------------------+
|  +- Header Global (ver 00-navegacao-global.md) ------------------------------------------+       |
|  |  [Logo NMS]   Subestacao: SE Exemplo      Mon: * RUNNING           operador@empresa.com   |   |
|  +----------------------------------------------------------------------------------------+       |
+----------+---------------------------------------------------------------------------------------+
| SIDEBAR  |                                                                                       |
|          |  +- Barra de Filtros ---------------------------------------------------------+       |
| > Topol. |  |                                                                           |       |
|   Alm (2)|  |  {Publisher v}  {Subscriber v}  {Subnet v}  {Status v}        [Limpar]    |       |
|   Sinc   |  |                                                                           |       |
|   Red    |  +----------------------------------------------------------------------------+       |
| > Proto  |                                                                                       |
|   GOOSE  |  +- Tabela de Streams SV (80 registros) ------------------------------------+        |
|   SV     |  |                                                                           |       |
|   MMS    |  |  Publisher ^| SV ID            | APP ID | Dest MAC          | VLAN   | ...|       |
|   SNMP   |  |  ----------+------------------+--------+-------------------+--------+-----|       |
|   Cfg    |  |  MU2_3T3   | MU2_3T30101      | 0x4000 | 01:0c:cd:04:00:99 | 0x04B2 | ...|       |
|          |  |  MU1_3L1   | MU1_3L10101      | 0x4000 | 01:0c:cd:04:00:8b | 0x0461 | ...|       |
|          |  |  ----------+------------------+--------+-------------------+--------+-----|       |
|          |  |  MU1_0P3   | MU1_0P30201      | 0x4000 | 01:0c:cd:04:01:6f | 0x0460 | ...|       |
|          |  |  MU1_0P3   | MU1_0P30101      | 0x4000 | 01:0c:cd:04:00:a7 | 0x0460 | ...|       |
|          |  |  MU1_3P1   | MU1_3P10101      | 0x4000 | 01:0c:cd:04:00:88 | 0x0462 | ...|       |
|          |  |  MU2_0P3   | MU2_0P30101      | 0x4000 | 01:0c:cd:04:00:a8 | 0x04C4 | ...|       |
|          |  |  MU2_3L1   | MU2_3L10101      | 0x4000 | 01:0c:cd:04:00:8c | 0x04C5 | ...|       |
|          |  |  MU1_0T3   | MU1_0T30101      | 0x4000 | 01:0c:cd:04:00:9a | 0x0463 | ...|       |
|          |  |                                                                           |       |
|          |  |  --- colunas continuam (scroll horizontal ou layout responsivo) ---        |       |
|          |  |                                                                           |       |
|          |  |  ...| Rate | Freq | Subnet | Subscriber(s)      | Status                 |       |
|          |  |  ...+------+------+--------+--------------------+------------------------|       |
|          |  |  ...| 4800 | 60Hz | W01    | DUCD_3T3, DUPC_3B  | * Normal               |       |
|          |  |  ...| 4800 | 60Hz | W01    | -                  | o Sem subscriber       |       |
|          |  |  ...+------+------+--------+--------------------+------------------------|       |
|          |  |  ...| 4800 | 60Hz | W01    | -                  | o Sem subscriber       |       |
|          |  |  ...| 4800 | 60Hz | W01    | -                  | o Sem subscriber       |       |
|          |  |  ...| 4800 | 60Hz | W01    | -                  | o Sem subscriber       |       |
|          |  |  ...| 4800 | 60Hz | W01    | -                  | o Sem subscriber       |       |
|          |  |  ...| 4800 | 60Hz | W01    | -                  | o Sem subscriber       |       |
|          |  |  ...| 4800 | 60Hz | W01    | -                  | o Sem subscriber       |       |
|          |  |                                                                           |       |
|          |  +----------------------------------------------------------------------------+       |
|          |                                                                                       |
|          |  < Anterior  Pagina 1 de 2  Proxima >    Exibindo 50 de 80                            |
|          |                                                                                       |
| [Logout] |                                                                                       |
+----------+---------------------------------------------------------------------------------------+
```

**Detalhamento da tabela SV:**

- **Colunas:** Publisher, SV ID (`sv_id`), APP ID (`app_id`), Dest MAC (`dest_mac`), VLAN (`vlan_id`), Sampling Rate (`sampling_rate`), Nominal Freq (`nominal_freq`), Subnet (`subnetwork`), Subscriber(s) (`subscribers[]`), Status
- **Ordenacao padrao:** Publisher (ascendente)
- **Colunas ordenaveis:** Publisher, SV ID, APP ID, Subnet
- **Agrupamento visual:** Streams COM subscribers aparecem primeiro, separadas das SEM subscriber por divisor visual
- **Linhas sem subscriber:** Exibem `—` na coluna Subscriber. VLAN pode exibir `—` se nao definido. Linha com opacidade reduzida (muted)
- **Clique na linha:** Abre o drawer de detalhe do stream SV
- **Paginacao:** pageSize=50 (2 paginas para 80 streams)

**Barra de filtros:**

- **Publisher:** Dropdown com todos os IEDs publishers (`Todos`, `MU1_0P3`, `MU2_3T3`, `MU1_3L1`, ...)
- **Subscriber:** Dropdown com todos os IEDs subscribers (`Todos`, `DUCD_3T3`, `DUPC_3B`, ..., `Sem subscriber`)
- **Subnet:** Dropdown com todas as sub-redes (`Todas`, `W01`, ...)
- **Status:** Dropdown com estados (`Todos`, `Normal`, `Sem subscriber`)
- **Botao Limpar:** Reseta todos os filtros
- Filtros aplicados client-side sobre dados ja carregados

---

### Drawer de detalhe do stream SV

Abre pela direita quando o usuario clica em uma linha da tabela. Ocupa aproximadamente 40% da largura da area de conteudo. Combina dados estaticos do SCD com metricas runtime do parser SV.

**Exemplo com dados do MU1_0P30201 (MU1_0P3 → broadcast):**

```
                              +----------------------------------------------+
                              | [X]                                          |
                              |                                              |
                              |  * MU1_0P30201 -- MU1_0P3                    |
                              |  ----------------------------------------    |
                              |                                              |
                              |  -- Time Window --                           |
                              |                                              |
                              |  Inicio:  27/03/2026 19:19:00                |
                              |  Fim:     27/03/2026 19:19:13                |
                              |                                              |
                              |  -- Identificacao --                         |
                              |                                              |
                              |  Publisher:       MU1_0P3                    |
                              |  SV ID:           MU1_0P30201               |
                              |  APP ID:          0x4000                     |
                              |  Dest MAC:        01:0c:cd:04:01:6f          |
                              |  VLAN ID:         0x0460                     |
                              |  VLAN Priority:   4                          |
                              |  Dataset:         16 entradas                |
                              |  Conf Rev:        1                          |
                              |  Smp Synch:       global                     |
                              |  Sampling Rate:   4800 Hz                    |
                              |  Nominal Freq:    60 Hz                      |
                              |  Subnet:          W01                        |
                              |                                              |
                              |  -- Subscribers --                           |
                              |                                              |
                              |  Nenhum subscriber configurado (broadcast)   |
                              |                                              |
                              |  -- Contadores --                            |
                              |                                              |
                              |  +---------------------+--------+--------+  |
                              |  | Metrica             | eth0   | eth1   |  |
                              |  +---------------------+--------+--------+  |
                              |  | total_samples       | 60634  | 60636  |  |
                              |  | lost_packets        | N 27   | N 25   |  |
                              |  |   (1a ocorrencia)   | 27/03  | 27/03  |  |
                              |  |                     | 19:19  | 19:19  |  |
                              |  | out_of_order        | 0      | 0      |  |
                              |  | duplicates          | 0      | 0      |  |
                              |  +---------------------+--------+--------+  |
                              |                                              |
                              |  -- Packet Interval --                       |
                              |                                              |
                              |  +---------------------+--------+--------+  |
                              |  | Metrica             | eth0   | eth1   |  |
                              |  +---------------------+--------+--------+  |
                              |  | nominal (us)        | 208.33 | 208.33 |  |
                              |  | min (us)            | 6.91   | 5.82   |  |
                              |  |   (timestamp)       | 19:19  | 19:19  |  |
                              |  |                     | :12    | :12    |  |
                              |  | max (us)            |N13381  |N13156  |  |
                              |  |   (timestamp)       | 19:19  | 19:19  |  |
                              |  |                     | :04    | :04    |  |
                              |  | avg (us)            | 208.43 | 208.42 |  |
                              |  | std (us) -- jitter  | 133.66 | 135.81 |  |
                              |  +---------------------+--------+--------+  |
                              |                                              |
                              |  -- Processing Time --                       |
                              |  ! Requer sincronizacao PTP com mesmo        |
                              |    Grandmaster das MUs. Sem PTP, valores     |
                              |    representam apenas offset entre clocks.   |
                              |                                              |
                              |  +---------------------+--------+--------+  |
                              |  | Metrica             | eth0   | eth1   |  |
                              |  +---------------------+--------+--------+  |
                              |  | min (us)            | 233937 | 233981 |  |
                              |  | max (us)            | 247338 | 247382 |  |
                              |  |   (timestamp)       | 19:19  | 19:19  |  |
                              |  |                     | :04    | :04    |  |
                              |  | avg (us)            | 234082 | 234126 |  |
                              |  | std (us)            | 476.45 | 476.51 |  |
                              |  +---------------------+--------+--------+  |
                              |                                              |
                              |  -- Clock Drift --                           |
                              |                                              |
                              |  +---------------------+--------+--------+  |
                              |  | Metrica             | eth0   | eth1   |  |
                              |  +---------------------+--------+--------+  |
                              |  | min (us)            | -95.13 | -99.58 |  |
                              |  |   (timestamp)       | 19:19  | 19:19  |  |
                              |  |                     | :09    | :09    |  |
                              |  | max (us)            | 748.16 | 743.72 |  |
                              |  |   (timestamp)       | 19:19  | 19:19  |  |
                              |  |                     | :04    | :04    |  |
                              |  +---------------------+--------+--------+  |
                              |                                              |
                              |  -- Qualidade --                             |
                              |                                              |
                              |  quality_bits:  [good, test_mode]            |
                              |  simulated:     [false]                      |
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

**Detalhamento do drawer SV:**

- **Cabecalho:** Indicador de estado (icone colorido) + SV ID + Publisher. Botao `[X]` para fechar.
- **Secao Time Window:** Inicio (`start`) e fim (`end`) da captura PCAP. Fonte: `GET /protocols/SV/metrics` → `time_window`.
- **Secao Identificacao:** Dados estaticos do SCD e metricas — sv_id, app_id, dest_mac, vlan_id, vlan_priority, dataset (entries), conf_rev, smp_synch, sampling_rate, nominal_freq, subnet. Fonte: `GET /scds/{scdId}/SV` + metricas.
- **Secao Subscribers:** Tabela com IEDs assinantes e status de recepcao. Fonte: SCD (subscribers[]).
- **Secao Contadores:** Tabela com colunas **eth0 | eth1**. Exibe total_samples, lost_packets (+ts), out_of_order (+ts), duplicates (+ts). Valores > 0 para perdas/erros destacados em vermelho. Fonte: `GET /protocols/SV/metrics` → `stats.{iface}.counters`.
- **Secao Packet Interval:** Tabela com colunas **eth0 | eth1**. Exibe nominal_us, min_us (+ts), max_us (+ts), avg_us, std_us (jitter). O campo `std_us` representa o PDV (Packet Delay Variation) — definicao classica de jitter. Valores extremos de min/max destacados se muito distantes do nominal. Fonte: `stats.{iface}.packet_interval`.
- **Secao Processing Time:** Tabela com colunas **eth0 | eth1**. Exibe min_us, max_us (+ts), avg_us, std_us. **Aviso obrigatorio:** banner ! acima da tabela informando que esta metrica requer sincronizacao PTP com o mesmo Grandmaster das MUs. Sem PTP, os valores representam apenas offset entre clocks. Fonte: `stats.{iface}.processing_time`.
- **Secao Clock Drift:** Tabela com colunas **eth0 | eth1**. Exibe min_us (+ts), max_us (+ts). Mede a variacao relativa do tempo de chegada do smpCnt=0. Primeira observacao e referencia (drift = 0). Fonte: `stats.{iface}.clock_drift`.
- **Secao Qualidade:** Lista de arrays observados — quality_bits[], simulated[]. Se contem valores anormais (ex: `test_mode`, `invalid`), destacar. Fonte: campos raiz da metrica.
- **Secao Redundancia:** Tabela com colunas **eth0 | eth1**. Exibe prp_lans[] e hsr_paths[]. Diagnostico PRP/HSR abaixo da tabela. Fonte: `stats.{iface}.prp_lans`, `stats.{iface}.hsr_paths`.

---

### Drawer de stream SV com alerta (jitter alto)

Quando metricas excedem limiares, um banner de alerta aparece no topo do drawer:

```
                              +----------------------------------------------+
                              | [X]                                          |
                              |                                              |
                              |  ! MU2_3T30101 -- MU2_3T3                    |
                              |  ----------------------------------------    |
                              |                                              |
                              |  +- ! ----------------------------------+    |
                              |  | Jitter (std_us) elevado              |    |
                              |  | eth0: 487.1 us   eth1: 492.3 us     |    |
                              |  | Nominal: 208.33 us                   |    |
                              |  +--------------------------------------+    |
                              |                                              |
                              |  (secoes de identificacao, subscribers,      |
                              |   contadores, packet interval, etc.          |
                              |   identicas ao drawer padrao)                |
                              |                                              |
                              +----------------------------------------------+
```

---

### Drawer sem subscriber / sem metricas

Mesmas variantes do drawer GOOSE (ver `05a-goose.md`):
- **Sem subscriber:** Secao Subscribers exibe "Nenhum subscriber configurado". Sem metricas.
- **Sem metricas:** Secoes de metricas exibem "Metricas indisponiveis. Monitoramento inativo ou dados ainda nao coletados."

---

### Estado sem SCD ativo

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
|   SV     |          |          (icone: onda / sinal amostrado)            |                      |
|   MMS    |          |                                                     |                      |
|   SNMP   |          |      Nenhum SCD ativo.                              |                      |
|   Cfg    |          |                                                     |                      |
|          |          |      Os streams Sampled Values serao exibidos        |                      |
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
| **Tabela de streams SV** | Tabela paginada (pageSize=50) com dados do SCD. Colunas ordenaveis. Clique abre drawer | `GET /scds/{scdId}/SV` |
| **Drawer de detalhe SV** | Painel lateral ~40% largura. 9 secoes estruturadas: Time Window, Identificacao, Subscribers, Contadores (eth0/eth1), Packet Interval (eth0/eth1), Processing Time (eth0/eth1), Clock Drift (eth0/eth1), Qualidade, Redundancia (eth0/eth1) | SCD (estatico) + `GET /protocols/SV/metrics` (runtime) |
| **Tabela por interface (eth0/eth1)** | Padrao reutilizado em 5 secoes (Contadores, Packet Interval, Processing Time, Clock Drift, Redundancia). Colunas por interface para comparacao visual | `GET /protocols/SV/metrics` → `stats` |
| **Banner de alerta (jitter)** | Banner no topo do drawer quando std_us e muito elevado. Icone ! + descricao + valores medidos | Comparacao client-side entre metrica e nominal |
| **Aviso PTP (Processing Time)** | Banner ! fixo acima da tabela Processing Time informando necessidade de sincronizacao PTP | Estatico |
| **Diagnostico PRP/HSR** | Texto abaixo da tabela de redundancia interpretando combinacao de prp_lans/hsr_paths | Logica client-side |
| **Linha muted (sem subscriber)** | Linha com opacidade reduzida e `—` na coluna Subscriber | Dados do SCD |
| **Estado vazio (sem SCD)** | Tela centralizada com icone, mensagem e link para `/settings/scd` | `GET /scds?latest=true` |
| **Paginacao** | Padrao conforme `00-navegacao-global.md` secao 6.1. PageSize=50 | `meta` da resposta |
| **Banner de erro** | Erro inline no topo da area de conteudo | HTTP 4xx/5xx |

---

## Dados e Endpoints

### Dados estaticos (SCD)

| # | Metodo | Endpoint | Uso na tela | Campos utilizados |
|---|---|---|---|---|
| 1 | `GET` | `/api/v1/scds?latest=true` | Verificar se ha SCD ativo | `scdId`, `status` |
| 2 | `GET` | `/api/v1/scds/{scdId}/SV` | Tabela principal: lista de streams com dados estaticos | `data[]` — cada stream contem: `publisher_ied`, `cb_name`, `sv_id`, `dataset`, `dataset_entries`, `dest_mac`, `app_id`, `vlan_id`, `vlan_priority`, `conf_rev`, `asdu_count`, `subscribers[]`, `subnetwork`, `source_file` |

### Metricas runtime (PCAP)

| # | Metodo | Endpoint | Uso na tela | Campos utilizados |
|---|---|---|---|---|
| 3 | `GET` | `/api/v1/protocols/SV/metrics` | Drawer: metricas estruturadas do parser SV | `time_window` (start, end), `interfaces[]`, `invalid_packets`, `metrics[]` — cada metrica contem: `sv_id`, `app_id`, `dest_mac`, `vlan_id[]`, `vlan_priority[]`, `dataset`, `dataset_entries`, `conf_rev[]`, `smp_synch[]`, `sampling_rate`, `nominal_freq`, `quality_bits[]`, `simulated[]`, `interfaces[]`, `stats.{iface}` (counters, packet_interval, processing_time, clock_drift, prp_lans, hsr_paths) |

### Dados persistidos

| # | Metodo | Endpoint | Uso na tela | Campos utilizados |
|---|---|---|---|---|
| 4 | `GET` | `/api/v1/protocols/SV/data` | Alternativa ao endpoint SCD para dados SV persistidos | `items[]` (ProtocolDataRecord) com `data` (object) |

### Estrutura detalhada das metricas SV (por interface)

**Counters (`stats.{iface}.counters`):**

| Campo | Tipo | Descricao |
|---|---|---|
| `total_samples` | int | Total de amostras recebidas |
| `lost_packets` | int | Pacotes perdidos (gaps no smpCnt) |
| `lost_packets_ts` | datetime/null | Timestamp da primeira perda |
| `out_of_order` | int | Pacotes fora de ordem |
| `out_of_order_ts` | datetime/null | Timestamp da primeira ocorrencia |
| `duplicates` | int | Pacotes duplicados (mesmo smpCnt) |
| `duplicates_ts` | datetime/null | Timestamp da primeira duplicata |

**Packet Interval (`stats.{iface}.packet_interval`):**

| Campo | Tipo | Descricao |
|---|---|---|
| `nominal_us` | float | Intervalo nominal esperado (us). 208.33 para 60Hz, 250.00 para 50Hz |
| `min_us` | float | Menor intervalo entre pacotes (us) |
| `min_ts` | datetime | Timestamp do menor intervalo |
| `max_us` | float | Maior intervalo entre pacotes (us) |
| `max_ts` | datetime | Timestamp do maior intervalo |
| `avg_us` | float | Intervalo medio (us) |
| `std_us` | float | Desvio padrao do intervalo (us) — jitter (PDV) |

**Processing Time (`stats.{iface}.processing_time`):**

| Campo | Tipo | Descricao |
|---|---|---|
| `min_us` | float | Menor processing time (us) |
| `max_us` | float | Maior processing time (us) |
| `max_ts` | datetime | Timestamp do maior processing time |
| `avg_us` | float | Processing time medio (us) |
| `std_us` | float | Desvio padrao (us) |

> **IMPORTANTE:** Processing time requer que o NMS esteja sincronizado via PTP com o mesmo Grandmaster das MUs. Sem sincronizacao PTP, os valores representam apenas o offset entre os clocks e nao tem significado pratico.

**Clock Drift (`stats.{iface}.clock_drift`):**

| Campo | Tipo | Descricao |
|---|---|---|
| `min_us` | float | Menor drift observado (us) |
| `min_ts` | datetime | Timestamp do menor drift |
| `max_us` | float | Maior drift observado (us) |
| `max_ts` | datetime | Timestamp do maior drift |

> O Clock Drift mede a variacao relativa do tempo de chegada do `smpCnt=0`. A primeira observacao e usada como referencia (drift = 0). Drift crescente indica clock do observador adiantando; decrescente indica atrasando; saltos bruscos indicam problema de sincronizacao na MU.

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
   GET /scds/{scdId}/SV
   +- Renderizar tabela com dados estaticos (80 streams paginados)
   +- Popular opcoes dos filtros (publishers, subscribers, subnets)

> Etapa 3 - Metricas runtime (sob demanda, para drawer)
   GET /protocols/SV/metrics
   +- Dados carregados quando usuario abre o drawer (clique na linha)
   +- Correlacao por sv_id entre SCD e metricas

> Polling continuo (a cada 15-30 segundos):
   GET /protocols/SV/metrics
   +- Se drawer aberto: atualizar todas as secoes de metricas
```

**Cache no client:**

- Dados do SCD (etapas 1-2) nao mudam apos ativacao — cachear com TTL longo
- Metricas (etapa 3) sao dinamicas — atualizar via polling quando drawer aberto
- Filtros aplicados client-side

---

### Exemplos de JSON (para designs Figma)

Os exemplos abaixo representam dados realistas para popular os componentes visuais.

#### Exemplo: Linha da tabela (dados SCD — 1 stream SV com subscribers)

```json
{
  "publisher_ied": "MU2_3T3",
  "ld_inst": "MU01",
  "cb_name": "MU2_3T30101",
  "sv_id": "MU2_3T3MU01/LLN0$MS$MU2_3T30101",
  "dataset": "MU2_3T3MU01/LLN0$Meas1",
  "dataset_entries": 16,
  "conf_rev": 1,
  "app_id": "0x4000",
  "dest_mac": "01:0c:cd:04:00:99",
  "vlan_id": "0x04B2",
  "vlan_priority": 4,
  "asdu_count": 1,
  "subscribers": ["DUCD_3T3", "DUPC_3B"],
  "subnetwork": "W01",
  "source_file": "MU2_3T3_20260327.CID"
}
```

#### Exemplo: Linha da tabela — sem subscriber (broadcast)

```json
{
  "publisher_ied": "MU1_0P3",
  "ld_inst": "MU02",
  "cb_name": "MU1_0P30201",
  "sv_id": "MU1_0P3MU02/LLN0$MS$MU1_0P30201",
  "dataset": "MU1_0P3MU02/LLN0$Meas1",
  "dataset_entries": 16,
  "conf_rev": 1,
  "app_id": "0x4000",
  "dest_mac": "01:0c:cd:04:01:6f",
  "vlan_id": "0x0460",
  "vlan_priority": 4,
  "asdu_count": 1,
  "subscribers": [],
  "subnetwork": "W01",
  "source_file": "MU1_0P3_20260327.CID"
}
```

#### Exemplo: Drawer completo — metricas SV (1 stream com 2 interfaces, dados normais)

```json
{
  "time_window": {
    "start": "2026-03-27T19:19:00.825923Z",
    "end": "2026-03-27T19:19:13.463593Z"
  },
  "interfaces": ["eth0", "eth1"],
  "invalid_packets": 0,
  "metrics": [
    {
      "sv_id": "MU1_0P30201",
      "app_id": "0x4000",
      "dest_mac": "01:0c:cd:04:01:6f",
      "vlan_id": ["0x0460"],
      "vlan_priority": [4],
      "dataset": "MU1_0P3MU02/LLN0$Meas1",
      "dataset_entries": 16,
      "conf_rev": [1],
      "smp_synch": ["global"],
      "sampling_rate": 4800,
      "nominal_freq": 60,
      "quality_bits": ["good"],
      "simulated": [false],
      "interfaces": ["eth0", "eth1"],
      "stats": {
        "eth0": {
          "prp_lans": ["A"],
          "hsr_paths": [],
          "counters": {
            "total_samples": 60634,
            "lost_packets": 27,
            "lost_packets_ts": "2026-03-27T19:19:02.066777Z",
            "out_of_order": 0,
            "out_of_order_ts": null,
            "duplicates": 0,
            "duplicates_ts": null
          },
          "packet_interval": {
            "nominal_us": 208.33,
            "min_us": 6.91,
            "min_ts": "2026-03-27T19:19:12.244919Z",
            "max_us": 13381.24,
            "max_ts": "2026-03-27T19:19:04.201602Z",
            "avg_us": 208.43,
            "std_us": 133.66
          },
          "processing_time": {
            "min_us": 233936.79,
            "max_us": 247338.29,
            "max_ts": "2026-03-27T19:19:04.201922Z",
            "avg_us": 234082.22,
            "std_us": 476.45
          },
          "clock_drift": {
            "min_us": -95.13,
            "min_ts": "2026-03-27T19:19:09.234006Z",
            "max_us": 748.16,
            "max_ts": "2026-03-27T19:19:04.234849Z"
          }
        },
        "eth1": {
          "prp_lans": ["B"],
          "hsr_paths": [],
          "counters": {
            "total_samples": 60636,
            "lost_packets": 25,
            "lost_packets_ts": "2026-03-27T19:19:02.066801Z",
            "out_of_order": 0,
            "out_of_order_ts": null,
            "duplicates": 0,
            "duplicates_ts": null
          },
          "packet_interval": {
            "nominal_us": 208.33,
            "min_us": 5.82,
            "min_ts": "2026-03-27T19:19:12.244953Z",
            "max_us": 13156.45,
            "max_ts": "2026-03-27T19:19:04.201634Z",
            "avg_us": 208.42,
            "std_us": 135.81
          },
          "processing_time": {
            "min_us": 233980.52,
            "max_us": 247382.06,
            "max_ts": "2026-03-27T19:19:04.201965Z",
            "avg_us": 234125.99,
            "std_us": 476.51
          },
          "clock_drift": {
            "min_us": -99.58,
            "min_ts": "2026-03-27T19:19:09.234040Z",
            "max_us": 743.72,
            "max_ts": "2026-03-27T19:19:04.234844Z"
          }
        }
      }
    }
  ]
}
```

#### Exemplo: Drawer com anomalias (jitter elevado + perdas + quality degradada)

```json
{
  "sv_id": "MU2_3T30101",
  "app_id": "0x4000",
  "dest_mac": "01:0c:cd:04:00:99",
  "vlan_id": ["0x04B2"],
  "vlan_priority": [4],
  "dataset": "MU2_3T3MU01/LLN0$Meas1",
  "dataset_entries": 16,
  "conf_rev": [1],
  "smp_synch": ["local"],
  "sampling_rate": 4800,
  "nominal_freq": 60,
  "quality_bits": ["good", "questionable", "inaccurate"],
  "simulated": [false],
  "interfaces": ["eth0", "eth1"],
  "stats": {
    "eth0": {
      "prp_lans": ["A"],
      "hsr_paths": [],
      "counters": {
        "total_samples": 45230,
        "lost_packets": 142,
        "lost_packets_ts": "2026-03-27T19:19:01.234567Z",
        "out_of_order": 8,
        "out_of_order_ts": "2026-03-27T19:19:03.456789Z",
        "duplicates": 3,
        "duplicates_ts": "2026-03-27T19:19:05.678901Z"
      },
      "packet_interval": {
        "nominal_us": 208.33,
        "min_us": 2.15,
        "min_ts": "2026-03-27T19:19:08.123456Z",
        "max_us": 52847.92,
        "max_ts": "2026-03-27T19:19:01.234567Z",
        "avg_us": 210.87,
        "std_us": 487.12
      },
      "processing_time": {
        "min_us": 234012.45,
        "max_us": 248567.89,
        "max_ts": "2026-03-27T19:19:01.234890Z",
        "avg_us": 234156.78,
        "std_us": 512.34
      },
      "clock_drift": {
        "min_us": -245.67,
        "min_ts": "2026-03-27T19:19:10.234567Z",
        "max_us": 1523.45,
        "max_ts": "2026-03-27T19:19:01.234567Z"
      }
    },
    "eth1": {
      "prp_lans": ["B"],
      "hsr_paths": [],
      "counters": {
        "total_samples": 45228,
        "lost_packets": 144,
        "lost_packets_ts": "2026-03-27T19:19:01.234590Z",
        "out_of_order": 6,
        "out_of_order_ts": "2026-03-27T19:19:03.456812Z",
        "duplicates": 5,
        "duplicates_ts": "2026-03-27T19:19:05.679012Z"
      },
      "packet_interval": {
        "nominal_us": 208.33,
        "min_us": 1.89,
        "min_ts": "2026-03-27T19:19:08.123478Z",
        "max_us": 53012.45,
        "max_ts": "2026-03-27T19:19:01.234590Z",
        "avg_us": 210.92,
        "std_us": 492.34
      },
      "processing_time": {
        "min_us": 234056.12,
        "max_us": 248612.34,
        "max_ts": "2026-03-27T19:19:01.234912Z",
        "avg_us": 234200.45,
        "std_us": 515.67
      },
      "clock_drift": {
        "min_us": -250.12,
        "min_ts": "2026-03-27T19:19:10.234590Z",
        "max_us": 1528.90,
        "max_ts": "2026-03-27T19:19:01.234590Z"
      }
    }
  }
}
```

#### Exemplo: Deteccao automatica de frequencia

**60 Hz (4800 Hz sampling rate):**
```json
{ "sampling_rate": 4800, "nominal_freq": 60, "packet_interval": { "nominal_us": 208.33 } }
```

**50 Hz (4000 Hz sampling rate):**
```json
{ "sampling_rate": 4000, "nominal_freq": 50, "packet_interval": { "nominal_us": 250.00 } }
```

---

## Fluxos de Interacao

### 1. Carregamento da pagina

```
> Usuario acessa /sampled-values via sidebar
> Frontend executa GET /api/v1/scds?latest=true
> Se nao ha SCD ativo:
     > Exibir estado vazio com link < Ir para Configuracao >
     > Fim do fluxo
> Se ha SCD ativo:
     > GET /api/v1/scds/{scdId}/SV
     > Renderizar tabela com 80 streams (pagina 1, 50 registros)
     > Popular filtros com valores extraidos do SCD
```

### 2. Clique em linha da tabela (abrir drawer)

```
> Usuario clica em uma linha (ex: MU1_0P30201 - MU1_0P3)
> Frontend coleta dados estaticos do stream do cache SCD
> Frontend executa GET /api/v1/protocols/SV/metrics
> Correlacionar metrica pelo campo sv_id
> Drawer abre pela direita com animacao slide-in
> Exibe secoes estruturadas:
     - Time Window (start/end da captura)
     - Identificacao (dados SCD + metricas)
     - Subscribers (IEDs)
     - Contadores (tabela eth0/eth1)
     - Packet Interval (tabela eth0/eth1) — inclui jitter (std_us)
     - Processing Time (tabela eth0/eth1) — com aviso PTP
     - Clock Drift (tabela eth0/eth1)
     - Qualidade (arrays observados)
     - Redundancia (tabela eth0/eth1 + diagnostico PRP/HSR)
> Iniciar polling de metricas para manter drawer atualizado
```

### 3. Atualizacao periodica (polling)

```
> A cada 15-30 segundos (apenas se drawer aberto):
     > GET /api/v1/protocols/SV/metrics
> Frontend compara dados novos com dados anteriores
> Se houve mudanca:
     > Atualizar todas as secoes de metricas no drawer
```

### 4. Aplicar filtros na tabela

```
> Usuario seleciona filtro (ex: Publisher = "MU1_0P3")
> Tabela filtra client-side (dados ja carregados do SCD)
> Paginacao recalcula total de resultados
> Combinar multiplos filtros
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
| **Com SCD, sem metricas** | SCD ativo + monitoramento STOPPED ou sem dados | Tabela com dados estaticos do SCD. Drawer: "Metricas indisponiveis" |
| **Sem SCD ativo** | `GET /scds?latest=true` nao retorna SCD ativo | Estado vazio centralizado com link para `/settings/scd` |
| **Carregando** | Requisicoes em andamento | Skeleton/shimmer na tabela |
| **Erro de API** | HTTP 500 ou erro de rede | Banner de erro inline: "Erro ao carregar dados Sampled Values." + [Tentar novamente] |

### Estados por linha da tabela

| Estado | Condicao | Visual |
|---|---|---|
| **Normal** | Stream com subscribers configurados no SCD | * Verde na coluna Status |
| **Sem subscriber** | Lista de subscribers vazia no SCD | o Cinza na coluna Status. Linha muted |

### Estados do drawer

| Estado | Condicao | Visual |
|---|---|---|
| **Aberto com dados completos** | Stream selecionado + metricas disponiveis | Todas as 9 secoes preenchidas |
| **Aberto sem metricas** | Stream selecionado + monitoramento inativo | Secoes estaticas preenchidas. Metricas: "Metricas indisponiveis" |
| **Aberto sem subscriber** | Stream sem subscribers no SCD | Secao Subscribers: "Nenhum subscriber configurado". Sem metricas |
| **Carregando** | Metricas sendo buscadas | Skeleton/shimmer nas secoes de metricas |
| **Com alerta (jitter)** | std_us muito elevado em packet_interval | Banner ! no topo do drawer com valores medidos |
| **Com anomalia (perdas)** | lost_packets > 0, out_of_order > 0 ou duplicates > 0 | Valores destacados em vermelho nas tabelas eth0/eth1 |

---

## Permissoes por Role

| Elemento | ADMIN | OPERATOR | VIEWER |
|---|---|---|---|
| Visualizar tabela SV | ✓ | ✓ | ✓ |
| Aplicar filtros | ✓ | ✓ | ✓ |
| Abrir drawer de detalhe | ✓ | ✓ | ✓ |
| Visualizar metricas runtime | ✓ | ✓ | ✓ |
| Visualizar redundancia PRP/HSR | ✓ | ✓ | ✓ |
| Link « Ir para Configuracao » (estado vazio) | ✓ Visivel e ativo | Link visivel (navega para `/settings/scd`, onde botoes de acao serao desabilitados) | **Oculto** (VIEWER nao configura) |

**Notas:**

- Esta pagina e exclusivamente de **leitura** — nao ha acoes de escrita.
- A unica diferenca por role e a visibilidade do link para Configuracao no estado vazio.

---

## Referencias

- `00-navegacao-global.md` — Layout master, header, sidebar, drawer, convencoes visuais, paginacao e RBAC
- `01-tela-inicial.md` — Topologia de rede (drawer de edge referencia `/sampled-values` para detalhes do stream)
- `05a-goose.md` — Pagina GOOSE (estrutura similar, mesma familia de protocolos)
- `05c-mms.md` — Pagina MMS (mesma familia de protocolos)
- `07-configuracao.md` — Tela de configuracao em `/settings/scd` (destino do link no estado vazio)
- `docs/parsers/sv-parser.md` — Parser SV: estrutura JSON, metricas por interface, packet interval, processing time, clock drift, qualidade, redundancia PRP/HSR
- `docs/examples/example-sv.json` — Exemplo real de saida do parser SV

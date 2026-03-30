# Tela 3 — Sincronismo Temporal (PTP)

## Objetivo

Visualizar o status de sincronizacao temporal PTP (Precision Time Protocol) dos IEDs da subestacao IEC 61850. Esta tela combina **dados estaticos do SCD** (nos logicos LTMS — Logical Node for Time Management Supervision) com **metricas PTP em tempo real** coletadas pelo motor de monitoramento, permitindo que o operador acompanhe a acuracia temporal de cada IED e identifique rapidamente degradacoes ou perdas de sincronismo.

A subestacao monitorada possui 42 IEDs — incluindo DUCD_3T3 (C60), DUPC_3L1 (L90), DUPC_0B (B30), MU1_0P3 (MU), MU2_3T3 (MU), PUPC_3P1 (C60) — todos equipados com o no logico **LTMS1** que expoe os Data Attributes `TmAcc` (acuracia temporal em nanossegundos) e `TmSrc` (fonte de tempo: GPS, PTP, NTP, etc.). A frequencia do sistema e 60 Hz (padrao brasileiro/norte-americano).

---

## Wireframe

### Estado principal (com dados)

```
+-------------------------------------------------------------------------------------------------+
|  +--- Header Global (ver 00-navegacao-global.md) ----------------------------------------+      |
|  |  [Logo NMS]   Subestacao: SE Exemplo   Mon: * RUNNING          operador@empresa.com   |      |
|  +---------------------------------------------------------------------------------------+      |
+----------+--------------------------------------------------------------------------------------+
| SIDEBAR  |                                                                                      |
|          |  +--- Grandmaster Clock ----------------------------------------------------  +      |
|   Topol. |  |                                                                            |      |
|   Alm    |  |  ID: 001e8c-fffe-650da0            Status: * Sincronizado                  |      |
| > Sinc   |  |  Ultima atualizacao: 27/03/2026 14:32:01                                   |      |
|   Red    |  |                                                                            |      |
|   Com    |  +----------------------------------------------------------------------------+      |
|   SNMP   |                                                                                      |
|   Cfg    |  +--- IEDs - Status PTP ----------------------------------------------------- +      |
|          |  |                                                                            |      |
|          |  |  +- DUCD_3T3 ---------------- +     +- DUPC_3L1 ---------------- +         |      |
|          |  |  | * Sincronizado             |     | * Sincronizado             |         |      |
|          |  |  | Acuracia: 120 ns           |     | Acuracia: 95 ns            |         |      |
|          |  |  | Fonte: PTP                 |     | Fonte: PTP                 |         |      |
|          |  |  | GE Vernova . C60           |     | GE Vernova . L90           |         |      |
|          |  |  +----------------------------+     +----------------------------+         |      |
|          |  |                                                                            |      |
|          |  |  +- DUPC_0B ----------------- +     +- MU1_0P3 ----------------- +         |      |
|          |  |  | * Sincronizado             |     | ! Degradado                |         |      |
|          |  |  | Acuracia: 80 ns            |     | Acuracia: 5200 ns          |         |      |
|          |  |  | Fonte: PTP                 |     | Fonte: PTP                 |         |      |
|          |  |  | GE Vernova . B30           |     | GE Vernova . MU            |         |      |
|          |  |  +----------------------------+     +----------------------------+         |      |
|          |  |                                                                            |      |
|          |  |  +- MU2_3T3 ----------------- +     +- PUPC_3P1 ---------------- +         |      |
|          |  |  | o Sem dados                |     | * Sincronizado             |         |      |
|          |  |  | Acuracia: -                |     | Acuracia: 150 ns           |         |      |
|          |  |  | Fonte: -                   |     | Fonte: PTP                 |         |      |
|          |  |  | GE Vernova . MU            |     | GE Vernova . C60           |         |      |
|          |  |  +----------------------------+     +----------------------------+         |      |
|          |  |                                                                            |      |
|          |  |  Nota: Grid 3x2 responsivo. Cada card e clicavel (abre drawer).            |      |
|          |  |  Layout se ajusta a 1 coluna em telas estreitas.                           |      |
|          |  |                                                                            |      |
|          |  +----------------------------------------------------------------------------+      |
|          |                                                                                      |
|          |  +--- Acuracia PTP ao longo do tempo ---------------------------------------- +      |
|          |  |                                                                            |      |
|          |  |  ns                                                                        |      |
|          |  |  5000 +                          /\                                        |      |
|          |  |  4000 +                         /  \                                       |      |
|          |  |  3000 +                        /    \                                      |      |
|          |  |  2000 +         /\            /      \               -- MU1_0P3            |      |
|          |  |  1000 +--------/--\----------/--------\---- -- Limiar (1000 ns)            |      |
|          |  |   500 +-------/----\--------/----------\--                                 |      |
|          |  |     0 +------------------------------------ -- DUCD_3T3                    |      |
|          |  |       +------------------------------------ -- DUPC_3L1                    |      |
|          |  |        10:00   11:00   12:00   13:00   14:00                               |      |
|          |  |                                                                            |      |
|          |  |  +----------------------------------------------------------------------+  |      |
|          |  |  |  {IED v}           [De: ____]    [Ate: ____]    [Limpar]             |  |      |
|          |  |  |  Opcoes:                                                             |  |      |
|          |  |  |  o Todos                                                             |  |      |
|          |  |  |  o DUCD_3T3                                                          |  |      |
|          |  |  |  o DUPC_3L1                                                          |  |      |
|          |  |  |  o DUPC_0B                                                           |  |      |
|          |  |  |  o MU1_0P3                                                           |  |      |
|          |  |  |  o MU2_3T3                                                           |  |      |
|          |  |  |  o PUPC_3P1                                                          |  |      |
|          |  |  +----------------------------------------------------------------------+  |      |
|          |  |                                                                            |      |
|          |  |  Nota: Grafico de linhas com uma serie por IED. Eixo Y = acuracia (ns),    |      |
|          |  |  eixo X = tempo. Linha tracejada horizontal indica o limiar configurado.   |      |
|          |  |  Hover em ponto exibe tooltip com timestamp, IED e valor.                  |      |
|          |  |                                                                            |      |
|          |  +----------------------------------------------------------------------------+      |
|          |                                                                                      |
| [Logout] |                                                                                      |
+----------+--------------------------------------------------------------------------------------+
```

**Detalhamento dos componentes do estado principal:**

- **Card Grandmaster Clock:** Barra informativa no topo da area de conteudo exibindo o identificador do relogio grandmaster PTP, status de sincronizacao e timestamp da ultima atualizacao recebida. O `grandmasterId` vem do campo `grandmasterId` da resposta de metricas PTP.
- **Grid de IED cards (3x2):** Cada card representa um IED com no LTMS. Exibe indicador de estado (icone colorido + texto), acuracia temporal (`TmAcc` em ns), fonte de tempo (`TmSrc`), vendor e tipo do IED. Clique no card abre o drawer de detalhe. O grid se adapta a 1 coluna em telas estreitas.
- **Grafico de serie temporal:** Grafico de linhas mostrando a evolucao da acuracia PTP (eixo Y, em nanossegundos) ao longo do tempo (eixo X). Cada IED e representado por uma serie (cor distinta). Uma linha tracejada horizontal indica o limiar de acuracia aceitavel. Filtros permitem selecionar IED especifico e intervalo de tempo (`fromUtc`/`toUtc`).

---

### Estado do drawer (detalhe do IED)

Abre pela direita quando o usuario clica em um card de IED. Ocupa aproximadamente 40% da largura da area de conteudo. Os cards e o grafico permanecem visiveis ao fundo (com overlay escurecido opcional).

**Exemplo com dados reais do DUCD_3T3:**

```
                              +------------------------------------------+
                              | [X]                                      |
                              |                                          |
                              |  * DUCD_3T3                              |
                              |  ---------------------------------       |
                              |                                          |
                              |  -- Informacoes do IED --                |
                              |                                          |
                              |  Vendor:         GE Vernova              |
                              |  Tipo:           C60                     |
                              |  Firmware:       8.60                    |
                              |                                          |
                              |  -- Sincronismo (LTMS1) --               |
                              |                                          |
                              |  TmAcc (Acuracia):    120 ns             |
                              |  TmSrc (Fonte):       PTP                |
                              |  Status:              * Sincronizado     |
                              |                                          |
                              |  -- Metricas PTP --                      |
                              |                                          |
                              |  +------------------+----------------+   |
                              |  | Chave            | Valor          |   |
                              |  +------------------+----------------+   |
                              |  | offset           | 45 ns          |   |
                              |  | delay            | 230 ns         |   |
                              |  | stepCount        | 0              |   |
                              |  | grandmasterId    | 001e8c-ff...   |   |
                              |  | clockClass       | 6              |   |
                              |  +------------------+----------------+   |
                              |                                          |
                              |  Nota: tabela chave-valor dinamica.      |
                              |  O conteudo varia conforme as metricas   |
                              |  PTP coletadas (additionalProperties).   |
                              |                                          |
                              |  -- Historico de Acuracia --             |
                              |                                          |
                              |  +-----------------------------------+   |
                              |  |  ns                               |   |
                              |  |  200 +  /\    /\                  |   |
                              |  |  150 + /  \--/  \   /\            |   |
                              |  |  100 +/------------\/--\----      |   |
                              |  |   50 +                            |   |
                              |  |    0 +----------------------------|   |
                              |  |      10:00  11:00  12:00  13:00   |   |
                              |  +-----------------------------------+   |
                              |                                          |
                              |  (mini grafico de acuracia mostrando     |
                              |   ultimas 4 horas deste IED)             |
                              |                                          |
                              +------------------------------------------+
```

**Detalhamento do drawer:**

- **Cabecalho:** Indicador de estado (icone colorido) + nome do IED. Botao `[X]` para fechar.
- **Secao Informacoes do IED:** Campos estaticos extraidos do SCD — vendor, tipo, firmware. Fonte: `GET /scds/{scdId}/IEDS`.
- **Secao Sincronismo (LTMS1):** Dados do no logico LTMS do IED. Exibe `TmAcc` (acuracia em nanossegundos), `TmSrc` (fonte de tempo), e status derivado (sincronizado, degradado, sem sincronismo). Fonte: `GET /scds/{scdId}/IEDS` (dados estaticos do LTMS) cruzados com metricas em tempo real.
- **Secao Metricas PTP:** Tabela chave-valor dinamica renderizada a partir do campo `metrics` (additionalProperties) do `MetricPoint`. O conteudo varia conforme as metricas coletadas pelo agente PTP — o frontend nao faz hardcode de chaves especificas. Fonte: `GET /protocols/PTP/metrics`.
- **Secao Historico de Acuracia:** Mini grafico de linhas mostrando a evolucao da acuracia temporal (`TmAcc`) deste IED nas ultimas 4 horas. Escopo reduzido em comparacao ao grafico principal da tela. Fonte: `GET /protocols/PTP/metrics?fromUtc=...&toUtc=...` filtrado por `hostId` do IED.

---

### Estado sem dados

Duas variantes de estado vazio sao possiveis nesta tela:

**Variante 1 — Sem SCD ativo (monitoramento inativo):**

```
+-----------------------------------------------------------------------------------------------+
|  +--- Header Global -------------------------------------------------------------------+      |
|  |  [Logo NMS]   Subestacao: -              Mon: o STOPPED         admin@empresa.com   |      |
|  +-------------------------------------------------------------------------------------+      |
+----------+------------------------------------------------------------------------------------+
| SIDEBAR  |                                                                                    |
|          |                                                                                    |
|   Topol. |                                                                                    |
|   Alm    |                                                                                    |
| > Sinc   |                                                                                    |
|   Red    |                                                                                    |
|   Com    |          +-----------------------------------------------------+                   |
|   SNMP   |          |                                                     |                   |
|   Cfg    |          |              (icone de relogio                      |                   |
|          |          |               com sinal de parado)                  |                   |
|          |          |                                                     |                   |
|          |          |      Monitoramento nao esta ativo.                  |                   |
|          |          |                                                     |                   |
|          |          |      Para visualizar os dados de sincronismo        |                   |
|          |          |      PTP, ative o monitoramento na tela             |                   |
|          |          |      de Configuracao.                               |                   |
|          |          |                                                     |                   |
|          |          |      < Ir para Configuracao >                       |                   |
|          |          |                                                     |                   |
|          |          +-----------------------------------------------------+                   |
|          |                                                                                    |
| [Logout] |                                                                                    |
+----------+------------------------------------------------------------------------------------+
```

**Variante 2 — Monitoramento ativo, mas sem metricas PTP coletadas:**

```
+------------------------------------------------------------------------------------------------+
|  +--- Header Global -------------------------------------------------------------------+       |
|  |  [Logo NMS]   Subestacao: SE Exemplo    Mon: * RUNNING       operador@empresa.com    |       |
|  +-------------------------------------------------------------------------------------+       |
+----------+-------------------------------------------------------------------------------------+
| SIDEBAR  |                                                                                     |
|          |                                                                                     |
|   Topol. |                                                                                     |
|   Alm    |                                                                                     |
| > Sinc   |                                                                                     |
|   Red    |                                                                                     |
|   Com    |          +-----------------------------------------------------+                    |
|   SNMP   |          |                                                     |                    |
|   Cfg    |          |              (icone de relogio                      |                    |
|          |          |               com sinal de espera)                  |                    |
|          |          |                                                     |                    |
|          |          |      Nenhuma metrica PTP coletada ainda.            |                    |
|          |          |                                                     |                    |
|          |          |      O monitoramento esta ativo, mas ainda nao      |                    |
|          |          |      foram recebidos dados PTP dos IEDs. As         |                    |
|          |          |      metricas serao exibidas automaticamente        |                    |
|          |          |      quando estiverem disponiveis.                  |                    |
|          |          |                                                     |                    |
|          |          +-----------------------------------------------------+                    |
|          |                                                                                     |
| [Logout] |                                                                                     |
+----------+-------------------------------------------------------------------------------------+
```

**Detalhamento dos estados vazios:**

- **Variante 1:** Exibida quando nao ha SCD ativo (`GET /scds?latest=true` nao retorna SCD com `status == ACTIVE`) ou quando o monitoramento esta parado (`state == STOPPED`). O link `<< Ir para Configuracao >>` navega para `/configuracao` (Tela 7). A visibilidade do link depende do role — VIEWER nao ve o link (apenas a mensagem informativa).
- **Variante 2:** Exibida quando o monitoramento esta ativo (`state == RUNNING`) mas a chamada `GET /protocols/PTP/metrics` retorna lista vazia (`items == []`). Sem link de acao — a mensagem informa que os dados serao exibidos automaticamente quando disponiveis. Esta e uma situacao transitoria (ex: monitoramento recem-iniciado).

---

## Componentes

| Componente | Descricao | Fonte de Dados |
|---|---|---|
| **Card Grandmaster Clock** | Barra informativa no topo da area de conteudo. Exibe: `grandmasterId` (identificador do relogio mestre PTP), status de sincronizacao (icone colorido + texto) e timestamp da ultima atualizacao. | `GET /protocols/PTP/metrics` → campo `grandmasterId` do `PtpMetricsIngestRequest`; `timestampUtc` do `MetricPoint` mais recente |
| **Grid de IED cards** | Layout de cards 2x2 (responsivo para 1 coluna). Cada card representa um IED com no LTMS1 e exibe: nome, indicador de estado (icone + texto), acuracia (`TmAcc` em ns), fonte de tempo (`TmSrc`), vendor e tipo do IED. Cards sao clicaveis. | `GET /scds/{scdId}/IEDS` (dados estaticos: vendor, tipo, LTMS) + `GET /protocols/PTP/metrics` (dados dinamicos: TmAcc, TmSrc) |
| **Card IED individual** | Card dentro do grid. Exibe cabecalho com nome do IED + indicador de estado, valor de acuracia, fonte de tempo, e rodape com vendor e tipo. Borda lateral com cor do estado. | Mesmo do grid acima |
| **Indicador de estado do IED** | Icone + texto indicando o estado de sincronismo: `● Sincronizado` (verde), `⚠ Degradado` (amarelo), `✕ Sem sincronismo` (vermelho), `○ Sem dados` (cinza) | Derivado de `TmAcc` vs limiar configurado |
| **Grafico de serie temporal** | Grafico de linhas (area principal). Eixo Y: acuracia em nanossegundos. Eixo X: tempo. Uma serie (linha) por IED com cor distinta. Linha tracejada horizontal indicando o limiar de acuracia. Suporte a hover (tooltip com timestamp, IED, valor). | `GET /protocols/PTP/metrics?fromUtc=...&toUtc=...` → `MetricPointListResponse.items[]` |
| **Linha de limiar no grafico** | Linha horizontal tracejada no grafico indicando o limiar de acuracia aceitavel (ex: 1000 ns). Valores acima do limiar indicam degradacao. | Valor configuravel (futura implementacao) ou constante definida no frontend |
| **Filtro de IED no grafico** | Dropdown `{IED v}` acima do grafico para selecionar qual IED exibir (ou todos). Opcoes: "Todos", "DUCD_3T3", "DUPC_3L1", "DUPC_0B", "MU1_0P3", "MU2_3T3", "PUPC_3P1" (e demais IEDs). | Lista de IEDs de `GET /scds/{scdId}/IEDS` |
| **Filtro de periodo no grafico** | Campos `[De: ____]` e `[Ate: ____]` com date-time picker. Mapeiam para `fromUtc`/`toUtc` na chamada da API. Botao `[Limpar]` reseta ao periodo padrao (ultimas 4 horas). | Input do usuario → parametros `fromUtc`/`toUtc` da API |
| **Drawer de detalhe do IED** | Painel lateral direito (~40% largura). Contem secoes: informacoes do IED, sincronismo (LTMS), metricas PTP (tabela chave-valor), historico de acuracia (mini grafico). | Multiplos endpoints (ver secao Dados) |
| **Tabela de metricas PTP no drawer** | Tabela chave-valor dinamica renderizada a partir do campo `metrics` (additionalProperties) do `MetricPoint`. Chaves comuns: offset, delay, stepCount, grandmasterId, clockClass. O frontend nao faz hardcode de chaves. | `GET /protocols/PTP/metrics` → `MetricPoint.metrics` filtrado por IED |
| **Mini grafico de historico no drawer** | Grafico de linhas compacto mostrando a evolucao da acuracia temporal deste IED nas ultimas 4 horas. Sem filtros — escopo fixo. | `GET /protocols/PTP/metrics?fromUtc=...&toUtc=...` filtrado por `hostId` |
| **Estado vazio (monitoramento inativo)** | Tela centralizada quando nao ha SCD ativo ou monitoramento parado. Icone de relogio + mensagem + link `<< Ir para Configuracao >>`. | `GET /scds?latest=true` + `GET /monitoring` |
| **Estado vazio (sem metricas PTP)** | Tela centralizada quando monitoramento ativo mas sem dados PTP coletados. Icone de relogio + mensagem informativa (sem link de acao). | `GET /protocols/PTP/metrics` retorna `items == []` |
| **Banner de erro de API** | Banner vermelho inline no topo da area de conteudo. Exibe mensagem de erro + detalhes tecnicos + botao `[X]` para fechar. | HTTP 5xx ou erro de rede |

---

## Dados e Endpoints

| # | Metodo | Endpoint | Uso na tela | Campos utilizados | Exemplo de chamada |
|---|---|---|---|---|---|
| 1 | `GET` | `/api/v1/scds/{scdId}/IEDS` | Dados estaticos dos IEDs incluindo nos LTMS (vendor, tipo, firmware, logical nodes). Cacheavel — so muda quando o SCD ativo muda. | `data` (object) — contem por IED: `vendor`, `type`, `firmware`, logical nodes (incluindo LTMS1 com `TmAcc`, `TmSrc`) | `GET /api/v1/scds/181b236a/IEDS` → Dados de DUCD_3T3, DUPC_3L1, DUPC_0B, MU1_0P3, MU2_3T3, PUPC_3P1 (e demais 42 IEDs) com seus nos LTMS1 |
| 2 | `GET` | `/api/v1/protocols/PTP/metrics` | Serie temporal de metricas PTP. Utilizado para popular o grafico de acuracia e os valores atuais nos cards. Chamada periodica (polling). | Query: `fromUtc`, `toUtc`, `page`, `pageSize`, `sort`, `order`. Response: `MetricPointListResponse` → `items[]` (cada item: `timestampUtc`, `hostId`, `metrics` com additionalProperties) | `GET /api/v1/protocols/PTP/metrics?fromUtc=2026-03-27T10:00:00Z&toUtc=2026-03-27T14:32:00Z&page=1&pageSize=200&sort=timestampUtc&order=asc` |
| 3 | `GET` | `/api/v1/protocols/PTP/data` | Dados PTP persistidos e historicos. Utilizado para consulta de dados consolidados. | Query: `type`, `scdId`, `page`, `pageSize`, `sort`, `order`. Response: `ScdProtocolDetailsResponse` → `data` (object, additionalProperties) | `GET /api/v1/protocols/PTP/data?scdId=181b236a&page=1&pageSize=50` |

**Schemas relevantes:**

| Schema | Campos | Observacoes |
|---|---|---|
| `PtpMetricsIngestRequest` | `scdId` (uuid), `timestampUtc` (datetime), `timeWindow` ({ startUtc, endUtc }), `grandmasterId` (string), `metrics` (object, additionalProperties:true) | Define a estrutura de ingestao de metricas PTP. O campo `metrics` e dinamico — chaves variam conforme o agente de coleta |
| `MetricPoint` | `timestampUtc` (datetime), `hostId` (string), `metrics` (object, additionalProperties:true) | Ponto individual de metrica. `hostId` identifica o IED. `metrics` contem pares chave-valor dinamicos (ex: offset, delay, stepCount) |
| `MetricPointListResponse` | `items` (MetricPoint[]), `meta` ({ page, pageSize, total }) | Resposta paginada para serie temporal de metricas |
| `ScdProtocolDetailsResponse` | `scdId` (uuid), `status` (ScdStatus), `substationName` (string), `parsedAtUtc` (datetime), `sourceFiles` (string[]), `data` (object, additionalProperties:true) | Dados persistidos de protocolo. Campo `data` e especifico por protocolo (PTP neste caso) |

**Estrategia de carregamento de dados:**

```
> Etapa 1 - Dados essenciais (bloqueante)
   GET /scds?latest=true
   +- Se nao ha SCD ativo > exibir estado vazio variante 1 (fim do fluxo)
   +- Se ha SCD ativo > obter scdId

> Etapa 2 - Dados estaticos do SCD (cacheavel)
   GET /scds/{scdId}/IEDS
   +- Extrair nos LTMS1 de cada IED (TmAcc, TmSrc)
   +- Renderizar skeleton dos cards de IED com dados estaticos (vendor, tipo)

> Etapa 3 - Metricas PTP em tempo real (polling periodico)
   GET /protocols/PTP/metrics?fromUtc={4h atras}&toUtc={agora}
   +- Se items == []: exibir estado vazio variante 2
   +- Se items != []:
      +- Extrair grandmasterId > popular card Grandmaster Clock
      +- Agrupar MetricPoints por hostId > popular cards de IED com TmAcc e TmSrc atuais
      +- Renderizar grafico de serie temporal com dados agrupados por IED

> Polling continuo (a cada 30-60 segundos):
   GET /protocols/PTP/metrics?fromUtc={janela}&toUtc={agora}
   +- Atualizar cards de IED (valores atuais)
   +- Atualizar card Grandmaster Clock
   +- Atualizar grafico de serie temporal (append novos pontos)
   (Nao re-renderizar tudo - apenas atualizar dados dinamicos)
```

**Cache no client:**

- Os dados do SCD (etapa 2) nao mudam apos ativacao — cachear com TTL longo ou ate troca de SCD ativo.
- As metricas PTP (etapa 3) sao dinamicas e devem ser atualizadas via polling periodico (30-60 segundos).
- O grafico mantem em memoria os pontos ja carregados e faz append dos novos, evitando recarga completa.

---

## Fluxos de Interacao

### 1. Carregamento inicial da pagina

```
> Usuario acessa /sincronismo
> Frontend executa GET /api/v1/scds?latest=true
> Se nao ha SCD ativo:
     > Exibir estado vazio variante 1 ("Monitoramento nao esta ativo")
     > Fim do fluxo
> Se ha SCD ativo:
     > GET /api/v1/scds/{scdId}/IEDS
     > Renderizar skeleton dos cards com dados estaticos (nome, vendor, tipo)
     > GET /api/v1/protocols/PTP/metrics?fromUtc={4h atras}&toUtc={agora}&pageSize=200
     > Se items == []:
          > Exibir estado vazio variante 2 ("Nenhuma metrica PTP coletada ainda")
     > Se items != []:
          > Popular card Grandmaster Clock com grandmasterId e ultimo timestamp
          > Popular cards de IED com TmAcc, TmSrc e status derivado
          > Renderizar grafico de serie temporal
          > Iniciar polling periodico
```

### 2. Clique em card de IED (abrir drawer)

```
> Usuario clica em um card de IED (ex: DUCD_3T3)
> Frontend coleta dados estaticos do IED ja carregados (cache etapa 2)
> Frontend filtra MetricPoints por hostId == "DUCD_3T3" (cache etapa 3)
> Drawer abre pela direita com animacao de slide-in
> Exibe:
     - Informacoes do IED: vendor, tipo, firmware
     - Sincronismo (LTMS1): TmAcc, TmSrc, status
     - Metricas PTP: tabela chave-valor dinamica com o MetricPoint mais recente
     - Historico: mini grafico de acuracia (ultimas 4 horas deste IED)
```

### 3. Filtrar grafico por IED

```
> Usuario seleciona um IED no dropdown {IED v} (ex: "MU1_0P3")
> Grafico atualiza para exibir apenas a serie do IED selecionado
> As outras series ficam ocultas (mas os dados permanecem em memoria)
> Se selecionar "Todos": todas as series sao exibidas novamente
```

### 4. Filtrar grafico por periodo

```
> Usuario preenche [De: ____] e [Ate: ____] com date-time picker
> Frontend executa GET /protocols/PTP/metrics?fromUtc={de}&toUtc={ate}&pageSize=200
> Grafico re-renderiza com os dados do novo periodo
> Cards de IED nao sao afetados (sempre mostram o valor mais recente)
```

### 5. Limpar filtros do grafico

```
> Usuario clica [Limpar]
> Dropdown IED volta para "Todos"
> Campos De/Ate sao limpos
> Grafico volta a exibir periodo padrao (ultimas 4 horas) com todos os IEDs
> GET /protocols/PTP/metrics?fromUtc={4h atras}&toUtc={agora}&pageSize=200
```

### 6. Fechar drawer

```
> Usuario clica [X] no drawer ou clica fora do drawer (no overlay)
> Drawer fecha com animacao de slide-out
> Area de conteudo principal volta ao tamanho completo
```

### 7. Atualizacao periodica (polling)

```
> A cada 30-60 segundos:
     > GET /api/v1/protocols/PTP/metrics?fromUtc={janela}&toUtc={agora}
> Frontend compara dados novos com dados anteriores
> Se houve mudanca:
     > Atualizar valores nos cards de IED (TmAcc, TmSrc, status)
     > Atualizar card Grandmaster Clock (ultimo timestamp)
     > Append novos pontos no grafico de serie temporal
     > Se o drawer esta aberto: atualizar secoes de metricas e historico
```

---

## Estados

| Estado | Condicao | Visual |
|---|---|---|
| **Sincronismo normal** | `TmAcc` dentro do limiar de acuracia aceitavel (ex: <= 1000 ns) | `● Sincronizado` — icone verde no card, borda lateral verde |
| **Sincronismo degradado** | `TmAcc` acima do limiar de acuracia mas dentro de margem toleravel (ex: 1000-10000 ns) | `⚠ Degradado` — icone amarelo no card, borda lateral amarela |
| **Sem sincronismo** | `TmAcc` muito alto (ex: > 10000 ns) ou `TmSrc` diferente de PTP (indica perda de fonte PTP) | `✕ Sem sincronismo` — icone vermelho no card, borda lateral vermelha |
| **Sem dados** | Monitoramento ativo mas nenhuma metrica PTP recebida para este IED | `○ Sem dados` — icone cinza no card, campos exibem "—", borda lateral cinza |
| **Sem SCD ativo** | Nenhum SCD com `status == ACTIVE` | Estado vazio global (variante 1): icone de relogio + mensagem + link para Configuracao |
| **Sem metricas PTP** | Monitoramento ativo (`state == RUNNING`) mas `GET /protocols/PTP/metrics` retorna `items == []` | Estado vazio global (variante 2): icone de relogio + mensagem informativa sem link |
| **Grandmaster sincronizado** | Metricas PTP sendo recebidas com `grandmasterId` consistente | `● Sincronizado` — icone verde no card Grandmaster Clock |
| **Grandmaster ausente** | Nenhuma metrica PTP com `grandmasterId` ou ID inconsistente | `⚠ Sem grandmaster` — icone amarelo no card Grandmaster Clock |
| **Carregando** | Requisicoes em andamento (etapas 1-3) | Skeleton/shimmer nos cards e area do grafico |
| **Erro de API** | HTTP 5xx ou erro de rede | Banner de erro vermelho inline no topo da area de conteudo: "Erro ao carregar dados de sincronismo PTP. Tente novamente." + botao `[Tentar novamente]` |
| **Erro de API no drawer** | Falha ao carregar dados do drawer | Mensagem de erro dentro do drawer + botao `[Tentar novamente]` |

**Faixas de acuracia (referencia para o designer):**

| Faixa | Classificacao | Indicador |
|---|---|---|
| 0 - 1000 ns | Normal (sincronizado) | ● Verde |
| 1001 - 10000 ns | Degradado | ⚠ Amarelo |
| > 10000 ns | Sem sincronismo | ✕ Vermelho |
| Sem dado | Sem metricas | ○ Cinza |

**Nota:** Os limiares acima sao valores de referencia para o wireframe. Os valores exatos poderao ser configuraveis em versao futura da aplicacao.

---

## Permissoes por Role

| Elemento | ADMIN | OPERATOR | VIEWER |
|---|---|---|---|
| Visualizar card Grandmaster Clock | ✓ | ✓ | ✓ |
| Visualizar cards PTP dos IEDs | ✓ | ✓ | ✓ |
| Ver detalhe do IED (drawer) | ✓ | ✓ | ✓ |
| Ver metricas PTP no drawer | ✓ | ✓ | ✓ |
| Ver historico de acuracia no drawer | ✓ | ✓ | ✓ |
| Filtrar grafico por IED | ✓ | ✓ | ✓ |
| Filtrar grafico por periodo | ✓ | ✓ | ✓ |
| Link << Ir para Configuracao >> (estado vazio) | ✓ Visivel e ativo | Link visivel (navega para Tela 7, onde botoes de acao serao desabilitados por role) | **Oculto** (VIEWER nao configura — remover o link, manter apenas a mensagem informativa) |

**Notas:**

- Esta tela e exclusivamente de **visualizacao** (somente leitura) — nao ha acoes de escrita (nao ha botoes de start/stop, ACK, configuracao, etc.).
- A unica diferenca por role e a visibilidade do link `<< Ir para Configuracao >>` no estado vazio (variante 1), seguindo o mesmo padrao da Tela 1 (Topologia).
- Todos os filtros (dropdown de IED, periodo do grafico) sao funcionalidades de visualizacao e estao disponiveis para todos os roles.

---

## Referencias

- `00-navegacao-global.md` — Layout master, header, sidebar, drawer, componentes reutilizaveis, convencoes visuais e RBAC
- `01-tela-inicial.md` — IEDs na topologia de rede (os mesmos 42 IEDs aparecem nesta tela com foco em sincronismo PTP)
- `02-alarmes.md` — Alarmes do tipo `PTP_SYNC_LOSS` podem ser gerados quando o sincronismo PTP e perdido
- `07-configuracao.md` — Configuracao do monitoramento (start/stop), upload de SCD, destino do link no estado vazio
- `docs/parsed-scd/scd.md` — Dados reais da subestacao: 42 IEDs com nos LTMS1 (TmAcc, TmSrc)

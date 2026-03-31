# Tela 1 — Topologia de Rede

## Objetivo

Visualizar a topologia de rede da subestacao com overlay de status em tempo real. Acessivel via rota `/topology` na sidebar de navegacao.

A tela apresenta os **IEDs** (Intelligent Electronic Devices) e **proxy nodes** da subestacao organizados pelas **sub-redes** (Station Bus, Process Bus) definidas no SCD ativo, com indicadores visuais de:

- **Tipo** de cada no (mu, ied, other, proxy) — determina forma/icone
- **Estado** de cada no (normal, falha, ausente, inesperado) — determina cor/borda
- Contagem de alarmes ativos por IED (badges)
- Estado global do monitoramento (RUNNING, STOPPED, etc.)
- Conexoes e fluxos de dados entre nos (GOOSE, SV)

**Proxy nodes** sao IEDs referenciados em streams GOOSE/SV (como publishers ou subscribers) mas cujo arquivo CID nao foi carregado no parser. Possuem dados limitados — sem vendor, tipo, IPs — mas sua presenca e conexoes sao conhecidas. O monitoramento parcial e possivel via LGOS/LSVS dos subscribers reais.

**Exemplo do wireframe:** A subestacao SE Exemplo possui 42 IEDs — incluindo DUCD_3T3 (C60, rele de protecao), DUPC_3L1 (L90, rele de linha), DUPC_0B (B30, rele de disjuntor), MU1_0P3 (merging unit), MU2_3T3 (merging unit) e PUPC_3P1 (C60, rele de protecao). O wireframe mostra uma amostra representativa de 8 IEDs; a subestacao real possui 42 IEDs em 9 subnets, alem de 161 proxy nodes, distribuidos em 9 sub-redes (W01, W02, W1, W2, W2_GE, W3, W4, W4_GE, W5).

**Escala real:** Em subestacoes de producao, a topologia pode conter 42 IEDs reais + 161 proxies = 203 nos, distribuidos em 9 sub-redes, com 237 streams GOOSE e 80 streams SV. A interface deve suportar esta escala via filtros, zoom semantico e toggle de proxies.

---

## Wireframe

### Estado principal (com SCD ativo e monitoramento)

```
+--------------------------------------------------------------------------------------------------+
|  +- Header Global (ver 00-navegacao-global.md) ------------------------------------------+       |
|  |  [Logo NMS]   Subestacao: SE Exemplo   Mon: * RUNNING           operador@empresa.com   |      |
|  +----------------------------------------------------------------------------------------+      |
+----------+---------------------------------------------------------------------------------------+
| SIDEBAR  |                                                                                       |
|          |  +- Indicador Global ---------------------------------------------------------+       |
| > Topol. |  |  * Monitoramento: RUNNING desde 27/03/2026 10:00                          |        |
|   Alm (2)|  |  42 IEDs | 161 Proxies | 9 Redes                                         |         |
|   Sinc   |  +------------------------------------------------------------------------+  |        |
|   Red    |                                                                                       |
|   Com    |  +- Filtros/Controles --------------------------------------------------------+       |
|   SNMP   |  | {Subnet: Todas v}  {Tipo: Todos v}  [x] Mostrar proxies   [____Buscar____]|        |
|   Cfg    |  +----------------------------------------------------------------------------+       |
|          |                                                                                       |
|          |  +- Area de Topologia (Grafo de Rede) ----------------------------------------+       |
|          |  |                                                                            |       |
|          |  |  +- Station Bus (W1, W2) -- 8-MMS ----------------------------------+     |        |
|          |  |  |                                                                   |     |       |
|          |  |  |  +-------------------+            +-------------------+            |     |      |
|          |  |  |  | * DUCD_3T3  [ied] |            | * DUPC_3L1  [ied] |            |     |      |
|          |  |  |  | GE Multilin       |            | GE Multilin       |            |     |      |
|          |  |  |  | C60 v8.60         |            | L90 v8.60         |            |     |      |
|          |  |  |  | ! 2 alarmes       |            | 0 alarmes         |            |     |      |
|          |  |  |  +---------+---------+            +---------+---------+            |     |      |
|          |  |  |            |                                |                      |     |      |
|          |  |  |  +-------------------+            +-------------------+            |     |      |
|          |  |  |  | * DUPC_0B   [ied] |            | * PUPC_3P1  [ied] |            |     |      |
|          |  |  |  | GE Multilin       |            | GE Multilin       |            |     |      |
|          |  |  |  | B30 v8.60         |            | C60 v8.60         |            |     |      |
|          |  |  |  | 0 alarmes         |            | 0 alarmes         |            |     |      |
|          |  |  |  +---------+---------+            +---------+---------+            |     |      |
|          |  |  |            |                                |                      |     |      |
|          |  |  +------------+--------------------------------+----------------------+     |      |
|          |  |               |                                |                            |      |
|          |  |  +- Process Bus (W01) -- Layer 2 ------------------------------------------+       |
|          |  |  |            |                                |                            |      |
|          |  |  |            | P1         GOOSE/SV            | P1                         |      |
|          |  |  |  +---------+---------+            +---------+---------+                  |      |
|          |  |  |  | * MU1_0P3    [mu] |            | * MU2_3T3    [mu] |                  |      |
|          |  |  |  | GE                | -- SV -->  | GE                |                  |      |
|          |  |  |  | MU v1.0           | <--GOOSE-- | MU v1.0           |                  |      |
|          |  |  |  | 0 alarmes         |            | 0 alarmes         |                  |      |
|          |  |  |  +---------+---------+            +---------+---------+                  |      |
|          |  |  |            |  -- SV -->  DUCD_3T3, DUPC_3L1                              |      |
|          |  |  |            |  <-- GOOSE --  DUCD_3T3                                     |      |
|          |  |  |                                                                          |      |
|          |  |  |  +-------------------+            +--------------------+                 |      |
|          |  |  |  | * PUCD_3T3  [ied] |            | * UAD_3L1  [other] |                 |      |
|          |  |  |  | GE Multilin       |            | GE Multilin        |                 |      |
|          |  |  |  | C60 v8.60         |            | UAD v1.0           |                 |      |
|          |  |  |  | 0 alarmes         |            | o AUSENTE           |                 |     |
|          |  |  |  +-------------------+            +--------------------+                 |      |
|          |  |  |                                                                          |      |
|          |  |  +--------------------------------------------------------------------------+      |
|          |  |                                                                            |       |
|          |  |  +- Redes Auxiliares ---------------------------------------------------+          |
|          |  |  |                                                                     |           |
|          |  |  |  +- W02 (Backup) --------+      +- W4_GE -----------------+        |            |
|          |  |  |  |  (MU1_0P3)            |      |  (DUCD_3T3, MU2_3T3)    |        |            |
|          |  |  |  |  Ethernet_2           |      |  Process GOOSE          |        |            |
|          |  |  |  +-----------------------+      +--------------------------+        |           |
|          |  |  |                                                                     |           |
|          |  |  +---------------------------------------------------------------------+           |
|          |  |                                                                            |       |
|          |  |  Nota: Este diagrama e CONCEITUAL. O grafo real sera renderizado por        |      |
|          |  |  uma biblioteca de grafos (Canvas/WebGL). Os nos que aparecem em            |      |
|          |  |  multiplas redes sao representados como um unico no do grafo com            |      |
|          |  |  conexoes para cada sub-rede. As zonas (Station Bus, Process Bus, etc.)     |      |
|          |  |  agrupam visualmente os nos conectados a cada sub-rede.                     |      |
|          |  |                                                                            |       |
|          |  |  Proxy nodes (borda tracejada) sao exibidos por padrao com visual muted.   |       |
|          |  |  O toggle "Mostrar proxies" permite oculta-los. Em zoom baixo, proxies     |       |
|          |  |  colapsam primeiro. Na escala real (200+ nos), os filtros por subnet e      |      |
|          |  |  tipo de no sao essenciais para navegacao.                                 |       |
|          |  |                                                                            |       |
|          |  +----------------------------------------------------------------------------+       |
|          |                                                                                       |
|          |  +- Legenda ------------------------------------------------------------------+       |
|          |  |                                                                            |       |
|          |  |  TIPOS DE NO:                                                              |       |
|          |  |  [mu]  Merging Unit (publica SV)     [ied]  IED (assina SV, nao publica)   |       |
|          |  |  [other] Outro (sem SV)              [proxy] Proxy (sem CID carregado)     |       |
|          |  |                                                                            |       |
|          |  |  ESTADOS (cor/borda -- aplicam-se a nos reais mu/ied/other):                |      |
|          |  |  * Normal (verde, borda solida)      o Ausente (cinza, borda tracejada)    |       |
|          |  |  N Falha (vermelho, borda solida)    ! Inesperado (laranja, pontilhada)    |       |
|          |  |  . Proxy (cinza claro, borda tracejada fina, opacidade reduzida)           |       |
|          |  |                                                                            |       |
|          |  |  CONEXOES:                                                                 |       |
|          |  |  --- SV -->  Fluxo Sampled Values    --- GOOSE -->  Fluxo GOOSE            |       |
|          |  |  (clique na conexao abre drawer de edge com detalhes do stream)            |       |
|          |  |                                                                            |       |
|          |  +----------------------------------------------------------------------------+       |
|          |                                                                                       |
| [Logout] |                                                                                       |
+----------+---------------------------------------------------------------------------------------+
```

**Detalhamento da area de topologia:**

- **Zonas de rede**: Cada sub-rede e representada como uma regiao visual (container com borda e label). As zonas agrupam os nos conectados aquela sub-rede.
- **Nos de IED (reais)**: Cada IED real (tipo mu/ied/other) e representado como um card/caixa contendo: indicador de tipo `[mu]`/`[ied]`/`[other]`, nome, vendor, tipo/config version e badge de alarmes. Um IED que aparece em multiplas redes (ex: DUCD_3T3 em W1, W2, W01, W4_GE) e renderizado como um unico no do grafo com arestas para cada zona.
- **Nos de Proxy**: Representados como cards com borda tracejada (`+- - -+`), opacidade reduzida, cinza claro. Exibem apenas nome e label `[proxy]` + "(sem CID)". Sem vendor, tipo, badges de alarme. Posicionados dentro das zonas de rede onde suas edges existem (inferido do campo `subnet` da edge).
- **Conexoes (arestas)**: Linhas entre nos indicando fluxos de dados. Setas indicam direcao (publisher -> subscriber). Rotuladas com o protocolo (SV, GOOSE). **Clicaveis** — abrem o drawer de edge com detalhes do stream.
- **Layout automatico**: O grafo utiliza layout automatico (force-directed ou hierarquico). O designer deve prever que a disposicao exata dos nos pode variar — o importante e a organizacao por zonas e a visibilidade das conexoes.
- **Interacao**: Zoom e pan na area do grafo. Clique em um no de IED abre o drawer de detalhe do IED. Clique em um proxy abre o drawer de proxy. Clique em uma edge abre o drawer de edge.

**Indicador global de monitoramento:**

- Posicionado acima dos filtros, em uma barra informativa.
- Exibe: estado do monitoramento (icone + texto colorido), data/hora de inicio, contagem total de IEDs, proxies e redes.
- Cores do estado conforme `00-navegacao-global.md` secao 3.

**Barra de filtros/controles:**

- Posicionada entre o indicador global e a area de topologia.
- **Filtro Subnet**: Dropdown com todas as sub-redes (`Todas`, `W1`, `W2`, `W4`, etc.). Filtra nos e edges visiveis no grafo.
- **Filtro Tipo**: Dropdown com tipos de no (`Todos`, `mu`, `ied`, `other`, `proxy`). Filtra nos por classificacao.
- **Toggle Proxies**: Checkbox `[x] Mostrar proxies`. Ativado por padrao. Ao desmarcar, oculta todos os nos proxy e suas edges.
- **Busca**: Campo de texto para busca por nome de IED. Destaca o no encontrado no grafo (zoom + highlight).

---

### Drawer de detalhe do IED (no real)

Abre pela direita quando o usuario clica em um no de IED real (tipo mu/ied/other) na topologia. Ocupa aproximadamente 40% da largura da area de conteudo. A topologia permanece visivel ao fundo (com overlay escurecido opcional).

**Exemplo com dados reais do DUCD_3T3:**

```
                              +--------------------------------------------+
                              | [X]                                        |
                              |                                            |
                              |  * DUCD_3T3                         [ied]  |
                              |  -----------------------------------       |
                              |                                            |
                              |  -- Informacoes do IED --                  |
                              |                                            |
                              |  Tipo:           ied                       |
                              |  Vendor:         GE Multilin               |
                              |  Modelo:         C60                       |
                              |  Descricao:      UR                        |
                              |  Config Version: 8.60                      |
                              |  Logical Devices: 3                        |
                              |  Pontos MMS:     42                        |
                              |    (LGOS: 20, LSVS: 12, LTMS: 10)          |
                              |                                            |
                              |  -- Redes / IPs --                         |
                              |                                            |
                              |  +--------+------------------+             |
                              |  | Rede   | IP               |             |
                              |  +--------+------------------+             |
                              |  | W1     | 172.30.184.149   |             |
                              |  | W2     | 172.30.184.149   |             |
                              |  | W01    | (Layer 2)        |             |
                              |  | W4_GE  | (Layer 2)        |             |
                              |  +--------+------------------+             |
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
                              |  (navega para /alarms filtrado por         |
                              |   DUCD_3T3)                                |
                              |                                            |
                              |  -- Monitoramento --                       |
                              |                                            |
                              |  +-----------+----------+----------+       |
                              |  | Protocolo | Target   | Status   |       |
                              |  +-----------+----------+----------+       |
                              |  | SV        | LDTM1    | * running|       |
                              |  | GOOSE     | GoCB01   | * running|       |
                              |  | PTP       | DUCD_3T3 | * running|       |
                              |  +-----------+----------+----------+       |
                              |                                            |
                              +--------------------------------------------+
```

**Detalhamento do drawer de IED:**

- **Cabecalho**: Indicador de estado (icone colorido) + nome do IED + badge de tipo `[ied]`/`[mu]`/`[other]`. Botao `[X]` para fechar.
- **Secao Informacoes**: Campos estaticos extraidos do SCD — tipo (mu/ied/other), vendor, modelo (`ied_type`), descricao (`desc`), config version (`config_version`), contagem de Logical Devices (`logical_device_count`), pontos MMS monitorados (`data_attribute_count`) com breakdown por classe (LGOS, LSVS, LTMS). Fonte: `GET /scds/{scdId}/IEDS`.
- **Secao Redes / IPs**: Tabela com as sub-redes onde o IED esta conectado (`subnets[]`) e seus enderecos IP (`ips[]`). Fonte: `GET /scds/{scdId}/NETWORKS` → nodes.
- **Secao Alarmes Ativos**: Mini-tabela com os alarmes nao reconhecidos deste IED. Exibe severidade (badge colorido) e resumo. Link `« Ver todos os alarmes deste IED »` navega para `/alarms` com filtro pre-aplicado pelo IED. Fonte: `GET /alarms?ack=false` filtrado client-side pelo IED.
- **Secao Monitoramento**: Tabela com os itens de monitoramento ativos para este IED. Exibe protocolo, target e status. Fonte: `GET /monitoring` → campo `monitorings[]` filtrado pelo IED.

---

### Drawer de detalhe do Proxy

Abre pela direita quando o usuario clica em um no de proxy na topologia. Mesmo tamanho e posicao do drawer de IED, mas com conteudo reduzido — proxy nodes nao possuem CID carregado, portanto nao ha vendor, modelo, IPs ou alarmes proprios.

**Exemplo com dados do DUCD_3T1:**

```
                              +--------------------------------------------+
                              | [X]                                        |
                              |                                            |
                              |  . DUCD_3T1                      [proxy]   |
                              |  -----------------------------------       |
                              |                                            |
                              |  -- Informacoes Limitadas --               |
                              |                                            |
                              |  Tipo:           Proxy (sem CID)           |
                              |  Descricao:      GE_Digital_Energy_UR      |
                              |                  URPC_GOOSE_Proxy          |
                              |  Arquivo origem: PUPC_3B.CID               |
                              |                                            |
                              |  Nota: Este no e um proxy -- referenciado  |
                              |  em streams GOOSE/SV mas sem arquivo CID   |
                              |  carregado. Dados detalhados nao           |
                              |  disponiveis.                              |
                              |                                            |
                              |  -- Conexoes (edges) --                    |
                              |                                            |
                              |  +----------+--------+------------------+  |
                              |  | Protocolo| De/Para| Stream           |  |
                              |  +----------+--------+------------------+  |
                              |  | GOOSE    | ->DUCD | TxGOOSE1         |  |
                              |  | GOOSE    | ->MU1  | TxGOOSE3         |  |
                              |  +----------+--------+------------------+  |
                              |                                            |
                              |  -- Monitoramento Parcial --               |
                              |                                            |
                              |  Monitoramento indireto via LGOS/LSVS      |
                              |  dos subscribers reais. Permite validar    |
                              |  confRev e presenca/ausencia do stream.    |
                              |  Sem metricas proprias do publisher.       |
                              |                                            |
                              +--------------------------------------------+
```

**Detalhamento do drawer de proxy:**

- **Cabecalho**: Indicador `·` (muted, cinza claro) + nome do proxy + badge `[proxy]`. Botao `[X]` para fechar.
- **Secao Informacoes Limitadas**: Apenas tipo ("Proxy (sem CID)"), descricao (`desc` do diagnostics.info.ieds_proxy) e arquivo de origem (`source_file`). **Sem** vendor, modelo, IPs, contagem de LDs ou pontos MMS. Nota explicativa sobre a natureza do proxy.
- **Secao Conexoes**: Tabela com as edges onde este proxy participa (como publisher ou subscriber). Exibe protocolo, direcao (-> ou <-) e nome do IED na outra ponta, stream ID. Fonte: `GET /scds/{scdId}/NETWORKS` → edges filtradas pelo proxy.
- **Secao Monitoramento Parcial**: Texto explicativo (nao tabela). Informa que o monitoramento e feito indiretamente via LGOS/LSVS dos subscribers reais. Permite validar confRev e presenca do stream, mas sem metricas do publisher.
- **SEM secao de alarmes** — proxies nao geram alarmes proprios no NMS.

---

### Drawer de detalhe da Edge (stream)

Abre pela direita quando o usuario clica em uma conexao (aresta) entre nos na topologia. Exibe os dados detalhados do stream GOOSE ou SV representado por aquela edge. Metricas de runtime detalhadas ficam nas telas de protocolo (`/goose`, `/sampled-values`).

**Exemplo com dados de um stream GOOSE:**

```
                              +--------------------------------------------+
                              | [X]                                        |
                              |                                            |
                              |  --- GOOSE -->                             |
                              |  DUCD_3T3 -> MU1_0P3                       |
                              |  -----------------------------------       |
                              |                                            |
                              |  -- Informacoes do Stream --               |
                              |                                            |
                              |  Protocolo:      GOOSE                     |
                              |  Stream ID:      TxGOOSE1                  |
                              |  Control Block:  GoCB01                    |
                              |  Publisher:      DUCD_3T3                  |
                              |  Subscribers:    MU1_0P3                   |
                              |                                            |
                              |  -- Parametros de Rede --                  |
                              |                                            |
                              |  Dest MAC:       01:0c:cd:01:00:00         |
                              |  APP ID:         0x0000                    |
                              |  VLAN ID:        0x0837                    |
                              |  Conf Rev:       1                         |
                              |  Subnet:         W4                        |
                              |                                            |
                              |  -- Navegacao --                           |
                              |                                            |
                              |  < Ver detalhes em GOOSE >                 |
                              |  (navega para /goose filtrado por          |
                              |   stream TxGOOSE1)                         |
                              |                                            |
                              +--------------------------------------------+
```

**Detalhamento do drawer de edge:**

- **Cabecalho**: Icone do protocolo (seta GOOSE ou SV) + direcao + nomes dos IEDs (publisher -> subscriber(s)). Botao `[X]` para fechar.
- **Secao Informacoes do Stream**: Protocolo (`protocol`), Stream ID (`stream_id` — GoID para GOOSE ou SvID para SV), Control Block (`cb_name`), Publisher (`from`), Subscribers (`to[]`). Fonte: `GET /scds/{scdId}/NETWORKS` → edges.
- **Secao Parametros de Rede**: Dest MAC (`dest_mac`), APP ID (`app_id`), VLAN ID (`vlan_id`), Conf Rev (`conf_rev`), Subnet (`subnet`). Fonte: mesma edge.
- **Secao Navegacao**: Link para a tela de protocolo correspondente: `< Ver detalhes em GOOSE >` navega para `/goose` filtrado pelo stream, ou `< Ver detalhes em Sampled Values >` navega para `/sampled-values` filtrado pelo stream. Metricas de runtime (contadores, timing, PRP/HSR) ficam nessas telas.

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
|          |         |      de um arquivo SCD em Configuracao.             |                   |
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
- O link `« Ir para Configuracao »` navega para `/settings/scd`, onde o usuario pode fazer upload e ativar um SCD.
- O header exibe `Subestacao: —` (traco) e o monitoramento como `○ STOPPED` quando nao ha SCD ativo.
- A visibilidade do link `« Ir para Configuracao »` depende do role do usuario (ver secao Permissoes).

---

## Componentes

| Componente | Descricao | Fonte de Dados |
|---|---|---|
| **Indicador global de monitoramento** | Barra informativa acima dos filtros exibindo estado do monitoramento (icone + texto colorido), data/hora de inicio (`sinceUtc`), contagem de IEDs, proxies e redes | `GET /monitoring` → `state`, `sinceUtc`; `GET /scds/{scdId}/summary` → `ieds`, `ieds_proxy`, `subnets` |
| **Barra de filtros/controles** | Barra entre indicador global e area de topologia. Contem: filtro por subnet (dropdown), filtro por tipo de no (dropdown), toggle mostrar/ocultar proxies (checkbox), campo de busca por nome | `GET /scds/{scdId}/summary` → `type{}` (para opcoes do dropdown tipo), subnets (para dropdown subnet). Filtros aplicados client-side |
| **Area de topologia (grafo)** | Area principal renderizada por biblioteca de grafos (Canvas/WebGL). Suporta zoom, pan e clique em nos/edges. Organizada por zonas de rede. Zoom semantico para 200+ nos | `GET /scds/{scdId}/NETWORKS` → `nodes`, `edges`, `subnets` |
| **No de IED (real)** | Card dentro do grafo representando um IED real (tipo mu/ied/other). Exibe: badge de tipo `[mu]`/`[ied]`/`[other]`, indicador de estado (cor/borda), nome, vendor, modelo/config version, badge de alarmes ativos | `GET /scds/{scdId}/NETWORKS` → nodes (estatico) + `GET /alarms?ack=false` (dinamico) + `GET /monitoring` (dinamico) |
| **No de Proxy** | Card muted dentro do grafo representando um proxy node. Visual: opacidade reduzida, borda tracejada fina, cinza claro. Exibe apenas nome e badge `[proxy]` + "(sem CID)". Sem vendor, modelo, badges de alarme. Colapsam primeiro em zoom baixo | `GET /scds/{scdId}/NETWORKS` → nodes onde `type == "proxy"` |
| **Zona de rede** | Container visual agrupando nos pertencentes a uma mesma sub-rede. Exibe label com nome e tipo da rede (ex: "Station Bus (W1) — 8-MMS") | `GET /scds/{scdId}/NETWORKS` → `subnets` |
| **Conexoes (arestas)** | Linhas entre nos representando fluxos de dados. Setas indicam direcao (publisher → subscriber). Rotuladas com protocolo (SV, GOOSE). **Clicaveis** — abrem o drawer de edge | `GET /scds/{scdId}/NETWORKS` → `edges` |
| **Legenda** | Barra inferior com tres secoes: tipos de no (mu/ied/other/proxy), estados (normal/falha/ausente/inesperado/proxy), conexoes (SV/GOOSE) | Estatico |
| **Drawer de detalhe do IED** | Painel lateral direito (~40% largura) com informacoes detalhadas do IED real selecionado. Secoes: informacoes (tipo, vendor, modelo, desc, config version, pontos MMS), redes/IPs, alarmes ativos, monitoramento | `GET /scds/{scdId}/IEDS`, `GET /scds/{scdId}/NETWORKS` → nodes, `GET /alarms?ack=false`, `GET /monitoring` |
| **Drawer de detalhe do Proxy** | Painel lateral direito (~40% largura) com informacoes limitadas do proxy selecionado. Secoes: informacoes limitadas (tipo, descricao, arquivo origem), conexoes (edges), monitoramento parcial (texto). Sem alarmes | `GET /scds/{scdId}/NETWORKS` → nodes/edges, `diagnostics.info.ieds_proxy` |
| **Drawer de detalhe da Edge** | Painel lateral direito (~40% largura) com detalhes do stream GOOSE/SV selecionado. Secoes: informacoes do stream (protocol, stream_id, cb_name, from, to[]), parametros de rede (dest_mac, app_id, vlan_id, conf_rev, subnet), link de navegacao para tela de protocolo | `GET /scds/{scdId}/NETWORKS` → edges |
| **Secao Alarmes no drawer IED** | Mini-tabela de alarmes ativos do IED selecionado + link para `/alarms` filtrada | `GET /alarms?ack=false` filtrado por IED |
| **Secao Monitoramento no drawer IED** | Tabela de itens de monitoramento ativos para o IED selecionado (protocolo, target, status) | `GET /monitoring` → `monitorings[]` filtrado por IED |
| **Estado vazio** | Tela centralizada quando nao ha SCD ativo. Icone ilustrativo + mensagem + link para `/settings/scd` | `GET /scds?latest=true` retorna vazio ou sem status ACTIVE |

---

## Dados e Endpoints

| # | Metodo | Endpoint | Uso na tela | Campos utilizados | Exemplo |
|---|---|---|---|---|---|
| 1 | `GET` | `/api/v1/scds?latest=true` | Obter o SCD ativo | `scdId`, `status` | Retorna o SCD mais recente; se `status != ACTIVE`, exibir estado vazio |
| 2 | `GET` | `/api/v1/scds/{scdId}/summary` | Nome da subestacao, contagens, opcoes de filtro | `substationName`, `ieds` (int), `ieds_proxy` (int), `subnets` (int), `type` (object: mu/ied/other/proxy counts), `goose_streams` (int), `sv_streams` (int), `mms_points` (object), `subscriptions` (object), `diagnostics` (object), `vendors` (string[]), `source_files` (int) | `GET /api/v1/scds/181b236a/summary` → `substationName: "SE Exemplo"`, `ieds: 42`, `ieds_proxy: 161`, `subnets: 9`, `type: {mu: 16, ied: 20, other: 6, proxy: 161}` |
| 3 | `GET` | `/api/v1/scds/{scdId}/IEDS` | Detalhes completos de cada IED real (para o drawer de IED) | `data[]` — cada IED contem: `ied_name`, `vendor`, `desc`, `type`, `config_version`, `logical_device_count`, `data_attribute_count`, `logical_nodes` (LGOS[], LSVS[], LTMS[]) | `GET /api/v1/scds/181b236a/IEDS` → `[{ied_name: "DUCD_3T3", vendor: "GE Multilin", type: "C60", config_version: "8.60", logical_device_count: 3, data_attribute_count: 42, ...}]` |
| 4 | `GET` | `/api/v1/scds/{scdId}/NETWORKS` | Topologia de rede (grafo completo com nos, edges, subnets) | `data.nodes[]`, `data.edges[]`, `data.subnets[]` — ver estrutura detalhada abaixo | `GET /api/v1/scds/181b236a/NETWORKS` → 203 nos (42 reais + 161 proxies), 269 edges, 9 subnets |
| 5 | `GET` | `/api/v1/monitoring` | Estado global do monitoramento + lista de itens monitorados | `enabled`, `state` (STOPPED/STARTING/RUNNING/STOPPING/ERROR), `sinceUtc`, `monitorings[]` (id, protocol, target, status, updatedAtUtc) | `GET /api/v1/monitoring` → `state: "RUNNING"`, `sinceUtc: "2026-03-27T13:00:00Z"`, `monitorings: [{protocol: "SV", target: "LDTM1", status: "running"}, ...]` |
| 6 | `GET` | `/api/v1/alarms?ack=false` | Alarmes ativos nao reconhecidos (para badges nos IEDs e secao do drawer) | `items[]` (alarmId, timestampUtc, type, severity, summary, details, occurrences, ack, pcapId) | `GET /api/v1/alarms?ack=false` → `items: [{type: "LOSS_OF_SIGNAL", severity: "CRITICAL", summary: "Perda de sinal SV stream LDTM1"}, ...]` |

### Estrutura detalhada do endpoint NETWORKS

O endpoint `/api/v1/scds/{scdId}/NETWORKS` retorna o grafo completo para visualizacao. A estrutura `data` contem tres arrays:

**Node (no do grafo):**

```json
{
  "id": "DUCD_3T3",
  "type": "ied",
  "vendor": "GE Multilin",
  "ied_type": "C60",
  "ips": ["172.30.184.149"],
  "subnets": ["W01", "W1", "W2", "W4_GE"]
}
```

| Campo | Tipo | Descricao |
|---|---|---|
| `id` | string | Nome do IED (unico) |
| `type` | enum | Classificacao: `mu` (publica SV), `ied` (assina SV, nao publica), `other` (sem SV), `proxy` (sem CID) |
| `vendor` | string | Fabricante (vazio se proxy) |
| `ied_type` | string | Tipo/modelo do IED (vazio se proxy) |
| `ips` | string[] | Enderecos IP (vazio se proxy) |
| `subnets` | string[] | SubNetworks onde o no esta conectado (pode ser vazio para proxies — inferir da edge) |

**Edge (aresta/stream):**

```json
{
  "id": "goose_MU1_0P3_FastGOOSE1",
  "protocol": "goose",
  "from": "MU1_0P3",
  "to": ["DUCD_3T3"],
  "stream_id": "FastGOOSE1",
  "cb_name": "GoCB01",
  "dest_mac": "01:0c:cd:01:00:00",
  "app_id": "0x0000",
  "vlan_id": "0x0837",
  "conf_rev": 1,
  "subnet": "W01"
}
```

| Campo | Tipo | Descricao |
|---|---|---|
| `id` | string | Identificador unico: `{protocol}_{publisher}_{cb_name}` |
| `protocol` | enum | `goose` ou `sv` |
| `from` | string | IED publicador |
| `to` | string[] | Lista de IEDs assinantes |
| `stream_id` | string | GoID (GOOSE) ou SvID (SV) |
| `cb_name` | string | Nome do Control Block |
| `dest_mac` | string | MAC multicast de destino |
| `app_id` | string | APPID em hex |
| `vlan_id` | string | VLAN ID em hex |
| `conf_rev` | integer | Configuration Revision |
| `subnet` | string | SubNetwork onde o stream e publicado |

**Subnet (sub-rede):**

```json
{
  "id": "W1",
  "name": "W1",
  "type": "8-MMS",
  "desc": "",
  "ieds": ["DUCD_3T3", "DUPC_3L1", "DUPC_0B", "PUPC_3P1"]
}
```

| Campo | Tipo | Descricao |
|---|---|---|
| `id` | string | Identificador (igual ao name) |
| `name` | string | Nome da SubNetwork |
| `type` | string | Tipo (8-MMS, Process Bus, etc) |
| `desc` | string | Descricao |
| `ieds` | string[] | Lista de IEDs reais conectados (sem proxies) |

**Schemas relevantes:**

| Schema | Campos | Observacoes |
|---|---|---|
| `ScdSummaryResponse` | `scdId` (uuid), `status` (ScdStatus), `substationName` (string), `parsedAtUtc` (datetime), `sourceFiles` (int), `ieds` (int), `ieds_proxy` (int), `subnets` (int), `goose_streams` (int), `sv_streams` (int), `type` (object), `mms_points` (object), `subscriptions` (object), `diagnostics` (object), `vendors` (string[]) | Resposta leve com metricas agregadas — campos alinhados com output do parser SCD |
| `NetworksResponse` | `nodes` (NetworkNode[]), `edges` (NetworkEdge[]), `subnets` (NetworkSubnet[]) | Grafo completo para visualizacao da topologia |
| `NetworkNode` | `id` (string), `type` (mu/ied/other/proxy), `vendor` (string), `ied_type` (string), `ips` (string[]), `subnets` (string[]) | No do grafo — proxy nodes tem vendor/ied_type/ips vazios |
| `NetworkEdge` | `id` (string), `protocol` (goose/sv), `from` (string), `to` (string[]), `stream_id` (string), `cb_name` (string), `dest_mac` (string), `app_id` (string), `vlan_id` (string), `conf_rev` (int), `subnet` (string) | Aresta do grafo — representa um stream GOOSE ou SV |
| `NetworkSubnet` | `id` (string), `name` (string), `type` (string), `desc` (string), `ieds` (string[]) | Sub-rede — `ieds` contem apenas IEDs reais |
| `IedDetailResponse` | `ied_name` (string), `vendor` (string), `desc` (string), `type` (string), `config_version` (string), `logical_device_count` (int), `data_attribute_count` (int), `logical_nodes` (object: LGOS[], LSVS[], LTMS[]) | Detalhe completo do IED real — usado no drawer |
| `MonitoringStatusResponse` | `enabled` (boolean), `state` (MonitoringState), `sinceUtc` (datetime), `monitorings` (MonitoringItem[]), `stoppedAtUtc` (datetime/null), `stoppedBy` (string/null) | Singleton — estado global |
| `MonitoringItem` | `id` (uuid), `protocol` (ProtocolEnum), `target` (string), `status` (string), `updatedAtUtc` (datetime) | Item individual monitorado |
| `Alarm` | `alarmId` (uuid), `timestampUtc` (datetime), `type` (string), `severity` (LOW/MEDIUM/MAJOR/CRITICAL), `summary` (string), `details` (object), `occurrences` (integer), `ack` (boolean), `pcapId` (uuid/null) | `details` e dinamico (additionalProperties) |

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
   +- Exibir indicador global: contagens de IEDs, proxies, redes
   +- Exibir nome da subestacao no header
   +- Popular opcoes dos filtros (subnets, tipos de no)

> Etapa 3 - Grafo completo (bloqueante)
   GET /scds/{scdId}/NETWORKS
   +- Renderizar grafo: nos (reais + proxies), edges, zonas de rede
   +- Aplicar visual diferenciado para proxies (muted, borda tracejada)
   +- Proxies filtrados client-side pelo toggle "Mostrar proxies"

> Etapa 4 - Detalhes dos IEDs (lazy load, para drawers)
   GET /scds/{scdId}/IEDS
   +- Dados carregados em background para enriquecer drawers de IED
   +- Nao bloqueia renderizacao do grafo

> Etapa 5 - Overlay em tempo real (paralelo, polling periodico)
   GET /monitoring               > aplicar estado de monitoramento nos nos reais
   GET /alarms?ack=false         > aplicar badges de alarme nos nos reais
   Nota: proxies NAO recebem overlay de estado/alarme (apenas dados estaticos do SCD)

> Polling continuo (a cada 15-30 segundos):
   GET /monitoring               > atualizar overlay de monitoramento
   GET /alarms?ack=false         > atualizar badges de alarme
   (Nao re-renderizar a topologia inteira - apenas atualizar os atributos visuais dos nos)
```

**Cache no client:**

- Os dados do SCD (etapas 1-4) nao mudam apos ativacao — podem ser cacheados com TTL longo ou ate invalidacao manual.
- Os dados de monitoramento e alarmes (etapa 5) sao dinamicos e devem ser atualizados via polling periodico.
- Filtros (subnet, tipo, toggle proxies) sao aplicados client-side sobre os dados ja carregados — nao disparam novas requisicoes.

---

## Fluxos de Interacao

### 1. Carregamento da pagina

```
> Usuario acessa /topology via sidebar ou navegacao direta
> Frontend executa GET /api/v1/scds?latest=true
> Se nao ha SCD ativo:
     > Exibir estado vazio com link < Ir para Configuracao >
     > Fim do fluxo
> Se ha SCD ativo:
     > GET /api/v1/scds/{scdId}/summary
     > Exibir indicador global (contagens IEDs, proxies, redes)
     > Exibir substationName no header
     > Popular filtros (dropdown subnet, dropdown tipo)
     > GET /scds/{scdId}/NETWORKS
     > Renderizar grafo: nos reais + proxies (com visual muted), edges, zonas
     > Background: GET /scds/{scdId}/IEDS (para drawers)
     > Paralelo: GET /monitoring + GET /alarms?ack=false
     > Aplicar overlay: estados dos nos reais + badges de alarme
     > Iniciar polling periodico (etapa 5)
```

### 2. Clique em um no de IED real (abrir drawer)

```
> Usuario clica em um no de IED real (tipo mu/ied/other) na topologia (ex: DUCD_3T3)
> Frontend coleta dados do IED ja carregados (cache das etapas 3-4)
> Frontend filtra alarmes ativos para este IED (cache da etapa 5)
> Frontend filtra itens de monitoramento para este IED (cache da etapa 5)
> Drawer abre pela direita com animacao de slide-in
> Exibe:
     - Informacoes: tipo, vendor, modelo, descricao, config version, pontos MMS (LGOS/LSVS/LTMS)
     - Redes / IPs (subnets[] e ips[])
     - Alarmes ativos com severidade e resumo
     - Itens de monitoramento com protocolo, target e status
```

### 2b. Clique em um no de proxy (abrir drawer de proxy)

```
> Usuario clica em um no de proxy na topologia (ex: DUCD_3T1)
> Frontend coleta dados do proxy de NETWORKS (nodes) e diagnostics.info.ieds_proxy
> Frontend filtra edges onde este proxy participa (como from ou to)
> Drawer abre pela direita com animacao de slide-in
> Exibe:
     - Informacoes limitadas: tipo "Proxy (sem CID)", descricao, arquivo origem
     - Nota explicativa sobre natureza do proxy
     - Conexoes: tabela de edges (protocolo, direcao, stream ID)
     - Monitoramento parcial: texto sobre LGOS/LSVS dos subscribers
> Nota: SEM secao de alarmes -- proxies nao geram alarmes proprios
```

### 2c. Clique em uma edge (abrir drawer de edge)

```
> Usuario clica em uma conexao (aresta) entre nos na topologia
> Frontend coleta dados da edge de NETWORKS (edges)
> Drawer abre pela direita com animacao de slide-in
> Exibe:
     - Informacoes do stream: protocolo, stream_id, cb_name, publisher, subscribers
     - Parametros de rede: dest_mac, app_id, vlan_id, conf_rev, subnet
     - Link de navegacao: < Ver detalhes em GOOSE > ou < Ver detalhes em Sampled Values >
       (navega para /goose ou /sampled-values filtrado pelo stream)
> Nota: metricas de runtime (contadores, timing, PRP/HSR) ficam nas telas de protocolo
```

### 3. Navegar para alarmes do IED (a partir do drawer)

```
> Usuario clica < Ver todos os alarmes deste IED > no drawer de IED
> Frontend navega para /alarms com filtro pre-aplicado (rota interna da aplicacao):
     /alarms?ied={iedName}
> Tela de Alarmes abre e filtra client-side pelos alarmes do IED selecionado
  Nota: A API nao possui parametro de filtro por IED. O frontend deve carregar
  os alarmes e filtrar localmente pelo campo `summary` ou `details` que contem
  o nome do IED, ou implementar logica de correlacao propria.
```

### 4. Navegar para Configuracao (a partir do estado vazio)

```
> Usuario clica < Ir para Configuracao > no estado vazio
> Frontend navega para /settings/scd
> Tela de Configuracao exibe opcoes de upload e ativacao de SCD
```

### 5. Atualizacao periodica (polling)

```
> A cada 15-30 segundos:
     > GET /api/v1/monitoring
     > GET /api/v1/alarms?ack=false
> Frontend compara dados novos com dados anteriores
> Se houve mudanca:
     > Atualizar icones de estado nos nos reais (sem re-renderizar o grafo)
     > Atualizar badges de contagem de alarmes nos nos reais
     > Proxies NAO sao afetados pelo polling (sem estado dinamico)
     > Se o drawer de IED esta aberto: atualizar secoes de alarmes e monitoramento
> Se o estado global de monitoramento mudou:
     > Atualizar indicador global no header e na barra informativa
```

### 6. Zoom e navegacao na topologia

```
> Usuario usa scroll/pinch para zoom in/out na area de topologia
> Zoom semantico (4 niveis para suportar 200+ nos):
     - Zoom muito baixo: apenas zonas de rede com labels e contagem de nos
     - Zoom baixo: nos reais visiveis (nome apenas), proxies colapsados/ocultos
     - Zoom medio: todos os nos visiveis, labels resumidos (nome + tipo)
     - Zoom alto: detalhes completos (vendor, config version, badges de alarme)
> Usuario arrasta para pan (mover a area visivel do grafo)
> Double-click em zona de rede: zoom para enquadrar a zona
> Proxies colapsam primeiro -- em zoom baixo, sao os primeiros a serem ocultados
```

### 7. Filtros e controles

```
> Filtro por subnet:
     > Usuario seleciona subnet no dropdown (ex: "W4")
     > Grafo filtra para exibir apenas nos e edges pertencentes a W4
     > Zonas de outras subnets sao ocultadas
     > Selecionar "Todas" restaura a visao completa

> Filtro por tipo de no:
     > Usuario seleciona tipo no dropdown (ex: "mu")
     > Grafo filtra para exibir apenas nos do tipo selecionado e suas edges
     > Selecionar "Todos" restaura a visao completa

> Toggle proxies:
     > Usuario desmarca checkbox "Mostrar proxies"
     > Todos os nos com type=="proxy" sao ocultos, junto com suas edges
     > Checkbox marcado por padrao (proxies visiveis)

> Busca por nome:
     > Usuario digita nome parcial de IED no campo de busca (ex: "DUCD")
     > Grafo destaca o(s) no(s) encontrado(s) com highlight visual
     > Zoom automatico para centralizar o no encontrado
     > Limpar busca restaura visualizacao normal

> Nota: Todos os filtros sao aplicados client-side sobre dados ja carregados.
  Multiplos filtros podem ser combinados (ex: subnet "W4" + tipo "mu" + proxies ocultos).
```

---

## Estados

### Estados dos nos (tipo × estado)

Os nos do grafo possuem duas dimensoes visuais independentes:

**Tipo do no** (determinado pelo campo `type` do parser SCD):

| Tipo | Criterio | Icone/Forma |
|---|---|---|
| **mu** (Merging Unit) | Publica SV (SampledValueControl presente) | Badge `[mu]` |
| **ied** | Assina SV mas nao publica | Badge `[ied]` |
| **other** | Nao publica nem assina SV | Badge `[other]` |
| **proxy** | Referenciado em edges mas CID nao carregado | Badge `[proxy]`, visual muted (ver abaixo) |

**Estado do no** (determinado dinamicamente pelo monitoramento e alarmes — aplica-se apenas a nos reais mu/ied/other):

| Estado | Condicao | Visual |
|---|---|---|
| **Normal** | No real definido no SCD + presente no monitoramento + sem alarmes ativos | ● Verde, borda solida |
| **Falha** | No real com alarmes ativos (`ack=false`) | ✕ Vermelho, borda solida, badge com contagem de alarmes |
| **Ausente** | No real definido no SCD mas nao encontrado nos itens de monitoramento | ○ Cinza, borda tracejada |
| **Inesperado** | Dispositivo presente nos itens de monitoramento mas nao definido no SCD | ⚠ Laranja, borda pontilhada |

**Estado fixo do proxy:**

| Estado | Condicao | Visual |
|---|---|---|
| **Proxy (dados limitados)** | No com `type == "proxy"` | · Cinza claro, borda tracejada fina, opacidade reduzida. Estado fixo — nao muda com monitoramento/alarmes |

**Prioridade de estados para nos reais (quando multiplas condicoes se aplicam):**
1. Falha (maior prioridade — se ha alarmes, exibir como falha independente de outros fatores)
2. Inesperado
3. Ausente
4. Normal (menor prioridade)

Nota: Proxies nao participam da logica de prioridade — possuem estado visual fixo.

### Estados da tela

| Estado | Condicao | Visual |
|---|---|---|
| **Topologia ativa** | SCD ativo + dados carregados | Grafo completo com nos reais, proxies (muted), zonas, conexoes e overlays |
| **Carregando** | Requisicoes em andamento (etapas 1-3) | Skeleton/shimmer na area de topologia |
| **Sem SCD ativo** | `GET /scds?latest=true` nao retorna SCD ativo | Estado vazio com icone, mensagem e link para `/settings/scd` |
| **Erro de API** | Falha na comunicacao com o backend | Banner de erro inline no topo da area de conteudo: "Erro ao carregar topologia. Tente novamente." + botao [Tentar novamente] |
| **Monitoramento parado** | `state == STOPPED` | Indicador global `○ STOPPED` (cinza). Nos reais sem overlay de status (apenas dados estaticos do SCD). Proxies inalterados |
| **Monitoramento em erro** | `state == ERROR` | Indicador global `⚠ ERROR` (vermelho). Banner de alerta na barra informativa |
| **Filtro aplicado** | Um ou mais filtros ativos (subnet, tipo, proxies ocultos) | Grafo exibe subset de nos/edges. Indicador visual de filtro ativo na barra de filtros |

### Estados do drawer de IED

| Estado | Condicao | Visual |
|---|---|---|
| **Aberto com dados** | IED real selecionado, dados carregados | Drawer completo com todas as secoes preenchidas |
| **Sem alarmes** | IED sem alarmes ativos | Secao "Alarmes Ativos" exibe: "Nenhum alarme ativo" (texto cinza). Link para `/alarms` ainda visivel |
| **Sem monitoramento** | IED nao presente nos itens de monitoramento | Secao "Monitoramento" exibe: "IED nao monitorado" (texto cinza, estado Ausente) |
| **Carregando** | Dados ainda sendo processados | Skeleton/shimmer dentro do drawer |

### Estados do drawer de Proxy

| Estado | Condicao | Visual |
|---|---|---|
| **Aberto com dados** | Proxy selecionado | Drawer com informacoes limitadas, conexoes e monitoramento parcial |
| **Sem conexoes** | Proxy sem edges associadas | Secao "Conexoes" exibe: "Nenhuma conexao encontrada" (texto cinza) |

### Estados do drawer de Edge

| Estado | Condicao | Visual |
|---|---|---|
| **Aberto com dados** | Edge selecionada | Drawer com informacoes do stream, parametros de rede e link de navegacao |

---

## Permissoes por Role

| Elemento | ADMIN | OPERATOR | VIEWER |
|---|---|---|---|
| Visualizar topologia | ✓ | ✓ | ✓ |
| Zoom e pan no grafo | ✓ | ✓ | ✓ |
| Filtros (subnet, tipo, toggle proxies, busca) | ✓ | ✓ | ✓ |
| Ver detalhe do IED (drawer) | ✓ | ✓ | ✓ |
| Ver detalhe do Proxy (drawer) | ✓ | ✓ | ✓ |
| Ver detalhe da Edge (drawer) | ✓ | ✓ | ✓ |
| Ver alarmes no drawer de IED | ✓ | ✓ | ✓ |
| Ver monitoramento no drawer de IED | ✓ | ✓ | ✓ |
| Link « Ver todos os alarmes » | ✓ | ✓ | ✓ |
| Link « Ver detalhes em GOOSE/SV » (drawer edge) | ✓ | ✓ | ✓ |
| Link « Ir para Configuracao » (estado vazio) | ✓ Visivel e ativo | Link visivel (navega para `/settings/scd`, onde botoes de acao serao desabilitados) | **Oculto** (VIEWER nao configura — remover o link, manter apenas a mensagem informativa) |

**Notas:**

- Esta tela e exclusivamente de **visualizacao** — nao ha acoes de escrita (nao ha botoes de start/stop, ACK, upload, etc.).
- A unica diferenca por role e a visibilidade do link para Configuracao no estado vazio.
- O botao de start/stop do monitoramento **nao esta presente nesta tela** — ele pertence a `/settings/monitoring`.
- Todos os novos elementos (drawer proxy, drawer edge, filtros, toggle proxies) sao acessiveis por todos os roles.

---

## Referencias

- `00-navegacao-global.md` — Layout master, header, sidebar, drawer, convencoes visuais e RBAC
- `09-dashboard.md` — Tela inicial (Dashboard) — landing page apos login, substitui a topologia como tela inicial
- `02-alarmes.md` — Tela de alarmes (destino do link « Ver todos os alarmes deste IED »)
- `05-comunicacao-dados.md` — Telas de protocolo: `/goose` e `/sampled-values` (destino do link no drawer de edge)
- `07-configuracao.md` — Tela de configuracao em `/settings/scd` (destino do link « Ir para Configuracao » no estado vazio)
- `08-autenticacao.md` — Tela de login (apos autenticacao, redirect vai para Dashboard `/`, nao mais para Topologia)
- `docs/parsers/scd-parser.md` — Parser SCD: estrutura de Networks (nodes, edges, subnets), Proxy Nodes, Summary
- `docs/examples/example-scd.json` — Exemplo real: 42 IEDs, 161 proxies, 9 subnets, 269 edges
- `docs/structure-proposal/frontend-recommendations.md` — Estrategia de renderizacao de topologia (carregamento progressivo, virtualizacao, cache, zoom semantico)

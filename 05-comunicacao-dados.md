# Tela 5 — Comunicacao de Dados (GOOSE, SV, MMS)

## Objetivo

Visualizar os mapas Publisher/Subscriber de GOOSE e Sampled Values (SV), alem do status de conexoes MMS, combinando dados estaticos extraidos do SCD com metricas em tempo real coletadas pelo motor de monitoramento. A tela possui **3 sub-visoes** navegaveis por tabs horizontais:

1. **GOOSE** — Mapa de control blocks GOOSE com publishers, subscribers, datasets e metricas
2. **Sampled Values (SV)** — Mapa de streams SV com jitter, perda de pacotes e duplicatas
3. **MMS** — Conexoes MMS com status e metricas dinamicas

A subestacao monitorada possui 4 IEDs — L90_DIG (rele de protecao de linha), MU320E_LAB (merging unit), MU_360_1 (merging unit) e T60_DIG (rele de protecao de transformador) — com **11 control blocks GOOSE** e **6 streams SV** distribuidos em 6 sub-redes (W1, W2, W4, W01, W02, W03).

---

## Wireframe

### Navegacao entre sub-visoes (tabs)

As tres sub-visoes sao acessadas por tabs horizontais posicionadas no topo da area de conteudo, logo abaixo do header global. Apenas uma tab fica ativa por vez.

```
+----------------------------------------------------------------------------------------------+
|  +--- Header Global (ver 00-navegacao-global.md) -----------------------------------------+  |
|  |  [Logo NMS]   Subestacao: MU_360_1     Mon: * RUNNING                  {user} [Sair]   |  |
|  +----------------------------------------------------------------------------------------+  |
+----------+-----------------------------------------------------------------------------------+
| SIDEBAR  |                                                                                   |
|          |  +--- Tabs ---------------------------------------------------------------+       |
| > Topol. |  |                                                                        |       |
|   Alm (2)|  |   [ GOOSE ]        [ Sampled Values ]        [ MMS ]                   |       |
|   Sinc   |  |   =========                                                            |       |
|   Red    |  |   (ativo)                                                              |       |
| > Com    |  +------------------------------------------------------------------------+       |
|   SNMP   |                                                                                   |
|   Cfg    |  +--- Conteudo da sub-visao ativa ----------------------------------------+       |
|          |  |                                                                        |       |
|          |  |                    (renderiza conforme tab selecionada)                |       |
|          |  |                                                                        |       |
|          |  +------------------------------------------------------------------------+       |
|          |                                                                                   |
| [Logout] |                                                                                   |
+----------+-----------------------------------------------------------------------------------+
```

**Comportamento das tabs:**

- A tab ativa possui sublinhado/borda inferior com cor de destaque e texto em negrito
- As tabs inativas possuem texto normal e sem sublinhado
- Ao clicar em uma tab, o conteudo abaixo troca sem recarregar a pagina
- A tab padrao ao acessar `/comunicacao` e **GOOSE**
- A URL pode refletir a tab ativa via query param (ex: `/comunicacao?tab=sv`)

---

### Sub-visao 5.1 — GOOSE

#### Wireframe — estado principal (com SCD ativo e metricas)

```
+-----------------------------------------------------------------------------------------------+
|                                                                                               |
|  +--- Barra de Filtros ---------------------------------------------------------------- +     |
|  |                                                                                      |     |
|  |  {Publisher v}    {Subscriber v}    {Sub-rede v}    {Status v}          [Limpar]     |     |
|  |                                                                                      |     |
|  |  Opcoes Publisher: Todos, L90_DIG, MU320E_LAB, MU_360_1, T60_DIG                     |     |
|  |  Opcoes Subscriber: Todos, L90_DIG, MU320E_LAB, MU_360_1, T60_DIG, (Sem subscr.)     |     |
|  |  Opcoes Sub-rede: Todas, W01, W1, W2, W4                                             |     |
|  |  Opcoes Status: Todos, Normal, Warning, Erro                                         |     |
|  |                                                                                      |     |
|  +--------------------------------------------------------------------------------------+     |
|                                                                                               |
|  +--- Tabela de Control Blocks GOOSE (11 registros) ------------------------------------+     |
|  |                                                                                      |     |
|  |  Publisher ^ | Control Block | GOOSE ID                         | Dataset | MAC      |     |
|  |  ------------+---------------+----------------------------------+---------+----------|     |
|  |  MU_360_1    | GCB01         | MU_360_1LDSUIED/LLN0$GO$GCB01    | 64 ent. | 01:0c:.  |     |
|  |  MU_360_1    | GCB03         | MU_360_1LDSUIED/LLN0$GO$GCB03    | 18 ent. | 01:0c:.  |     |
|  |  MU_360_1    | GCB04         | MU_360_1LDSUIED/LLN0$GO$GCB04    | 16 ent. | 01:0c:.  |     |
|  |  MU_360_1    | GCB05         | MU_360_1LDSUIED/LLN0$GO$GCB05    |  6 ent. | 01:0c:.  |     |
|  |  L90_DIG     | GoCB01        | TxGOOSE1                         | 36 ent. | 01:0c:.  |     |
|  |  ------------+---------------+----------------------------------+---------+----------|     |
|  |  L90_DIG     | GoCB02        | TxGOOSE2                         |  2 ent. | -        |     |
|  |  MU320E_LAB  | FastGOOSE1    | MU320E_LABGO01                   | 63 ent. | -        |     |
|  |  MU320E_LAB  | GOCB1         | GOID_BER                         |  8 ent. | -        |     |
|  |  MU320E_LAB  | GOCB2         | GOID_Fixed                       |  8 ent. | -        |     |
|  |  T60_DIG     | GoCB01        | T60_DIG_G01                      |  2 ent. | -        |     |
|  |  T60_DIG     | GoCB02        | T60_DIG_G02                      |  2 ent. | -        |     |
|  |                                                                                      |     |
|  |  --- continuacao das colunas (scroll horizontal ou layout responsivo) ---            |     |
|  |                                                                                      |     |
|  |  VLAN     | Sub-rede | Subscriber(s)       | Status                                  |     |
|  |  ---------+----------+---------------------+---------------------------------------- |     |
|  |  0x0000   | W01      | L90_DIG             | * Normal                                |     |
|  |  0x0000   | W01      | L90_DIG             | * Normal                                |     |
|  |  0x0000   | W01      | L90_DIG             | * Normal                                |     |
|  |  0x0000   | W01      | L90_DIG             | * Normal                                |     |
|  |  0x0837   | W4       | MU_360_1            | * Normal                                |     |
|  |  ---------+----------+---------------------+---------------------------------------- |     |
|  |  -        | W2       | -                   | o Sem subscriber                        |     |
|  |  -        | W01      | -                   | o Sem subscriber                        |     |
|  |  -        | W01      | -                   | o Sem subscriber                        |     |
|  |  -        | W01      | -                   | o Sem subscriber                        |     |
|  |  -        | W1       | -                   | o Sem subscriber                        |     |
|  |  -        | W4       | -                   | o Sem subscriber                        |     |
|  |                                                                                      |     |
|  +--------------------------------------------------------------------------------------+     |
|                                                                                               |
|  < Anterior  Pagina 1 de 1  Proxima >    Exibindo 11 de 11                                    |
|                                                                                               |
+-----------------------------------------------------------------------------------------------+
```

**Detalhamento da tabela GOOSE:**

- **Colunas:** Publisher, Control Block, GOOSE ID, Dataset (contagem de entradas), MAC, VLAN, Sub-rede, Subscriber(s), Status
- **Ordenacao padrao:** Publisher (ascendente)
- **Colunas ordenaveis:** Publisher, Control Block, Dataset, Sub-rede
- **Agrupamento visual:** As linhas COM subscribers aparecem primeiro (separadas visualmente das linhas SEM subscriber por um divisor horizontal mais espesso ou fundo alternado)
- **Linhas sem subscriber:** Exibem `—` na coluna Subscriber, MAC e VLAN ficam vazios quando nao definidos no SCD. Linha com opacidade reduzida (muted) para diferenciacao visual
- **Coluna Status:** Combina dados estaticos do SCD com overlay de metricas em tempo real
- **Clique na linha:** Abre o drawer de detalhe do control block (descrito abaixo)
- **Truncamento:** GOOSE ID pode ser longo — truncar com ellipsis e tooltip com valor completo

#### Wireframe — drawer de detalhe do control block GOOSE

Abre pela direita quando o usuario clica em uma linha da tabela. Exemplo com GCB01:

```
                              +------------------------------------------+
                              | [X]                                      |
                              |                                          |
                              |  * GCB01 - MU_360_1                      |
                              |  ---------------------------------       |
                              |                                          |
                              |  -- Informacoes do Control Block --      |
                              |                                          |
                              |  Publisher:     MU_360_1                 |
                              |  Control Block: GCB01                    |
                              |  GOOSE ID:      MU_360_1LDSUIED/         |
                              |                 LLN0$GO$GCB01            |
                              |  MAC:           01:0c:cd:01:00:00        |
                              |  VLAN:          0x0000                   |
                              |  Sub-rede:      W01                      |
                              |                                          |
                              |  -- Dataset (64 entradas) --             |
                              |                                          |
                              |  +----------------------------------+    |
                              |  | Tipo              | Descricao    |    |
                              |  +----------------------------------+    |
                              |  | LPDI Ind1-Ind16   | Indicacoes   |    |
                              |  |                   | digitais     |    |
                              |  | XCBR, XSWI        | Posicoes de  |    |
                              |  |                   | chaves       |    |
                              |  | Alarmes           | Monitoramento|    |
                              |  | Leituras          | Analogicas   |    |
                              |  +----------------------------------+    |
                              |                                          |
                              |  -- Subscriber(s) --                     |
                              |                                          |
                              |  +----------------------------------+    |
                              |  | IED        | Status              |    |
                              |  +----------------------------------+    |
                              |  | L90_DIG    | * Recebendo         |    |
                              |  +----------------------------------+    |
                              |                                          |
                              |  -- Metricas em Tempo Real --            |
                              |  (dados do GooseMetricStream)            |
                              |                                          |
                              |  +----------------------------------+    |
                              |  | Chave             | Valor        |    |
                              |  +----------------------------------+    |
                              |  | controlBlock      | GCB01        |    |
                              |  | appId             | 0x0001       |    |
                              |  | dataset           | (ref.)       |    |
                              |  | LLN0              | {...}        |    |
                              |  | LCCH1             | {...}        |    |
                              |  | (demais campos    |              |    |
                              |  |  dinamicos)       |              |    |
                              |  +----------------------------------+    |
                              |                                          |
                              |  Nota: Os campos de metricas sao         |
                              |  dinamicos (additionalProperties).       |
                              |  Renderizar como tabela chave-valor.     |
                              |                                          |
                              +------------------------------------------+
```

#### Wireframe — drawer de control block SEM subscriber

Exemplo com GoCB02 (L90_DIG, sem subscriber):

```
                              +------------------------------------------+
                              | [X]                                      |
                              |                                          |
                              |  o GoCB02 - L90_DIG                      |
                              |  ---------------------------------       |
                              |                                          |
                              |  -- Informacoes do Control Block --      |
                              |                                          |
                              |  Publisher:     L90_DIG                  |
                              |  Control Block: GoCB02                   |
                              |  GOOSE ID:      TxGOOSE2                 |
                              |  MAC:           -                        |
                              |  VLAN:          -                        |
                              |  Sub-rede:      W2                       |
                              |                                          |
                              |  -- Dataset (2 entradas) --              |
                              |                                          |
                              |  (descricao do dataset conforme SCD)     |
                              |                                          |
                              |  -- Subscriber(s) --                     |
                              |                                          |
                              |  +----------------------------------+    |
                              |  |                                  |    |
                              |  |  Nenhum subscriber configurado   |    |
                              |  |  no SCD para este control block. |    |
                              |  |                                  |    |
                              |  +----------------------------------+    |
                              |                                          |
                              |  -- Metricas em Tempo Real --            |
                              |                                          |
                              |  (sem metricas disponiveis ou            |
                              |   exibir dados se monitoramento          |
                              |   estiver coletando)                     |
                              |                                          |
                              +------------------------------------------+
```

#### Wireframe — drawer para GoCB01 (L90_DIG → MU_360_1)

Exemplo com dados reais do fluxo reverso (rele enviando comandos para MU):

```
                              +------------------------------------------+
                              | [X]                                      |
                              |                                          |
                              |  * GoCB01 - L90_DIG                      |
                              |  ---------------------------------       |
                              |                                          |
                              |  -- Informacoes do Control Block --      |
                              |                                          |
                              |  Publisher:     L90_DIG                  |
                              |  Control Block: GoCB01                   |
                              |  GOOSE ID:      TxGOOSE1                 |
                              |  MAC:           01:0c:cd:01:00:00        |
                              |  VLAN:          0x0837                   |
                              |  Sub-rede:      W4                       |
                              |                                          |
                              |  -- Dataset (36 entradas) --             |
                              |                                          |
                              |  +----------------------------------+    |
                              |  | Tipo              | Descricao    |    |
                              |  +----------------------------------+    |
                              |  | Trip/Close        | Comandos de  |    |
                              |  |                   | disjuntor e  |    |
                              |  |                   | seccionadoras|    |
                              |  | SF6 Pressure      | Pressao SF6  |    |
                              |  | HMI Buttons       | Ativacoes de |    |
                              |  |                   | botoes HMI   |    |
                              |  +----------------------------------+    |
                              |                                          |
                              |  -- Subscriber(s) --                     |
                              |                                          |
                              |  +----------------------------------+    |
                              |  | IED        | Status              |    |
                              |  +----------------------------------+    |
                              |  | MU_360_1   | * Recebendo         |    |
                              |  +----------------------------------+    |
                              |                                          |
                              |  -- Metricas em Tempo Real --            |
                              |                                          |
                              |  +----------------------------------+    |
                              |  | Chave             | Valor        |    |
                              |  +----------------------------------+    |
                              |  | controlBlock      | GoCB01       |    |
                              |  | appId             | (valor)      |    |
                              |  | dataset           | (ref.)       |    |
                              |  | (campos dinamicos |              |    |
                              |  |  do monitoramento)|              |    |
                              |  +----------------------------------+    |
                              |                                          |
                              +------------------------------------------+
```

---

### Sub-visao 5.2 — Sampled Values (SV)

#### Wireframe — estado principal (com SCD ativo e metricas)

```
+-----------------------------------------------------------------------------------------------+
|                                                                                               |
|  +--- Barra de Filtros ---------------------------------------------------------------- +     |
|  |                                                                                      |     |
|  |  {Publisher v}    {Subscriber v}    {Sub-rede v}    {Status v}          [Limpar]     |     |
|  |                                                                                      |     |
|  |  Opcoes Publisher: Todos, MU320E_LAB, MU_360_1                                       |     |
|  |  Opcoes Subscriber: Todos, L90_DIG, T60_DIG, (Sem subscriber)                        |     |
|  |  Opcoes Sub-rede: Todas, W01                                                         |     |
|  |  Opcoes Status: Todos, Normal, Warning (jitter alto), Erro (perda de pacotes)        |     |
|  |                                                                                      |     |
|  +--------------------------------------------------------------------------------------+     |
|                                                                                               |
|  +--- Tabela de Streams SV (6 registros) -----------------------------------------------+     |
|  |                                                                                      |     |
|  |  Publisher ^ | Stream ID                | App ID  | MAC               | VLAN   | Taxa|     |
|  |  ------------+--------------------------+---------+-------------------+--------+-----|     |
|  |  MU_360_1    | LDTM1/F4800S2I4U4_1      | 0x4001  | 01:0c:cd:04:00:01 | 0x044F | 4800|     |
|  |  MU_360_1    | LDTM2/F4800S3I4U4_3      | 0x4003  | 01:0c:cd:04:00:03 | 0x044F | 4800|     |
|  |  ------------+--------------------------+---------+-------------------+--------+-----|     |
|  |  MU320E_LAB  | MU01/MU320E_LAB0101      | 0x4000  | 01:0c:cd:04:00:66 | -      | 4800|     |
|  |  MU320E_LAB  | MU02/MU320E_LAB0201      | 0x4000  | 01:0c:cd:04:01:2e | -      | 4800|     |
|  |  MU_360_1    | LDTM1/F4800S2I4U4_2      | 0x4002  | 01:0c:cd:04:00:02 | -      | 4800|     |
|  |  MU_360_1    | LDTM2/F4800S2I4U4_4      | 0x4004  | 01:0c:cd:04:00:04 | -      | 4800|     |
|  |                                                                                      |     |
|  |  --- continuacao das colunas ---                                                     |     |
|  |                                                                                      |     |
|  |  Sub-rede | Subscriber(s)       | Status          | Metricas (overlay)               |     |
|  |  ---------+---------------------+-----------------+----------------------------------|     |
|  |  W01      | L90_DIG, T60_DIG    | * Normal        | Jitter: 12.3 us  Perda: 0        |     |
|  |  W01      | L90_DIG, T60_DIG    | ! Jitter alto   | Jitter: 487.1 us !  Perda: 0     |     |
|  |  ---------+---------------------+-----------------+----------------------------------|     |
|  |  W01      | -                   | o Sem subscriber| -                                |     |
|  |  W01      | -                   | o Sem subscriber| -                                |     |
|  |  W01      | -                   | o Sem subscriber| -                                |     |
|  |  W01      | -                   | o Sem subscriber| -                                |     |
|  |                                                                                      |     |
|  +--------------------------------------------------------------------------------------+     |
|                                                                                               |
|  < Anterior  Pagina 1 de 1  Proxima >    Exibindo 6 de 6                                      |
|                                                                                               |
|  +--- Legenda de Status ------------------------------------------------------------    +     |
|  |  * Normal: metricas dentro dos limiares                                              |     |
|  |  ! Jitter alto: maxJitterUs acima do limiar configurado                              |     |
|  |  N Perda de pacotes: lostPackets > 0                                                 |     |
|  |  o Sem subscriber: stream sem assinantes configurados no SCD                         |     |
|  +--------------------------------------------------------------------------------------+     |
|                                                                                               |
+-----------------------------------------------------------------------------------------------+
```

**Detalhamento da tabela SV:**

- **Colunas:** Publisher, Stream ID, App ID, MAC, VLAN, Taxa (samples/s), Sub-rede, Subscriber(s), Status, Metricas (overlay)
- **Ordenacao padrao:** Publisher (ascendente)
- **Colunas ordenaveis:** Publisher, Stream ID, App ID, Sub-rede
- **Agrupamento visual:** Streams COM subscribers primeiro, separadas das SEM subscriber por divisor visual
- **Linhas sem subscriber:** Exibem `—` na coluna Subscriber, opacidade reduzida
- **Coluna Metricas (overlay):** Exibe resumo inline das metricas SV em tempo real — jitter medio e perda de pacotes. Quando os valores excedem limiares, a celula ganha destaque visual (fundo amarelo para warning, vermelho para erro)
- **Taxa:** Todas as streams neste SCD sao 4800 samples/s. Valor extraido do SCD
- **Clique na linha:** Abre o drawer com detalhes completos e grafico de metricas

#### Wireframe — drawer de detalhe do stream SV

Exemplo com LDTM1/F4800S2I4U4_1 (MU_360_1, com subscribers):

```
                              +------------------------------------------+
                              | [X]                                      |
                              |                                          |
                              |  * LDTM1/F4800S2I4U4_1 - MU_360_1        |
                              |  ---------------------------------       |
                              |                                          |
                              |  -- Informacoes do Stream --             |
                              |                                          |
                              |  Publisher:     MU_360_1                 |
                              |  Stream ID:     LDTM1/F4800S2I4U4_1      |
                              |  App ID:        0x4001                   |
                              |  MAC:           01:0c:cd:04:00:01        |
                              |  VLAN:          0x044F                   |
                              |  Sub-rede:      W01                      |
                              |  Taxa:          4800 samples/s           |
                              |  Dataset:       16 entradas              |
                              |                                          |
                              |  -- Subscriber(s) --                     |
                              |                                          |
                              |  +----------------------------------+    |
                              |  | IED        | Status              |    |
                              |  +----------------------------------+    |
                              |  | L90_DIG    | * Recebendo         |    |
                              |  | T60_DIG    | * Recebendo         |    |
                              |  +----------------------------------+    |
                              |                                          |
                              |  -- Metricas em Tempo Real --            |
                              |  (dados do SvMetricStream)               |
                              |                                          |
                              |  +----------------------------------+    |
                              |  | Metrica          | Valor         |    |
                              |  +----------------------------------+    |
                              |  | maxJitterUs      | 45.2 us       |    |
                              |  | avgJitterUs      | 12.3 us       |    |
                              |  | stdJitterUs      | 8.7 us        |    |
                              |  | lostPackets      | 0             |    |
                              |  | lostPacketsPct   | 0.00%         |    |
                              |  | duplicates       | 0             |    |
                              |  +----------------------------------+    |
                              |                                          |
                              |  -- Grafico de Jitter (tempo real) --    |
                              |                                          |
                              |  +----------------------------------+    |
                              |  |  us                              |    |
                              |  |  50 +          *                 |    |
                              |  |  40 +    *  *    *               |    |
                              |  |  30 +  *      *    *   *         |    |
                              |  |  20 + *  *          * * *        |    |
                              |  |  10 +*                   **  *   |    |
                              |  |   0 +------------------------    |    |
                              |  |     +------------------------    |    |
                              |  |      -5min            agora      |    |
                              |  |                                  |    |
                              |  |  --- maxJitter  - - avgJitter    |    |
                              |  +----------------------------------+    |
                              |                                          |
                              |  -- Campos Dinamicos --                  |
                              |  (additionalProperties)                  |
                              |                                          |
                              |  +----------------------------------+    |
                              |  | Chave            | Valor         |    |
                              |  +----------------------------------+    |
                              |  | (campos dinamicos|               |    |
                              |  |  variam por      |               |    |
                              |  |  implantacao)    |               |    |
                              |  +----------------------------------+    |
                              |                                          |
                              +------------------------------------------+
```

#### Wireframe — drawer de stream SV com alerta

Exemplo com stream que possui jitter elevado:

```
                              +------------------------------------------+
                              | [X]                                      |
                              |                                          |
                              |  ! LDTM2/F4800S3I4U4_3 - MU_360_1        |
                              |  ---------------------------------       |
                              |                                          |
                              |  +- ! ------------------------------+    |
                              |  | Jitter maximo acima do limiar    |    |
                              |  | maxJitterUs: 487.1 us            |    |
                              |  | Limiar configurado: 250 us       |    |
                              |  +----------------------------------+    |
                              |                                          |
                              |  -- Informacoes do Stream --             |
                              |                                          |
                              |  Publisher:     MU_360_1                 |
                              |  Stream ID:     LDTM2/F4800S3I4U4_3      |
                              |  App ID:        0x4003                   |
                              |  MAC:           01:0c:cd:04:00:03        |
                              |  VLAN:          0x044F                   |
                              |  Sub-rede:      W01                      |
                              |  Taxa:          4800 samples/s           |
                              |  Dataset:       16 entradas              |
                              |                                          |
                              |  -- Subscriber(s) --                     |
                              |                                          |
                              |  +----------------------------------+    |
                              |  | IED        | Status              |    |
                              |  +----------------------------------+    |
                              |  | L90_DIG    | * Recebendo         |    |
                              |  | T60_DIG    | * Recebendo         |    |
                              |  +----------------------------------+    |
                              |                                          |
                              |  -- Metricas em Tempo Real --            |
                              |                                          |
                              |  +----------------------------------+    |
                              |  | Metrica          | Valor         |    |
                              |  +----------------------------------+    |
                              |  | maxJitterUs      | ! 487.1 us    |    |
                              |  | avgJitterUs      | 89.4 us       |    |
                              |  | stdJitterUs      | 112.6 us      |    |
                              |  | lostPackets      | 0             |    |
                              |  | lostPacketsPct   | 0.00%         |    |
                              |  | duplicates       | 2             |    |
                              |  +----------------------------------+    |
                              |                                          |
                              |  -- Grafico de Jitter --                 |
                              |                                          |
                              |  (grafico com linha de limiar em         |
                              |   vermelho tracejado a 250 us)           |
                              |                                          |
                              +------------------------------------------+
```

---

### Sub-visao 5.3 — MMS

#### Wireframe — estado principal

A sub-visao MMS exibe conexoes MMS com um schema extensivel (`MmsMetricsIngestRequest` possui `additionalProperties`). Como o modelo de dados MMS e definido em tempo de implantacao, a interface deve ser generica e renderizar campos dinamicos.

```
+----------------------------------------------------------------------------------------------+
|                                                                                              |
|  +--- Barra de Filtros -------------------------------------------------------------+        |
|  |                                                                                  |        |
|  |  {Conexao v}    {Status v}    [De: ____] [Ate: ____]                    [Limpar] |        |
|  |                                                                                  |        |
|  +----------------------------------------------------------------------------------+        |
|                                                                                              |
|  +--- Tabela de Conexoes MMS -------------------------------------------------------+        |
|  |                                                                                  |        |
|  |  Conexao ^                   | Status          | Ultima Atualizacao              |        |
|  |  ----------------------------+-----------------+---------------------------------|        |
|  |  L90_DIG/MMS_Server          | * Conectado     | 19/02/2026 14:32                |        |
|  |  T60_DIG/MMS_Server          | * Conectado     | 19/02/2026 14:30                |        |
|  |  MU_360_1/MMS_Server         | ! Timeout       | 19/02/2026 14:15                |        |
|  |  MU320E_LAB/MMS_Server       | o Desconectado  | 19/02/2026 09:00                |        |
|  |                                                                                  |        |
|  +----------------------------------------------------------------------------------+        |
|                                                                                              |
|  < Anterior  Pagina 1 de 1  Proxima >    Exibindo 4 de 4                                     |
|                                                                                              |
+----------------------------------------------------------------------------------------------+
```

**Detalhamento da tabela MMS:**

- **Colunas:** Conexao, Status, Ultima Atualizacao
- **Natureza generica:** Como o schema MMS e extensivel (`additionalProperties`), a tabela exibe apenas os campos base. Detalhes adicionais sao mostrados no drawer
- **Clique na linha:** Abre o drawer com metricas dinamicas

#### Wireframe — drawer de detalhe da conexao MMS

```
                              +------------------------------------------+ 
                              | [X]                                      | 
                              |                                          | 
                              |  * L90_DIG - MMS                         | 
                              |  ---------------------------------       | 
                              |                                          | 
                              |  -- Informacoes da Conexao --            | 
                              |                                          | 
                              |  Conexao:        L90_DIG/MMS_Server      | 
                              |  Status:         * Conectado             | 
                              |  Ultima Atualizacao: 19/02/2026 14:32    | 
                              |                                          | 
                              |  -- Metricas (campos dinamicos) --       | 
                              |  (additionalProperties do                | 
                              |   MmsMetricsIngestRequest)               | 
                              |                                          | 
                              |  +-----------------------------------+   | 
                              |  | Chave              | Valor        |   | 
                              |  +-----------------------------------+   | 
                              |  | (campos definidos  |              |   | 
                              |  |  em tempo de       |              |   | 
                              |  |  implantacao -     |              |   | 
                              |  |  renderizar todas  |              |   | 
                              |  |  as chaves do      |              |   | 
                              |  |  objeto de         |              |   | 
                              |  |  metricas como     |              |   | 
                              |  |  tabela chave-     |              |   | 
                              |  |  valor)            |              |   | 
                              |  +-----------------------------------+   | 
                              |                                          | 
                              |  Nota: O modelo de dados MMS e           | 
                              |  extensivel. Os campos acima variam      | 
                              |  conforme a configuracao do agente       | 
                              |  de monitoramento. Renderizar como       | 
                              |  componente de campos dinamicos          | 
                              |  (ver 00-navegacao-global.md, 6.7).      | 
                              |                                          | 
                              +------------------------------------------+ 
```

---

### Estado sem SCD ativo

Exibido quando a chamada `GET /scds?latest=true` nao retorna nenhum SCD com status `ACTIVE`. Substitui toda a area de conteudo (tabs + tabelas) por um estado vazio centralizado.

```
+-----------------------------------------------------------------------------------------------+
|  +--- Header Global (ver 00-navegacao-global.md) -----------------------------------------+   |
|  |  [Logo NMS]   Subestacao: -               Mon: o STOPPED          admin@empresa.com    |   |
|  +----------------------------------------------------------------------------------------+   |
+----------+------------------------------------------------------------------------------------+
| SIDEBAR  |                                                                                    |
|          |                                                                                    |
| > Topol. |                                                                                    |
|   Alm    |                                                                                    |
|   Sinc   |                                                                                    |
|   Red    |                                                                                    |
| > Com    |          +-----------------------------------------------------+                   |
|   SNMP   |          |                                                     |                   |
|   Cfg    |          |          (icone: setas bidirecionais /              |                   |
|          |          |           comunicacao de dados)                     |                   |
|          |          |                                                     |                   |
|          |          |      Nenhum SCD ativo.                              |                   |
|          |          |                                                     |                   |
|          |          |      Os mapas de comunicacao GOOSE, SV e MMS        |                   |
|          |          |      serao exibidos apos a ativacao de um           |                   |
|          |          |      arquivo SCD na tela de Configuracao.           |                   |
|          |          |                                                     |                   |
|          |          |      < Ir para Configuracao >                       |                   |
|          |          |                                                     |                   |
|          |          +-----------------------------------------------------+                   |
|          |                                                                                    |
|          |                                                                                    |
| [Logout] |                                                                                    |
+----------+------------------------------------------------------------------------------------+
```

**Detalhamento:**

- O icone ilustrativo deve representar comunicacao de dados (setas bidirecionais, fluxos de rede)
- A mensagem explica que o SCD e necessario para gerar os mapas de comunicacao
- O link `« Ir para Configuracao »` navega para `/configuracao` (Tela 7)
- O header exibe `Subestacao: —` (traco) e o monitoramento como `○ STOPPED`
- A visibilidade do link `« Ir para Configuracao »` depende do role (ver secao Permissoes)

---

## Componentes

| Componente | Descricao | Fonte de Dados |
|---|---|---|
| **Tabs de navegacao** | 3 tabs horizontais: GOOSE, Sampled Values, MMS. Tab ativa com sublinhado e texto em negrito | Estado local (client-side) |
| **Barra de filtros (GOOSE)** | Dropdowns: Publisher, Subscriber, Sub-rede, Status. Botao Limpar | Valores extraidos do SCD (IEDs, sub-redes) + estados de metricas |
| **Barra de filtros (SV)** | Dropdowns: Publisher, Subscriber, Sub-rede, Status. Botao Limpar | Valores extraidos do SCD + estados de metricas |
| **Barra de filtros (MMS)** | Dropdown Conexao, Status. Campos De/Ate para periodo temporal. Botao Limpar | Dados de metricas MMS |
| **Tabela de control blocks GOOSE** | Tabela com 11 linhas (dados do SCD), colunas ordenaveis, clique abre drawer | `GET /api/v1/scds/{scdId}/GOOSE` + `GET /api/v1/protocols/GOOSE/metrics` |
| **Tabela de streams SV** | Tabela com 6 linhas (dados do SCD), colunas ordenaveis, coluna de metricas inline, clique abre drawer | `GET /api/v1/scds/{scdId}/SV` + `GET /api/v1/protocols/SV/metrics` |
| **Tabela de conexoes MMS** | Tabela generica com conexoes MMS, colunas ordenaveis, clique abre drawer | `GET /api/v1/protocols/MMS/metrics` + `GET /api/v1/protocols/MMS/data` |
| **Overlay de metricas (GOOSE)** | Tabela chave-valor dinamica dentro do drawer, renderizando campos de `GooseMetricStream` incluindo `additionalProperties` | `GET /api/v1/protocols/GOOSE/metrics` |
| **Overlay de metricas (SV)** | Metricas inline na tabela (resumo) e no drawer (detalhado): maxJitterUs, avgJitterUs, stdJitterUs, lostPackets, lostPacketsPct, duplicates | `GET /api/v1/protocols/SV/metrics` |
| **Overlay de metricas (MMS)** | Tabela chave-valor dinamica no drawer, campos variam por implantacao | `GET /api/v1/protocols/MMS/metrics` |
| **Grafico de jitter (SV)** | Grafico de linha temporal dentro do drawer de stream SV. Eixo X: tempo (ultimos 5 minutos). Eixo Y: microsegundos. Linhas para maxJitter e avgJitter. Linha tracejada vermelha para limiar | `GET /api/v1/protocols/SV/metrics` (serie temporal) |
| **Indicador de alerta (SV)** | Banner no topo do drawer quando metricas excedem limiares. Icone ⚠ + descricao do alerta + valor medido vs. limiar | Comparacao client-side entre metrica e limiar configurado |
| **Drawer de detalhe (GOOSE)** | Painel lateral ~40% largura. Secoes: info do CB, dataset, subscribers, metricas dinamicas | SCD (estatico) + metricas (tempo real) |
| **Drawer de detalhe (SV)** | Painel lateral ~40% largura. Secoes: info do stream, subscribers, metricas, grafico de jitter, campos dinamicos | SCD (estatico) + metricas (tempo real) |
| **Drawer de detalhe (MMS)** | Painel lateral ~40% largura. Secoes: info da conexao, metricas dinamicas (chave-valor) | Metricas MMS |
| **Linha com destaque de alerta** | Linha da tabela com fundo amarelo/vermelho sutil quando metricas indicam anomalia (jitter alto, perda de pacotes) | Comparacao client-side |
| **Linha muted (sem subscriber)** | Linha com opacidade reduzida e `—` na coluna Subscriber para control blocks/streams sem assinantes | Dados do SCD |
| **Estado vazio (sem SCD)** | Tela centralizada com icone, mensagem e link para Configuracao | `GET /scds?latest=true` retorna vazio ou sem ACTIVE |
| **Paginacao** | Padrao conforme `00-navegacao-global.md` secao 6.1 | `meta` de cada resposta de API |
| **Banner de erro** | Erro inline no topo da area de conteudo | HTTP 4xx/5xx ou erro de rede |

---

## Dados e Endpoints

### Dados estaticos (SCD)

| # | Metodo | Endpoint | Uso na tela | Campos utilizados |
|---|---|---|---|---|
| 1 | `GET` | `/api/v1/scds?latest=true` | Verificar se ha SCD ativo | `scdId`, `status` |
| 2 | `GET` | `/api/v1/scds/{scdId}/GOOSE` | Mapa completo de GOOSE: control blocks, datasets, subscribers | `data` (object) — contem lista de 11 CBs com publisher, cbName, gooseId, datasetEntries, mac, vlan, subnet, subscribers |
| 3 | `GET` | `/api/v1/scds/{scdId}/SV` | Mapa completo de SV: streams, subscribers | `data` (object) — contem lista de 6 streams com publisher, streamId, appId, mac, vlan, sampleRate, subnet, subscribers |

**Schema:** `ScdProtocolDetailsResponse`

```
{
  scdId: uuid,
  status: ScdStatus,
  substationName: string,
  parsedAtUtc: datetime,
  sourceFiles: string[],
  data: object    // conteudo varia por ScdViewEnum (GOOSE, SV, IEDS, NETWORKS, VALIDATIONS)
}
```

### Metricas em tempo real

| # | Metodo | Endpoint | Uso na tela | Parametros | Schema de resposta |
|---|---|---|---|---|---|
| 4 | `GET` | `/api/v1/protocols/GOOSE/metrics` | Overlay de metricas GOOSE na tabela e drawer | `fromUtc`, `toUtc`, `page`, `pageSize` | `MetricPointListResponse` { items: MetricPoint[], meta: PageMeta } |
| 5 | `GET` | `/api/v1/protocols/SV/metrics` | Overlay de metricas SV: jitter, perda de pacotes, duplicatas | `fromUtc`, `toUtc`, `page`, `pageSize` | `MetricPointListResponse` |
| 6 | `GET` | `/api/v1/protocols/MMS/metrics` | Metricas de conexoes MMS | `fromUtc`, `toUtc`, `page`, `pageSize` | `MetricPointListResponse` |

### Dados persistidos

| # | Metodo | Endpoint | Uso na tela | Parametros | Schema de resposta |
|---|---|---|---|---|---|
| 7 | `GET` | `/api/v1/protocols/GOOSE/data` | Dados GOOSE persistidos (historico) | `fromUtc`, `toUtc`, `page`, `pageSize` | `ProtocolDataListResponse` { items: ProtocolDataRecord[], meta: PageMeta } |
| 8 | `GET` | `/api/v1/protocols/SV/data` | Dados SV persistidos (historico) | `fromUtc`, `toUtc`, `page`, `pageSize` | `ProtocolDataListResponse` |
| 9 | `GET` | `/api/v1/protocols/MMS/data` | Dados MMS persistidos (historico) | `fromUtc`, `toUtc`, `page`, `pageSize` | `ProtocolDataListResponse` |

**Schemas de metricas:**

| Schema | Campos | Observacoes |
|---|---|---|
| `GooseMetricStream` | `controlBlock` (string, required), `appId` (string, required), `dataset` (string, required), + `additionalProperties` (nos IEC 61850: LLN0, LCCH1, etc.) | Campos dinamicos — renderizar como tabela chave-valor |
| `SvMetricStream` | `svId` (string, required), `appId` (string, required), `destMac` (string, required), `vlanId` (string), `duplicates` (integer), `maxJitterUs` (number), `avgJitterUs` (number), `stdJitterUs` (number), `lostPackets` (integer), `lostPacketsPct` (number), + `additionalProperties` | Metricas fixas de qualidade + campos dinamicos |
| `MmsMetricsIngestRequest` | Extensivel — schema definido em tempo de implantacao (`additionalProperties`) | Renderizar integralmente como campos dinamicos |
| `ProtocolDataRecord` | `id` (uuid), `protocol` (string), `type` (string), `scdId` (uuid), `createdAtUtc` (datetime), `data` (object) | Registro generico de dado persistido |
| `MetricPointListResponse` | `items` (MetricPoint[]), `meta` (PageMeta) | Listagem paginada |
| `ProtocolDataListResponse` | `items` (ProtocolDataRecord[]), `meta` (PageMeta) | Listagem paginada |

### Estrategia de carregamento de dados

```
> Etapa 1 - Verificacao de SCD (bloqueante)
   GET /scds?latest=true
   +- Se nao ha SCD ativo > exibir estado vazio (fim do fluxo)
   +- Se ha SCD ativo > obter scdId

> Etapa 2 - Mapa estatico do protocolo ativo (bloqueante)
   GET /scds/{scdId}/GOOSE   (se tab GOOSE ativa)
   GET /scds/{scdId}/SV      (se tab SV ativa)
   +- Renderizar tabela com dados estaticos do SCD

> Etapa 3 - Overlay de metricas (paralelo, nao-bloqueante)
   GET /protocols/GOOSE/metrics   (se tab GOOSE)
   GET /protocols/SV/metrics      (se tab SV)
   GET /protocols/MMS/metrics     (se tab MMS)
   +- Aplicar metricas sobre os dados estaticos
   +- Destacar linhas com anomalias

> Polling continuo (a cada 15-30 segundos):
   GET /protocols/{protocolo}/metrics
   +- Atualizar overlay de metricas sem re-renderizar tabela inteira
   +- Atualizar grafico de jitter no drawer (se aberto)
```

**Cache no client:**

- Os dados do SCD (etapas 1-2) nao mudam apos ativacao — cachear com TTL longo
- Os dados de metricas (etapa 3) sao dinamicos e atualizados via polling periodico
- Ao trocar de tab, carregar mapa SCD do novo protocolo (se ainda nao cacheado) + iniciar polling de metricas do protocolo correspondente

---

## Fluxos de Interacao

### 1. Carregamento da pagina

```
> Usuario acessa /comunicacao
> Frontend executa GET /api/v1/scds?latest=true
> Se nao ha SCD ativo:
     > Exibir estado vazio com link < Ir para Configuracao >
     > Fim do fluxo
> Se ha SCD ativo:
     > Exibir tabs (GOOSE ativo por padrao)
     > GET /api/v1/scds/{scdId}/GOOSE
     > Renderizar tabela de control blocks com dados do SCD (11 linhas)
     > Paralelo: GET /api/v1/protocols/GOOSE/metrics
     > Aplicar overlay de metricas nas linhas da tabela
     > Iniciar polling periodico de metricas (a cada 15-30 segundos)
```

### 2. Troca de tab (GOOSE → SV → MMS)

```
> Usuario clica na tab "Sampled Values"
> Tab GOOSE desativa, tab SV ativa (visual: sublinhado muda)
> Parar polling de metricas GOOSE
> Se dados SV do SCD ja cacheados:
     > Renderizar tabela SV imediatamente
> Senao:
     > GET /api/v1/scds/{scdId}/SV
     > Renderizar tabela de streams SV (6 linhas)
> GET /api/v1/protocols/SV/metrics
> Aplicar overlay de metricas (jitter, perda de pacotes)
> Destacar linhas com anomalias (fundo amarelo/vermelho)
> Iniciar polling periodico de metricas SV

> Mesmo comportamento ao trocar para tab MMS:
     > Nao ha mapa SCD para MMS (schema extensivel)
     > GET /api/v1/protocols/MMS/metrics
     > GET /api/v1/protocols/MMS/data
     > Renderizar tabela de conexoes MMS
     > Iniciar polling periodico
```

### 3. Clique em linha da tabela (abrir drawer)

```
> Usuario clica em uma linha da tabela (ex: GCB01 na tab GOOSE)
> Drawer abre pela direita com animacao slide-in (~40% da largura)
> Exibe secoes conforme protocolo:
     GOOSE: info do CB, dataset, subscribers, metricas dinamicas
     SV: info do stream, subscribers, metricas, grafico de jitter
     MMS: info da conexao, metricas dinamicas
> Se metricas disponiveis:
     > Renderizar tabela chave-valor com dados atuais
     > Atualizar a cada ciclo de polling
     > Para SV: renderizar grafico de jitter temporal
> Se metricas indisponiveis (monitoramento parado ou sem dados):
     > Exibir mensagem: "Metricas indisponiveis - monitoramento inativo"
```

### 4. Atualizacao periodica de metricas (polling)

```
> A cada 15-30 segundos:
     > GET /api/v1/protocols/{protocolo_ativo}/metrics
> Frontend compara dados novos com dados anteriores
> Se houve mudanca:
     > Atualizar coluna Status e overlay de metricas na tabela
     > Atualizar destaque visual de linhas com anomalias
     > Se drawer aberto: atualizar metricas e grafico
> Para SV especificamente:
     > Verificar lostPackets > 0 > destacar com N (vermelho)
     > Verificar maxJitterUs > limiar > destacar com ! (amarelo)
     > Verificar duplicates > 0 > indicar no drawer
```

### 5. Aplicar filtros na tabela

```
> Usuario seleciona filtro (ex: Publisher = "MU_360_1")
> Tabela filtra client-side (dados ja carregados do SCD)
> Linhas que nao correspondem ao filtro ficam ocultas
> Paginacao recalcula total de resultados
> Combinar multiplos filtros (publisher + sub-rede + status)
> Botao [Limpar] reseta todos os filtros

> Filtro por Status (ex: "Warning"):
     > Exibir apenas linhas com metricas em estado de alerta
     > Requer que overlay de metricas esteja carregado
```

---

## Estados

### Estados gerais da tela

| Estado | Condicao | Visual |
|---|---|---|
| **Com SCD + metricas** | SCD ativo + monitoramento RUNNING + metricas disponiveis | Tabs visiveis, tabelas com dados do SCD e overlay de metricas em tempo real. Colunas de status com indicadores coloridos. Linhas com anomalias destacadas |
| **Com SCD, sem metricas** | SCD ativo + monitoramento STOPPED ou sem dados de metricas | Tabs visiveis, tabelas com dados estaticos do SCD (control blocks, streams). Coluna de metricas vazia ou com `—`. Mensagem sutil: "Monitoramento inativo — metricas indisponiveis" |
| **Sem SCD ativo** | `GET /scds?latest=true` nao retorna SCD ativo | Tabs ocultas. Estado vazio centralizado com icone, mensagem e link `« Ir para Configuracao »` |
| **Carregando** | Requisicoes em andamento (mapa SCD ou metricas) | Skeleton/shimmer na tabela. Tabs visiveis mas conteudo em carregamento |
| **Erro de API** | HTTP 500 ou erro de rede em qualquer endpoint | Banner de erro vermelho inline no topo: "Erro ao carregar dados de comunicacao. Tente novamente." + botao [Tentar novamente] |

### Estados por linha da tabela (GOOSE e SV)

| Estado | Condicao | Visual |
|---|---|---|
| **Normal** | Control block/stream com subscribers + metricas dentro dos limiares | ● Verde na coluna Status. Linha com aparencia padrao |
| **Sem subscriber** | Lista de subscribers vazia no SCD | ○ Cinza na coluna Status. Texto `—` na coluna Subscriber. Linha com opacidade reduzida (muted). MAC e VLAN podem exibir `—` se nao definidos |
| **Warning (jitter alto)** | SV: `maxJitterUs` acima do limiar configurado | ⚠ Amarelo na coluna Status. Fundo da linha com tom amarelo sutil. Valor de jitter com icone ⚠ |
| **Erro (perda de pacotes)** | SV: `lostPackets > 0` | ✕ Vermelho na coluna Status. Fundo da linha com tom vermelho sutil. Valor de lostPackets destacado |
| **Erro (perda de pacotes + jitter)** | SV: `lostPackets > 0` E `maxJitterUs` acima do limiar | ✕ Vermelho na coluna Status (prioridade maior que warning). Ambas as metricas destacadas |
| **Sem metricas** | Monitoramento inativo ou metricas nao recebidas para este CB/stream | `—` na coluna de metricas. Status baseado apenas em dados do SCD |

### Estados do drawer

| Estado | Condicao | Visual |
|---|---|---|
| **Aberto com dados completos** | CB/stream selecionado + metricas disponiveis | Todas as secoes preenchidas: info, dataset/subscribers, metricas, grafico (SV) |
| **Aberto sem metricas** | CB/stream selecionado + monitoramento inativo | Secoes de info e dataset/subscribers preenchidas. Secao de metricas: "Metricas indisponiveis — monitoramento inativo" |
| **Aberto com alerta** | SV: metricas excedem limiares | Banner de alerta no topo do drawer (fundo amarelo/vermelho). Metricas exibidas com destaque |
| **Carregando** | Dados sendo processados | Skeleton/shimmer dentro do drawer |

### Estados da sub-visao MMS

| Estado | Condicao | Visual |
|---|---|---|
| **Com dados** | Metricas MMS disponiveis | Tabela de conexoes com status e timestamps |
| **Sem dados** | Nenhuma metrica MMS recebida | Estado vazio na area da tab: "Nenhuma conexao MMS registrada. O agente de monitoramento ainda nao coletou dados MMS." |
| **Conexao com timeout** | Metrica indica timeout ou indisponibilidade | ⚠ Amarelo na coluna Status |
| **Conexao desconectada** | Metrica indica desconexao | ○ Cinza na coluna Status |

---

## Permissoes por Role

| Elemento | ADMIN | OPERATOR | VIEWER |
|---|---|---|---|
| Visualizar tabs (GOOSE, SV, MMS) | ✓ | ✓ | ✓ |
| Visualizar tabelas de dados | ✓ | ✓ | ✓ |
| Aplicar filtros | ✓ | ✓ | ✓ |
| Abrir drawer de detalhe | ✓ | ✓ | ✓ |
| Visualizar metricas em tempo real | ✓ | ✓ | ✓ |
| Visualizar grafico de jitter (SV) | ✓ | ✓ | ✓ |
| Link « Ir para Configuracao » (estado vazio) | ✓ Visivel e ativo | Link visivel (navega para Tela 7, onde botoes de acao serao desabilitados) | **Oculto** (VIEWER nao configura — remover o link, manter apenas a mensagem informativa) |

**Notas:**

- Esta tela e exclusivamente de **leitura** — nao ha acoes de escrita (nao ha botoes de start/stop, ACK, upload, etc.)
- A unica diferenca por role e a visibilidade do link para Configuracao no estado vazio
- Todos os roles possuem acesso identico a todas as funcionalidades de visualizacao, filtros e drawers

---

## Referencias

- `00-navegacao-global.md` — Layout master, header, sidebar, drawer, convencoes visuais, paginacao, campos dinamicos (secao 6.7) e RBAC
- `01-tela-inicial.md` — Topologia de rede que exibe os mesmos IEDs e fluxos GOOSE/SV em formato de grafo
- `07-configuracao.md` — Tela de configuracao (destino do link « Ir para Configuracao » no estado vazio; upload e ativacao de SCD)
- `docs/parsed-scd/scd.md` — Dados reais da subestacao: 4 IEDs, 6 sub-redes, 11 control blocks GOOSE, 6 streams SV

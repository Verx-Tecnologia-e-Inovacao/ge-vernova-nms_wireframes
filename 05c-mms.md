# Tela 5c — MMS

## Objetivo

Visualizar conexoes MMS da subestacao com status e metricas dinamicas. Acessivel via rota `/mms` na sidebar de navegacao (submenu Protocols).

O modelo de dados MMS e extensivel (`MmsMetricsIngestRequest` possui `additionalProperties`). Como o schema e definido em tempo de implantacao, a interface deve ser generica e renderizar campos dinamicos.

---

## Wireframe

### Estado principal

```
+--------------------------------------------------------------------------------------------------+
|  +- Header Global (ver 00-navegacao-global.md) ------------------------------------------+       |
|  |  [Logo NMS]   Subestacao: MU_360_1    Mon: * RUNNING           operador@empresa.com   |       |
|  +----------------------------------------------------------------------------------------+       |
+----------+---------------------------------------------------------------------------------------+
| SIDEBAR  |                                                                                       |
|          |  +- Barra de Filtros ---------------------------------------------------------+       |
| > Topol. |  |                                                                           |       |
|   Alm (2)|  |  {Conexao v}    {Status v}    [De: ____] [Ate: ____]              [Limpar]|       |
|   Sinc   |  |                                                                           |       |
|   Red    |  +----------------------------------------------------------------------------+       |
| > Proto  |                                                                                       |
|   GOOSE  |  +- Tabela de Conexoes MMS --------------------------------------------------+       |
|   SV     |  |                                                                           |       |
|   MMS    |  |  Conexao ^                   | Status          | Ultima Atualizacao       |       |
|   SNMP   |  |  ----------------------------+-----------------+--------------------------|       |
|   Cfg    |  |  L90_DIG/MMS_Server          | * Conectado     | 19/02/2026 14:32         |       |
|          |  |  T60_DIG/MMS_Server          | * Conectado     | 19/02/2026 14:30         |       |
|          |  |  MU_360_1/MMS_Server         | ! Timeout       | 19/02/2026 14:15         |       |
|          |  |  MU320E_LAB/MMS_Server       | o Desconectado  | 19/02/2026 09:00         |       |
|          |  |                                                                           |       |
|          |  +----------------------------------------------------------------------------+       |
|          |                                                                                       |
|          |  < Anterior  Pagina 1 de 1  Proxima >    Exibindo 4 de 4                              |
|          |                                                                                       |
| [Logout] |                                                                                       |
+----------+---------------------------------------------------------------------------------------+
```

**Detalhamento da tabela MMS:**

- **Colunas:** Conexao, Status, Ultima Atualizacao
- **Natureza generica:** Como o schema MMS e extensivel (`additionalProperties`), a tabela exibe apenas os campos base. Detalhes adicionais sao mostrados no drawer
- **Clique na linha:** Abre o drawer com metricas dinamicas

**Barra de filtros:**

- **Conexao:** Dropdown com todas as conexoes MMS
- **Status:** Dropdown com estados (`Todos`, `Conectado`, `Timeout`, `Desconectado`)
- **De / Ate:** Campos de data/hora para periodo temporal (`fromUtc`, `toUtc`)
- **Botao Limpar:** Reseta todos os filtros

---

### Drawer de detalhe da conexao MMS

```
                              +----------------------------------------------+
                              | [X]                                          |
                              |                                              |
                              |  * L90_DIG — MMS                             |
                              |  ----------------------------------------    |
                              |                                              |
                              |  -- Informacoes da Conexao --                |
                              |                                              |
                              |  Conexao:           L90_DIG/MMS_Server       |
                              |  Status:            * Conectado              |
                              |  Ultima Atualizacao: 19/02/2026 14:32        |
                              |                                              |
                              |  -- Metricas (campos dinamicos) --           |
                              |  (additionalProperties do                    |
                              |   MmsMetricsIngestRequest)                   |
                              |                                              |
                              |  +-----------------------------------+       |
                              |  | Chave              | Valor        |       |
                              |  +-----------------------------------+       |
                              |  | (campos definidos   |              |       |
                              |  |  em tempo de        |              |       |
                              |  |  implantacao -      |              |       |
                              |  |  renderizar todas   |              |       |
                              |  |  as chaves do       |              |       |
                              |  |  objeto de          |              |       |
                              |  |  metricas como      |              |       |
                              |  |  tabela chave-      |              |       |
                              |  |  valor)             |              |       |
                              |  +-----------------------------------+       |
                              |                                              |
                              |  Nota: O modelo de dados MMS e               |
                              |  extensivel. Os campos acima variam          |
                              |  conforme a configuracao do agente           |
                              |  de monitoramento. Renderizar como           |
                              |  componente de campos dinamicos              |
                              |  (ver 00-navegacao-global.md, 6.7).          |
                              |                                              |
                              +----------------------------------------------+
```

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
|   SV     |          |          (icone: terminal / conexao MMS)            |                      |
|   MMS    |          |                                                     |                      |
|   SNMP   |          |      Nenhuma conexao MMS registrada.                |                      |
|   Cfg    |          |                                                     |                      |
|          |          |      O agente de monitoramento ainda nao            |                      |
|          |          |      coletou dados MMS.                             |                      |
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
- MMS nao depende diretamente do SCD — o estado vazio e exibido quando nao ha metricas MMS coletadas.

---

## Componentes

| Componente | Descricao | Fonte de Dados |
|---|---|---|
| **Barra de filtros** | Dropdowns: Conexao, Status. Campos De/Ate para periodo temporal. Botao Limpar | Dados de metricas MMS |
| **Tabela de conexoes MMS** | Tabela com conexoes MMS, colunas ordenaveis, clique abre drawer | `GET /protocols/MMS/metrics` + `GET /protocols/MMS/data` |
| **Drawer de detalhe MMS** | Painel lateral ~40% largura. Secoes: info da conexao, metricas dinamicas (chave-valor) | Metricas MMS |
| **Campos dinamicos (chave-valor)** | Tabela renderizando todas as chaves do objeto de metricas como pares chave-valor. Componente generico conforme `00-navegacao-global.md` secao 6.7 | `additionalProperties` do MmsMetricsIngestRequest |
| **Estado vazio (sem dados)** | Tela centralizada com icone, mensagem e link para `/settings/scd` | Nenhuma metrica MMS coletada |
| **Paginacao** | Padrao conforme `00-navegacao-global.md` secao 6.1 | `meta` da resposta |
| **Banner de erro** | Erro inline no topo da area de conteudo | HTTP 4xx/5xx |

---

## Dados e Endpoints

| # | Metodo | Endpoint | Uso na tela | Campos utilizados |
|---|---|---|---|---|
| 1 | `GET` | `/api/v1/protocols/MMS/metrics` | Metricas de conexoes MMS | `items[]` (MetricPoint) com `timestampUtc`, `hostId`, `metrics` (object, additionalProperties) |
| 2 | `GET` | `/api/v1/protocols/MMS/data` | Dados MMS persistidos (historico) | `items[]` (ProtocolDataRecord) com `data` (object) |

**Schemas:**

| Schema | Campos | Observacoes |
|---|---|---|
| `MmsMetricsIngestRequest` | Extensivel — schema definido em tempo de implantacao (`additionalProperties`) | Renderizar integralmente como campos dinamicos |
| `MetricPoint` | `timestampUtc` (datetime), `hostId` (string), `metrics` (object, additionalProperties) | Ponto de metrica generico |
| `ProtocolDataRecord` | `id` (uuid), `protocol` (string), `type` (string), `scdId` (uuid), `createdAtUtc` (datetime), `data` (object) | Registro generico de dado persistido |

### Exemplos de JSON (para designs Figma)

#### Exemplo: Linha da tabela — conexao MMS conectada

```json
{
  "timestampUtc": "2026-02-19T14:32:00Z",
  "hostId": "L90_DIG/MMS_Server",
  "metrics": {
    "status": "connected",
    "uptime_seconds": 86400,
    "requests_total": 12450,
    "requests_failed": 0,
    "avg_response_ms": 12.5,
    "last_error": null
  }
}
```

#### Exemplo: Linha da tabela — conexao MMS com timeout

```json
{
  "timestampUtc": "2026-02-19T14:15:00Z",
  "hostId": "MU_360_1/MMS_Server",
  "metrics": {
    "status": "timeout",
    "uptime_seconds": 3600,
    "requests_total": 890,
    "requests_failed": 45,
    "avg_response_ms": 2350.8,
    "last_error": "connection timeout after 5000ms"
  }
}
```

#### Exemplo: Linha da tabela — conexao MMS desconectada

```json
{
  "timestampUtc": "2026-02-19T09:00:00Z",
  "hostId": "MU320E_LAB/MMS_Server",
  "metrics": {
    "status": "disconnected",
    "uptime_seconds": 0,
    "requests_total": 0,
    "requests_failed": 0,
    "avg_response_ms": null,
    "last_error": "connection refused"
  }
}
```

#### Exemplo: Drawer — metricas dinamicas completas (additionalProperties)

```json
{
  "timestampUtc": "2026-02-19T14:32:00Z",
  "hostId": "L90_DIG/MMS_Server",
  "metrics": {
    "status": "connected",
    "uptime_seconds": 86400,
    "requests_total": 12450,
    "requests_failed": 0,
    "avg_response_ms": 12.5,
    "max_response_ms": 145.2,
    "active_associations": 2,
    "named_variables_read": 8920,
    "named_variables_written": 3530,
    "reports_sent": 156,
    "file_transfers": 0,
    "last_error": null,
    "server_version": "MMS/1.0",
    "tls_enabled": true
  }
}
```

> **Nota:** Os campos acima sao exemplos ilustrativos. O schema real MMS e definido em tempo de implantacao (`additionalProperties`). Os campos variam conforme a configuracao do agente de monitoramento. A interface deve renderizar todos os campos recebidos como tabela chave-valor.

---

### Estrategia de carregamento

```
> Etapa 1 - Carregar metricas MMS (bloqueante)
   GET /protocols/MMS/metrics
   GET /protocols/MMS/data
   +- Se nenhum dado retornado > exibir estado vazio
   +- Se ha dados > renderizar tabela de conexoes

> Polling continuo (a cada 15-30 segundos):
   GET /protocols/MMS/metrics
   +- Atualizar status e timestamps na tabela
   +- Se drawer aberto: atualizar metricas dinamicas
```

---

## Fluxos de Interacao

### 1. Carregamento da pagina

```
> Usuario acessa /mms via sidebar
> Frontend executa GET /api/v1/protocols/MMS/metrics
> Se nenhum dado MMS:
     > Exibir estado vazio
     > Fim do fluxo
> Se ha dados:
     > Renderizar tabela de conexoes MMS
     > Iniciar polling periodico
```

### 2. Clique em linha da tabela (abrir drawer)

```
> Usuario clica em uma linha (ex: L90_DIG/MMS_Server)
> Drawer abre pela direita com animacao slide-in
> Exibe:
     - Informacoes da conexao (nome, status, ultima atualizacao)
     - Metricas dinamicas como tabela chave-valor (additionalProperties)
```

### 3. Atualizacao periodica (polling)

```
> A cada 15-30 segundos:
     > GET /api/v1/protocols/MMS/metrics
> Atualizar status e timestamps na tabela
> Se drawer aberto: atualizar metricas dinamicas
```

### 4. Aplicar filtros

```
> Filtros de conexao e status aplicados client-side
> Filtros de periodo (De/Ate) aplicados como query params na API (fromUtc/toUtc)
> Botao [Limpar] reseta todos os filtros
```

---

## Estados

### Estados gerais da tela

| Estado | Condicao | Visual |
|---|---|---|
| **Com dados** | Metricas MMS disponiveis | Tabela de conexoes com status e timestamps |
| **Sem dados** | Nenhuma metrica MMS recebida | Estado vazio: "Nenhuma conexao MMS registrada. O agente de monitoramento ainda nao coletou dados MMS." |
| **Carregando** | Requisicoes em andamento | Skeleton/shimmer na tabela |
| **Erro de API** | HTTP 500 ou erro de rede | Banner de erro inline |

### Estados por linha da tabela

| Estado | Condicao | Visual |
|---|---|---|
| **Conectado** | Metrica indica conexao ativa | * Verde na coluna Status |
| **Timeout** | Metrica indica timeout ou indisponibilidade | ! Amarelo na coluna Status |
| **Desconectado** | Metrica indica desconexao | o Cinza na coluna Status |

### Estados do drawer

| Estado | Condicao | Visual |
|---|---|---|
| **Aberto com dados** | Conexao selecionada + metricas disponiveis | Secoes de info e metricas dinamicas preenchidas |
| **Carregando** | Dados sendo processados | Skeleton/shimmer dentro do drawer |

---

## Permissoes por Role

| Elemento | ADMIN | OPERATOR | VIEWER |
|---|---|---|---|
| Visualizar tabela MMS | ✓ | ✓ | ✓ |
| Aplicar filtros | ✓ | ✓ | ✓ |
| Abrir drawer de detalhe | ✓ | ✓ | ✓ |
| Visualizar metricas dinamicas | ✓ | ✓ | ✓ |
| Link « Ir para Configuracao » (estado vazio) | ✓ Visivel e ativo | Link visivel (navega para `/settings/scd`) | **Oculto** |

**Notas:**

- Esta pagina e exclusivamente de **leitura** — nao ha acoes de escrita.
- A unica diferenca por role e a visibilidade do link para Configuracao no estado vazio.

---

## Referencias

- `00-navegacao-global.md` — Layout master, header, sidebar, drawer, convencoes visuais, paginacao, campos dinamicos (secao 6.7) e RBAC
- `05a-goose.md` — Pagina GOOSE (mesma familia de protocolos)
- `05b-sampled-values.md` — Pagina Sampled Values (mesma familia de protocolos)
- `07-configuracao.md` — Tela de configuracao em `/settings/scd` (destino do link no estado vazio)

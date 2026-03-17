# Tela 9 — Dashboard (Tela Inicial)

## Objetivo

Apresentar uma visao geral da saude da subestacao ao operador imediatamente apos o login. Esta e a tela inicial da aplicacao — o usuario e redirecionado automaticamente para ca apos autenticacao bem-sucedida (ver `08-autenticacao.md`).

O dashboard e uma tela **puramente informativa** — somente leitura, sem acoes, sem navegacao a partir dos cards. Seu proposito e condensar os dados mais relevantes do SCD ativo e do monitoramento em tempo real em uma unica visao, organizada em 3 faixas por prioridade:

1. **Status critico** — estado do monitoramento, alarmes ativos por severidade, diagnostics do SCD
2. **Visao geral de dispositivos e comunicacao** — IEDs, streams GOOSE/SV, subscriptions
3. **Informacoes contextuais** — MMS points, vendors, source files

Todos os roles (ADMIN, OPERATOR, VIEWER) visualizam o mesmo conteudo. A unica diferenca por role e a visibilidade do botao para Settings no estado vazio (sem SCD ativo).

A topologia de rede, anteriormente a tela inicial, passa a estar disponivel em `/topology` (ver `01-tela-inicial.md`).

---

## Wireframe

### Estado principal (completo — SCD ativo + monitoramento RUNNING)

```
+--------------------------------------------------------------------------------------------------+
|  +- Header Global (ver 00-navegacao-global.md) --------------------------------------------+     |
|  |  [Logo NMS]   Subestacao: SubstationXYZ   Mon: * RUNNING        operador@empresa.com    |     |
|  +------------------------------------------------------------------------------------------+     |
+----------+----------------------------------------------------------------------------------------+
| SIDEBAR  |                                                                                        |
|          |  +- Faixa 1 — Status Critico --------------------------------------------------- +     |
| > Dash.  |  |                                                                               |     |
|   Topol. |  |  +- Barra de Monitoramento (full-width) -------------------------------- +    |     |
|   Alm(28)|  |  |  * RUNNING desde 19/02/2026 10:00                                     |    |     |
|   Sinc   |  |  +--------------------------------------------------------------------  -+    |     |
|   Red    |  |                                                                               |     |
|   Proto  |  |  +- Alarmes Ativos ---------------+   +- Diagnostics SCD ---------------+    |     |
|     GOOSE|  |  |                                 |   |                                 |    |     |
|     SV   |  |  |  N CRITICAL          3          |   |  N Errors              38       |    |     |
|     MMS  |  |  |  ! MAJOR             5          |   |  ! Warnings            93       |    |     |
|   Config |  |  |  o MEDIUM           12          |   |  o Info             3.095       |    |     |
|     SCD  |  |  |  o LOW               8          |   |                                 |    |     |
|     Mon  |  |  |                                 |   |                                 |    |     |
|     MIB  |  |  +---------------------------------+   +---------------------------------+    |     |
|          |  |                                                                               |     |
|          |  +------------------------------------------------------------------------------------+|
|          |                                                                                        |
|          |  +- Faixa 2 — Dispositivos e Comunicacao ----------------------------------------+     |
|          |  |                                                                               |     |
|          |  |  +- IEDs ---------------+  +- Streams ---------+  +- Subscriptions --------+  |     |
|          |  |  |                      |  |                   |  |                        |  |     |
|          |  |  |  Reais           42  |  |  GOOSE       237  |  |  Validos        2.682  |  |     |
|          |  |  |  Proxy          161  |  |  SV           80  |  |  Invalidos          0  |  |     |
|          |  |  |                      |  |                   |  |                        |  |     |
|          |  |  |  Por tipo:           |  |                   |  |                        |  |     |
|          |  |  |  mu:      16         |  |                   |  |                        |  |     |
|          |  |  |  ied:     20         |  |                   |  |                        |  |     |
|          |  |  |  other:    6         |  |                   |  |                        |  |     |
|          |  |  |  proxy:  161         |  |                   |  |                        |  |     |
|          |  |  |                      |  |                   |  |                        |  |     |
|          |  |  +----------------------+  +-------------------+  +------------------------+  |     |
|          |  |                                                                               |     |
|          |  +-------------------------------------------------------------------------------+     |
|          |                                                                                        |
|          |  +- Faixa 3 — Informacoes Contextuais -------------------------------------------+     |
|          |  |                                                                               |     |
|          |  |  +- MMS Points ---------+  +- Vendors e Arquivos -------------------------+   |     |
|          |  |  |                      |  |                                              |   |     |
|          |  |  |  Total       5.370   |  |  Vendors:  GE, GE Multilin                  |   |     |
|          |  |  |  LGOS        4.880   |  |  Source files:  42                           |   |     |
|          |  |  |  LSVS          408   |  |                                              |   |     |
|          |  |  |  LTMS           82   |  |                                              |   |     |
|          |  |  |                      |  |                                              |   |     |
|          |  |  +----------------------+  +----------------------------------------------+   |     |
|          |  |                                                                               |     |
|          |  +-------------------------------------------------------------------------------+     |
|          |                                                                                        |
| [Logout] |                                                                                        |
+----------+----------------------------------------------------------------------------------------+
```

**Detalhamento das faixas:**

- **Barra de monitoramento (full-width)**: Icone de estado colorido + texto do estado (RUNNING/STOPPED/ERROR) + data/hora de inicio (`sinceUtc` convertido para horario local). Cores conforme `00-navegacao-global.md` secao 3.
- **Card Alarmes Ativos**: Contagem de alarmes nao reconhecidos (`ack=false`) agrupados por severidade. Cada linha exibe badge colorido de severidade + label + contagem. Severidades: CRITICAL, MAJOR, MEDIUM, LOW com badges coloridos conforme design system.
- **Card Diagnostics SCD**: Contagem de diagnosticos do SCD por nivel (error, warning, info). Dados estaticos do summary — nao mudam enquanto o SCD estiver ativo.
- **Card IEDs**: Total de IEDs reais (com CID carregado) e proxies (referenciados sem CID). Distribuicao por tipo (mu, ied, other, proxy).
- **Card Streams**: Contagem total de control blocks GOOSE e SV definidos no SCD.
- **Card Subscriptions**: Total de subscriptions validos e invalidos. Indica saude das referencias ExtRef no SCD.
- **Card MMS Points**: Pontos MMS monitorados por classe (LGOS para GOOSE, LSVS para SV, LTMS para tempo).
- **Card Vendors e Arquivos**: Lista de fabricantes encontrados no SCD e contagem de arquivos fonte processados.
- **Numeros grandes**: Valores acima de 999 devem ser formatados com separador de milhar (ex: 2.682, 5.370, 3.095).

---

### Estado monitoramento parado (SCD ativo + STOPPED)

```
+--------------------------------------------------------------------------------------------------+
|  +- Header Global -----------------------------------------------------------------------+       |
|  |  [Logo NMS]   Subestacao: SubstationXYZ   Mon: o STOPPED        operador@empresa.com  |       |
|  +----------------------------------------------------------------------------------------+       |
+----------+----------------------------------------------------------------------------------------+
| SIDEBAR  |                                                                                        |
|          |  +- Faixa 1 — Status Critico ---------------------------------------------------+      |
| > Dash.  |  |                                                                              |      |
|   Topol. |  |  +- Barra de Monitoramento (full-width) --------------------------------+    |      |
|   Alm    |  |  |  o STOPPED desde 19/02/2026 15:30                                    |    |      |
|   Sinc   |  |  +------------------------------------------------------------------  --+    |      |
|   Red    |  |                                                                              |      |
|   Proto  |  |  +- Alarmes Ativos ---------------+   +- Diagnostics SCD ---------------+   |      |
|   Config |  |  |                                 |   |                                 |   |      |
|          |  |  |  Monitoramento parado.           |   |  N Errors              38       |   |      |
|          |  |  |  Alarmes indisponiveis.          |   |  ! Warnings            93       |   |      |
|          |  |  |                                 |   |  o Info             3.095       |   |      |
|          |  |  |                                 |   |                                 |   |      |
|          |  |  +---------------------------------+   +---------------------------------+   |      |
|          |  |                                                                              |      |
|          |  +------------------------------------------------------------------------------+      |
|          |                                                                                        |
|          |  +- Faixa 2 — Dispositivos e Comunicacao ----------------------------------------+     |
|          |  |  (conteudo identico ao estado completo — dados do SCD sao estaticos)           |     |
|          |  +-------------------------------------------------------------------------------+     |
|          |                                                                                        |
|          |  +- Faixa 3 — Informacoes Contextuais -------------------------------------------+     |
|          |  |  (conteudo identico ao estado completo — dados do SCD sao estaticos)           |     |
|          |  +-------------------------------------------------------------------------------+     |
|          |                                                                                        |
| [Logout] |                                                                                        |
+----------+----------------------------------------------------------------------------------------+
```

**Diferencas em relacao ao estado completo:**

- Header: indicador de monitoramento exibe `o STOPPED` em cinza (ao inves de `* RUNNING` em verde).
- Barra de monitoramento: exibe `o STOPPED desde {data/hora}` em cinza.
- Card Alarmes Ativos: substituido por mensagem `Monitoramento parado. Alarmes indisponiveis.` em texto cinza. Sem contadores de severidade — alarmes dependem do monitoramento ativo para serem gerados.
- Card Diagnostics SCD: permanece inalterado (dados estaticos do SCD, nao dependem do monitoramento).
- Faixas 2 e 3: permanecem inalteradas (dados estaticos do SCD).
- Badge de alarmes na sidebar: sem contagem (monitoramento parado nao gera novos alarmes).

---

### Estado sem SCD ativo

Exibido quando a chamada `GET /scds?latest=true` nao retorna nenhum SCD com status `ACTIVE`, ou quando nao ha nenhum SCD cadastrado no sistema.

```
+--------------------------------------------------------------------------------------------------+
|  +- Header Global -----------------------------------------------------------------------+       |
|  |  [Logo NMS]   Subestacao: —                Mon: o STOPPED        admin@empresa.com     |       |
|  +----------------------------------------------------------------------------------------+       |
+----------+----------------------------------------------------------------------------------------+
| SIDEBAR  |                                                                                        |
|          |                                                                                        |
| > Dash.  |                                                                                        |
|   Topol. |                                                                                        |
|   Alm    |                                                                                        |
|   Sinc   |                                                                                        |
|   Red    |         +-----------------------------------------------------+                        |
|   Proto  |         |                                                     |                        |
|   Config |         |           (icone de dashboard/graficos               |                        |
|          |         |            com indicadores vazios)                   |                        |
|          |         |                                                     |                        |
|          |         |    Nenhum SCD ativo encontrado.                      |                        |
|          |         |                                                     |                        |
|          |         |    O dashboard sera exibido apos a ativacao          |                        |
|          |         |    de um arquivo SCD na tela de Configuracao.        |                        |
|          |         |                                                     |                        |
|          |         |    [Ir para Configuracao]                            |                        |
|          |         |                                                     |                        |
|          |         +-----------------------------------------------------+                        |
|          |                                                                                        |
|          |                                                                                        |
| [Logout] |                                                                                        |
+----------+----------------------------------------------------------------------------------------+
```

**Detalhamento:**

- O icone ilustrativo deve representar um dashboard/painel com graficos vazios.
- A mensagem explica que o SCD e necessario para popular os dados do dashboard.
- O botao `[Ir para Configuracao]` navega para `/settings/scd` (Tela 7, sub-rota SCD), onde o usuario pode fazer upload e ativar um SCD.
- O header exibe `Subestacao: —` (traco) e o monitoramento como `o STOPPED` quando nao ha SCD ativo.
- A visibilidade do botao `[Ir para Configuracao]` depende do role do usuario (ver secao Permissoes).

---

## Componentes

| Componente | Descricao | Fonte de Dados |
|---|---|---|
| **Barra de monitoramento** | Barra full-width no topo da faixa 1. Exibe icone de estado colorido + texto do estado (RUNNING/STOPPED/ERROR) + data/hora de inicio (`sinceUtc` convertido para horario local). Cores do estado conforme `00-navegacao-global.md` secao 3 | `GET /monitoring` → `state`, `sinceUtc` |
| **Card Alarmes Ativos** | Card na faixa 1. Lista contagem de alarmes nao reconhecidos por severidade (CRITICAL, MAJOR, MEDIUM, LOW). Cada severidade com badge colorido conforme design system. Quando monitoramento parado: exibe mensagem `Monitoramento parado. Alarmes indisponiveis.` em cinza | `GET /alarms?ack=false&severity={X}&pageSize=1` (4 chamadas paralelas) → `meta.total` por severidade |
| **Card Diagnostics SCD** | Card na faixa 1. Exibe contagem de diagnosticos por nivel (error, warning, info) com indicador visual de saude. Dados estaticos do SCD — nao mudam enquanto o SCD estiver ativo | `GET /scds/{scdId}/summary` → `diagnostics.error`, `diagnostics.warning`, `diagnostics.info` |
| **Card IEDs** | Card na faixa 2. Exibe total de IEDs reais e proxies, com distribuicao por tipo (mu, ied, other, proxy) | `GET /scds/{scdId}/summary` → `ieds`, `ieds_proxy`, `type` |
| **Card Streams** | Card na faixa 2. Exibe contagem de control blocks GOOSE e SV | `GET /scds/{scdId}/summary` → `goose_streams`, `sv_streams` |
| **Card Subscriptions** | Card na faixa 2. Exibe total de subscriptions validos e invalidos | `GET /scds/{scdId}/summary` → `subscriptions.valid`, `subscriptions.invalid` |
| **Card MMS Points** | Card na faixa 3. Exibe pontos MMS monitorados: total e breakdown por classe (LGOS, LSVS, LTMS) | `GET /scds/{scdId}/summary` → `mms_points.total`, `mms_points.lgos`, `mms_points.lsvs`, `mms_points.ltms` |
| **Card Vendors e Arquivos** | Card na faixa 3. Lista de fabricantes (vendors) encontrados no SCD e contagem de arquivos fonte processados | `GET /scds/{scdId}/summary` → `vendors[]`, `source_files` |
| **Estado vazio** | Tela centralizada quando nao ha SCD ativo. Icone ilustrativo de dashboard vazio + mensagem + botao `[Ir para Configuracao]` (ADMIN/OPERATOR visivel, VIEWER oculto) | `GET /scds?latest=true` retorna vazio ou sem status ACTIVE |
| **Banner de erro** | Banner inline vermelho no topo da area de conteudo quando ha falha de comunicacao com a API | Qualquer endpoint retornando HTTP 5xx ou erro de rede |

---

## Dados e Endpoints

| # | Metodo | Endpoint | Uso na tela | Campos utilizados | Exemplo |
|---|---|---|---|---|---|
| 1 | `GET` | `/api/v1/scds?latest=true` | Obter o SCD ativo (bloqueante — determina se dashboard exibe dados ou estado vazio) | `scdId`, `status` | `GET /api/v1/scds?latest=true` → se `status != ACTIVE`, exibir estado vazio |
| 2 | `GET` | `/api/v1/scds/{scdId}/summary` | Dados estaticos do SCD: IEDs, streams, subscriptions, diagnostics, vendors, MMS points, source files | `ieds`, `ieds_proxy`, `goose_streams`, `sv_streams`, `type{}`, `mms_points{}`, `subscriptions{}`, `diagnostics{}`, `vendors[]`, `source_files` | `GET /api/v1/scds/a1b2c3d4/summary` → preenche faixas 2, 3 e card diagnostics |
| 3 | `GET` | `/api/v1/monitoring` | Estado do monitoramento em tempo real | `state` (RUNNING/STOPPED/ERROR), `sinceUtc` | `GET /api/v1/monitoring` → `state: "RUNNING"`, `sinceUtc: "2026-02-19T13:00:00Z"` |
| 4 | `GET` | `/api/v1/alarms?ack=false&severity={X}&pageSize=1` | Contagem de alarmes por severidade (4 chamadas paralelas: CRITICAL, MAJOR, MEDIUM, LOW) | `meta.total` (contagem total de alarmes para a severidade filtrada) | `GET /api/v1/alarms?ack=false&severity=CRITICAL&pageSize=1` → `meta.total: 3` |

**Nota tecnica — contagem de alarmes por severidade:**

A API de alarmes nao fornece contadores agregados por severidade. Para obter as contagens sem transferir todos os dados de alarmes, o frontend deve executar 4 chamadas paralelas com `pageSize=1`, uma por severidade, e usar o campo `meta.total` da resposta como contador:

```
Paralelo:
  GET /api/v1/alarms?ack=false&severity=CRITICAL&pageSize=1  → meta.total = 3
  GET /api/v1/alarms?ack=false&severity=MAJOR&pageSize=1     → meta.total = 5
  GET /api/v1/alarms?ack=false&severity=MEDIUM&pageSize=1    → meta.total = 12
  GET /api/v1/alarms?ack=false&severity=LOW&pageSize=1       → meta.total = 8
```

Esta abordagem e mais escalavel que buscar todos os alarmes e contar client-side, pois transfere dados minimos (1 item + meta por chamada).

**Schemas relevantes:**

| Schema | Campos | Observacoes |
|---|---|---|
| `ScdSummaryResponse` | `scdId` (uuid), `status` (ScdStatus), `substationName` (string), `parsedAtUtc` (datetime), `sourceFiles` (string[]), `ieds` (ScdSummaryIed[]), `networks` (ScdSummaryNetwork[]) | Resposta do summary — para o dashboard, os campos agregados do JSON completo sao usados (ver secao Summary da doc do parser SCD) |
| `MonitoringStatusResponse` | `enabled` (boolean), `state` (MonitoringState), `sinceUtc` (datetime), `monitorings` (MonitoringItem[]), `stoppedAtUtc` (datetime/null), `stoppedBy` (string/null) | Singleton — estado global do monitoramento |
| `AlarmListResponse` | `items` (AlarmRecord[]), `meta` ({ page, pageSize, total }) | Apenas `meta.total` e usado no dashboard (contagem por severidade) |
| `AlarmSeverity` | LOW, MEDIUM, MAJOR, CRITICAL | Enum para filtro e badge colorido |
| `MonitoringState` | STOPPED, STARTING, RUNNING, STOPPING, ERROR | Enum para estado do monitoramento |

---

### Estrategia de carregamento de dados

O carregamento e feito em etapas para garantir uma experiencia progressiva:

```
> Etapa 1 - SCD ativo (bloqueante)
   GET /scds?latest=true
   +- Se nao ha SCD ativo → exibir estado vazio (fim do fluxo)
   +- Se ha SCD ativo → obter scdId, prosseguir

> Etapa 2 - Dados do dashboard (paralelo)
   GET /scds/{scdId}/summary    → preencher faixas 2 e 3 + card diagnostics
   GET /monitoring               → preencher barra de monitoramento
   GET /alarms?ack=false&severity=CRITICAL&pageSize=1  ┐
   GET /alarms?ack=false&severity=MAJOR&pageSize=1     │ → preencher card alarmes
   GET /alarms?ack=false&severity=MEDIUM&pageSize=1    │
   GET /alarms?ack=false&severity=LOW&pageSize=1       ┘

> Etapa 3 - Polling continuo (a cada 15 segundos)
   GET /monitoring               → atualizar barra de monitoramento
   GET /alarms (4 chamadas)      → atualizar card alarmes
   (Nao re-buscar summary — dados do SCD sao estaticos)
```

**Cache no client:**

- Os dados do SCD (etapa 2, summary) nao mudam apos ativacao — podem ser cacheados com TTL longo ou ate invalidacao manual (novo SCD ativado).
- Os dados de monitoramento e alarmes (etapas 2-3) sao dinamicos e devem ser atualizados via polling a cada 15 segundos.

---

## Fluxos de Interacao

### 1. Carregamento da pagina

```
> Usuario acessa / (dashboard) apos login
> Frontend executa GET /api/v1/scds?latest=true
> Se nao ha SCD ativo:
     > Exibir estado vazio com botao [Ir para Configuracao]
     > Botao visivel para ADMIN/OPERATOR, oculto para VIEWER
     > Fim do fluxo
> Se ha SCD ativo:
     > Exibir skeleton/shimmer nas 3 faixas
     > Paralelo: GET /scds/{scdId}/summary + GET /monitoring + 4x GET /alarms (por severidade)
     > Conforme cada resposta chega, preencher o card correspondente (renderizacao progressiva)
     > Apos todas as respostas: dashboard completo
     > Iniciar polling periodico (etapa 3)
```

### 2. Navegacao para Settings (estado vazio)

```
> Usuario (ADMIN ou OPERATOR) clica [Ir para Configuracao] no estado vazio
> Frontend navega para /settings/scd (Tela 7, sub-rota SCD)
> Tela 7 exibe opcoes de upload e ativacao de SCD
```

### 3. Atualizacao periodica (polling)

```
> A cada 15 segundos:
     > GET /api/v1/monitoring
     > 4x GET /api/v1/alarms?ack=false&severity={X}&pageSize=1
> Frontend compara dados novos com dados anteriores
> Se houve mudanca:
     > Atualizar barra de monitoramento (estado + data/hora)
     > Atualizar contadores de alarmes por severidade
> Se o estado de monitoramento mudou para STOPPED:
     > Substituir card alarmes por mensagem "Monitoramento parado. Alarmes indisponiveis."
     > Atualizar header global: indicador muda para o STOPPED (cinza)
> Se o estado de monitoramento mudou para ERROR:
     > Atualizar header global: indicador muda para X ERROR (vermelho)
     > Barra de monitoramento exibe X ERROR em vermelho
```

### 4. Mudanca de estado do monitoramento

```
> Monitoramento muda de RUNNING para STOPPED (detectado via polling):
     > Barra de monitoramento: icone muda para o STOPPED cinza
     > Card alarmes: substituido por mensagem "Monitoramento parado. Alarmes indisponiveis."
     > Faixas 2 e 3: sem alteracao (dados estaticos do SCD)

> Monitoramento muda de STOPPED para RUNNING (detectado via polling):
     > Barra de monitoramento: icone muda para * RUNNING verde
     > Card alarmes: volta a exibir contadores por severidade
     > Re-executar 4x GET /alarms imediatamente (sem esperar proximo ciclo de polling)
```

### 5. Ativacao de SCD (transicao empty → complete)

```
> Usuario ativa SCD em outra aba/sessao ou via tela de configuracao
> No proximo ciclo de polling (ou ao retornar ao dashboard):
     > GET /api/v1/scds?latest=true retorna SCD com status ACTIVE
     > Dashboard transiciona do estado vazio para o estado completo
     > Executa carregamento completo (etapa 2 da estrategia)
     > Nota: se o usuario fez a ativacao na mesma sessao (navegou para /settings/scd e voltou),
       o carregamento da pagina (fluxo 1) ja detecta o SCD ativo normalmente.
```

---

## Estados

| Estado | Condicao | Visual |
|---|---|---|
| **Completo** | SCD ativo + monitoramento RUNNING + dados carregados | Todas as 3 faixas preenchidas com dados reais. Barra de monitoramento `* RUNNING` em verde |
| **Monitoramento parado** | SCD ativo + monitoramento STOPPED | Faixas 2-3 preenchidas normalmente. Barra de monitoramento `o STOPPED` em cinza. Card alarmes substituido por mensagem `Monitoramento parado. Alarmes indisponiveis.` em cinza. Card diagnostics inalterado |
| **Monitoramento em erro** | SCD ativo + monitoramento ERROR | Faixas 2-3 preenchidas normalmente. Barra de monitoramento `X ERROR` em vermelho |
| **Sem SCD ativo** | Nenhum SCD com status ACTIVE | Estado vazio centralizado: icone de dashboard vazio + mensagem + botao `[Ir para Configuracao]` (ADMIN/OPERATOR visivel, VIEWER oculto). Header exibe `Subestacao: —` e `o STOPPED` |
| **Carregando** | Requisicoes em andamento (etapas 1-2) | Skeleton/shimmer nas 3 faixas. Cada card e preenchido individualmente conforme sua resposta chega |
| **Erro de API** | Falha de comunicacao com o backend | Banner de erro inline no topo da area de conteudo: `Erro ao carregar dashboard. Tente novamente.` + botao `[Tentar novamente]`. Cards que falharam exibem indicador de erro individual |

---

## Permissoes por Role

| Elemento | ADMIN | OPERATOR | VIEWER |
|---|---|---|---|
| Visualizar Dashboard | ✓ | ✓ | ✓ |
| Ver barra de monitoramento | ✓ | ✓ | ✓ |
| Ver card alarmes ativos | ✓ | ✓ | ✓ |
| Ver card diagnostics | ✓ | ✓ | ✓ |
| Ver faixa 2 (IEDs, streams, subscriptions) | ✓ | ✓ | ✓ |
| Ver faixa 3 (MMS points, vendors, arquivos) | ✓ | ✓ | ✓ |
| Botao `[Ir para Configuracao]` (estado vazio) | ✓ Visivel e ativo | ✓ Visivel e ativo | **Oculto** (VIEWER nao configura) |

**Notas:**

- Esta tela e exclusivamente de **visualizacao** — nao ha acoes de escrita (nao ha botoes de start/stop, ACK, upload, etc.).
- A unica diferenca por role e a visibilidade do botao `[Ir para Configuracao]` no estado vazio.
- O botao de start/stop do monitoramento **nao esta presente nesta tela** — ele pertence a Tela 7 (Configuracao).

---

## Dados de Exemplo (para reprodução no Figma)

Os JSONs abaixo representam as respostas reais de cada endpoint usado pelo dashboard, com dados de uma subestação real (42 IEDs + 161 proxies). Usar estes valores exatos para popular a tela no Figma.

### Endpoint 1 — `GET /api/v1/scds?latest=true`

Retorna o SCD ativo. O campo `scdId` é usado como parâmetro para o endpoint de summary.

```json
{
  "items": [
    {
      "scdId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
      "fileName": "SubstationXYZ_2026.scd",
      "sha256": "e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855",
      "status": "ACTIVE",
      "createdAtUtc": "2026-02-18T09:15:00Z",
      "parsedAtUtc": "2026-02-18T09:15:42Z",
      "activatedAtUtc": "2026-02-18T09:20:00Z"
    }
  ],
  "meta": {
    "page": 1,
    "pageSize": 1,
    "total": 1
  }
}
```

---

### Endpoint 2 — `GET /api/v1/scds/{scdId}/summary`

Dados estáticos do SCD. Preenche faixas 2 e 3, e o card de diagnostics na faixa 1.

```json
{
  "scdId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "status": "ACTIVE",
  "substationName": "SubstationXYZ",
  "parsedAtUtc": "2026-02-18T09:15:42Z",
  "sourceFiles": [
    "SubstationXYZ_2026.scd",
    "L90_DIG-Lab_20260129.CID",
    "MU_360_1_20260129.CID",
    "T60_DIG_20260129.CID"
  ],
  "summary": {
    "ieds": 42,
    "ieds_proxy": 161,
    "goose_streams": 237,
    "sv_streams": 80,
    "subnets": 9,
    "type": {
      "mu": 16,
      "ied": 20,
      "other": 6,
      "proxy": 161
    },
    "mms_points": {
      "total": 5370,
      "lgos": 4880,
      "lsvs": 408,
      "ltms": 82
    },
    "subscriptions": {
      "total": 2682,
      "valid": 2682,
      "invalid": 0
    },
    "diagnostics": {
      "info": 3095,
      "warning": 93,
      "error": 38
    },
    "vendors": ["GE", "GE Multilin"],
    "source_files": 42
  }
}
```

**Mapeamento dos campos para os cards:**

| Card | Campos do JSON |
|------|---------------|
| **Diagnostics SCD** | `summary.diagnostics.error` → 38, `summary.diagnostics.warning` → 93, `summary.diagnostics.info` → 3.095 |
| **IEDs** | `summary.ieds` → 42 (reais), `summary.ieds_proxy` → 161 (proxy), `summary.type` → mu:16, ied:20, other:6, proxy:161 |
| **Streams** | `summary.goose_streams` → 237, `summary.sv_streams` → 80 |
| **Subscriptions** | `summary.subscriptions.valid` → 2.682, `summary.subscriptions.invalid` → 0 |
| **MMS Points** | `summary.mms_points.total` → 5.370, `.lgos` → 4.880, `.lsvs` → 408, `.ltms` → 82 |
| **Vendors e Arquivos** | `summary.vendors` → ["GE", "GE Multilin"], `summary.source_files` → 42 |

---

### Endpoint 3 — `GET /api/v1/monitoring`

Estado do monitoramento em tempo real. Preenche a barra de monitoramento na faixa 1.

**Estado RUNNING:**

```json
{
  "enabled": true,
  "state": "RUNNING",
  "sinceUtc": "2026-02-19T13:00:00Z",
  "monitorings": [
    {
      "id": "f1a2b3c4-d5e6-7890-abcd-111111111111",
      "protocol": "SV",
      "target": "LDTM1",
      "status": "running",
      "updatedAtUtc": "2026-02-19T14:32:00Z"
    },
    {
      "id": "f1a2b3c4-d5e6-7890-abcd-222222222222",
      "protocol": "GOOSE",
      "target": "GoCB01",
      "status": "running",
      "updatedAtUtc": "2026-02-19T14:32:00Z"
    },
    {
      "id": "f1a2b3c4-d5e6-7890-abcd-333333333333",
      "protocol": "PTP",
      "target": "L90_DIG",
      "status": "running",
      "updatedAtUtc": "2026-02-19T14:32:00Z"
    }
  ],
  "stoppedAtUtc": null,
  "stoppedBy": null
}
```

**Dados para o Figma (estado RUNNING):**
- Icone: `*` (verde)
- Texto: `RUNNING desde 19/02/2026 10:00` (sinceUtc convertido para BRT UTC-3)

**Estado STOPPED (para wireframe alternativo):**

```json
{
  "enabled": false,
  "state": "STOPPED",
  "sinceUtc": "2026-02-19T18:30:00Z",
  "monitorings": [],
  "stoppedAtUtc": "2026-02-19T18:30:00Z",
  "stoppedBy": "admin@empresa.com"
}
```

**Dados para o Figma (estado STOPPED):**
- Icone: `o` (cinza)
- Texto: `STOPPED desde 19/02/2026 15:30` (sinceUtc convertido para BRT UTC-3)

---

### Endpoint 4 — `GET /api/v1/alarms?ack=false&severity={X}&pageSize=1`

Contagem de alarmes por severidade. Quatro chamadas paralelas, uma por severidade. O campo `meta.total` é o contador usado no card.

**Chamada 1 — CRITICAL:**

```json
{
  "items": [
    {
      "scdId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
      "alarm": {
        "alarmId": "b1c2d3e4-f5a6-7890-bcde-111111111111",
        "timestampUtc": "2026-02-19T17:32:01Z",
        "type": "LOSS_OF_SIGNAL",
        "severity": "CRITICAL",
        "summary": "Perda de sinal SV stream MU_360_1 LDTM1",
        "details": {
          "streamId": "LDTM1/F4800S2I4U4_1",
          "publisher": "MU_360_1",
          "subscribers": "L90_DIG, T60_DIG"
        },
        "occurrences": 3,
        "ack": false,
        "pcapId": "c1d2e3f4-a5b6-7890-cdef-111111111111"
      }
    }
  ],
  "meta": {
    "page": 1,
    "pageSize": 1,
    "total": 3
  }
}
```

**Chamada 2 — MAJOR:**

```json
{
  "items": [
    {
      "scdId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
      "alarm": {
        "alarmId": "b1c2d3e4-f5a6-7890-bcde-222222222222",
        "timestampUtc": "2026-02-19T17:30:15Z",
        "type": "JITTER_EXCEEDED",
        "severity": "MAJOR",
        "summary": "Jitter SV acima do limiar MU320E_LAB",
        "details": {
          "streamId": "LDTM1/MSVCB01",
          "measuredJitter": "2.5ms",
          "threshold": "1.0ms"
        },
        "occurrences": 1,
        "ack": false,
        "pcapId": null
      }
    }
  ],
  "meta": {
    "page": 1,
    "pageSize": 1,
    "total": 5
  }
}
```

**Chamada 3 — MEDIUM:**

```json
{
  "items": [
    {
      "scdId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
      "alarm": {
        "alarmId": "b1c2d3e4-f5a6-7890-bcde-333333333333",
        "timestampUtc": "2026-02-19T16:10:44Z",
        "type": "PACKET_LOSS",
        "severity": "MEDIUM",
        "summary": "Perda de pacotes GOOSE GCB01 MU_360_1",
        "details": {
          "controlBlock": "MU_360_1Master/LLN0$GO$GoCB01",
          "lossRate": "0.5%",
          "windowSeconds": 300
        },
        "occurrences": 12,
        "ack": false,
        "pcapId": "c1d2e3f4-a5b6-7890-cdef-333333333333"
      }
    }
  ],
  "meta": {
    "page": 1,
    "pageSize": 1,
    "total": 12
  }
}
```

**Chamada 4 — LOW:**

```json
{
  "items": [
    {
      "scdId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
      "alarm": {
        "alarmId": "b1c2d3e4-f5a6-7890-bcde-444444444444",
        "timestampUtc": "2026-02-19T15:05:22Z",
        "type": "CONF_REV_MISMATCH",
        "severity": "LOW",
        "summary": "confRev mismatch GOOSE L90_DIG GoCB02",
        "details": {
          "controlBlock": "L90_DIGMaster/LLN0$GO$GoCB02",
          "expectedConfRev": 1,
          "receivedConfRev": 2
        },
        "occurrences": 1,
        "ack": false,
        "pcapId": null
      }
    }
  ],
  "meta": {
    "page": 1,
    "pageSize": 1,
    "total": 8
  }
}
```

**Resumo para o card Alarmes Ativos (Figma):**

| Severidade | Badge | Contagem (`meta.total`) |
|---|---|---|
| CRITICAL | N (badge colorido) | 3 |
| MAJOR | ! (badge colorido) | 5 |
| MEDIUM | o (badge colorido) | 12 |
| LOW | o (badge colorido) | 8 |

---

### Resumo — Todos os valores para popular no Figma

**Header:**
- Subestacao: `SubstationXYZ`
- Monitoramento: `* RUNNING` (verde)
- Usuario: `operador@empresa.com`

**Faixa 1 — Status Critico:**
- Barra de monitoramento: `RUNNING desde 19/02/2026 10:00`
- Alarmes: CRITICAL 3, MAJOR 5, MEDIUM 12, LOW 8
- Diagnostics: Errors 38, Warnings 93, Info 3.095

**Faixa 2 — Dispositivos e Comunicacao:**
- IEDs: Reais 42, Proxy 161 | Por tipo: mu 16, ied 20, other 6, proxy 161
- Streams: GOOSE 237, SV 80
- Subscriptions: Validos 2.682, Invalidos 0

**Faixa 3 — Informacoes Contextuais:**
- MMS Points: Total 5.370, LGOS 4.880, LSVS 408, LTMS 82
- Vendors: GE, GE Multilin
- Source files: 42

---

## Referencias

- `00-navegacao-global.md` — Layout master, header, sidebar, convencoes visuais e RBAC
- `01-tela-inicial.md` — Topologia de rede (anteriormente tela inicial, agora em `/topology`)
- `02-alarmes.md` — Tela de alarmes (detalhamento dos alarmes exibidos como contadores no dashboard)
- `07-configuracao.md` — Tela de configuracao (destino do botao `[Ir para Configuracao]` no estado vazio)
- `08-autenticacao.md` — Tela de login (origem do redirect para esta tela apos autenticacao)
- `docs/extras-2026-03-16/2.2. Documentacao SCD.md` — Parser SCD, estrutura do Summary, Diagnostics, Proxy Nodes
- `docs/api/swagger-nms-v1.0.1.yaml` — Especificacao REST API (endpoints de monitoring, alarms, SCDs)

# 00 — Navegação Global, Layout Master e Convenções

## 1. Introdução

Este documento define os padrões visuais e de navegação compartilhados por todas as 8 telas do NMS (Network Monitoring System) para subestações IEC 61850. O público-alvo é o **designer UI/UX** responsável pela criação dos mockups no Figma.

Todos os wireframes de tela (documentos `01` a `08`) fazem referência a este documento como fonte de verdade para:

- Estrutura do layout master (header, sidebar, área de conteúdo)
- Componentes reutilizáveis (tabelas, drawers, filtros, paginação, estados vazios)
- Legenda de símbolos ASCII usados nos wireframes
- Regras de RBAC (permissões por role)
- Convenções de timestamp e formatação

---

## 2. Layout Master

O layout é composto por três regiões fixas — **Header**, **Sidebar** e **Área de Conteúdo** — presentes em todas as telas autenticadas. A única exceção é a **Tela 8 (Autenticação)**, que renderiza apenas a área de conteúdo centralizada, sem header nem sidebar.

```
+-----------------------------------------------------------------+
| HEADER                                                          |
| [Logo NMS]   Subestação: {nome}     Mon: * RUNNING    {user}    |
+----------+------------------------------------------------------+
| SIDEBAR  |                                                      |
|          |              ÁREA DE CONTEÚDO                        |
| 1. Top   |             (varia por tela)                         |
| 2. Alm   |                                                      |
| 3. PTP   |                                                      |
| 4. Red   |                                                      |
| 5. Com   |       +------------------------------+               |
| 6. SNM   |       |    DRAWER (painel lateral)   |               |
| 7. Cfg   |       |    (abre pela direita)       |               |
|          |       +------------------------------+               |
| [Logout] |                                                      |
+----------+------------------------------------------------------+
```

**Observações:**

- O header ocupa toda a largura superior da tela.
- A sidebar é fixa à esquerda, com largura constante e scroll próprio quando necessário.
- A área de conteúdo ocupa o espaço restante e possui scroll independente.
- O drawer, quando aberto, sobrepõe parcialmente a área de conteúdo pela direita.

---

## 3. Header

O header é uma barra horizontal fixa no topo, presente em todas as telas autenticadas.

### Componentes (da esquerda para a direita)

| Componente | Posição | Descrição | Fonte de Dados |
|---|---|---|---|
| **Logo NMS** | Canto esquerdo | Logotipo do sistema. Clique retorna à tela inicial (`/`). | Estático |
| **Subestação ativa** | Centro-esquerda | Nome da subestação sendo monitorada. Exibe o campo `substationName` do SCD ativo. | `GET /api/v1/scds?latest=true` → obter `scdId` → `GET /api/v1/scds/{scdId}/summary` → campo `substationName` |
| **Status do monitoramento** | Centro-direita | Indicador visual do estado atual do motor de monitoramento. | `GET /api/v1/monitoring` → campo `state` |
| **Usuário logado** | Direita | `displayName` extraído dos claims do JWT. | Token JWT (client-side) |

### Estados do monitoramento (campo `state`)

| Estado | Indicador Visual | Cor |
|---|---|---|
| `RUNNING` | `● RUNNING` | Verde |
| `STARTING` | `● STARTING` | Verde (pulsante) |
| `STOPPED` | `○ STOPPED` | Cinza |
| `STOPPING` | `○ STOPPING` | Cinza (pulsante) |
| `ERROR` | `⚠ ERROR` | Vermelho |

### Logout

O botão de logout fica posicionado no **rodapé da sidebar** (não no header). A ação de logout executa:

```
DELETE /api/v1/sessions/current
```

Após o logout, o usuário é redirecionado para a tela de autenticação (`/login`).

---

## 4. Sidebar (Menu de Navegação)

A sidebar é um painel vertical fixo à esquerda, presente em todas as telas autenticadas. Na **Tela 8 (Autenticação)** a sidebar não é exibida.

### Itens do menu

| # | Item | Ícone sugerido | Rota | Observação |
|---|---|---|---|---|
| 1 | Topologia | rede/grafo | `/` | Tela inicial. Exibida ao fazer login ou clicar no logo. |
| 2 | Alarmes | sino | `/alarmes` | Badge com contagem de alarmes não reconhecidos: `GET /api/v1/alarms?ack=false` → `meta.total`. |
| 3 | Sincronismo | relógio | `/sincronismo` | Sincronismo temporal (PTP). |
| 4 | Redundância | escudo | `/redundancia` | **Desabilitado/oculto** até a API HSR/PRP ser definida. |
| 5 | Comunicação | setas bidirecionais | `/comunicacao` | Dados de comunicação: GOOSE, SV, MMS. |
| 6 | SNMP | terminal/rede | `/snmp` | Monitoramento SNMP de equipamentos de rede. |
| 7 | Configuração | engrenagem | `/configuracao` | Upload de SCD, gerenciamento de MIBs, controle de monitoramento. |
| — | Logout | saída | — | Posicionado no rodapé da sidebar, separado visualmente dos itens de navegação. |

### Comportamento

- O item ativo (tela atual) deve ter destaque visual claro (fundo colorido, borda lateral ou ícone preenchido).
- A sidebar é **persistente** — sempre visível, sem opção de colapsar.
- O badge de alarmes no item "Alarmes" é atualizado periodicamente (polling ou evento) e exibe apenas a contagem de alarmes com `ack=false`.
- O item "Redundância" deve ficar oculto ou desabilitado (com tooltip explicativo) enquanto a API HSR/PRP não estiver implementada.

---

## 5. Convenções dos Wireframes ASCII

Todos os documentos de wireframe (`01` a `08`) utilizam a seguinte legenda de símbolos:

```
+------------------+   Bloco/container com borda
|  Conteúdo                                    |
+----------------------------------------------+

[Botão]                Botão clicável
[Botão (desabilitado)] Botão desabilitado por role/estado

< Link de navegação >  Link para outra tela

(...expandível...)     Área colapsável/expansível

> Ação                 Fluxo de interação (resultado de clique)

*  Estado ativo        Indicador de status (verde)
o  Estado inativo      Indicador de status (cinza)
!  Alerta/warning     Indicador de status (amarelo)
X  Erro/fechar        Indicador de erro (vermelho) / Botão fechar [X]

{Filtro v}             Dropdown/select
[____]                 Campo de texto/input
(*) / o                  Radio button (selecionado / não)
[x] / [ ]                  Checkbox (marcado / não)

< 1 2 3 ... N >        Paginação
```

---

## 6. Padrões Comuns (Componentes Reutilizáveis)

Os componentes abaixo são referenciados por múltiplas telas. O designer deve criar componentes Figma reutilizáveis para cada um deles.

---

### 6.1 Paginação

Todas as listagens da API são paginadas com os seguintes parâmetros:

| Parâmetro | Tipo | Padrão | Descrição |
|---|---|---|---|
| `page` | integer | `1` | Número da página (1-based) |
| `pageSize` | integer | `50` | Itens por página (máximo 200) |
| `sort` | string | varia | Campo de ordenação |
| `order` | string | `desc` | Direção: `asc` ou `desc` |

A resposta da API inclui o objeto `meta` com a estrutura:

```json
{
  "meta": {
    "page": 1,
    "pageSize": 50,
    "total": 237
  }
}
```

**Wireframe da paginação:**

```
< Anterior  Página 1 de 5  Próxima >   Exibindo 50 de 237
```

- O botão "Anterior" fica desabilitado na primeira página.
- O botão "Próxima" fica desabilitado na última página.
- O total de páginas é calculado no frontend: `Math.ceil(total / pageSize)`.
- A contagem "Exibindo X de Y" informa quantos itens estão visíveis na página atual versus o total.

---

### 6.2 Drawer (Painel Lateral)

O drawer é o padrão principal para exibição de detalhes. **Não são usados modais/pop-ups em nenhuma tela do sistema** — toda visualização de detalhe é feita via drawer.

**Características:**

- Abre pela **direita**, sobrepondo parcialmente a área de conteúdo.
- Largura: aproximadamente **40% da área de conteúdo**.
- Botão de fechar `[X]` no canto superior direito do drawer.
- O conteúdo por trás do drawer pode ficar levemente escurecido (overlay opcional).
- O drawer possui scroll próprio para conteúdos longos.

**Wireframe do drawer:**

```
                    +---------------------------+
                    | [X]                       |
                    | Título do Item            |
                    | ---------------------     |
                    | Campo: Valor              |
                    | Campo: Valor              |
                    |                           |
                    | [Ação Principal]          |
                    +---------------------------+
```

---

### 6.3 Tabela de Dados

Padrão para todas as listagens tabulares do sistema.

**Características:**

- Colunas com suporte a **ordenação** (indicada pelas setas `▲`/`▼`).
- Clique na linha abre o drawer com os detalhes do item.
- Paginação posicionada abaixo da tabela (conforme seção 6.1).
- Linhas podem ter destaque visual por severidade ou estado (ex: alarmes críticos em vermelho).

**Wireframe da tabela:**

```
+-----------+------------+----------+---------+
| Coluna ^  | Coluna     | Coluna v | Coluna  |
+-----------+------------+----------+---------+
| dado      | dado       | dado     | dado    |
| dado      | dado       | dado     | dado    |
+-----------+------------+----------+---------+
< Anterior  Página 1 de N  Próxima >
```

---

### 6.4 Barra de Filtros

Posicionada **acima da tabela de dados**, permite ao usuário refinar os resultados exibidos.

**Características:**

- Filtros são aplicados como query parameters na chamada à API.
- O botão "Limpar" reseta todos os filtros para o estado padrão.
- Filtros de data utilizam campos `De` / `Até` (mapeados para `fromUtc` / `toUtc` na API).

**Wireframe da barra de filtros:**

```
+-----------------------------------------------------+
| {Filtro 1 v}  {Filtro 2 v}  [De: ____] [Até: ____]  |
|                                         [Limpar]    |
+-----------------------------------------------------+
```

---

### 6.5 Estado Vazio

Exibido quando uma listagem não possui itens para mostrar (filtro sem resultados, nenhum dado cadastrado, etc.).

**Características:**

- Centralizado na área de conteúdo.
- Ícone ilustrativo relacionado ao contexto (ex: sino para alarmes, engrenagem para configuração).
- Mensagem descritiva do motivo do estado vazio.
- Link de ação sugerida quando aplicável (ex: "Faça upload de um arquivo SCD").

**Wireframe do estado vazio:**

```
+---------------------------------------------+
|                                             |
|          (ícone ilustrativo)                |
|                                             |
|     Mensagem descritiva do estado vazio     |
|     < Link de ação sugerida >               |
|                                             |
+---------------------------------------------+
```

---

### 6.6 Feedback de Erro

Mensagens de erro são exibidas **inline** (nunca em modal). O padrão é um banner colorido no topo da área de conteúdo.

**Características:**

- Ícone de alerta à esquerda.
- Mensagem de erro descritiva.
- Detalhes técnicos quando aplicável (código de erro, endpoint, etc.).
- Botão de fechar `[X]` à direita.
- Cores: vermelho para erros, amarelo para warnings.

**Wireframe do feedback de erro:**

```
+- ! ----------------------------------------- +
| Mensagem de erro descritiva                  |
| Detalhes técnicos (quando aplicável)    [X]  |
+----------------------------------------------+
```

---

### 6.7 Campos Dinâmicos (chave-valor)

Alguns campos da API possuem `additionalProperties: true`, o que significa que a estrutura de chaves não é fixa. Exemplos:

- `Alarm.details` — varia por tipo de alarme
- `MetricPoint.metrics` — pares chave-valor que diferem por protocolo
- `Scd.parsed` — conteúdo completo do SCD parseado
- `ScdProtocolDetailsResponse.data` — específico por protocolo

Esses campos devem ser renderizados como **tabela chave-valor dinâmica**, sem componentes hardcoded para chaves específicas.

**Wireframe de campos dinâmicos:**

```
+------------------+--------------------+
| Chave            | Valor              |
+------------------+--------------------+
| campo1           | valor1             |
| campo2           | valor2             |
+------------------+--------------------+
```

---

## 7. Permissões por Role (RBAC)

O sistema possui três roles de acesso, definidos pelo enum `UserRole` da API:

| Role | Descrição | Ações permitidas |
|---|---|---|
| **ADMIN** | Administrador do sistema | Todas as ações: configuração (upload de SCD, gerenciamento de MIBs), controle de monitoramento (start/stop), reconhecimento de alarmes (ACK), visualização de todas as telas. |
| **OPERATOR** | Operador da subestação | Controle de monitoramento (start/stop), reconhecimento de alarmes (ACK), visualização de todas as telas. **Sem acesso** a configuração de SCD ou MIBs. |
| **VIEWER** | Visualizador somente-leitura | Apenas visualização. Nenhuma ação de escrita (sem start/stop, sem ACK, sem configuração). |

### Regra geral de exibição

- Botões e ações **não disponíveis** para o role do usuário devem ser **ocultados** (não exibidos).
- **Exceção**: quando o usuário precisa saber que a ação existe, mas não tem permissão, o botão deve ser exibido como **desabilitado** com tooltip explicativo (ex: "Apenas administradores podem realizar esta ação").
- A decisão entre ocultar ou desabilitar deve ser avaliada caso a caso em cada wireframe de tela.

### Resumo de permissões por funcionalidade

| Funcionalidade | ADMIN | OPERATOR | VIEWER |
|---|---|---|---|
| Visualizar todas as telas | Sim | Sim | Sim |
| Start/Stop monitoramento | Sim | Sim | Nao |
| ACK alarmes | Sim | Sim | Nao |
| Upload de SCD | Sim | Nao | Nao |
| Confirmar/Rejeitar SCD | Sim | Nao | Nao |
| Gerenciar MIBs | Sim | Nao | Nao |
| Download de PCAPs | Sim | Sim | Sim |

---

## 8. Timestamps

### Convenção da API

- Todos os timestamps retornados pela API estão em **UTC** no formato **ISO 8601** (ex: `2026-02-19T14:30:00Z`).
- Os campos seguem a convenção de nomenclatura `*Utc` (ex: `createdAtUtc`, `updatedAtUtc`, `timestampUtc`, `parsedAtUtc`).

### Exibição no Frontend

- O frontend é responsável por converter os timestamps UTC para o **fuso horário local do usuário** antes de exibir.
- Formato de exibição sugerido (padrão brasileiro): `DD/MM/YYYY HH:mm:ss`
- Exemplos:
  - API retorna: `2026-02-19T14:30:00Z`
  - Exibição (BRT, UTC-3): `19/02/2026 11:30:00`

### Recomendação para o designer

- Prever espaço suficiente para o formato `DD/MM/YYYY HH:mm:ss` (19 caracteres) em todas as colunas de data/hora nas tabelas.
- Em contextos com espaço limitado (badges, cards), considerar formato abreviado: `DD/MM HH:mm`.

---

## 9. Referência entre Documentos

Este é o índice completo dos wireframes do NMS. Cada documento detalha uma tela específica e referencia este documento (`00`) para os padrões compartilhados.

| Arquivo | Tela | Rota | Descrição |
|---|---|---|---|
| `01-tela-inicial.md` | Topologia de Rede | `/` | Visualização da topologia da subestação com IEDs, redes e fluxos GOOSE/SV. Tela inicial após login. |
| `02-alarmes.md` | Alarmes | `/alarmes` | Listagem, filtros, detalhes e reconhecimento (ACK) de alarmes do sistema. |
| `03-sincronismo-temporal.md` | Sincronismo Temporal (PTP) | `/sincronismo` | Monitoramento de sincronização PTP (Precision Time Protocol) dos IEDs. |
| `04-redundancia-rede.md` | Redundância de Rede (HSR/PRP) | `/redundancia` | Monitoramento de protocolos de redundância de rede. **PENDENTE** — aguardando definição da API. |
| `05-comunicacao-dados.md` | Comunicação de Dados | `/comunicacao` | Monitoramento de comunicação GOOSE, SV e MMS entre IEDs. |
| `06-snmp.md` | SNMP | `/snmp` | Monitoramento SNMP de switches e equipamentos de rede da subestação. |
| `07-configuracao.md` | Configuração | `/configuracao` | Upload/gerenciamento de SCD, controle de monitoramento e gerenciamento de MIBs. |
| `08-autenticacao.md` | Autenticação | `/login` | Tela de login. Única tela sem header e sidebar. |

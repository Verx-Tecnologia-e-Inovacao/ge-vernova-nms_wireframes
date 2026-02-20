# Tela 7 — Configuracao

## Objetivo

Gerenciar os tres pilares de configuracao do NMS (Network Monitoring System) para subestacoes IEC 61850: **upload e ativacao de arquivos SCD/CID** (que definem a topologia e os pontos de monitoramento da subestacao), **controle do motor de monitoramento** (start/stop) e **gerenciamento de MIBs** para traducao de OIDs no monitoramento SNMP.

Esta tela possui **3 secoes internas** navegaveis por tabs horizontais:

1. **Gerenciamento de SCD** — Upload, parse, confirmacao/rejeicao e ativacao de arquivos SCD/CID
2. **Controle de Monitoramento** — Visualizacao de estado e start/stop do motor de monitoramento
3. **Gerenciamento de MIBs** — CRUD de arquivos MIB utilizados pelo agente SNMP

A subestacao monitorada possui 4 IEDs — L90_DIG (rele de protecao de linha), MU320E_LAB (merging unit), MU_360_1 (merging unit) e T60_DIG (rele de protecao de transformador) — distribuidos em 6 sub-redes (W1, W2, W4, W01, W02, W03), com protocolos GOOSE, SV, PTP, MMS, SNMP e SYSTEM.

---

## Wireframe

### Navegacao entre secoes (tabs)

As tres secoes sao acessadas por tabs horizontais posicionadas no topo da area de conteudo, logo abaixo do header global. Apenas uma tab fica ativa por vez.

```
+----------------------------------------------------------------------------------------------+
|                                                                                              |
|  +--- Header Global (ver 00-navegacao-global.md) -----------------------------------------+  |
|  |  [Logo NMS]   Subestacao: MU_360_1     Mon: * RUNNING                  {user} [Sair]   |  |
|  +----------------------------------------------------------------------------------------+  |
|                                                                                              |
|  +--- Tabs -------------------------------------------------------------------------------+  |
|  |                                                                                        |  |
|  |   [ SCD ]        [ Monitoramento ]        [ MIBs ]                                     |  |
|  |   =======                                                                              |  |
|  |   (ativo)                                                                              |  |
|  +----------------------------------------------------------------------------------------+  |
|                                                                                              |
|  +--- Conteudo da secao ativa ------------------------------------------------------------+  |
|  |                                                                                        |  |
|  |                         (renderiza conforme secao selecionada)                         |  |
|  |                                                                                        |  |
|  +----------------------------------------------------------------------------------------+  |
|                                                                                              |
+----------------------------------------------------------------------------------------------+
```

**Comportamento das tabs:**

- A tab ativa possui sublinhado/borda inferior com cor de destaque e texto em negrito
- As tabs inativas possuem texto normal e sem sublinhado
- Ao clicar em uma tab, o conteudo abaixo troca sem recarregar a pagina
- A tab padrao ao acessar `/configuracao` e **SCD**
- A URL pode refletir a tab ativa via query param (ex: `/configuracao?tab=monitoramento`)

---

### Secao 7.1 — Gerenciamento de SCD

#### Ciclo de vida do SCD

Antes de detalhar o wireframe, e importante compreender o ciclo de vida completo de um arquivo SCD/CID no sistema:

```
                +---------------+                                      
                |  UPLOADED     | ---- Upload do arquivo (POST /scds)  
                +--------+------+                                      
                         |                                             
                         | Parser automatico (async)                   
                         |                                             
                +--------+------+                                      
                |               |                                      
                v                 v                                    
          +-----------+  +--------------+                              
          |   PARSED  |  |  STALLED     | ---- Parser falhou ou timeout
          +-----+-----+  +------+-------+                              
                |               |                                      
          +-----+--------+      |                                      
          |              |      |                                      
          v            v         v                                     
   +------------+  +------------+                                      
   |   ACTIVE   |  |  REJECTED  | ---- Decisao do administrador        
   +------+-----+  +------------+                                      
          |                                                            
          | Quando outro SCD e ativado                                 
          v                                                            
   +------------+                                                      
   |  INACTIVE  |                                                      
   +------------+                                                      
                                                                       
   Qualquer status (exceto ACTIVE) > DELETED (soft delete)             
```

#### Sugestao de cores para badges de status

| Status | Cor | Indicador |
|---|---|---|
| UPLOADED | Azul | Upload realizado, aguardando parse |
| PARSED | Amarelo | Parse concluido, aguardando decisao |
| STALLED | Laranja | Parse falhou ou travou |
| ACTIVE | Verde | SCD ativo no sistema |
| INACTIVE | Cinza | SCD desativado (substituido) |
| REJECTED | Vermelho | SCD rejeitado pelo administrador |
| DELETED | Cinza escuro | SCD removido (soft delete) |

#### Wireframe — estado principal (com dados)

```
+----------------------------------------------------------------------------------------------+
|                                                                                              |
|  +--- Area de Upload (visivel apenas para ADMIN) -----------------------------------------+  |
|  |                                                                                        |  |
|  |   Tipo: {SCD v}    [Selecionar arquivo SCD/CID...]    [Upload]                         |  |
|  |                                                                                        |  |
|  |   Opcoes do dropdown Tipo:                                                             |  |
|  |     o SCD - Substation Configuration Description (arquivo completo)                    |  |
|  |     o CID - Configured IED Description (arquivo individual por IED)                    |  |
|  |                                                                                        |  |
|  +----------------------------------------------------------------------------------------+  |
|                                                                                              |
|  +--- Barra de Filtros -------------------------------------------------------------------+  |
|  |                                                                                        |  |
|  |   {Status v}          [Buscar: ____]                                      [Limpar]     |  |
|  |                                                                                        |  |
|  |   Opcoes Status:                                                                       |  |
|  |     o Todos                                                                            |  |
|  |     o UPLOADED                                                                         |  |
|  |     o PARSED                                                                           |  |
|  |     o STALLED                                                                          |  |
|  |     o ACTIVE                                                                           |  |
|  |     o INACTIVE                                                                         |  |
|  |     o REJECTED                                                                         |  |
|  |                                                                                        |  |
|  +----------------------------------------------------------------------------------------+  |
|                                                                                              |
|  +--- Tabela de SCDs -------------------------------------------------------------------- +  |
|  |                                                                                        |  |
|  |  Arquivo ^               | Status     | Criado em v      | Parseado em      | Ativ.    |  |
|  |  ------------------------+------------+------------------+------------------+----------|  |
|  |  MU_360_1_20260129.CID   | * ACTIVE   | 19/02/2026 10:00 | 19/02/2026 10:01 | 19/02    |  |
|  |                          | (verde)    |                  |                  | 10:05    |  |
|  |  ------------------------+------------+------------------+------------------+----------|  |
|  |  T60_DIG_20260129.CID    | o PARSED   | 18/02/2026 14:30 | 18/02/2026 14:31 |   -      |  |
|  |                          | (amarelo)  |                  |                  |          |  |
|  |  ------------------------+------------+------------------+------------------+----------|  |
|  |  L90_DIG-Lab_20260129.CID| ! STALLED  | 17/02/2026 09:15 |       -          |   -      |  |
|  |                          | (laranja)  |                  |                  |          |  |
|  |  ------------------------+------------+------------------+------------------+----------|  |
|  |  MU320E_20250918.CID     | o INACTIVE | 15/01/2026 08:00 | 15/01/2026 08:02 | 15/01    |  |
|  |                          | (cinza)    |                  |                  | 08:10    |  |
|  |                                                                                        |  |
|  +----------------------------------------------------------------------------------------+  |
|                                                                                              |
|  < Anterior  Pagina 1 de 1  Proxima >    Exibindo 4 de 4                                     |
|                                                                                              |
+----------------------------------------------------------------------------------------------+
```

**Comportamento da tabela:**

- Ordenacao padrao: `createdAtUtc` descendente (mais recente primeiro)
- Colunas ordenaveis: Arquivo, Criado em
- Clique na linha abre a **expansao inline** com o resumo do SCD (detalhado abaixo)
- A coluna Status exibe badge colorido conforme tabela de cores
- Campos de data nulos exibem traco (`—`)

#### Wireframe — expansao inline (detalhe do SCD selecionado)

Ao clicar em uma linha da tabela de SCDs, uma area de expansao abre abaixo da linha selecionada, exibindo o resumo parseado do SCD (`GET /scds/{scdId}/summary`).

```
+-----------------------------------------------------------------------------------------------+
|                                                                                               |
|  +--- Tabela de SCDs --------------------------------------------------------------------+    |
|  |                                                                                       |    |
|  |  Arquivo ^               | Status     | Criado em v      | Parseado em      | Ativ.  |     |
|  |  ------------------------+------------+------------------+------------------+--------|     |
|  |  MU_360_1_20260129.CID   | * ACTIVE   | 19/02/2026 10:00 | 19/02/2026 10:01 | 19/02  |     |
|  |  v (expandido)           | (verde)    |                  |                  | 10:05  |     |
|  |                                                                                       |    |
|  |  +--- Resumo do SCD ---------------------------------------------------------------+  |    |
|  |  |                                                                                 |  |    |
|  |  |  Subestacao: MU_360_1                                                           |  |    |
|  |  |  Status: * ACTIVE                                                               |  |    |
|  |  |  Parseado em: 19/02/2026 10:01                                                  |  |    |
|  |  |  Ativado em: 19/02/2026 10:05                                                   |  |    |
|  |  |  Arquivos-fonte: MU_360_1_20260129.CID                                          |  |    |
|  |  |                                                                                 |  |    |
|  |  |  +--- IEDs (4) -------------------------------------------------------------+  |  |     |
|  |  |  |                                                                           |  |  |    |
|  |  |  |  Nome IED       | Fabricante      | Pontos Monitorados                   |  |  |     |
|  |  |  |  ---------------+-----------------+--------------------------------------  |  |  |   |
|  |  |  |  L90_DIG        | GE Multilin     | 24                                   |  |  |     |
|  |  |  |  MU320E_LAB     | GE              | 18                                   |  |  |     |
|  |  |  |  MU_360_1       | GE Vernova      | 32                                   |  |  |     |
|  |  |  |  T60_DIG        | GE Vernova      | 20                                   |  |  |     |
|  |  |  |                                                                           |  |  |    |
|  |  |  +--------------------------------------------------------------------------|  |  |     |
|  |  |                                                                                 |  |    |
|  |  |  +--- Redes (6) ------------------------------------------------------------+  |  |     |
|  |  |  |                                                                           |  |  |    |
|  |  |  |  Nome Rede | Tipo          | Dispositivos Conectados                     |  |  |     |
|  |  |  |  ----------+---------------+---------------------------------------------  |  |  |   |
|  |  |  |  W1        | 8-MMS         | L90_DIG, T60_DIG                            |  |  |     |
|  |  |  |  W2        | 8-MMS         | L90_DIG, T60_DIG                            |  |  |     |
|  |  |  |  W4        | SubNetwork    | L90_DIG, T60_DIG                            |  |  |     |
|  |  |  |  W01       | Process Bus   | MU320E_LAB, MU_360_1, L90_DIG              |  |  |      |
|  |  |  |  W02       | SubNetwork    | MU320E_LAB                                  |  |  |     |
|  |  |  |  W03       | IP            | MU_360_1                                    |  |  |     |
|  |  |  |                                                                           |  |  |    |
|  |  |  +--------------------------------------------------------------------------|  |  |     |
|  |  |                                                                                 |  |    |
|  |  |  +--- Acoes (visiveis conforme status e role) -------------------------------+  |  |    |
|  |  |  |                                                                           |  |  |    |
|  |  |  |   (Para este SCD ACTIVE: nenhuma acao disponivel)                         |  |  |    |
|  |  |  |                                                                           |  |  |    |
|  |  |  +--------------------------------------------------------------------------|  |  |     |
|  |  |                                                                                 |  |    |
|  |  +---------------------------------------------------------------------------------+  |    |
|  |                                                                                       |    |
|  |  ------------------------+------------+------------------+------------------+--------|     |
|  |  T60_DIG_20260129.CID    | o PARSED   | 18/02/2026 14:30 | 18/02/2026 14:31 |   -    |     |
|  |  (...)                                                                                |    |
|  |                                                                                       |    |
|  +---------------------------------------------------------------------------------------+    |
|                                                                                               |
+-----------------------------------------------------------------------------------------------+
```

#### Wireframe — acoes por status do SCD (visivel apenas para ADMIN)

As acoes disponiveis na area de expansao variam conforme o status do SCD:

**SCD com status PARSED:**

```
+--- Acoes -------------------------------------------------------------------------+
|                                                                                   |
|   [Confirmar]    [Rejeitar]    [Excluir]                                          |
|                                                                                   |
|   Confirmar: ativa este SCD como configuracao da subestacao                       |
|   Rejeitar: requer preenchimento de motivo (campo reason)                         |
|   Excluir: soft delete do SCD                                                     |
|                                                                                   |
+-----------------------------------------------------------------------------------+
```

**SCD com status STALLED:**

```
+--- Acoes -------------------------------------------------------------------------+
|                                                                                   |
|   [Rejeitar]    [Excluir]                                                         |
|                                                                                   |
|   Rejeitar: requer preenchimento de motivo (campo reason)                         |
|   Excluir: soft delete do SCD                                                     |
|                                                                                   |
+-----------------------------------------------------------------------------------+
```

**SCD com status UPLOADED:**

```
+--- Acoes -------------------------------------------------------------------------+
|                                                                                   |
|   [Excluir]                                                                       |
|                                                                                   |
|   (Aguardando parse automatico - nenhuma outra acao disponivel)                   |
|                                                                                   |
+-----------------------------------------------------------------------------------+
```

**SCD com status ACTIVE:**

```
+--- Acoes -------------------------------------------------------------------------+
|                                                                                   |
|   (Nenhuma acao disponivel - SCD ativo nao pode ser removido nem alterado)        |
|                                                                                   |
+-----------------------------------------------------------------------------------+
```

**SCD com status INACTIVE:**

```
+--- Acoes -------------------------------------------------------------------------+
|                                                                                   |
|   [Excluir]                                                                       |
|                                                                                   |
+-----------------------------------------------------------------------------------+
```

**SCD com status REJECTED ou DELETED:**

```
+--- Acoes -------------------------------------------------------------------------+
|                                                                                   |
|   (Nenhuma acao disponivel)                                                       |
|                                                                                   |
+-----------------------------------------------------------------------------------+
```

#### Wireframe — confirmacao de rejeicao (inline)

Ao clicar em [Rejeitar], um formulario inline aparece na area de acoes solicitando o motivo:

```
+--- Rejeitar SCD ------------------------------------------------------------------+
|                                                                                   |
|   Motivo da rejeicao (obrigatorio):                                               |
|   +-----------------------------------------------------------------------+       |
|   | [____________________________________________________________]        |       |
|   +-----------------------------------------------------------------------+       |
|                                                                                   |
|   Notas adicionais (opcional):                                                    |
|   +-----------------------------------------------------------------------+       |
|   | [____________________________________________________________]        |       |
|   +-----------------------------------------------------------------------+       |
|                                                                                   |
|   [Confirmar Rejeicao]    [Cancelar]                                              |
|                                                                                   |
+-----------------------------------------------------------------------------------+
```

#### Wireframe — confirmacao de exclusao (inline)

Ao clicar em [Excluir], uma confirmacao inline substitui os botoes de acao:

```
+--- Confirmar Exclusao ------------------------------------------------------------+
|                                                                                   |
|   ! Tem certeza que deseja excluir o SCD "T60_DIG_20260129.CID"?                  |
|      Esta acao nao pode ser desfeita.                                             |
|                                                                                   |
|   [Sim, Excluir]    [Cancelar]                                                    |
|                                                                                   |
+-----------------------------------------------------------------------------------+
```

#### Wireframe — estado vazio (nenhum SCD)

```
+----------------------------------------------------------------------------------------------+
|                                                                                              |
|  +--- Area de Upload --------------------------------------------------------------------+   |
|  |   Tipo: {SCD v}    [Selecionar arquivo SCD/CID...]    [Upload]                        |   |
|  +---------------------------------------------------------------------------------------+   |
|                                                                                              |
|                                                                                              |
|                            +----------------------------------+                              |
|                            |                                  |                              |
|                            |    (icone: arquivo/engrenagem)   |                              |
|                            |                                  |                              |
|                            |  Nenhum SCD cadastrado           |                              |
|                            |                                  |                              |
|                            |  Faca upload de um arquivo       |                              |
|                            |  SCD ou CID para configurar      |                              |
|                            |  a subestacao.                   |                              |
|                            |                                  |                              |
|                            +----------------------------------+                              |
|                                                                                              |
+----------------------------------------------------------------------------------------------+
```

#### Wireframe — estado vazio (role VIEWER/OPERATOR, sem permissao de upload)

```
+----------------------------------------------------------------------------------------------+
|                                                                                              |
|   (Area de upload oculta para VIEWER e OPERATOR)                                             |
|                                                                                              |
|                            +----------------------------------+                              |
|                            |                                  |                              |
|                            |    (icone: arquivo/engrenagem)   |                              |
|                            |                                  |                              |
|                            |  Nenhum SCD cadastrado           |                              |
|                            |                                  |                              |
|                            |  Solicite a um administrador     |                              |
|                            |  o upload de um arquivo SCD.     |                              |
|                            |                                  |                              |
|                            +----------------------------------+                              |
|                                                                                              |
+----------------------------------------------------------------------------------------------+
```

---

### Secao 7.2 — Controle de Monitoramento

#### Maquina de estados do monitoramento

```
+----------+            +----------+           +----------+ 
| STOPPED  | -------->  | STARTING | --------> | RUNNING  | 
| (cinza)  |  Iniciar   | (amarelo |  sucesso  | (verde)  | 
|          |            |  pulsa)  |           |          | 
+----------+            +-----+----+           +-----+----+ 
     ^                        |                      |      
     |                        | falha                | Parar
     |                        v                      v      
     |                  +----------+           +----------+ 
     |                  |  ERROR   |           | STOPPING | 
     |                  |(vermelho)|           | (amarelo | 
     |                  +----------+           |  pulsa)  | 
     |                                         +-----+----+ 
     |                                               |      
     +-----------------------------------------------+      
                        sucesso                             
```

#### Wireframe — estado RUNNING

```
+----------------------------------------------------------------------------------------------+
|                                                                                              |
|  +--- Card de Status -----------------------------------------------------------------+      |
|  |                                                                                    |      |
|  |   +----------------------------------------------------------------------------+   |      |
|  |   |                                                                            |   |      |
|  |   |   Estado do Monitoramento                                                  |   |      |
|  |   |                                                                            |   |      |
|  |   |   * RUNNING                                                                |   |      |
|  |   |   (indicador verde, ponto solido)                                          |   |      |
|  |   |                                                                            |   |      |
|  |   |   Ativo desde: 19/02/2026 10:05                                            |   |      |
|  |   |                                                                            |   |      |
|  |   |   [Parar Monitoramento]                                                    |   |      |
|  |   |   (visivel para ADMIN e OPERATOR; oculto para VIEWER)                      |   |      |
|  |   |                                                                            |   |      |
|  |   +----------------------------------------------------------------------------+   |      |
|  |                                                                                    |      |
|  +------------------------------------------------------------------------------------+      |
|                                                                                              |
|  +--- Tabela de Itens Monitorados ----------------------------------------------------+      |
|  |                                                                                    |      |
|  |   Protocolo   | Alvo                         | Status    | Atualizado em           |      |
|  |   ------------+------------------------------+-----------+-------------------------+      |
|  |   SV          | MU_360_1/LDTM1/F4800S2I4U4   | * OK      | 19/02/2026 14:32        |      |
|  |   SV          | MU320E_LAB/LDInst1/svDataSet | * OK      | 19/02/2026 14:32        |      |
|  |   GOOSE       | L90_DIG/GCB01                | * OK      | 19/02/2026 14:31        |      |
|  |   GOOSE       | T60_DIG/GCB01                | * OK      | 19/02/2026 14:31        |      |
|  |   PTP         | MU_360_1/PTP_Clock           | * OK      | 19/02/2026 14:30        |      |
|  |   MMS         | L90_DIG/MMS_Server           | * OK      | 19/02/2026 14:30        |      |
|  |   MMS         | T60_DIG/MMS_Server           | * OK      | 19/02/2026 14:30        |      |
|  |   SNMP        | switch-w1/161                | ! WARN    | 19/02/2026 14:29        |      |
|  |   SYSTEM      | nms-agent/health             | * OK      | 19/02/2026 14:32        |      |
|  |                                                                                    |      |
|  +------------------------------------------------------------------------------------+      |
|                                                                                              |
|  < Anterior  Pagina 1 de 1  Proxima >    Exibindo 9 de 9                                     |
|                                                                                              |
+----------------------------------------------------------------------------------------------+
```

**Legenda de status dos itens monitorados:**

| Status | Indicador | Cor |
|---|---|---|
| OK | ● | Verde |
| WARN | ⚠ | Amarelo |
| ERROR | ✕ | Vermelho |
| UNKNOWN | ○ | Cinza |

#### Wireframe — estado STOPPED

```
+-----------------------------------------------------------------------------------------------+
|                                                                                               |
|  +--- Card de Status -----------------------------------------------------------------+       |
|  |                                                                                    |       |
|  |   +----------------------------------------------------------------------------+   |       |
|  |   |                                                                            |   |       |
|  |   |   Estado do Monitoramento                                                  |   |       |
|  |   |                                                                            |   |       |
|  |   |   o STOPPED                                                                |   |       |
|  |   |   (indicador cinza, ponto vazio)                                           |   |       |
|  |   |                                                                            |   |       |
|  |   |   Parado desde: 19/02/2026 09:00                                           |   |       |
|  |   |   Parado por: admin@empresa.com                                            |   |       |
|  |   |                                                                            |   |       |
|  |   |   [Iniciar Monitoramento]                                                  |   |       |
|  |   |   (visivel para ADMIN e OPERATOR; oculto para VIEWER)                      |   |       |
|  |   |                                                                            |   |       |
|  |   +----------------------------------------------------------------------------+   |       |
|  |                                                                                    |       |
|  +------------------------------------------------------------------------------------+       |
|                                                                                               |
|  +--- Tabela de Itens Monitorados ----------------------------------------------------+       |
|  |                                                                                    |       |
|  |                            (tabela vazia ou exibindo                               |       |
|  |                             ultimo estado conhecido)                               |       |
|  |                                                                                    |       |
|  |   Protocolo   | Alvo                         | Status    | Atualizado em           |       |
|  |   ------------+------------------------------+-----------+-------------------------+       |
|  |   SV          | MU_360_1/LDTM1/F4800S2I4U4   | o UNKNOWN | 19/02/2026 09:00        |       |
|  |   SV          | MU320E_LAB/LDInst1/svDataSet | o UNKNOWN | 19/02/2026 09:00        |       |
|  |   (... demais itens com status UNKNOWN ...)                                        |       |
|  |                                                                                    |       |
|  +------------------------------------------------------------------------------------+       |
|                                                                                               |
+-----------------------------------------------------------------------------------------------+
```

#### Wireframe — estado STARTING/STOPPING (transicao)

```
+--- Card de Status -----------------------------------------------------------------+
|                                                                                    |
|   +----------------------------------------------------------------------------+   |
|   |                                                                            |   |
|   |   Estado do Monitoramento                                                  |   |
|   |                                                                            |   |
|   |   o STARTING...                                                            |   |
|   |   (indicador amarelo pulsante, com spinner)                                |   |
|   |                                                                            |   |
|   |   [Iniciar Monitoramento (desabilitado)]                                   |   |
|   |   (botao desabilitado durante transicao)                                   |   |
|   |                                                                            |   |
|   +----------------------------------------------------------------------------+   |
|                                                                                    |
+------------------------------------------------------------------------------------+
```

#### Wireframe — estado ERROR

```
+--- Card de Status -----------------------------------------------------------------+
|                                                                                    |
|   +----------------------------------------------------------------------------+   |
|   |                                                                            |   |
|   |   Estado do Monitoramento                                                  |   |
|   |                                                                            |   |
|   |   ! ERROR                                                                  |   |
|   |   (indicador vermelho)                                                     |   |
|   |                                                                            |   |
|   |   Em erro desde: 19/02/2026 14:15                                          |   |
|   |                                                                            |   |
|   |   [Iniciar Monitoramento]                                                  |   |
|   |   (permite tentar reiniciar)                                               |   |
|   |                                                                            |   |
|   +----------------------------------------------------------------------------+   |
|                                                                                    |
+------------------------------------------------------------------------------------+
```

---

### Secao 7.3 — Gerenciamento de MIBs

#### Wireframe — estado principal (com dados)

```
+-----------------------------------------------------------------------------------------------+
|                                                                                               |
|  +--- Barra de Acoes (visivel apenas para ADMIN) -----------------------------------------+   |
|  |                                                                                        |   |
|  |   Buscar: [____________________________]                             [Nova MIB]        |   |
|  |                                                                                        |   |
|  +----------------------------------------------------------------------------------------+   |
|                                                                                               |
|  +--- Barra de Busca (visivel para OPERATOR e VIEWER - sem botao Nova MIB) ---------------+   |
|  |                                                                                        |   |
|  |   Buscar: [____________________________]                                               |   |
|  |                                                                                        |   |
|  +----------------------------------------------------------------------------------------+   |
|                                                                                               |
|  +--- Tabela de MIBs ---------------------------------------------------------------- +       |
|  |                                                                                    |       |
|  |   Nome ^             | Versao | Descricao                    | Criado em  | Acoes  |       |
|  |   -------------------+--------+------------------------------+------------+--------|       |
|  |   IF-MIB             | 2.2    | Interfaces de rede           | 15/01/2026 | [E][X] |       |
|  |                      |        |                              |    08:00   |        |       |
|  |   -------------------+--------+------------------------------+------------+--------|       |
|  |   SNMPv2-MIB         | 1.0    | Entidades SNMP v2            | 15/01/2026 | [E][X] |       |
|  |                      |        |                              |    08:01   |        |       |
|  |   -------------------+--------+------------------------------+------------+--------|       |
|  |   ENTITY-MIB         | 3.0    | Entidades fisicas do equip.  | 16/01/2026 | [E][X] |       |
|  |                      |        |                              |    09:30   |        |       |
|  |   -------------------+--------+------------------------------+------------+--------|       |
|  |   BRIDGE-MIB         | -      | Switching e bridging         | 18/02/2026 | [E][X] |       |
|  |                      |        |                              |    14:00   |        |       |
|  |                                                                                    |       |
|  |   Legenda Acoes: [E] = Editar   [X] = Excluir                                      |       |
|  |   (Coluna Acoes visivel apenas para ADMIN; oculta para OPERATOR e VIEWER)          |       |
|  |                                                                                    |       |
|  +------------------------------------------------------------------------------------+       |
|                                                                                               |
|  < Anterior  Pagina 1 de 1  Proxima >    Exibindo 4 de 4                                      |
|                                                                                               |
+-----------------------------------------------------------------------------------------------+
```

**Comportamento da tabela:**

- Ordenacao padrao: `createdAtUtc` descendente
- Colunas ordenaveis: Nome, Criado em
- Campo de busca (`q`) filtra por nome da MIB
- Coluna Versao exibe traco (`—`) quando `version` e `null`
- Coluna Descricao trunca texto longo com ellipsis
- Coluna Acoes e visivel apenas para role ADMIN

#### Wireframe — formulario de nova MIB (expansao inline)

Ao clicar em [Nova MIB], um formulario inline aparece acima da tabela:

```
+--- Nova MIB --------------------------------------------------------------------------+
|                                                                                       |
|   Nome (obrigatorio):                                                                 |
|   [____________________________________________________________]                      |
|                                                                                       |
|   Versao (opcional):                                                                  |
|   [____________________________]                                                      |
|                                                                                       |
|   Descricao (opcional):                                                               |
|   [____________________________________________________________]                      |
|                                                                                       |
|   Conteudo da MIB (obrigatorio):                                                      |
|   +-------------------------------------------------------------------------------+   |
|   |                                                                               |   |
|   |  (textarea com altura expandida - aceita o conteudo textual do arquivo MIB)   |   |
|   |  (aproximadamente 10-15 linhas visiveis, com scroll interno)                  |   |
|   |                                                                               |   |
|   |  Exemplo:                                                                     |   |
|   |  IF-MIB DEFINITIONS ::= BEGIN                                                 |   |
|   |  IMPORTS                                                                      |   |
|   |      MODULE-IDENTITY, OBJECT-TYPE, Counter32, ...                             |   |
|   |  ...                                                                          |   |
|   |                                                                               |   |
|   +-------------------------------------------------------------------------------+   |
|                                                                                       |
|   [Salvar]    [Cancelar]                                                              |
|                                                                                       |
+---------------------------------------------------------------------------------------+
```

#### Wireframe — formulario de edicao (expansao inline abaixo da linha)

Ao clicar em [Ed] (Editar) em uma linha, o formulario aparece inline abaixo da linha selecionada, pre-populado com os valores atuais:

```
+--- Editar MIB: IF-MIB ----------------------------------------------------------------+
|                                                                                       |
|   Nome:                                                                               |
|   [IF-MIB_________________________________________________]                           |
|                                                                                       |
|   Versao:                                                                             |
|   [2.2_________________________]                                                      |
|                                                                                       |
|   Descricao:                                                                          |
|   [Interfaces de rede___________________________________________]                     |
|                                                                                       |
|   Conteudo da MIB:                                                                    |
|   +-------------------------------------------------------------------------------+   |
|   |  IF-MIB DEFINITIONS ::= BEGIN                                                 |   |
|   |  IMPORTS                                                                      |   |
|   |      MODULE-IDENTITY, OBJECT-TYPE, Counter32, ...                             |   |
|   |  (conteudo atual da MIB carregado)                                            |   |
|   +-------------------------------------------------------------------------------+   |
|                                                                                       |
|   [Salvar Alteracoes]    [Cancelar]                                                   |
|                                                                                       |
+---------------------------------------------------------------------------------------+
```

#### Wireframe — confirmacao de exclusao (inline)

Ao clicar em [Ex] (Excluir), uma confirmacao inline substitui a linha de acoes:

```
+--- Confirmar Exclusao -----------------------------------------------------------------+
|                                                                                        |
|   ! Tem certeza que deseja excluir a MIB "IF-MIB"?                                     |
|      Esta acao nao pode ser desfeita.                                                  |
|                                                                                        |
|   [Sim, Excluir]    [Cancelar]                                                         |
|                                                                                        |
+----------------------------------------------------------------------------------------+
```

#### Wireframe — estado vazio (nenhuma MIB)

```
+-----------------------------------------------------------------------------------------------+
|                                                                                               |
|  +--- Barra de Acoes ---------------------------------------------------------------- +       |
|  |   Buscar: [____________________________]                         [Nova MIB]        |       |
|  +------------------------------------------------------------------------------------+       |
|                                                                                               |
|                            +----------------------------------+                               |
|                            |                                  |                               |
|                            |    (icone: terminal/rede)        |                               |
|                            |                                  |                               |
|                            |  Nenhuma MIB cadastrada          |                               |
|                            |                                  |                               |
|                            |  Adicione MIBs para habilitar    |                               |
|                            |  a traducao de OIDs no           |                               |
|                            |  monitoramento SNMP.             |                               |
|                            |                                  |                               |
|                            +----------------------------------+                               |
|                                                                                               |
+-----------------------------------------------------------------------------------------------+
```

---

## Componentes

| Componente | Fonte de dados | Formato de exibicao |
|---|---|---|
| **Tabs de navegacao** | Estado local (client-side) | 3 tabs: SCD, Monitoramento, MIBs. Tab ativa com sublinhado e texto em negrito |
| **Area de upload SCD** | Input file + dropdown | Dropdown para tipo (SCD/CID) + seletor de arquivo + botao Upload |
| **Dropdown Tipo SCD** | Valores fixos: SCD, CID | Select com opcoes pre-definidas |
| **Dropdown Status SCD** | `ScdStatus` enum | Select com opcao "Todos" como padrao; cada opcao com cor indicativa |
| **Tabela de SCDs** | `GET /api/v1/scds` | Tabela com linhas expandiveis |
| **Coluna Arquivo** | `scd.fileName` | Texto; nome do arquivo original |
| **Coluna Status SCD** | `scd.status` | Badge colorido: UPLOADED (azul), PARSED (amarelo), STALLED (laranja), ACTIVE (verde), INACTIVE (cinza), REJECTED (vermelho), DELETED (cinza escuro) |
| **Coluna Criado em** | `scd.createdAtUtc` | Formato local: `DD/MM/AAAA HH:mm` (convertido de UTC) |
| **Coluna Parseado em** | `scd.parsedAtUtc` | Formato local: `DD/MM/AAAA HH:mm` ou `—` se null |
| **Coluna Ativado em** | `scd.activatedAtUtc` | Formato local: `DD/MM/AAAA HH:mm` ou `—` se null |
| **Expansao de resumo SCD** | `GET /api/v1/scds/{scdId}/summary` → `ScdSummaryResponse` | Tabelas de IEDs e redes + botoes de acao |
| **Tabela de IEDs** | `summary.ieds[]` | Colunas: Nome IED, Fabricante, Pontos Monitorados |
| **Tabela de Redes** | `summary.networks[]` | Colunas: Nome Rede, Tipo, Dispositivos Conectados |
| **Botao Confirmar** | `POST /api/v1/scds/{scdId}/confirmations` | Botao primario; visivel para ADMIN quando status=PARSED |
| **Botao Rejeitar** | `POST /api/v1/scds/{scdId}/confirmations` | Botao de alerta; visivel para ADMIN quando status=PARSED ou STALLED. Exige campo reason |
| **Botao Excluir SCD** | `DELETE /api/v1/scds/{scdId}` | Botao destrutivo; visivel para ADMIN quando status != ACTIVE |
| **Formulario de rejeicao** | Input do usuario | Campos: reason (obrigatorio), notes (opcional) |
| **Card de status monitoramento** | `GET /api/v1/monitoring` → `MonitoringStatusResponse` | Card com indicador visual de estado, data desde, e botao de acao |
| **Indicador de estado** | `monitoring.state` | ● RUNNING (verde), ○ STOPPED (cinza), ⚠ ERROR (vermelho), ○ STARTING (amarelo pulsante), ○ STOPPING (amarelo pulsante) |
| **Botao Iniciar** | `PATCH /api/v1/monitoring` `{enabled: true}` | Visivel quando STOPPED ou ERROR; oculto para VIEWER |
| **Botao Parar** | `PATCH /api/v1/monitoring` `{enabled: false}` | Visivel quando RUNNING; oculto para VIEWER |
| **Tabela de itens monitorados** | `monitoring.monitorings[]` | Colunas: Protocolo, Alvo, Status, Atualizado em |
| **Badge de protocolo** | `item.protocol` | Badge com texto: SV, GOOSE, PTP, MMS, SNMP, SYSTEM |
| **Campo de busca MIBs** | Query param `q` | Input de texto que filtra por nome da MIB |
| **Tabela de MIBs** | `GET /api/v1/mibs` | Tabela com colunas: Nome, Versao, Descricao, Criado em, Acoes |
| **Botao Nova MIB** | — | Abre formulario inline acima da tabela; visivel apenas para ADMIN |
| **Formulario de MIB** | Input do usuario | Campos: name (obrigatorio), version, description, content (textarea, obrigatorio) |
| **Botao Editar MIB** | `PUT /api/v1/mibs/{mibId}` | Abre formulario inline pre-populado; visivel apenas para ADMIN |
| **Botao Excluir MIB** | `DELETE /api/v1/mibs/{mibId}` | Confirmacao inline; visivel apenas para ADMIN |
| **Paginacao** | `meta` de cada listagem | Padrao conforme `00-navegacao-global.md` secao 6.1 |
| **Banner de erro** | HTTP 4xx/5xx ou erro de rede | Banner vermelho inline no topo da area de conteudo |
| **Estado vazio** | `meta.total == 0` | Ilustracao centralizada + mensagem descritiva conforme `00-navegacao-global.md` secao 6.5 |

---

## Dados e Endpoints

### SCD

| Endpoint | Metodo | Uso na tela | Campos utilizados | Exemplo de chamada |
|---|---|---|---|---|
| `/api/v1/scds` | GET | Listar SCDs (paginado, filtrado) | Query: `latest`, `status`, `page`, `pageSize`, `sort`, `order` | `GET /api/v1/scds?page=1&pageSize=50&order=desc&sort=createdAtUtc` |
| `/api/v1/scds` | GET | Filtrar por status | Query: `status=PARSED` | `GET /api/v1/scds?status=PARSED&page=1&pageSize=50` |
| `/api/v1/scds` | POST | Upload de arquivo SCD/CID | Body: multipart/form-data (`file` + `scdType`: SCD ou CID) | `POST /api/v1/scds` com `Content-Type: multipart/form-data` |
| `/api/v1/scds/{scdId}` | GET | Obter SCD por ID | Response: `Scd` | `GET /api/v1/scds/a1b2c3d4-...` |
| `/api/v1/scds/{scdId}/summary` | GET | Obter resumo do SCD (IEDs, redes) | Response: `ScdSummaryResponse` | `GET /api/v1/scds/a1b2c3d4-.../summary` |
| `/api/v1/scds/{scdId}/confirmations` | POST | Confirmar ou rejeitar SCD | Body: `{ decision: "CONFIRM"\|"REJECT", performedBy, notes?, reason? }` | `POST /api/v1/scds/a1b2c3d4-.../confirmations` |
| `/api/v1/scds/{scdId}` | DELETE | Excluir SCD (soft delete) | — | `DELETE /api/v1/scds/a1b2c3d4-...` |

**Schemas SCD:**

| Schema | Campos | Observacoes |
|---|---|---|
| `Scd` | `scdId` (uuid), `fileName` (string), `sha256` (string), `status` (ScdStatus), `createdAtUtc` (datetime), `parsedAtUtc` (datetime\|null), `activatedAtUtc` (datetime\|null), `activatedBy` (string\|null), `rejectedAtUtc` (datetime\|null), `rejectedBy` (string\|null), `parsed` (object\|null) | Campo `parsed` tem `additionalProperties: true` — nao exibido diretamente na UI |
| `ScdStatus` | UPLOADED, PARSED, STALLED, ACTIVE, INACTIVE, REJECTED, DELETED | Enum para badges e filtros |
| `ScdUploadResponse` | `scdId` (uuid), `createdAtUtc` (datetime), `fileName` (string), `sha256` (string), `status` (ScdStatus) | Retornado apos upload |
| `ScdSummaryResponse` | `scdId` (uuid), `status` (ScdStatus), `substationName` (string), `parsedAtUtc` (datetime), `sourceFiles` (string[]), `ieds` (ScdSummaryIed[]), `networks` (ScdSummaryNetwork[]) | Exibido na expansao inline |
| `ScdSummaryIed` | `iedName` (string), `vendor` (string), `monitoredDataPointsCount` (integer) | Linha na tabela de IEDs |
| `ScdSummaryNetwork` | `name` (string), `type` (string), `connectedDevices` (string[]) | Linha na tabela de redes |
| `ScdConfirmation` | `decision` ("CONFIRM"\|"REJECT"), `performedBy` (string), `notes` (string\|null), `reason` (string, obrigatorio quando REJECT) | Body do POST confirmations |

**Erros esperados (SCD):**

| Codigo | Endpoint | Situacao |
|---|---|---|
| 409 | `POST /scds` | Arquivo com mesmo hash SHA-256 ja existe |
| 409 | `POST /scds/{id}/confirmations` | SCD nao esta no status correto para confirmacao/rejeicao |
| 409 | `DELETE /scds/{id}` | SCD ativo nao pode ser removido |

### Monitoramento

| Endpoint | Metodo | Uso na tela | Campos utilizados | Exemplo de chamada |
|---|---|---|---|---|
| `/api/v1/monitoring` | GET | Obter status atual do monitoramento | Response: `MonitoringStatusResponse` | `GET /api/v1/monitoring` |
| `/api/v1/monitoring` | PATCH | Iniciar ou parar monitoramento | Body: `{ enabled: true\|false }` | `PATCH /api/v1/monitoring` |

**Schemas Monitoramento:**

| Schema | Campos | Observacoes |
|---|---|---|
| `MonitoringStatusResponse` | `enabled` (boolean), `state` (MonitoringState), `sinceUtc` (datetime), `monitorings` (MonitoringItem[]), `stoppedAtUtc` (datetime\|null), `stoppedBy` (string\|null) | Resposta principal |
| `MonitoringState` | STOPPED, STARTING, RUNNING, STOPPING, ERROR | Enum para indicador visual |
| `MonitoringItem` | `id` (string), `protocol` (ProtocolEnum), `target` (string), `status` (string), `updatedAtUtc` (datetime) | Linha na tabela de itens monitorados |
| `ProtocolEnum` | SV, GOOSE, PTP, MMS, SNMP, SYSTEM | Badge de protocolo |
| `MonitoringUpdateRequest` | `enabled` (boolean) | Body do PATCH |

**Erros esperados (Monitoramento):**

| Codigo | Endpoint | Situacao |
|---|---|---|
| 409 | `PATCH /monitoring` | Conflito de estado (ex: tentar iniciar quando ja esta STARTING) |

### MIBs

| Endpoint | Metodo | Uso na tela | Campos utilizados | Exemplo de chamada |
|---|---|---|---|---|
| `/api/v1/mibs` | GET | Listar MIBs (paginado, com busca) | Query: `q`, `page`, `pageSize`, `sort`, `order` | `GET /api/v1/mibs?page=1&pageSize=50&order=desc&sort=createdAtUtc` |
| `/api/v1/mibs` | GET | Buscar MIBs por nome | Query: `q=IF-MIB` | `GET /api/v1/mibs?q=IF-MIB&page=1&pageSize=50` |
| `/api/v1/mibs` | POST | Criar nova MIB | Body: `{ name, version?, description?, content }` | `POST /api/v1/mibs` |
| `/api/v1/mibs/{mibId}` | GET | Obter MIB por ID | Response: `Mib` | `GET /api/v1/mibs/a1b2c3d4-...` |
| `/api/v1/mibs/{mibId}` | PUT | Atualizar MIB | Body: `{ name?, version?, description?, content? }` | `PUT /api/v1/mibs/a1b2c3d4-...` |
| `/api/v1/mibs/{mibId}` | DELETE | Excluir MIB | — | `DELETE /api/v1/mibs/a1b2c3d4-...` |

**Schemas MIBs:**

| Schema | Campos | Observacoes |
|---|---|---|
| `Mib` | `mibId` (uuid), `name` (string), `version` (string\|null), `description` (string\|null), `content` (string), `createdAtUtc` (datetime), `updatedAtUtc` (datetime) | Conteudo e o texto completo da MIB |
| `CreateMibRequest` | `name` (string, obrigatorio), `version` (string\|null), `description` (string\|null), `content` (string, obrigatorio) | Body do POST |
| `UpdateMibRequest` | `name` (string), `version` (string\|null), `description` (string\|null), `content` (string) | Body do PUT |

**Erros esperados (MIBs):**

| Codigo | Endpoint | Situacao |
|---|---|---|
| 409 | `POST /mibs` | MIB com mesmo nome ja existe |

---

## Fluxos de Interacao

### 1. Upload de SCD

```
Usuario (ADMIN) acessa a tab SCD
    |
    v
Seleciona o tipo do arquivo no dropdown (SCD ou CID)
    |
    v
Clica em [Selecionar arquivo SCD/CID...] e escolhe um arquivo .scd ou .cid
    |
    v
Clica em [Upload]
    |
    v
> POST /api/v1/scds (multipart/form-data: file + scdType)
> Botao exibe spinner, campo de arquivo desabilitado
    |
    +-- Sucesso (201)
    |       |
    |       v
    |   Nova linha aparece na tabela com status UPLOADED (badge azul)
    |       |
    |       v
    |   Parser roda automaticamente em background (async)
    |       |
    |       +-- Parse bem-sucedido > status muda para PARSED (badge amarelo)
    |       |                         (polling ou WebSocket para detectar mudanca)
    |       |
    |       +-- Parse falhou > status muda para STALLED (badge laranja)
    |
    +-- Erro 409 (arquivo duplicado - mesmo SHA-256)
    |       |
    |       v
    |   Banner de erro inline: "Este arquivo ja foi enviado anteriormente."
    |
    +-- Erro 500 / erro de rede
            |
            v
        Banner de erro inline: "Erro ao enviar arquivo. Tente novamente."
        Botao reabilitado
```

### 2. Confirmar SCD (ativar)

```
Usuario (ADMIN) clica em uma linha com status PARSED na tabela
    |
    v
Expansao inline abre mostrando resumo do SCD
> GET /api/v1/scds/{scdId}/summary
    |
    v
Usuario analisa IEDs, redes e pontos de monitoramento
    |
    v
Clica em [Confirmar]
    |
    v
> POST /api/v1/scds/{scdId}/confirmations
  Body: { decision: "CONFIRM", performedBy: "{usuario_logado}" }
    |
    +-- Sucesso (200)
    |       |
    |       v
    |   Status do SCD muda para ACTIVE (badge verde)
    |       |
    |       v
    |   Se havia outro SCD ACTIVE, ele muda para INACTIVE (badge cinza)
    |       |
    |       v
    |   Feedback inline: "SCD ativado com sucesso." (badge verde, 5 segundos)
    |       |
    |       v
    |   Header global atualiza o nome da subestacao
    |
    +-- Erro 409 (estado invalido)
    |       |
    |       v
    |   Banner de erro: "SCD nao esta em estado valido para confirmacao."
    |   Recarrega dados do SCD
    |
    +-- Erro 500 / erro de rede
            |
            v
        Banner de erro: "Erro ao confirmar SCD. Tente novamente."
```

### 3. Rejeitar SCD

```
Usuario (ADMIN) clica em uma linha com status PARSED ou STALLED
    |
    v
Expansao inline abre mostrando resumo (se PARSED) ou dados basicos (se STALLED)
    |
    v
Clica em [Rejeitar]
    |
    v
Formulario inline de rejeicao aparece na area de acoes
    |
    v
Usuario preenche "Motivo da rejeicao" (obrigatorio) e "Notas" (opcional)
    |
    v
Clica em [Confirmar Rejeicao]
    |
    v
> POST /api/v1/scds/{scdId}/confirmations
  Body: { decision: "REJECT", performedBy: "{usuario_logado}", reason: "...", notes: "..." }
    |
    +-- Sucesso (200)
    |       |
    |       v
    |   Status do SCD muda para REJECTED (badge vermelho)
    |   Feedback inline: "SCD rejeitado." (badge amarelo, 5 segundos)
    |
    +-- Erro 409 (estado invalido)
    |       |
    |       v
    |   Banner de erro: "SCD nao esta em estado valido para rejeicao."
    |
    +-- Erro 500 / erro de rede
            |
            v
        Banner de erro: "Erro ao rejeitar SCD. Tente novamente."
```

### 4. Excluir SCD

```
Usuario (ADMIN) clica em [Excluir] na expansao de um SCD nao-ACTIVE
    |
    v
Confirmacao inline: "Tem certeza que deseja excluir o SCD '{fileName}'?"
    |
    +-- Clica [Sim, Excluir]
    |       |
    |       v
    |   > DELETE /api/v1/scds/{scdId}
    |       |
    |       +-- Sucesso (200/204)
    |       |       |
    |       |       v
    |       |   Linha removida da tabela (ou status muda para DELETED)
    |       |   Feedback inline: "SCD excluido." (5 segundos)
    |       |
    |       +-- Erro 409 (SCD ativo)
    |       |       |
    |       |       v
    |       |   Banner de erro: "SCD ativo nao pode ser removido."
    |       |
    |       +-- Erro 500 / erro de rede
    |               |
    |               v
    |           Banner de erro: "Erro ao excluir SCD. Tente novamente."
    |
    +-- Clica [Cancelar]
            |
            v
        Confirmacao fechada, acoes voltam ao estado original
```

### 5. Iniciar monitoramento

```
Usuario (ADMIN ou OPERATOR) acessa a tab Monitoramento
    |
    v
Card de status exibe o STOPPED
    |
    v
Clica em [Iniciar Monitoramento]
    |
    v
> PATCH /api/v1/monitoring { enabled: true }
> Botao entra em estado de loading (spinner)
    |
    +-- Sucesso (200)
    |       |
    |       v
    |   Card atualiza para o STARTING (indicador amarelo pulsante)
    |   Botao fica desabilitado durante transicao
    |       |
    |       v (polling do GET /monitoring detecta mudanca)
    |
    |   Estado muda para * RUNNING (indicador verde)
    |   Botao muda para [Parar Monitoramento]
    |   Header global atualiza: Mon: * RUNNING
    |   Tabela de itens monitorados popula com dados
    |
    +-- Erro 409 (conflito de estado)
    |       |
    |       v
    |   Banner de erro: "Monitoramento ja esta em transicao."
    |   Recarrega estado via GET /monitoring
    |
    +-- Erro 500 / erro de rede
            |
            v
        Banner de erro: "Erro ao iniciar monitoramento. Tente novamente."
        Botao reabilitado
```

### 6. Parar monitoramento

```
Usuario (ADMIN ou OPERATOR) visualiza card com * RUNNING
    |
    v
Clica em [Parar Monitoramento]
    |
    v
> PATCH /api/v1/monitoring { enabled: false }
> Botao entra em estado de loading (spinner)
    |
    +-- Sucesso (200)
    |       |
    |       v
    |   Card atualiza para o STOPPING (indicador amarelo pulsante)
    |   Botao fica desabilitado durante transicao
    |       |
    |       v (polling do GET /monitoring detecta mudanca)
    |
    |   Estado muda para o STOPPED (indicador cinza)
    |   Botao muda para [Iniciar Monitoramento]
    |   Header global atualiza: Mon: o STOPPED
    |   Exibe "Parado por: {usuario}" e "Parado desde: {data}"
    |
    +-- Erro 409 (conflito de estado)
    |       |
    |       v
    |   Banner de erro: "Monitoramento ja esta em transicao."
    |
    +-- Erro 500 / erro de rede
            |
            v
        Banner de erro: "Erro ao parar monitoramento. Tente novamente."
```

### 7. Criar MIB

```
Usuario (ADMIN) acessa a tab MIBs
    |
    v
Clica em [Nova MIB]
    |
    v
Formulario inline aparece acima da tabela
    |
    v
Preenche campos: name (obrigatorio), version, description, content (obrigatorio)
    |
    v
Clica em [Salvar]
    |
    v
> POST /api/v1/mibs
  Body: { name: "...", version: "...", description: "...", content: "..." }
    |
    +-- Sucesso (201)
    |       |
    |       v
    |   Formulario fecha
    |   Nova linha aparece na tabela de MIBs
    |   Feedback inline: "MIB criada com sucesso." (5 segundos)
    |
    +-- Erro 409 (nome duplicado)
    |       |
    |       v
    |   Mensagem de erro no formulario: "Ja existe uma MIB com este nome."
    |   Formulario permanece aberto para correcao
    |
    +-- Erro 500 / erro de rede
            |
            v
        Banner de erro: "Erro ao criar MIB. Tente novamente."
```

### 8. Editar MIB

```
Usuario (ADMIN) clica em [Ed] (Editar) em uma linha da tabela de MIBs
    |
    v
> GET /api/v1/mibs/{mibId} (para carregar conteudo completo)
    |
    v
Formulario inline abre abaixo da linha, pre-populado com dados atuais
    |
    v
Usuario altera campos desejados
    |
    v
Clica em [Salvar Alteracoes]
    |
    v
> PUT /api/v1/mibs/{mibId}
  Body: { name: "...", version: "...", description: "...", content: "..." }
    |
    +-- Sucesso (200)
    |       |
    |       v
    |   Formulario fecha
    |   Linha na tabela atualiza com novos dados
    |   Feedback inline: "MIB atualizada com sucesso." (5 segundos)
    |
    +-- Erro 500 / erro de rede
            |
            v
        Banner de erro: "Erro ao atualizar MIB. Tente novamente."
```

### 9. Excluir MIB

```
Usuario (ADMIN) clica em [Ex] (Excluir) em uma linha da tabela de MIBs
    |
    v
Confirmacao inline: "Tem certeza que deseja excluir a MIB '{name}'?"
    |
    +-- Clica [Sim, Excluir]
    |       |
    |       v
    |   > DELETE /api/v1/mibs/{mibId}
    |       |
    |       +-- Sucesso (200/204)
    |       |       |
    |       |       v
    |       |   Linha removida da tabela
    |       |   Feedback inline: "MIB excluida." (5 segundos)
    |       |
    |       +-- Erro 500 / erro de rede
    |               |
    |               v
    |           Banner de erro: "Erro ao excluir MIB. Tente novamente."
    |
    +-- Clica [Cancelar]
            |
            v
        Confirmacao fechada, linha volta ao estado original
```

---

## Estados

### Secao 7.1 — Gerenciamento de SCD

| Estado | Descricao | Comportamento Visual |
|---|---|---|
| **Com dados** | Lista de SCDs com itens retornados pela API | Tabela completa com linhas preenchidas, badges de status coloridos, paginacao visivel |
| **Vazio** | Nenhum SCD cadastrado (`meta.total == 0`) | Ilustracao centralizada + "Nenhum SCD cadastrado" + sugestao de upload (para ADMIN) ou "Solicite a um administrador" (para OPERATOR/VIEWER) |
| **Filtro sem resultados** | Filtro de status aplicado, nenhum SCD corresponde | Area de tabela vazia + "Nenhum SCD para o filtro selecionado" + [Limpar filtros] |
| **Upload em progresso** | `POST /scds` em andamento | Botao Upload com spinner, campo de arquivo desabilitado |
| **Upload concluido** | SCD recem enviado | Nova linha na tabela com status UPLOADED (badge azul) |
| **Parse em andamento** | SCD com status UPLOADED, parser rodando em background | Badge UPLOADED com indicador de progresso (opcional: spinner no badge) |
| **SCD parseado** | Status PARSED | Badge amarelo; acoes [Confirmar] e [Rejeitar] visiveis para ADMIN |
| **SCD travado** | Status STALLED (parse falhou) | Badge laranja; acoes [Rejeitar] e [Excluir] visiveis para ADMIN |
| **SCD ativo** | Status ACTIVE | Badge verde; nenhuma acao destrutiva disponivel |
| **SCD inativo** | Status INACTIVE | Badge cinza; acao [Excluir] disponivel para ADMIN |
| **SCD rejeitado** | Status REJECTED | Badge vermelho; nenhuma acao disponivel |
| **Confirmacao em progresso** | `POST confirmations` em andamento | Botao Confirmar/Rejeitar com spinner, desabilitado |
| **Erro de upload duplicado (409)** | Arquivo com mesmo hash ja existe | Banner de erro inline: "Este arquivo ja foi enviado anteriormente." |
| **Erro de confirmacao (409)** | SCD nao esta no estado correto | Banner de erro inline com mensagem descritiva |
| **Erro de exclusao (409)** | Tentou excluir SCD ativo | Banner de erro: "SCD ativo nao pode ser removido." |
| **Erro de API** | HTTP 500 ou erro de rede | Banner de erro vermelho inline no topo + botao [Tentar novamente] |
| **Carregando** | Requisicao em andamento (lista ou detalhe) | Skeleton/shimmer na tabela ou spinner na expansao |
| **Expansao aberta** | Linha da tabela selecionada com detalhe visivel | Area expandida abaixo da linha com resumo, IEDs, redes e acoes |

### Secao 7.2 — Controle de Monitoramento

| Estado | Descricao | Comportamento Visual |
|---|---|---|
| **RUNNING** | Monitoramento em execucao | Indicador ● verde, texto "RUNNING", campo "Ativo desde: {data}", botao [Parar Monitoramento], tabela de itens monitorados com dados atualizados |
| **STOPPED** | Monitoramento parado | Indicador ○ cinza, texto "STOPPED", campos "Parado desde: {data}" e "Parado por: {usuario}", botao [Iniciar Monitoramento], tabela de itens com status UNKNOWN |
| **STARTING** | Transicao para RUNNING | Indicador ○ amarelo pulsante com spinner, texto "STARTING...", botao desabilitado |
| **STOPPING** | Transicao para STOPPED | Indicador ○ amarelo pulsante com spinner, texto "STOPPING...", botao desabilitado |
| **ERROR** | Erro no motor de monitoramento | Indicador ⚠ vermelho, texto "ERROR", campo "Em erro desde: {data}", botao [Iniciar Monitoramento] (permite tentar reiniciar) |
| **Erro de API (409)** | Conflito ao tentar iniciar/parar | Banner de erro: "Monitoramento ja esta em transicao." |
| **Erro de API (500)** | Erro de servidor ou rede | Banner de erro vermelho inline + botao [Tentar novamente] |
| **Carregando** | GET /monitoring em andamento | Skeleton/shimmer no card de status e na tabela |

### Secao 7.3 — Gerenciamento de MIBs

| Estado | Descricao | Comportamento Visual |
|---|---|---|
| **Com dados** | Lista de MIBs com itens retornados | Tabela completa, paginacao visivel, coluna Acoes para ADMIN |
| **Vazio** | Nenhuma MIB cadastrada (`meta.total == 0`) | Ilustracao centralizada + "Nenhuma MIB cadastrada" + sugestao de criacao |
| **Busca sem resultados** | Filtro `q` aplicado, nenhuma MIB corresponde | Area de tabela vazia + "Nenhuma MIB encontrada para '{termo}'" + [Limpar busca] |
| **Formulario aberto (criacao)** | Formulario de Nova MIB visivel acima da tabela | Campos vazios, botoes [Salvar] e [Cancelar] |
| **Formulario aberto (edicao)** | Formulario de edicao visivel abaixo da linha selecionada | Campos pre-populados, botoes [Salvar Alteracoes] e [Cancelar] |
| **Salvando** | POST ou PUT em andamento | Botao Salvar com spinner, campos desabilitados |
| **Erro de nome duplicado (409)** | Nome de MIB ja existe | Mensagem de erro inline no formulario: "Ja existe uma MIB com este nome." |
| **Confirmacao de exclusao** | Usuario clicou [Excluir] | Confirmacao inline substitui os botoes de acao da linha |
| **Excluindo** | DELETE em andamento | Botao [Sim, Excluir] com spinner |
| **Erro de API** | HTTP 500 ou erro de rede | Banner de erro vermelho inline |
| **Carregando** | GET /mibs em andamento | Skeleton/shimmer na tabela |

---

## Permissoes por Role

| Elemento | ADMIN | OPERATOR | VIEWER |
|---|---|---|---|
| **Secao SCD** | | | |
| Ver lista de SCDs | ✓ | ✓ | ✓ |
| Filtrar SCDs por status | ✓ | ✓ | ✓ |
| Expandir detalhes do SCD (resumo, IEDs, redes) | ✓ | ✓ | ✓ |
| Area de upload SCD | Visivel | **Oculto** | **Oculto** |
| Botao [Confirmar] (SCD PARSED) | Visivel | **Oculto** | **Oculto** |
| Botao [Rejeitar] (SCD PARSED/STALLED) | Visivel | **Oculto** | **Oculto** |
| Botao [Excluir] (SCD nao-ACTIVE) | Visivel | **Oculto** | **Oculto** |
| **Secao Monitoramento** | | | |
| Ver status do monitoramento | ✓ | ✓ | ✓ |
| Ver tabela de itens monitorados | ✓ | ✓ | ✓ |
| Botao [Iniciar Monitoramento] | Visivel | Visivel | **Oculto** |
| Botao [Parar Monitoramento] | Visivel | Visivel | **Oculto** |
| **Secao MIBs** | | | |
| Ver lista de MIBs | ✓ | ✓ | ✓ |
| Buscar MIBs | ✓ | ✓ | ✓ |
| Botao [Nova MIB] | Visivel | **Oculto** | **Oculto** |
| Botao [Editar] por linha | Visivel | **Oculto** | **Oculto** |
| Botao [Excluir] por linha | Visivel | **Oculto** | **Oculto** |
| Coluna Acoes na tabela de MIBs | Visivel | **Oculto** | **Oculto** |

**Nota:** Elementos marcados como "Oculto" nao sao renderizados no DOM para os roles indicados. Nao sao exibidos como desabilitados — simplesmente nao aparecem. Isso evita confusao e reduz carga cognitiva para usuarios sem permissao.

---

## Referencias

- `00-navegacao-global.md` — Layout master, padroes de tabela, drawer, paginacao, barra de filtros, convencoes visuais globais, RBAC e regras de timestamp
- `01-tela-inicial.md` — Consome o SCD ativo para renderizar a topologia da subestacao. O nome da subestacao exibido no header vem do `substationName` do SCD ativo
- `06-snmp.md` — Consome as MIBs cadastradas nesta tela para traduzir OIDs em nomes legiveis no monitoramento SNMP
- `08-autenticacao.md` — Fluxo de login que define o role do usuario (ADMIN, OPERATOR, VIEWER), determinando quais acoes sao visiveis nesta tela

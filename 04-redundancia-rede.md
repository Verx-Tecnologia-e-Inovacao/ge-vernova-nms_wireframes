# Tela 4 — Redundância de Rede (HSR/PRP)

> **STATUS: PENDENTE DE DEFINIÇÃO DE API**
>
> Esta tela está **bloqueada** até que o time de backend defina como os protocolos HSR (High-availability Seamless Redundancy) e PRP (Parallel Redundant Protocol) serão expostos na API.

---

## Contexto

A norma IEC 62439-3 define dois protocolos de redundância de rede para subestações:

- **HSR** (High-availability Seamless Redundancy) — redundância em anel, sem perda de pacotes durante falhas
- **PRP** (Parallel Redundant Protocol) — redes paralelas independentes, sem perda de pacotes durante falhas

Estes protocolos são essenciais para garantir a disponibilidade da comunicação em subestações IEC 61850, especialmente nos barramentos de processo (Process Bus) onde trafegam GOOSE e Sampled Values.

---

## Por que está bloqueada

### 1. Ausência no ProtocolEnum da API

O `ProtocolEnum` da API v1.0.1 contém apenas:

```
SV, GOOSE, PTP, MMS, SNMP, SYSTEM
```

Não há valores `HSR` ou `PRP`. Portanto, não existem endpoints para:
- Ingestão de métricas HSR/PRP
- Consulta de dados HSR/PRP
- Alarmes com source HSR/PRP

### 2. Ausência no SCD

O SCD parseado (`docs/parsed-scd/scd.md`) não contém dados explícitos de HSR/PRP. As 6 sub-redes definidas (W1, W2, W4, W01, W02, W03) são tipadas como `8-MMS`, `IP` ou sem tipo especificado — nenhuma declara HSR ou PRP.

### 3. Fonte de dados indefinida

Não está claro de onde virão os dados de redundância:
- Polling SNMP dos switches (MIBs específicas de HSR/PRP)?
- Análise de tráfego pelo agente de captura?
- Declaração no SCD (extensão de configuração)?

---

## Perguntas em aberto

| # | Pergunta | Impacto |
|---|---|---|
| 1 | Os dados de HSR/PRP virão via SNMP dos switches? | Define se usamos os endpoints existentes de SNMP ou precisamos de novos |
| 2 | Será adicionado `HSR` e `PRP` ao `ProtocolEnum`? | Define se teremos endpoints dedicados `/protocols/HSR/metrics` etc. |
| 3 | O protocolo `SYSTEM` será usado como container genérico para HSR/PRP? | Alternativa sem mudança de API |
| 4 | O SCD incluirá informações de redundância no futuro? | Define se haverá dados estáticos para overlay |
| 5 | Quais métricas são relevantes? (ex: supervision frames, ring breaks, duplicates discarded) | Define os campos da tabela/dashboard |

---

## Sugestão de estrutura futura

Quando a API suportar HSR/PRP, esta tela deverá seguir o padrão das demais telas de protocolo e conter:

### Componentes esperados

| Componente | Descrição |
|---|---|
| Status geral da redundância | Indicador: ativa / degradada / inativa |
| Tabela de portas/interfaces | Status HSR/PRP por porta dos switches |
| Métricas de redundância | Supervision frames enviados/recebidos, ring breaks detectados, duplicatas descartadas |
| Gráfico temporal | Série temporal das métricas de redundância |
| Drawer de detalhe | Detalhe por switch/interface |

### Wireframe conceitual (sujeito a mudança)

```
+------------------------------------------------------------------+
|                                                                  |
|  [ HSR ]  [ PRP ]                                                |
|                                                                  |
|  +- Status Geral ------------------------------------------+     |
|  | Protocolo: HSR          Status: * Ativo                  |    |
|  | Topologia: Anel         Ring Breaks: 0                    |   |
|  +----------------------------------------------------------+    |
|                                                                  |
|  +----------+----------+---------------+----------+---------+    |
|  | Switch   | Porta    | Status        | Sup.Tx   | Sup.Rx  |    |
|  +----------+----------+---------------+----------+---------+    |
|  | SW-01    | Port 1   | * Forwarding  | 12340    | 12338   |    |
|  | SW-01    | Port 2   | * Forwarding  | 12340    | 12335   |    |
|  | SW-02    | Port 1   | * Forwarding  | 12340    | 12340   |    |
|  | SW-02    | Port 2   | ! Blocked     | 12340    | 0       |    |
|  +----------+----------+---------------+----------+---------+    |
|                                                                  |
|  +- Métricas de Redundância --------------------------------+    |
|  |  (gráfico temporal de supervision frames / ring breaks)   |   |
|  +----------------------------------------------------------+    |
|                                                                  |
+------------------------------------------------------------------+
```

### Endpoints esperados (hipotéticos)

| Método | Endpoint | Uso |
|---|---|---|
| GET | `/api/v1/protocols/HSR/metrics` | Métricas HSR em série temporal |
| GET | `/api/v1/protocols/HSR/data` | Dados HSR persistidos |
| GET | `/api/v1/protocols/PRP/metrics` | Métricas PRP em série temporal |
| GET | `/api/v1/protocols/PRP/data` | Dados PRP persistidos |

Ou, alternativamente, via SNMP:

| Método | Endpoint | Uso |
|---|---|---|
| GET | `/api/v1/protocols/SNMP/data?type=HSR` | Dados HSR via SNMP |
| GET | `/api/v1/protocols/SNMP/metrics` | Métricas SNMP incluindo OIDs de HSR/PRP |

---

## Ação necessária

**Alinhar com o time de backend** para definir:

1. Como HSR/PRP serão expostos na API (novo ProtocolEnum vs. SNMP vs. SYSTEM)
2. Quais métricas serão coletadas
3. Se haverá dados estáticos no SCD

Após essa definição, este documento será atualizado com wireframes completos seguindo o template das demais telas.

---

## Referências

- `00-navegacao-global.md` — Layout master e padrões comuns (item 4 da sidebar está desabilitado)
- `06-snmp.md` — Possível fonte de dados HSR/PRP via polling SNMP
- `07-configuracao.md` — Gerenciamento de MIBs (se dados vierem via SNMP)

# Wireframes — NMS (Network Monitoring System)

Documentos de definição de wireframes para as telas do NMS — sistema de monitoramento de subestações IEC 61850. Criados como insumo para o design de interfaces no Figma.

Cada documento contém:
- **Wireframes ASCII** com layout de cada tela e seus estados (com dados, vazio, erro)
- **Componentes** mapeados para campos exatos da API REST
- **Dados de exemplo reais** de uma subestação com 4 IEDs e 6 redes
- **Fluxos de interação** numerados (o que acontece ao clicar, filtrar, navegar)
- **Permissões por role** (ADMIN, OPERATOR, VIEWER)

## Por onde começar

1. Se voce nao e familiarizado com os termos tecnicos do dominio (IEC 61850, GOOSE, SV, SCD, IED, etc.), comece pelo **`glossary.md`** — ele explica todas as siglas e termos tecnicos usados nos documentos.
2. Em seguida, leia o **`00-navegacao-global.md`** — ele define o layout master (header + sidebar + area de conteudo), os componentes reutilizaveis (tabelas, drawers, filtros, paginacao) e as convencoes visuais. Todos os outros documentos fazem referencia a ele.

## Arquivos

| Arquivo | Tela | Descrição |
|---|---|---|
| `glossary.md` | — | Glossario de siglas e termos tecnicos do dominio |
| `00-navegacao-global.md` | — | Layout master, convencoes, padroes reutilizaveis, RBAC |
| `01-tela-inicial.md` | Topologia de Rede | Grafo de IEDs e redes com status em tempo real |
| `02-alarmes.md` | Alarmes | Lista, filtros, detalhe e reconhecimento (ACK) |
| `03-sincronismo-temporal.md` | Sincronismo PTP | Cards por IED + gráfico de acurácia temporal |
| `04-redundancia-rede.md` | Redundância HSR/PRP | **Placeholder** — API pendente de definição |
| `05-comunicacao-dados.md` | Comunicação | Mapas GOOSE, SV e MMS (3 sub-visões) |
| `06-snmp.md` | SNMP | Dispositivos de rede e OIDs traduzidos via MIB |
| `07-configuracao.md` | Configuração | Upload de SCD, controle de monitoramento, MIBs |
| `08-autenticacao.md` | Autenticação | Login e logout |

## Fontes de referência

| Arquivo | Conteúdo |
|---|---|
| `docs/api/swagger-nms-v1.0.1.yaml` | Especificação completa da API REST |
| `docs/parsed-scd/scd.md` | Análise do SCD real (4 IEDs, 6 redes, 11 GOOSE, 6 SV) |
| `docs/structure-proposal/Proposta de Telas.pdf` | Mapa de telas aprovado |
| `docs/structure-proposal/frontend-recommendations.md` | Recomendações de arquitetura frontend |

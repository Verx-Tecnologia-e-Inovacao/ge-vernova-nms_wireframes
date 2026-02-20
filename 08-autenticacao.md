# Tela 8 — Autenticacao

## Objetivo

Login e logout do sistema NMS (Network Monitoring System). Esta e a unica tela publica do sistema — nenhum bearer token e necessario para acessa-la. A tela de autenticacao **nao possui sidebar nem header** da aplicacao; o layout e limpo e centralizado, sem elementos de navegacao global.

Apos o login bem-sucedido, o usuario e redirecionado para a Tela 1 (Topologia). O logout e acionado a partir da sidebar (definido em `00-navegacao-global.md`) e retorna o usuario para esta tela.

---

## Wireframe

### Estado principal (formulario de login)

```
+--------------------------------------------------------------------+
|                                                                    |
|                 (tela inteira, sem sidebar/header)                 |
|                                                                    |
|                                                                    |
|                                                                    |
|                   +----------------------------+                   |
|                   |                            |                   |
|                   |        ##  NMS  ##         |                   |
|                   |    (Logo centralizado)     |                   |
|                   |                            |                   |
|                   |  Network Monitoring        |                   |
|                   |       System               |                   |
|                   |                            |                   |
|                   |  +----------------------+  |                   |
|                   |  | Usuario       [____] |  |                   |
|                   |  +----------------------+  |                   |
|                   |                            |                   |
|                   |  +----------------------+  |                   |
|                   |  | Senha         [____] |  |                   |
|                   |  +----------------------+  |                   |
|                   |                            |                   |
|                   |    [      Entrar      ]    |                   |
|                   |                            |                   |
|                   +----------------------------+                   |
|                                                                    |
|                             NMS v1.0.1                             |
|                                                                    |
+--------------------------------------------------------------------+
```

**Detalhamento:**

- **Fundo**: cor solida neutra (cinza claro ou branco), sem elementos decorativos
- **Card central**: container com sombra sutil, centralizado vertical e horizontalmente
- **Logo NMS**: logotipo da aplicacao no topo do card
- **Titulo**: "Network Monitoring System" abaixo do logo, tipografia secundaria
- **Campo Usuario**: input de texto, placeholder "Usuario", obrigatorio
- **Campo Senha**: input de password (mascarado), placeholder "Senha", obrigatorio
- **Botao Entrar**: botao primario, largura total do card, desabilitado enquanto os campos estiverem vazios
- **Versao**: texto pequeno abaixo do card, ex: "NMS v1.0.1"

---

### Estado de erro (credenciais invalidas)

```
+--------------------------------------------------------------------+
|                                                                    |
|                 (tela inteira, sem sidebar/header)                 |
|                                                                    |
|                                                                    |
|                                                                    |
|                   +----------------------------+                   |
|                   |                            |                   |
|                   |        ##  NMS  ##         |                   |
|                   |    (Logo centralizado)     |                   |
|                   |                            |                   |
|                   |  Network Monitoring        |                   |
|                   |       System               |                   |
|                   |                            |                   |
|                   |  +----------------------+  |                   |
|                   |  | Usuario   [admin___] |  |                   |
|                   |  +----------------------+  |                   |
|                   |                            |                   |
|                   |  +----------------------+  |                   |
|                   |  | Senha     [------__] |  |                   |
|                   |  +----------------------+  |                   |
|                   |                            |                   |
|                   |  ! Credenciais invalidas.  |                   |
|                   |    Verifique usuario e     |                   |
|                   |    senha.                  |                   |
|                   |                            |                   |
|                   |    [      Entrar      ]    |                   |
|                   |                            |                   |
|                   +----------------------------+                   |
|                                                                    |
|                             NMS v1.0.1                             |
|                                                                    |
+--------------------------------------------------------------------+
```

**Detalhamento:**

- Os campos **mantem os valores preenchidos** apos o erro (o campo de senha pode ser limpo ou mantido, a criterio do designer)
- A **mensagem de erro** aparece inline entre os campos e o botao, com icone de alerta e cor de destaque (vermelho/laranja)
- O botao Entrar permanece habilitado (os campos ainda contem valores)
- Para erro 400 (dados invalidos), a mensagem exibida e: "Dados invalidos. Verifique os campos."

---

### Estado autenticado (redirect)

Apos o login bem-sucedido (`POST /api/v1/sessions` retorna 200), o sistema:

1. Armazena `accessToken` e `refreshToken` no client (localStorage ou memoria)
2. Redireciona imediatamente para **Tela 1 — Topologia** (`01-tela-inicial.md`)

Nao ha estado visual intermediario de "boas-vindas" — o redirect e instantaneo. Durante a transicao, pode-se exibir um breve indicador de carregamento (spinner) no botao ou na tela.

---

## Componentes

| Componente | Tipo | Descricao |
|---|---|---|
| Logo NMS | Imagem/SVG | Logo da aplicacao, centralizado no topo do card de login |
| Titulo do sistema | Texto | "Network Monitoring System" — tipografia secundaria abaixo do logo |
| Campo Usuario | Input text | Obrigatorio, placeholder "Usuario", sem mascara |
| Campo Senha | Input password | Obrigatorio, placeholder "Senha", caracteres mascarados (••••) |
| Botao Entrar | Button (primary) | Submete o formulario. Estado `disabled` enquanto campos estiverem vazios. Exibe spinner durante requisicao |
| Mensagem de erro | Alert inline | Aparece abaixo dos campos quando o login falha (401 ou 400). Icone ⚠ + texto descritivo. Cor de destaque |
| Versao do sistema | Texto | Texto pequeno no rodape do card ou da pagina (ex: "NMS v1.0.1") |

---

## Dados e Endpoints

### Login

| Campo | Valor |
|---|---|
| Metodo | `POST` |
| Endpoint | `/api/v1/sessions` |
| Autenticacao | Nenhuma (endpoint publico, `security: []`) |
| Request body | `{ "username": "string", "password": "string" }` |
| Response 200 | `{ "tokenType": "Bearer", "accessToken": "string", "expiresInSeconds": 3600, "refreshToken": "string" }` |
| Response 401 | Credenciais invalidas |
| Response 400 | Dados invalidos (campos ausentes ou formato incorreto) |

### Logout

| Campo | Valor |
|---|---|
| Metodo | `DELETE` |
| Endpoint | `/api/v1/sessions/current` |
| Autenticacao | Bearer token (obrigatorio) |
| Request body | Nenhum |
| Response 204 | No Content — sessao encerrada com sucesso |

### Refresh Token

| Campo | Valor |
|---|---|
| Metodo | `POST` |
| Endpoint | `/api/v1/tokens/refresh` |
| Autenticacao | Nenhuma (endpoint publico, `security: []`) |
| Request body | `{ "refreshToken": "string" }` |
| Response 200 | `{ "accessToken": "string", "expiresInSeconds": 3600 }` |
| Response 401 | Refresh token invalido ou expirado |

### Resumo de endpoints

| Metodo | Endpoint | Uso | Security |
|---|---|---|---|
| `POST` | `/api/v1/sessions` | Login (criar sessao) | Publico |
| `DELETE` | `/api/v1/sessions/current` | Logout (encerrar sessao) | Bearer token |
| `POST` | `/api/v1/tokens/refresh` | Renovar access token | Publico |

> **Nota:** O `sessionId` no endpoint de logout utiliza o valor especial `current` para referenciar a sessao ativa do usuario autenticado.

---

## Fluxos de Interacao

### 1. Login

```
Usuario acessa o sistema (URL raiz ou rota protegida sem token)
    |
    v
Exibe formulario de login (estado principal)
    |
    v
Usuario preenche "Usuario" e "Senha"
    |
    v
Botao [Entrar] fica habilitado
    |
    v
Usuario clica [Entrar] ou pressiona Enter
    |
    v
> POST /api/v1/sessions { username, password }
> Botao exibe spinner, campos desabilitados
    |
    +-- Sucesso (200)
    |       |
    |       v
    |   Armazena accessToken e refreshToken no client
    |       |
    |       v
    |   Redireciona para Tela 1 (Topologia)
    |
    +-- Erro 401 (credenciais invalidas)
    |       |
    |       v
    |   Exibe mensagem inline: "! Credenciais invalidas. Verifique usuario e senha."
    |   Campos mantem valores, botao reabilitado
    |
    +-- Erro 400 (dados invalidos)
            |
            v
        Exibe mensagem inline: "! Dados invalidos. Verifique os campos."
        Campos mantem valores, botao reabilitado
```

### 2. Refresh automatico de token

```
Qualquer requisicao autenticada retorna 401 (token expirado)
    |
    v
Frontend intercepta o 401 (HTTP interceptor)
    |
    v
> POST /api/v1/tokens/refresh { refreshToken }
    |
    +-- Sucesso (200)
    |       |
    |       v
    |   Armazena novo accessToken
    |       |
    |       v
    |   Retenta a requisicao original com novo token
    |
    +-- Erro 401 (refresh token expirado)
            |
            v
        Limpa tokens armazenados no client
            |
            v
        Redireciona para tela de login
            |
            v
        Exibe mensagem: "! Sessao expirada. Faca login novamente."
```

### 3. Logout

```
Usuario clica [Logout] na sidebar (ver 00-navegacao-global.md)
    |
    v
> DELETE /api/v1/sessions/current
    |
    v
Limpa accessToken e refreshToken do client
    |
    v
Redireciona para tela de login (estado principal)
```

> **Nota:** O logout deve limpar os tokens do client independentemente da resposta do servidor (mesmo se o `DELETE` falhar, o usuario deve ser deslogado localmente).

---

## Estados

| Estado | Descricao | Comportamento Visual |
|---|---|---|
| Nao autenticado | Formulario de login visivel | Formulario limpo, campos vazios, botao desabilitado |
| Campos preenchidos | Usuario digitou nos dois campos | Botao [Entrar] muda de `disabled` para estado primario (ativo) |
| Carregando | Requisicao `POST /sessions` em andamento | Botao exibe spinner, texto muda para "Entrando...", campos ficam desabilitados |
| Erro de credencial | Servidor retornou 401 | Mensagem de erro inline com ⚠, campos mantem valores digitados, botao reabilitado |
| Erro de dados | Servidor retornou 400 | Mensagem de erro inline com ⚠, campos mantem valores digitados, botao reabilitado |
| Autenticado | Login bem-sucedido (200) | Redirect imediato para Tela 1 (Topologia) |
| Sessao expirada | Refresh token falhou (401 no refresh) | Redirect para login com mensagem "Sessao expirada. Faca login novamente." |

### Diagrama de estados

```
    +------------------+
    |  Nao autenticado | <------------------------------+
    |  (campos vazios) |                                 |
    +--------+---------+                                 |
             | usuario digita                            |
             v                                           |
    +------------------+                                 |
    |    Campos        |                                 |
    |  preenchidos     |                                 |
    +--------+---------+                                 |
             | clica [Entrar]                            |
             v                                           |
    +------------------+                                 |
    |   Carregando     |                                 |
    |   (spinner)      |                                 |
    +--------+---------+                                 |
             |                                           |
       +-----+------+                                    |
       |            |                                    |
       v            v                                    |
+------------+  +------------+                           |
|   Erro     |  | Autenticado|                           |
| (401/400)  |  |   (200)    |                           |
+-----+------+  +-----+------+                           |
      |               |                                  |
      |               v                                  |
      |        +-------------+    refresh falha    +-----+----------+
      |        |  Tela 1     | ----------------->  |    Sessao      |
      |        | (Topologia) |                     |   expirada     |
      |        +-------------+                     +----------------+
      |               |
      |               v logout
      +------>  Volta para "Nao autenticado"
```

---

## Permissoes por Role

Esta tela e **publica** — nao ha restricoes baseadas em role. Qualquer usuario (autenticado ou nao) pode acessar a tela de login.

| Acao | ADMIN | OPERATOR | VIEWER | Nao autenticado |
|---|---|---|---|---|
| Visualizar formulario de login | -- | -- | -- | Sim |
| Realizar login | -- | -- | -- | Sim |
| Refresh de token | Sim | Sim | Sim | -- |
| Logout | Sim | Sim | Sim | -- |

> **Nota:** Apos o login, o role do usuario (`ADMIN`, `OPERATOR`, `VIEWER`) determina as permissoes em todas as demais telas do sistema. Consultar `00-navegacao-global.md` para a matriz completa de permissoes.

---

## Regras e Observacoes Tecnicas

1. **Armazenamento de tokens**: O `accessToken` e o `refreshToken` devem ser armazenados de forma segura no client. Opcoes recomendadas:
   - `httpOnly cookies` (mais seguro, requer suporte do backend)
   - `localStorage` (mais simples, vulneravel a XSS)
   - `memoria` (mais seguro contra XSS, perde-se ao fechar a aba)

2. **Interceptor HTTP**: Implementar um interceptor global que:
   - Adiciona o header `Authorization: Bearer {accessToken}` em todas as requisicoes autenticadas
   - Intercepta respostas 401 e tenta refresh automatico
   - Enfileira requisicoes concorrentes durante o refresh para evitar multiplos refreshes simultaneos

3. **Redirect apos login**: Se o usuario tentou acessar uma rota protegida antes de autenticar, o sistema deve redireciona-lo para essa rota apos o login (e nao para a Tela 1 padrao).

4. **Validacao de campos**: Validacao client-side minima antes de submeter:
   - Ambos os campos devem estar preenchidos (nao vazios)
   - Nao ha requisitos de formato especifico para username ou senha no frontend

5. **Acessibilidade**:
   - Campos devem ter labels associados (`<label>`)
   - Suporte a navegacao por teclado (Tab entre campos, Enter para submeter)
   - Mensagens de erro devem ser anunciadas por screen readers (`aria-live="polite"`)

---

## Referencias

- `00-navegacao-global.md` — Layout master, sidebar, header e padroes comuns de navegacao
- `01-tela-inicial.md` — Tela de destino apos login bem-sucedido (Topologia da subestacao)

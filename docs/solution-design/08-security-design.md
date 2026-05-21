# 08 — Design de Segurança

> **Camada:** Experience | **Depende de:** 05-backend-design, 06-api-design

## Autenticação

### JWT de Sessão (Interface Web)

Emitido após login com sucesso via `POST /api/v1/auth/login`.

```
Header:  { "alg": "HS256", "typ": "JWT" }
Payload: {
  "sub": "joao.silva",
  "name": "João Silva",
  "type": "SESSION",
  "iat": 1716220800,
  "exp": 1716249600    ← 8 horas
}
```

- Segredo JWT configurado via variável de ambiente `JWT_SECRET` (mínimo 256 bits).
- O frontend armazena o token em memória (variável de estado React) — evita XSS via `localStorage`.
- Se a aba for fechada, o token é perdido e o usuário deve fazer login novamente.
- Endpoint de logout não precisa invalidar no servidor (stateless); a expiração de 8h é a garantia.

### Token de Integração (API Externa)

Emitido via `POST /api/v1/tokens` por usuário autenticado.

```
Header:  { "alg": "HS256", "typ": "JWT" }
Payload: {
  "sub": "joao.silva",
  "type": "INTEGRATION",
  "tokenId": 42,
  "exp": 1767225600    ← data configurada pelo usuário
}
```

- O valor do token (`sifu_tk_...`) é exibido uma única vez na criação.
- No banco, armazena apenas o hash SHA-256 do token (`token_hash`) — nunca o valor em claro.
- Na validação, o `JwtFilter` verifica: assinatura → expiração → status no banco (`ACTIVE`).
- Revogação via `DELETE /api/v1/tokens/{id}` marca status como `REVOKED` no banco.

### Fluxo de Validação por Requisição

```
Requisição HTTP
  ↓
JwtFilter (OncePerRequestFilter)
  ├── Extrai token do header Authorization: Bearer <token>
  ├── Verifica assinatura HMAC-SHA256
  ├── Verifica expiração
  ├── Se type=INTEGRATION: consulta banco para verificar status (ACTIVE)
  ├── Carrega usuário no SecurityContext
  └── Continua para o Controller

Se qualquer verificação falhar → 401 Unauthorized
```

## Proteção contra Força Bruta

- Após 5 tentativas de login inválidas consecutivas, a conta é bloqueada por 15 minutos.
- O campo `tentativas_login` é incrementado a cada falha e zerado em caso de sucesso.
- O campo `bloqueado_ate` registra até quando a conta está bloqueada.
- Retorna HTTP 423 (Locked) com mensagem de tempo restante.

## Criptografia de Senhas

- Algoritmo: **BCrypt** com custo (strength) 12.
- Armazenamento: apenas o hash no campo `senha_hash` — nunca a senha em texto claro.
- Validação de força de senha (mínimo 8 caracteres, ao menos 1 número, 1 maiúscula, 1 especial).

## Recuperação de Senha

1. `POST /api/v1/auth/recover-password { email }` → sempre retorna 204 (sem revelar se e-mail existe).
2. Sistema gera token de redefinição (UUID) com expiração de 1 hora, armazenado em tabela própria.
3. Envia e-mail com link `https://sifu/reset-password?token=<uuid>`.
4. `PUT /api/v1/auth/reset-password { resetToken, newPassword }` → valida token, redefine senha, invalida token.

## HTTPS

- Toda comunicação deve ocorrer sobre HTTPS em produção.
- O backend rejeita requisições HTTP em produção (configuração via `server.ssl` no Spring Boot ou proxy reverso).
- Em desenvolvimento local, HTTP é permitido para facilitar o ambiente Docker.

## CORS

O backend aceita requisições apenas da origem do frontend:

```
Origens permitidas: configurado via variável de ambiente CORS_ALLOWED_ORIGINS
Métodos: GET, POST, PUT, PATCH, DELETE, OPTIONS
Headers: Authorization, Content-Type
```

## Headers de Segurança HTTP

Configurados via Spring Security:

| Header | Valor |
|---|---|
| `X-Frame-Options` | `DENY` |
| `X-Content-Type-Options` | `nosniff` |
| `Strict-Transport-Security` | `max-age=31536000; includeSubDomains` |
| `Content-Security-Policy` | `default-src 'self'` |

## Rate Limiting

Endpoints de integração (`/api/v1/**` com token de integração) são limitados a **100 requisições por minuto** por token. Implementado via filtro Spring com contador em memória (ou Redis em Fase 2).

Resposta ao exceder: `429 Too Many Requests` com header `Retry-After`.

## Auditoria

Toda operação que altera estado é auditada pelo `AuditInterceptor` (AOP). O registro inclui:

| Campo | Origem |
|---|---|
| `usuario_login` | `SecurityContext` |
| `data_hora` | `NOW()` |
| `operacao` | Anotação `@Auditavel(operation = "ISSUE_COMMITMENT")` |
| `entidade` | Anotação `@Auditavel(entity = "Commitment")` |
| `entidade_id` | Retorno do método interceptado |
| `dados_antes` | Estado da entidade antes (serializado em JSON) |
| `dados_depois` | Estado da entidade depois (serializado em JSON) |
| `ip` | `HttpServletRequest.getRemoteAddr()` |
| `ug_id` | Contexto da operação |

Operações auditadas obrigatoriamente: todas as criações, atualizações e transições de status de documentos financeiros; login, logout, criação e revogação de tokens; criação e desativação de usuários.

A tabela `auditoria` é **append-only** — sem updates ou deletes jamais.

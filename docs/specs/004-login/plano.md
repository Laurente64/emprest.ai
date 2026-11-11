# 004 — Login e sessão · Plano

Status: APROVADO em 2026-11-11 pelo agente, por delegação do usuário
(spec.md, "Aprovação"). Tarefas: tarefas.md.

## Ponto de partida (conferido em 2026-11-11)
- back/ e front/ na main, limpos. Nenhum módulo usa o Prisma; não há
  guard, contexto de request nem filtro de erro.
- Supabase Auth: JWKS com chave ES256 publicada; cadastro público
  desligado (`disable_signup = true`).
- vibing/: a branch `estado-e-comandos` (AGENTS.md e rules) não foi
  mergeada. A branch `pratica-sdd` sai dela, e o PR leva os dois.

## Back

| peça | onde | o que faz |
|---|---|---|
| Configuração | `src/config/env.ts` | `DATABASE_URL`, `SUPABASE_URL` e `SUPABASE_ANON_KEY` entram no `envSchema`. Faltando, a API não sobe (ADR-001 §9). |
| Prisma | `src/prisma/` | `PrismaService` global, com a URL do ambiente (rules/secrets.md). Conecta na primeira query, não no boot. |
| Contexto | `src/modules/autenticacao/contexto-do-request.ts` | `AsyncLocalStorage` com tenant, usuário, membership e papel (ADR-001 §3). Um middleware abre o contexto; o guard preenche. |
| Token | `src/modules/autenticacao/verificador-de-token.ts` e `jwks.ts` | Recusa algoritmo simétrico, `alg: none` e token sem `kid`. Busca a JWKS com `fetch` e guarda 10 min; `kid` desconhecido força uma busca nova, e se continuar desconhecido é 401. Passa só a chave escolhida para o `supabase-js` `auth.getClaims(token, { keys })`, que confere assinatura e expiração; assim ele nunca cai no `getUser`. Depois confere `aud` e `iss` e devolve `sub` e `app_metadata.tenant_id`. |
| Guard | `src/modules/autenticacao/guarda-de-autenticacao.ts` | Global (`APP_GUARD`). `@Publica()` libera a rota (só `/v1/health`). 401 sem token válido; 403 sem tenant ou sem membership. |
| Membership | `src/modules/autenticacao/memberships.repository.ts` | Busca a membership por (tenant, user). Isola o Prisma (ADR-001 §3). |
| Erros | `src/problemas/` | Filtro global que transforma toda exceção em Problem Details (RFC 9457), e a classe `Problema` para as recusas com `type` próprio (`/problemas/<slug>`). |
| `/v1/eu` | `src/modules/eu/` | Controller, service e schema Zod. |
| OpenAPI | `src/openapi/` | Esquema `bearer` no documento; o `emit` usa valores de mentira para o ambiente, porque só monta o documento e não conecta em nada. |

Dependência nova: `@supabase/supabase-js` com versão fixa (D-02 da spec).

`SUPABASE_ANON_KEY` no back/: o cliente do `supabase-js` exige uma chave
para existir e a manda ao buscar a JWKS. É a chave pública (a mesma do
front), não a secreta. A secreta não entra no runtime da API.

### Testes
- `test/autenticacao.e2e-spec.ts` (supertest): a app Nest inteira, com a
  JWKS servida por um `http.createServer` local e tokens assinados no
  teste com `node:crypto` (ES256). O repositório de membership é
  trocado por uma versão em memória: este teste prova o guard, não o
  banco. O servidor local conta as chamadas a `/auth/v1/user` (CA-03
  exige zero). CA-01 a CA-09.
- O e2e da spec 002 passa a ter o ambiente de teste (setup do Jest e2e
  com valores locais), porque o módulo agora precisa das variáveis.

## Front

| peça | onde | o que faz |
|---|---|---|
| Cliente do Auth | `src/sessao/supabase.ts` | `createClient` com `VITE_SUPABASE_URL` e `VITE_SUPABASE_ANON_KEY`. |
| Sessão | `src/sessao/` | Hook que acompanha `onAuthStateChange`; rota protegida que manda para `/entrar` sem sessão. |
| Transporte | `src/api/http.ts` | Põe `Authorization: Bearer` com o token da sessão; erro da API vira `ErroDaApi` com o status e o Problem Details. |
| Estado de servidor | `src/main.tsx` | `QueryClientProvider` do TanStack Query (ADR-001 §2). |
| Telas | `src/pages/Entrar.tsx`, `src/pages/AreaLogada.tsx` | Login (React Hook Form + `zodResolver`) e a barra de topo do layout.md §6. |
| Landing | `src/pages/Landing.tsx` | Link "Entrar" (D-04). |
| CSP | `vercel.json` | `connect-src` ganha a URL do Supabase. |

Dependências novas, todas no ADR-001 §2 e §7, com versão fixa:
`@supabase/supabase-js`, `@tanstack/react-query`, `react-hook-form`,
`@hookform/resolvers`, `zod`.

### Testes
Playwright, com o preview apontando para `http://supabase.teste.local` e
`http://api.teste.local`, ambos interceptados (`page.route`). CA-10 a
CA-18; o CA-18 lê o `vercel.json`, porque o preview não aplica os
cabeçalhos da Vercel. O teste do CA-01 da spec 001 muda junto com a
emenda dela.

## Variáveis de ambiente (rules/secrets.md)

| variável | repositório | .env local | .env.example | Vercel |
|---|---|---|---|---|
| `DATABASE_URL` | back | já existe | já existe | usuário cadastra |
| `SUPABASE_URL` | back | agente pôs | agente põe | usuário cadastra |
| `SUPABASE_ANON_KEY` | back | agente põe | agente põe | usuário cadastra |
| `VITE_SUPABASE_URL` | front | agente pôs | agente põe | usuário cadastra |
| `VITE_SUPABASE_ANON_KEY` | front | agente pôs | agente põe | usuário cadastra |

Sem essas variáveis no painel, o deploy da API não sobe e o login do
front não funciona. O painel é do usuário (restrictions.md).

## Deploy
Push das branches gera preview. O merge é do usuário, na ordem: back
004 → front 004.

## Revisão das specs
Revisor de contexto limpo, 2026-11-11, sobre 004, 005 e 006. Achados e o
que foi feito: tarefas.md desta spec, "Revisão".

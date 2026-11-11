# 004 — Login e sessão · Tarefas

Uma por vez, com os checks de vibing/rules/checks.md ao fim de cada uma.
Nesta prática o usuário delegou as aprovações (spec.md, "Aprovação"): as
tarefas seguem sem parada entre elas, e o resultado de cada uma fica
registrado aqui.

## T1 — Back: configuração, Prisma, contexto e erros
- Branch `spec-004-login` no back/, a partir da main.
- `@supabase/supabase-js` com versão fixa.
- `envSchema` com `DATABASE_URL`, `SUPABASE_URL` e `SUPABASE_ANON_KEY`;
  `.env.example` sem valor.
- `PrismaService`, contexto do request, filtro de Problem Details.
- Pronto quando: `npm test`, `npm run test:e2e`, build e lint passam.

## T2 — Back: guard, verificador de token e `GET /v1/eu`
- Guard global com `@Publica()`, verificador de token, repositório de
  membership, módulo `eu`, esquema bearer no OpenAPI, `openapi.json`
  regerado.
- Testes CA-01 a CA-09.
- Pronto quando: `npm test`, `npm run test:e2e`, `npm run test:db`,
  build, lint e `openapi:check` passam.

## T3 — Front: sessão, login e área logada
- Branch `spec-004-login` no front/, a partir da main.
- Dependências, cliente do Auth, sessão, rota protegida, `/entrar`,
  `/app` com a barra, transporte com o token, CSP, link na landing,
  `.env.example`, cliente regerado (`api:gen`).
- Testes CA-10 a CA-18 e o teste do CA-01 emendado da spec 001.
- Pronto quando: `npm test`, build, lint e `api:check` passam.

## T4 — Verificação
- Verificador (vibing/agents/verificador-spec.md) com back/ e front/ nas
  branches `spec-004-login`. Rodada salva em verificacoes/.

## T5 — Push e PR
- Push das duas branches e links de PR (não há `gh` nesta máquina).

## T6 — Produção
- Depende do merge e das variáveis no painel da Vercel (usuário).
- Entrar no endereço da Vercel com a conta de teste. Quem fez e o
  resultado ficam no registro.

## Revisão (2026-11-11)
Um subagente revisor de contexto limpo leu as specs 004, 005 e 006 contra
o PRD, os ADRs, as rules e o enunciado e devolveu 37 itens. O que mudou:
- Aceitos e incorporados nas specs: CSP de login e de fotos com CA
  próprio (004 item 11, 006 item 16); token só com `kid` da JWKS, sem cair
  no `getUser`, conferindo `iss` (004 item 13, CA-03); `user_metadata`
  ignorado (CA-04); preflight de CORS (CA-09); 401 numa tela logada
  (item 9, CA-16); tabela única de variáveis (004); emenda formal da spec
  001; nota na spec 003; quem escreve em produção e o conflito com o
  enunciado (005); conflito com restrictions.md sobre dado fora de
  migration (005 C-05); DDL e escolha do comando de geração (005 item 4 e
  L-06); CA da senha trocado por conferência do histórico (005);
  conta de teste opcional nos CAs (005 CA-05 e CA-07); `app_metadata`
  somado, não trocado (005 item 9); estoque tratado como na spec 003 (006
  L-07); `autoatendimento` na `audit_log`; "Limite de L itens"; fuso e
  relógio fixos nos testes; dados que sobram em produção (006 L-08);
  detalhes de paginação e de "X de Y".
- Não aceito como pedido: voltar as specs para rascunho. O usuário
  delegou a aprovação em 2026-11-10. Para não atribuir decisão do agente
  a ele, cada spec lista "Para o usuário confirmar".
- Mantido com registro: rate limit, helmet, pino e Sentry fora (004 D-08);
  o agente roda migration e seed (005, por decisão do usuário); a chave
  secreta no `back/.env` durante a prática (005, "Quem escreve em
  produção").

## Registro
(preenchido durante a execução)

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
- T1 (2026-11-11) — back `spec-004-login` 4ab998c. `npm test` 9 passed;
  `npm run test:e2e` 4 passed; build e lint sem erro; `openapi:check` em dia.
- T2 (2026-11-11) — back 24e400b. `npm run test:e2e` 13 passed (CA-01 a
  CA-09); `npm test` 9 passed; `npm run test:db` 104 passed; build e lint
  sem erro; `openapi:check` em dia. Achado durante a tarefa: o `getClaims`
  do `supabase-js` cai no `getUser` pela rede quando o algoritmo é HS ou o
  `kid` não está na JWKS; por isso a API escolhe a chave antes (item 13).
- T3 (2026-11-11) — front `spec-004-login` 734c5fd, feita por um subagente
  (fork do agente principal) só no front/. `npm test` 18 passed (8 da
  landing e 10 de CA-10 a CA-18); build e lint sem erro; `api:check` em
  dia. Decisões que a spec não cobria, tomadas na tarefa:
  - `vercel.json` ganha o rewrite de SPA (`/(.*)` → `/index.html`). Sem
    ele, abrir `/entrar` ou `/app` direto dá 404 na Vercel; o preview do
    Vite já faz isso e o teste não pegaria. Teste citando CA-13.
  - O cliente do Supabase é criado no primeiro uso: sem as variáveis no
    build, a landing continua abrindo.
  - Só o 400 do Auth mostra "E-mail ou senha incorretos."; outra falha
    mostra "Não foi possível entrar agora. Tente de novo."
  - "Sair" revoga o token no Supabase; depois de um 401 da API, a saída é
    só local (o servidor já recusou o token).
  - O TanStack Query não repete 4xx; repete no máximo duas vezes 5xx ou
    falha de rede.
  - Falha de `/v1/eu` que não seja 401 nem 403: "Não foi possível carregar
    os seus dados. Tente de novo." e "Sair".
  - Título e lead da coluna esquerda do login escritos na tarefa; rodapé
    "Operações · TI interno" a 55% (ADR-002 §7), não a 40% do layout.md.
  - Ficou como estava: os testes da landing de CA-02 a CA-08 (spec 001)
    citam o CA só em comentário, o que o checks.md não aceita; o bundle
    passou de 500 kB (aviso do Vite, não erro).
- T4 (2026-11-11) — verificação.
  - Rodada 1 (back 24e400b, front 734c5fd): NÃO PRONTA. CA-02: token RS256
    de chave RSA com o mesmo `kid` voltava 500. Causa: exceção do
    `getClaims` fora de `TokenInvalidoError`. Correção (agente principal):
    back b3733ce (exceção da conferência vira 401; `alg` do token igual ao
    da chave; CA-02 com os dois casos) e front bfe7458 ("Sair" com escopo
    `local`). A spec foi emendada nos itens que a rodada apontou como fora
    dela. Achado do agente na mesma correção: os testes unitários só
    passavam porque o Prisma Client lia o `back/.env`; passaram a carregar
    o ambiente de teste.
  - Rodada 2 (back b3733ce, front bfe7458): PRONTA.
  - Observação da rodada 2, fora dos CAs: o teto de uma busca da JWKS a
    cada 30 s não vale para requests simultâneos. Corrigido depois, no
    branch da spec 006 (busca em andamento compartilhada), sem rodada nova
    da 004 para isso.

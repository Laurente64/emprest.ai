# 004 — Login e sessão

Status: APROVADA em 2026-11-11 pelo agente, por delegação do usuário
(ver "Aprovação"). As decisões do agente estão em "Para o usuário
confirmar". Emendada em 2026-11-11 depois da rodada 1 do verificador
(itens 3, 5, 7, 8 e 13; CA-02 e CA-14): verificacoes/2026-11-11-rodada-1.md.
Origem: PRD v1, item 1 ("Login. Cada pessoa vê os próprios
empréstimos."). Pré-requisito das specs 005 e 006: o briefing de dados
iniciais exige que o professor "entra pelo login do sistema". O
enunciado da prática diz que o login já estava no ar; não estava
(AGENTS.md, "Ainda NÃO existe": autenticação, guards e contexto de
tenant).
Repositórios afetados: back/ e front/.

## Aprovação
O usuário pediu em 2026-11-10, antes de se ausentar, que o agente
aprovasse as specs por ele, com validação extra e o verificador no fim.
Antes da aprovação, um subagente revisor de contexto limpo leu as specs
004, 005 e 006 contra o PRD, os ADRs, as rules e o enunciado, e apontou
37 itens; o que mudou por causa deles está em tarefas.md, "Revisão".
Aprovação por delegação não transforma decisão do agente em decisão do
usuário: as decisões técnicas desta spec são do agente e ficam listadas
para o usuário confirmar ou desfazer.

## Objetivo
Quem tem conta entra com e-mail e senha, a API sabe quem é e em qual
tenant, e a pessoa chega a uma área logada. Conta só existe por ação de
administrador (ADR-001 §7, cadastro fechado): não há cadastro nesta spec.

## Comportamento

### Front
1. Rota `/entrar`, no layout.md §5.1: à esquerda a marca, o título, o
   lead e as três estatísticas (3 itens por pessoa, 14 dias de prazo, 0
   linhas para preencher); à direita "Entrar", E-mail, Senha, botão
   "Entrar" e a nota "Cada pessoa vê apenas os próprios empréstimos". O
   formato é validado no navegador antes de chamar o Auth (React Hook
   Form + `zodResolver`, ADR-001 §2). O schema desse formulário é escrito
   no front: login não passa pela nossa API, então não vem do
   `openapi.json`.
2. O login é feito pelo `supabase-js` no navegador, que guarda e renova
   a sessão (ADR-001 §7).
3. Credenciais recusadas: "E-mail ou senha incorretos.", sem dizer qual
   dos dois. A pessoa continua em `/entrar`. Outra falha do Auth (rede,
   indisponibilidade): "Não foi possível entrar agora. Tente de novo."
   (emenda de 2026-11-11).
4. Login aceito: vai para `/app`.
5. `/app` e o que estiver abaixo dele exigem sessão. Sem sessão, vai
   para `/entrar`. Com sessão, `/entrar` vai para `/app`. O `vercel.json`
   reescreve toda rota para o `index.html` (SPA): sem isso, abrir
   `/entrar` ou `/app` direto na Vercel dá 404 (emenda de 2026-11-11).
6. A área logada tem a barra de topo do layout.md §6: marca, papel,
   iniciais, nome e "Sair". Nome e papel vêm da API (`GET /v1/eu`), não
   do token. Nesta spec a barra ainda não tem abas e o conteúdo é só
   "Bem-vindo, <nome>."; as abas e o catálogo entram na spec 006.
7. "Sair" encerra a sessão deste aparelho no `supabase-js` (escopo
   `local`; o padrão global derrubaria a pessoa em todos os aparelhos) e
   volta para `/entrar` (emenda de 2026-11-11).
8. Conta válida no Auth mas sem vínculo com o tenant (API responde 403):
   a área logada mostra "Sua conta ainda não tem acesso ao EMPREST.AI.
   Fale com Operações." e o botão "Sair". Outra falha de `/v1/eu`: "Não
   foi possível carregar os seus dados. Tente de novo." e "Sair" (emenda
   de 2026-11-11).
9. A API respondendo 401 numa tela logada (sessão revogada ou renovação
   falhou): o front encerra a sessão e vai para `/entrar`.
10. A landing (`/`) ganha um link "Entrar" para `/entrar`. Ela continua
    abrindo sem sessão. Ver D-04.
11. A CSP do `vercel.json` libera, em `connect-src`, exatamente a origem
    do projeto Supabase (`https://owbckpmncpilerfmlxnz.supabase.co`), sem
    curinga. Sem isso o navegador bloqueia o login na Vercel, e o
    Playwright, que roda no preview sem esses cabeçalhos, não percebe.

### Back
12. Toda rota exige `Authorization: Bearer <access token do Supabase>`,
    exceto `GET /v1/health`, que continua pública (spec 002).
13. O token só é aceito se: o algoritmo for assimétrico (ES256 ou RS256)
    com `kid`; o `kid` estiver na JWKS do projeto (ADR-001 §7), guardada
    em cache por 10 minutos e buscada de novo quando o `kid` não aparece,
    no máximo uma vez a cada 30 segundos (sem esse teto, cada token com
    `kid` inventado seria uma chamada à JWKS; depois de uma rotação de
    chave, o `kid` novo pode esperar até 30 s; emenda de 2026-11-11); o
    `alg` do token ser o da chave; a assinatura bater com essa chave; não estiver expirado;
    `aud` for `authenticated`; e `iss` for `<SUPABASE_URL>/auth/v1`.
    Fora disso, 401. A API nunca pergunta ao Supabase se o token vale
    (sem `getUser`): a decisão é pela chave pública.
14. O tenant vem do claim `app_metadata.tenant_id` do token (ADR-001 §6:
    "Claim do JWT, resolvido no guard, nunca do body"). Valor em body,
    query, header ou `user_metadata` (que a própria pessoa consegue
    alterar) é ignorado.
15. Com o token válido, o guard busca a membership (tenant do claim,
    `user_id` = `sub`). Sem claim de tenant ou sem membership: 403. Com
    membership, `tenant_id`, `user_id`, `membership_id` e papel ficam no
    contexto do request (`AsyncLocalStorage`, ADR-001 §3).
16. `GET /v1/eu` responde 200 com `{ "nome": <display_name>, "papel":
    <role> }` da membership do contexto.
17. Todo erro da API segue RFC 9457 (ADR-001 §4): `application/problem+json`
    com `type`, `title`, `status` e `detail`. O `type` é `about:blank`
    para erro genérico ou `/problemas/<motivo>` para recusa com motivo
    próprio (referência relativa, que a RFC permite). Os desta spec:
    `/problemas/nao-autenticado`, `/problemas/token-invalido` e
    `/problemas/sem-acesso`.
18. CORS: o preflight da origem do front com o cabeçalho `authorization`
    é aceito; de outra origem, a resposta não traz
    `access-control-allow-origin` (ADR-001 §7).

## Variáveis de ambiente (as três specs)
Regra: vibing/rules/secrets.md. O painel da Vercel é do usuário.

| variável | repositório | quem lê | `.env` local | `.env.example` | Vercel |
|---|---|---|---|---|---|
| `DATABASE_URL` | back | API, Prisma CLI | sim | sim | sim (usuário) |
| `DIRECT_URL` | back | Prisma CLI | sim | sim | não: o deploy não migra |
| `FRONT_ORIGIN` | back | API | sim | sim | já existe |
| `SUPABASE_URL` | back | API (JWKS, URL das fotos), seed, script de imagens | sim | sim | sim (usuário) |
| `SUPABASE_ANON_KEY` | back | API (o cliente do `supabase-js` exige uma chave; é a pública) | sim | sim | sim (usuário) |
| `SUPABASE_SERVICE_ROLE_KEY` | back | só seed e envio de imagens | sim | sim | não: o runtime da API não a usa |
| `SEED_PROFESSOR_SENHA`, `SEED_TESTE_EMAIL`, `SEED_TESTE_SENHA`, `SEED_TESTE_NOME` | back | só seed | sim | sim | não |
| `PIXABAY_API_KEY` | back | só script de imagens | sim | sim | não |
| `VITE_API_URL` | front | build | sim | sim | já existe |
| `VITE_SUPABASE_URL` | front | build (login) | sim | sim | sim (usuário) |
| `VITE_SUPABASE_ANON_KEY` | front | build (login): só a chave pública | sim | sim | sim (usuário) |

Conflito com secrets.md item 3 ("três lugares ou em nenhum"): as
variáveis só de seed e de script não vão para a Vercel, porque o deploy
não roda seed nem script, e a chave secreta no painel da API seria
exposição sem uso. Ficam em dois lugares.

## Decisões do agente (por delegação)
- D-01 Tenant no `app_metadata` do usuário do Supabase Auth, que vai
  para o token como claim. Só a chave secreta grava `app_metadata` (o
  seed da spec 005 grava); a pessoa não consegue alterar. Alternativas:
  Custom Access Token Hook (função no banco, migration, permissão para o
  role do Auth e configuração no painel); buscar a membership só pelo
  `sub`, sem claim de tenant (contraria a letra do ADR-001 §6).
  Limitação: um tenant por conta. O ADR-001 §6 prevê a mesma pessoa em
  mais de uma área; com este desenho, trocar de tenant exige
  administrador e novo login. Aceitável com um tenant só; revisar quando
  houver o segundo. As specs 005 e 006 dependem desta decisão.
- D-02 A conferência da assinatura usa `@supabase/supabase-js`
  (`auth.getClaims` com a chave já escolhida pela API, item 13). O
  ADR-001 §7 nomeia a biblioteca para a sessão no navegador; usá-la
  também no runtime da API é escolha do agente. Alternativas: `jose`
  (fora do ADR); WebCrypto à mão (criptografia escrita no projeto).
- D-03 Não existe "Entrar como Operações" (layout.md §5.1): o papel vem
  da membership (ADR-001 §6 e §7; ADR-002, "O que não decide").
- D-04 Conflito com a spec 001: o CA-01 dela diz "Nada pede login", e o
  "Fora do escopo" dela proíbe o link "Entrar". Sem o link, quem abre o
  endereço da Vercel (o professor, na correção) não acha o login. A spec
  001 é emendada (status, CA-01 e "Fora do escopo"), e o teste do CA-01
  dela passa a provar o CA-01 emendado.
- D-05 Sem cadastro, sem "esqueci minha senha" e sem troca de senha
  (ADR-001 §7, cadastro fechado). O cadastro público do Supabase Auth
  foi desligado pelo usuário no painel em 2026-11-10, por orientação do
  agente, e o agente conferiu (`disable_signup = true`).
- D-06 Não entram nesta spec: shadcn/ui (as telas usam Tailwind com os
  tokens do ADR-002; ele entra "na spec que precisar dele", ADR-002 §2)
  e Zustand (a sessão é do `supabase-js` e o resto é estado de servidor,
  no TanStack Query).
- D-07 Módulos novos com nomes em português (`autenticacao`, `eu`), como
  as rotas e os campos de resposta. A skill de TypeScript manda perguntar
  em módulo novo; o usuário estava ausente. Os models do Prisma
  continuam em inglês, como a spec 003 aprovou.
- D-08 Rate limit (ADR-001 §10). A spec 003 dizia que a tabela entraria
  "com o primeiro endpoint autenticado". Fica de fora desta prática, e
  helmet, pino e Sentry também (ADR-001 §9 e §10). É um desvio de spec
  aprovada e de ADR, registrado como pendência no ENTREGA.md (vibing/).
  O `POST /v1/emprestimos` da spec 006 é escrita e fica sem limite por
  pessoa além das regras de negócio.

## Para o usuário confirmar
D-01, D-02, D-04, D-07 e D-08. Cada uma tem alternativa defensável.

## Critérios de aceitação
Back: Jest e supertest. A JWKS é servida por um servidor local do
teste, com chave gerada no teste. Nunca o Supabase.
- CA-01 Rota protegida sem token responde 401.
- CA-02 Token assinado por outra chave com o mesmo `kid` (de curva
  elíptica ou RSA), expirado, com `aud` diferente de `authenticated`, com
  `iss` de outro projeto ou com carga que não é JSON responde 401.
- CA-03 Token HS256, com `alg: none` ou com `kid` que não está na JWKS
  responde 401, sem nenhuma chamada ao endpoint `/user` do Auth.
- CA-04 Token válido sem `app_metadata.tenant_id` (mesmo com um
  `user_metadata.tenant_id`), ou com tenant em que o `sub` não tem
  membership, responde 403.
- CA-05 Token válido com membership: `GET /v1/eu` responde 200 com o
  nome e o papel da membership.
- CA-06 O tenant é o do claim: um tenant diferente mandado no header
  `x-tenant-id`, na query `tenant_id` ou no body é ignorado. A mesma
  pessoa com membership só no tenant A e token com claim do tenant B
  recebe 403 e não vê nada de B (ADR-001 §8).
- CA-07 `GET /v1/health` responde 200 sem token.
- CA-08 Os erros 401 e 403 têm `content-type: application/problem+json`,
  os campos `type`, `title`, `status` e `detail`, e o `type` do item 17.
- CA-09 O preflight `OPTIONS` da origem do front pedindo o cabeçalho
  `authorization` é aceito; de outra origem, sem
  `access-control-allow-origin`.

Front: Playwright. Auth do Supabase e API interceptados.
- CA-10 Login aceito leva a `/app`, que mostra o nome e o papel vindos
  de `GET /v1/eu`.
- CA-11 Login recusado mostra "E-mail ou senha incorretos." e continua
  em `/entrar`.
- CA-12 E-mail em formato inválido ou senha vazia mostram mensagem no
  campo e não chamam o Auth.
- CA-13 `/app` sem sessão leva a `/entrar`.
- CA-14 "Sair" encerra a sessão deste aparelho (escopo `local`) e leva a
  `/entrar`; abrir `/app` depois leva a `/entrar`.
- CA-15 A chamada à API leva `Authorization: Bearer <access token da
  sessão>`.
- CA-16 Com a API respondendo 403 em `/v1/eu`, a área logada mostra a
  mensagem de conta sem acesso e o botão "Sair". Com 401, a sessão é
  encerrada e a tela vai para `/entrar`.
- CA-17 A landing tem o link "Entrar", que leva a `/entrar`, e continua
  abrindo sem sessão.
- CA-18 A CSP do `vercel.json` tem em `connect-src` a origem exata do
  projeto Supabase, sem curinga.

Os dois repositórios:
- CA-19 `npm run openapi:check` (back/) e `npm run api:check` (front/)
  terminam sem diferença.

Fora dos critérios, porque teste não toca produção: entrar de verdade no
endereço da Vercel. Depende do merge e das variáveis no painel (usuário);
a tarefa T6 registra quem fez e o resultado.

## Fora do escopo
- Cadastro, convite, recuperação e troca de senha (D-05).
- Abas, catálogo e empréstimos (spec 006).
- CASL: entra na spec 006, com a primeira regra de autorização.
- Rate limit, helmet, pino e Sentry (D-08).
- Papel de Operações e as abas dele.

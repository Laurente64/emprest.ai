# EMPREST.AI

Controle de empréstimo de equipamentos internos (notebooks, monitores,
cabos, câmeras), no lugar da planilha compartilhada. Para colaboradores,
que pedem e devolvem itens, e para Operações, que cadastra equipamentos
e registra devolução no balcão.
Negócio em vibing/docs/PRD.md. Stack decidida em vibing/docs/adr/001-stack.md.

## Onde olhar

O agente trabalha a partir de emprest.ai/, que não é repositório git.
Por isso o emprest.ai/CLAUDE.md não é versionado: é cópia de
vibing/setup/CLAUDE.root.md (ver vibing/README.md). Se ele faltar ou
estiver diferente da fonte, avise antes de começar a tarefa.

Dentro de emprest.ai/ há três repositórios separados:

- vibing/ — documentação e instruções. Este arquivo mora aqui.
- back/ — a API (repositório emprest.ai-back).
- front/ — a SPA (repositório emprest.ai-front).

Dentro de vibing/:

- docs/PRD.md — o que o negócio precisa. Não é spec.
- docs/adr/ — decisões técnicas já tomadas e o motivo delas.
- docs/specs/<NNN-funcionalidade>/ — spec, plano e tarefas de cada
  feature. Você rascunha os três; nada vale antes de eu aprovar.
- rules/ — como se trabalha aqui. Ver Procedimentos, abaixo.
- skills/<nome>/SKILL.md — como escrever um tipo de código. Ver
  Procedimentos, abaixo.
- agents/<nome>.md — instruções de um subagente. Ver Procedimentos,
  abaixo.
- layout.md — especificação de layout das telas. Arquivo de contexto,
  fora do git (.gitignore).
- handoffs/AAAA-MM-DD-nome-curto.md — o que uma sessão passa para a
  próxima. Fora do git (.gitignore). Ver rules/handoff.md.

## Comandos

Rodados pelo agente em 2026-09-23, nas specs 001 e 002. Os marcados
com "não rodado" estão no package.json mas ninguém executou ainda.

|                  | back/ | front/ |
|------------------|-------|--------|
| rodar o projeto  | `npm run start` · `npm run start:dev` não rodado | `npm run dev` não rodado; `npm run preview` roda |
| rodar os testes  | `npm test` (Jest), `npm run test:e2e` (supertest) e `npm run test:db` (Testcontainers; rodado em 2026-11-05, na spec 003; precisa do Docker ligado) | `npm test` (Playwright, sobe o preview sozinho) |
| buildar          | `npm run build` | `npm run build` |
| lint             | `npm run lint` | `npm run lint` |
| contrato da API  | `npm run openapi:emit` escreve · `npm run openapi:check` só confere | `npm run api:gen` escreve · `npm run api:check` só confere |
| gerar migration  | `npx prisma migrate diff --from-empty --to-schema-datamodel prisma/schema.prisma --script` (rodado em 2026-11-05, na spec 003; parte do banco vazio, então só serviu para a primeira migration) | não se aplica |

O `npm test` do front precisa do Chromium do Playwright, já instalado
nesta máquina. O `api:check` do front precisa do back/ clonado ao lado.

Só entram comandos que já rodaram nesta máquina. Se precisar de um que
não está aqui, proponha e espere — não o trate como verificado.

## Ambiente

- Banco: Supabase, um único projeto (ref owbckpmncpilerfmlxnz), e ele é
  PRODUÇÃO. Não há banco local nem banco de desenvolvimento.
- Toda alteração no Supabase é feita via migration do Prisma Migrate,
  aplicada só depois que eu aprovar o SQL. Ver vibing/rules/migration.md.
- Deploy: Vercel, um projeto por repositório, ligado ao repositório
  remoto. Push publica. main é produção.
- Git: você trabalha em branch, faz push da branch e abre PR. O merge é
  meu.
- Configuração vem de variável de ambiente. Ver o `.env.example` de cada
  repositório e vibing/rules/secrets.md.

## Precedência

Se dois artefatos discordarem sobre comportamento, a spec vence o código.
Se algo não estiver escrito em lugar nenhum, pergunte — não decida.

## Estado atual do projeto

Existe na main:
- vibing/: PRD, ADR-001, ADR-002, rules/, skills/, agents/ e specs 001,
  002 e 003.
- Os projetos na Vercel e o projeto Supabase.
- back/: NestJS com `GET /v1/health`, schema Zod, configuração validada
  no boot, CORS por allowlist, o `openapi.json` commitado e o adapter da
  Vercel. Prisma com `schema.prisma`, a migration do modelo de dados
  (tabelas com `tenant_id`, RLS e policies) e o seed. Testes em Jest,
  supertest e Testcontainers.
- front/: Vite com React e Tailwind, a rota `/` com a landing, o cliente
  gerado pelo orval, a CSP no `vercel.json` e oito testes de Playwright.
  `.env.example` com `VITE_API_URL`.
- O `.env` e o `.env.example` do banco moram no back/.

Ainda NÃO existe na main:
- Código da API que use o banco: nenhum módulo importa o Prisma ainda.
- Autenticação, guards, CASL e o contexto de tenant no request.
- Workflows de CI (GitHub Actions, previstos no ADR-001 §9). Enquanto
  não existirem, a checagem de contrato roda como comando, na mão.

Conferido com --version em 2026-09-21: Node 22.13.1; npm 11.12.0
(pnpm e yarn não instalados); Docker 29.3.0, só o cliente — o daemon
não foi conferido. Supabase CLI: não instalada.

Divergência aberta: pelo .env, o projeto Supabase está em us-west-2; o
ADR-001 §9 diz sa-east-1. Não corrija por conta própria — o ADR é meu.

## Procedimentos

As regras estão em vibing/rules/. Cada uma diz quando vale.

- Sempre: rules/restrictions.md e rules/checks.md.
- Antes de criar, alterar ou remover tabela, coluna, índice, constraint
  ou policy: rules/migration.md.
- Antes de criar ou usar variável de ambiente, instanciar o client do
  banco ou escrever código que lê configuração: rules/secrets.md.
- Quando eu pedir para salvar ou retomar um handoff: rules/handoff.md.

As skills estão em vibing/skills/. Leia a skill antes da tarefa que ela
cobre. Se ela conflitar com um ADR ou uma spec, PARE e avise, como
com qualquer regra (rules/restrictions.md).

- Antes de escrever, alterar ou revisar arquivo .ts ou .tsx em back/ ou
  front/: skills/typescript/SKILL.md.

Os subagentes estão em vibing/agents/. O Claude Code não os carrega de
lá, então eles não existem como tipo de agente. Para usar um, dispare um
subagente general-purpose, mande ler o arquivo e seguir o que está nele,
e passe as entradas que ele pede. O prompt é só isso: não copie nem
resuma o arquivo, nem acrescente lembretes ou pistas do que procurar.

- No passo 4 da ordem abaixo, ou quando eu pedir verificação, conferência
  de CA ou revisão contra a spec: agents/verificador-spec.md.

Ordem de uma funcionalidade:
1. Rascunhe a spec em vibing/docs/specs/<NNN-funcionalidade>/, a partir
   do PRD, com os critérios de aceitação numerados (CA-xx). PARE até eu
   aprovar.
2. Escreva o plano e as tarefas. PARE até eu aprovar.
3. Execute UMA tarefa, rode os checks, mostre o resultado e PARE.
4. Depois da última tarefa, antes de declarar a spec pronta, delegue a
   verificação a agents/verificador-spec.md. A spec só está pronta com
   "Veredito: PRONTA". Qualquer outro resultado volta para as tarefas, e
   quem corrige é o agente principal, nunca o verificador.

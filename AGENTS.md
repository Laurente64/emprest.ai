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
- layout.md — especificação de layout das telas. Arquivo de contexto,
  fora do git (.gitignore).

## Comandos

Rodados pelo agente em 2026-09-23, nas specs 001 e 002. Os marcados
com "não rodado" estão no package.json mas ninguém executou ainda.

|                  | back/ | front/ |
|------------------|-------|--------|
| rodar o projeto  | `npm run start` · `npm run start:dev` não rodado | `npm run dev` não rodado; `npm run preview` roda |
| rodar os testes  | `npm test` (Jest) e `npm run test:e2e` (supertest) | `npm test` (Playwright, sobe o preview sozinho) |
| buildar          | `npm run build` | `npm run build` |
| lint             | `npm run lint` | `npm run lint` |
| contrato da API  | `npm run openapi:emit` escreve · `npm run openapi:check` só confere | `npm run api:gen` escreve · `npm run api:check` só confere |

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
- Configuração vem de variável de ambiente. Ver vibing/.env.example e
  vibing/rules/secrets.md.

## Precedência

Se dois artefatos discordarem sobre comportamento, a spec vence o código.
Se algo não estiver escrito em lugar nenhum, pergunte — não decida.

## Estado atual do projeto

Existe:
- vibing/: PRD, ADR-001, rules/, specs 001 e 002.
- Os projetos na Vercel e o projeto Supabase.
- back/, na branch `api-saude`: NestJS com `GET /v1/health`, schema
  Zod, configuração validada no boot, CORS por allowlist, o
  `openapi.json` commitado e o adapter da Vercel. Testes em Jest e
  supertest.
- front/, na branch `landing-page`: Vite com React e Tailwind, a rota
  `/` com a landing, o cliente gerado pelo orval e oito testes de
  Playwright. `.env.example` com `VITE_API_URL`.
- O `.env` e o `.env.example` do banco agora moram no back/.

Nada disso está na main: são branches com PR aberto esperando merge.

Ainda NÃO existe:
- Banco, Prisma, schema, migrations e seed.
- Autenticação, multi-tenancy, CASL e RLS.
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

Ordem de uma funcionalidade:
1. Rascunhe a spec em vibing/docs/specs/<NNN-funcionalidade>/, a partir
   do PRD, com os critérios de aceitação numerados (CA-xx). PARE até eu
   aprovar.
2. Escreva o plano e as tarefas. PARE até eu aprovar.
3. Execute UMA tarefa, rode os checks, mostre o resultado e PARE.

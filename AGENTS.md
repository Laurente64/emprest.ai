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

Nenhum comando foi verificado nesta máquina ainda. back/ e front/ não
têm código nem package.json.

|                 | back/ | front/ |
|-----------------|-------|--------|
| rodar o projeto | —     | —      |
| rodar os testes | —     | —      |
| buildar         | —     | —      |

Só entram comandos que eu já rodei nesta máquina. Até lá, se precisar
de um comando, proponha e espere — não o trate como verificado.

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
- vibing/: PRD, ADR-001, rules/, .env.example.
- Os projetos na Vercel e o projeto Supabase.
- No remoto de front/ e de back/, só um commit de teste da Vercel:
  front/ tem um index.html "Hello Vercel"; back/ tem uma function
  Python em api/index.py (o ADR-001 §3 escolhe NestJS). As cópias
  locais ainda não foram sincronizadas.

Ainda NÃO existe:
- Código de aplicação.
- package.json, testes, schema Prisma, migrations, seed.
- Workflows de CI (GitHub Actions, previstos no ADR-001 §9).

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

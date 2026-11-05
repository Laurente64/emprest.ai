# 003 — Modelo de dados · Tarefas

Uma por vez. Cada tarefa termina com os checks de
vibing/rules/checks.md e PARA até o próximo "pode implementar".

## T1 — Branch
- Resolver a alteração não commitada no `tsconfig.json` do jeito que
  você escolher (D-04).
- Atualizar a main local com origin/main e criar a branch
  `modelo-de-dados` a partir dela, no back/.
- Pronto quando: `git status` limpo na branch nova.

## T2 — Prisma e Testcontainers instalados
- Instalar com versão fixa: `prisma` (dev) e `@prisma/client`, na
  versão de D-01, e `@testcontainers/postgresql` 11.14.0 (dev). Mostro
  o diff do `package.json` antes de commitar.
- `prisma/schema.prisma` só com `datasource` (`url` = `DATABASE_URL`,
  `directUrl` = `DIRECT_URL`) e `generator`, sem model.
- Pronto quando: `npx prisma validate`, build, lint e `npm test`
  passam.

## T3 — DDL pretendido (PARA)
- Escrever na resposta o DDL completo: tabelas, enums, índices,
  CHECKs, FKs (com D-02 e D-03), RLS, policies e o trigger da
  `audit_log`.
- Nenhum arquivo muda nesta tarefa.
- Pronto quando: você aprova ou corrige o DDL (migration.md, passo 1).

## T4 — Infra de teste de banco
- Conferir `docker info`.
- Config do Jest para teste de banco e o script `npm run test:db`.
- Setup que sobe o Postgres no Testcontainers, cria o `auth.users`
  mínimo e os roles `anon` e `authenticated`, roda
  `prisma migrate deploy` contra o container e derruba tudo no fim.
- Um teste de fumaça: o container sobe e responde.
- Pronto quando: `npm run test:db` passa com o teste de fumaça.

## T5 — Schema e migration
- Models no `schema.prisma`, conforme o DDL aprovado na T3.
- Mostrar o comando de geração (`prisma migrate diff`) e esperar o OK
  antes de rodar. Gerar o `migration.sql` e acrescentar o SQL bruto.
- Testes de CA-01 a CA-11 (e CA-02b, se D-03 for aprovado) contra o
  container.
- Pronto quando: `npm run test:db`, `npm test`, build e lint passam.
  Nada é aplicado em produção nesta tarefa.

## T6 — Seed
- `prisma/seed.ts` idempotente, criando o tenant `suporte_ti`, e a
  configuração `prisma.seed` no `package.json`.
- Teste de CA-12 contra o container: rodar o seed duas vezes deixa um
  tenant só.
- Pronto quando: `npm run test:db`, build e lint passam.

## T7 — SQL final (PARA)
- Mostrar o `migration.sql` inteiro.
- Pronto quando: você aprova o SQL (migration.md, passo 7).

## T8 — Aplicar em produção
Cada comando só roda depois do seu OK:
- `npx prisma migrate status` contra produção (só leitura).
- `npx prisma migrate deploy` contra produção. Colo a saída.
- `npx prisma db seed` contra produção. Colo a saída.
- Pronto quando: o SQL aplicado é o aprovado (rules/checks.md, passo 2),
  `migrate status` não mostra pendência e o
  arquivo aplicado é o mesmo commitado.

## T9 — Push e PR
- Push da branch `modelo-de-dados` e link do PR (não há `gh` nesta
  máquina).
- Pronto quando: o PR está aberto. O merge é seu.

## Fora do back/ (vibing/, separado)
- AGENTS.md: atualizar "Estado atual" (a spec 002 já está na main) e
  incluir na tabela de Comandos os comandos que rodarem nesta spec.
- rules/migration.md: o risco de `migrate diff` apagar objetos de SQL
  bruto (ver plano).

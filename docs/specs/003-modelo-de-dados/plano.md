# 003 — Modelo de dados · Plano

Status: APROVADO em 2026-10-26.
Spec: spec.md (aprovada em 2026-10-22). Tarefas: tarefas.md.

## Decisões de 2026-10-26
Respostas do usuário às decisões abaixo:
- D-01 Prisma 6.19.3, versão fixa.
- D-02 FK para `auth.users` como SQL bruto na migration. O teste cria
  um `auth.users` mínimo e os roles `anon` e `authenticated`.
- D-03 Toda FK entre tabelas de domínio inclui o `tenant_id`. Entra o
  CA-02b.
- D-04 A alteração no `tsconfig.json` do back/ vai para `git stash`
  antes da branch nova.
- D-05 Docker fica, só para teste local (ver D-05 abaixo).

## O que mudou na execução (2026-11-05)
O usuário pediu "todo o código no lugar, rápido" depois de ver o DDL da
T3. Isso foi lido como autorização para escrever o código local, e as
T4, T5 e T6 foram feitas de uma vez, sem parar entre elas. Nada foi
commitado. Não autoriza nada em produção: T7 e T8 continuam valendo.

Diferenças em relação ao que estava escrito. As da spec foram levadas para
ela em 2026-11-05, depois da primeira verificação (ver spec.md):
- `@testcontainers/postgresql` é 11.14.0, não 12.2.1: a linha 12 puxa
  `undici` 8, que exige Node >= 22.19, e esta máquina tem 22.13.1.
- `loans` referencia `equipments` por uma única FK de três colunas
  (`equipment_id`, `tenant_id`, `equipment_tracking`), no lugar de
  (id, tracking) mais (id, tenant_id). A proteção é a mesma (CA-02b e
  CA-04b).
- `audit_log.id` é `bigserial`, não identity: o Prisma não gera identity.
- `_prisma_migrations` também recebe RLS e policy.
- `recusado` também exige `decided_at` e `decided_by_membership_id`.
- `display_name` é obrigatório.
- `tracking` só é travado pelo banco depois que o equipamento tem
  empréstimo; antes disso, é o service que garante.
- `updated_at` é preenchido pelo Prisma, não por trigger.
- `memberships.user_id` usa ON DELETE RESTRICT. Quando o ADR decidir a
  exclusão de dados (LGPD), isso precisa ser revisto.
- O `migrate diff` gerou `CREATE SCHEMA IF NOT EXISTS "public"`, retirado
  da migration por não fazer nada e não estar no DDL.
- `prisma` entrou no `exclude` do `tsconfig.build.json`. Sem isso, o
  `prisma/seed.ts` mudaria a saída para `dist/src/` e quebraria o
  `api/index.js`, que importa de `../dist/`.

Limite do teste: o Postgres do teste é o 17, com uma imitação do Supabase
(`test/db/banco-de-teste.ts`). A versão de produção não foi conferida, e o
`auth.users` real só é exercitado na T8.

## Ponto de partida do back/ (conferido em 2026-10-22)
- origin/main já tem a spec 002 (PRs #1 e #2 mergeados). A main local
  está um commit atrás.
- A cópia local está na branch `tsconfig-sem-baseurl`, com uma
  alteração não commitada no `tsconfig.json` que REMOVE `"strict":
  true`. Não é desta spec e não sei de onde veio (ver D-04).
- Não há Prisma, schema nem migration.
- O daemon do Docker não está rodando (`docker info` falhou). Sem ele,
  Testcontainers não sobe e nenhum CA é verificável (ver D-05).

## Decisões que preciso de você antes de começar

### D-01 Versão do Prisma
O ADR-001 §5 descreve o Prisma com `url` e `directUrl` no schema e
`?pgbouncer=true&connection_limit=1` na URL. Isso é o Prisma 6. No
Prisma 7 mudou:
- a URL sai do schema e vai para um `prisma.config.ts`;
- o client passa a exigir um driver adapter (`@prisma/adapter-pg` +
  `pg`), duas bibliotecas fora do ADR;
- `pgbouncer=true` e `connection_limit=1` deixam de ser lidos da URL. O
  limite de 1 conexão vira opção do pool do adapter.

| Opção | Consequência |
|---|---|
| **Prisma 6.19.3, versão fixa (recomendada)** | Segue o ADR ao pé da letra, sem biblioteca nova. A linha 6 é a anterior à atual. |
| Prisma 7.10.1, versão fixa | Versão estável atual. Exige aceitar `@prisma/adapter-pg` e `pg` e reescrever como o ADR §5 é cumprido. Isso é seu (o ADR é seu). |
| Prisma 8 | Ainda em release candidate (`8.0.0-rc.25`), apesar de estar com a tag `latest` no npm. Descartado. |

### D-02 FK de `memberships.user_id` para `auth.users`
O schema `auth` é do Supabase e não existe num Postgres comum do
Testcontainers.
- **Recomendado:** a FK entra como SQL bruto na migration. O setup do
  teste cria, antes de migrar, um `auth.users (id uuid primary key)`
  mínimo e os roles `anon` e `authenticated`, imitando o Supabase. Esse
  SQL de setup existe só no teste e nunca vai para produção.
- Alternativa: sem FK. `user_id` vira um uuid solto, e uma membership
  pode apontar para usuário apagado.

### D-03 Isolamento de tenant nas FKs (além da spec)
A spec não diz isto. Proponho que toda FK entre tabelas de domínio
inclua o `tenant_id`: por exemplo, `loans (equipment_id, tenant_id) →
equipments (id, tenant_id)`. Assim o banco recusa um empréstimo de um
tenant apontando para equipamento de outro. Custa um índice único
(id, tenant_id) em cada tabela referenciada. Se aprovar, entra como
CA-02b.

### D-04 A alteração não commitada no `tsconfig.json` do back/
Ela tira o `strict` do TypeScript. Trocar de branch leva a alteração
junto, então antes de criar a branch desta spec preciso saber o que
fazer com ela:
- guardar com `git stash` e deixar a decisão para depois;
- você commita ou descarta antes.

Eu não descarto alteração que não é minha.

### D-05 Docker
Decidido em 2026-11-05: o Docker fica, só para teste local.

O motivo vem da aula 9 (`rules/tests.md`, ponto de partida): teste roda
contra o banco local, nunca contra o remoto, e o teste de política de
acesso não pode usar banco mocado. A aula não cita o Docker pelo nome:
ele é o meio de ter esse banco local aqui, pelo Testcontainers que o
ADR-001 §8 e o rules/checks.md já exigem. Nada de Docker vai para
produção ou para a Vercel.

Ligue o Docker Desktop. Eu confiro com `docker info` no início da T4.

## Stack (subconjunto do ADR-001)

| Item | Uso nesta spec |
|---|---|
| Prisma + Prisma Migrate (§5) | schema e a única migration. Versão: D-01 |
| SQL bruto na migration (§5) | RLS, policies, CHECKs, índices parciais, FKs compostas, FK para `auth.users`, trigger da `audit_log` |
| `tenant_id` em toda tabela (§6) | `memberships`, `equipments`, `loans`, `audit_log` |
| Seed idempotente (§5) | `prisma/seed.ts`, cria `suporte_ti` |
| Testcontainers (§8) | `@testcontainers/postgresql` 11.14.0, versão fixa, com Postgres real |
| Jest (§8) | os testes de banco, numa config separada |

Fica fora, como a spec diz: `PrismaService`, módulos Nest, endpoints,
auth, CASL e rate limit. Nenhum código de `src/` instancia o client do
banco nesta spec.

## Nomes
As tabelas e colunas estão em inglês e os valores de enum em
português, como a spec aprovou. No Prisma, os models seguem as tabelas
(`Tenant`, `Membership`, `Equipment`, `Loan`, `AuditLog`), com
`@@map`/`@map` para o nome em snake_case no banco. A skill de
TypeScript pede para perguntar o idioma de módulo novo: esta spec não
cria módulo Nest, então a pergunta fica para a primeira spec de
funcionalidade.

## Como a migration é gerada
`prisma migrate dev` está proibido (vibing/rules/migration.md), porque
pode propor reset. Proponho gerar o SQL sem tocar em banco nenhum:

```
npx prisma migrate diff --from-empty --to-schema-datamodel prisma/schema.prisma --script
```

A saída vai para `prisma/migrations/<timestamp>_modelo_de_dados/
migration.sql`. Depois eu acrescento o SQL bruto no fim do mesmo
arquivo. Esse comando nunca rodou nesta máquina: na T3 eu mostro o
comando e espero o seu OK antes de rodar.

**Risco conhecido:** o Prisma não modela índice parcial, CHECK, FK
composta para fora do schema, nem policy. Numa migration futura, o
`migrate diff` pode propor apagar esses objetos, porque eles não
existem no `schema.prisma`. Toda migration futura precisa ser lida
procurando `DROP INDEX`, `DROP CONSTRAINT` e `DROP POLICY` que ninguém
pediu. Proponho registrar isso em vibing/rules/migration.md, numa
tarefa separada do vibing/.

## Como os testes rodam
- Um script novo, `npm run test:db`, com config própria do Jest. Ele
  sobe um Postgres no Testcontainers, cria o `auth` mínimo e os roles
  (D-02), aplica a migration com `prisma migrate deploy` apontando para
  o container e roda os testes de CA-01 a CA-12.
- O `npm test` continua sem precisar de Docker.
- Aplicar a migration no container antes de produção é também o ensaio
  do SQL: se ele quebra, quebra no container.

## Produção
Ordem, cada passo parando em você:
1. DDL pretendido na resposta (migration.md, passo 1). Você aprova.
2. Migration gerada e testada no container. Mostro o SQL inteiro
   (passo 7). Você aprova.
3. `npx prisma migrate status` contra produção, só leitura, para
   confirmar que o banco não tem migration nenhuma.
4. `npx prisma migrate deploy` contra produção, usando o `.env` do
   back/. Colo a saída (rules/checks.md, passo 2).
5. `npx prisma db seed` contra produção, para criar `suporte_ti`.
   Comando aprovado à parte (decisão P-10 da spec).

O ADR-001 §9 diz que a migration roda num job do GitHub Actions. Esse
CI ainda não existe, então esta primeira migration é aplicada à mão,
pelo comando que o ADR nomeia. A lacuna é a mesma da spec 002.

## Variáveis de ambiente
`DATABASE_URL` e `DIRECT_URL` já estão no `.env.example` do back/.
Nesta spec só o Prisma CLI as lê, então elas não entram no `envSchema`
de `src/config/env.ts`. Se entrassem, a API deixaria de subir na
Vercel sem elas. Entram com o primeiro endpoint que usar o banco. Não
há variável nova e nada a fazer no painel da Vercel.

## Deploy
O push da branch gera um preview na Vercel. O build não usa o banco:
`prisma generate` não precisa do valor da URL. O merge é seu.

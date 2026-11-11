# 005 — Dados iniciais · Plano

Status: APROVADO em 2026-11-11 pelo agente, por delegação do usuário
(spec 004, "Aprovação"). Tarefas: tarefas.md.

## O que muda na spec 003
A spec 003 diz que o seed "não cria usuário nem equipamento" (P-10).
Esta spec muda isso. O teste do CA-12 da spec 003 rodava `prisma db
seed` pelo CLI; o CLI carrega o `back/.env`, e o seed novo chamaria a API
de administração do Supabase de verdade. Por isso o teste passa a chamar
a função do seed direto, com a imitação do Auth.

## Migration
- `schema.prisma`: `imagePath String? @map("image_path")` em `Equipment`.
- DDL pretendido: `ALTER TABLE "equipments" ADD COLUMN "image_path" TEXT;`
  Coluna nula, sem default: as linhas existentes ficam com nulo
  (migration.md, item 5). O código da main não lê a coluna, então nada
  quebra antes do merge (item 6).
- Geração sem banco: `npx prisma migrate diff --from-schema-datamodel
  <schema.prisma da main> --to-schema-datamodel prisma/schema.prisma
  --script`. O comando é novo nesta máquina; entra na tabela Comandos
  depois de rodar. A saída é lida procurando `DROP` que ninguém pediu
  (plano da spec 003, risco conhecido).
- Aprovação do SQL: o usuário delegou; um subagente revisor de contexto
  limpo confere o SQL contra esta spec e o migration.md antes do deploy.

## Seed

| arquivo | papel |
|---|---|
| `prisma/dados/equipamentos.json` | Os 12 da lista, copiados do enunciado. |
| `prisma/semear.ts` | `semear(prisma, authAdmin, config)`: tenant, equipamentos, contas e memberships, idempotente (spec, item 9). |
| `prisma/auth-admin.ts` | Interface da administração do Auth e a implementação com `supabase-js` (`auth.admin`, chave secreta). |
| `prisma/config-do-seed.ts` | Lê e valida o ambiente com Zod; falha antes de escrever (spec, item 10). |
| `prisma/seed.ts` | Só liga as peças e roda. É o que `prisma db seed` chama. |

Conta existente é encontrada por e-mail na lista de usuários do Auth
(`auth.admin.listUsers`, paginada; são poucas contas).

## Imagens (skill imagens-equipamentos)

| arquivo | papel |
|---|---|
| `scripts/imagens/pixabay.ts` | Busca, filtro de tags e verificação de origem. |
| `scripts/imagens/foto.ts` | `sharp`: 800×600 com `fit: cover`, JPEG; leitura de dimensões e sha256. |
| `scripts/imagens/registros.ts` | Tipos, leitura e validação de `creditos.json` e `revisao.json`. |
| `scripts/imagens/preparar.ts` | CLI `npm run imagens:preparar`. |
| `scripts/imagens/enviar.ts` | CLI `npm run imagens:enviar`: exige terminal interativo e o ref do projeto digitado; cria o bucket público se faltar; envia com upsert; confere a URL pública. Fala com o Storage por `fetch`, sem biblioteca nova. |

- `sharp` como devDependency, versão fixa (ADR-003).
- `scripts/` fora do build (`tsconfig.build.json`), como `prisma/`.
- O Jest unitário passa a olhar `src/`, `scripts/` e `prisma/`.
- Revisão visual: feita pelo agente nesta noite, por delegação, e
  registrada como "agente" no `revisao.json`. Vale até o usuário conferir.

## Testes
- `test/db/dados-iniciais.db-spec.ts` (Testcontainers): CA-01 a CA-07,
  com a imitação do Auth gravando em `auth.users` do banco de teste (a FK
  de `memberships.user_id` exige a linha).
- `scripts/imagens/*.spec.ts` (Jest): CA-08 a CA-13. Os testes do envio
  chamam só a conferência que roda antes de qualquer rede; nenhum teste
  carrega o `back/.env`.

## Produção
O usuário autorizou o agente a rodar as escritas de produção, com
registro verdadeiro (2026-11-10). Ordem:
1. `npx prisma migrate status` (só leitura).
2. `npx prisma migrate deploy`. Saída colada nas tarefas.
3. `npx prisma db seed`, duas vezes. Saídas coladas.
4. Contagem só de leitura (12 equipamentos, situação por patrimônio, 2
   memberships, 0 empréstimos).
5. Envio das fotos: `npm run imagens:enviar`, por uma pessoa no próprio
   terminal. O agente não roda: é o requisito "o agente não consegue
   enviar" da skill. Fica na lista do usuário.

## Variáveis de ambiente
Só no `back/.env` e no `.env.example`: o deploy não roda seed nem
script de imagem, então nada disso vai para a Vercel.
`PIXABAY_API_KEY`, `SUPABASE_SERVICE_ROLE_KEY`, `SEED_PROFESSOR_SENHA`,
`SEED_TESTE_EMAIL`, `SEED_TESTE_SENHA`, `SEED_TESTE_NOME`.

---
description: Procedimento para qualquer mudança de schema
globs: ["back/prisma/**", "back/src/modules/**"]
alwaysApply: false
---

# Mudança de schema
> leitor: agente

O único projeto Supabase é produção. Não há banco local nem de
desenvolvimento: toda migration que você aplica cai em dado real.
O dono do schema é o Prisma Migrate (ADR-001 §5).

## Quando
Qualquer tarefa que precise criar, alterar ou remover tabela, coluna,
índice, constraint ou política de acesso — inclusive quando a mudança
parecer trivial.

## Procedimento
1. Antes de gerar qualquer coisa: escreva o DDL pretendido na resposta
   e PARE. Eu aprovo ou corrijo.
2. Gere a migration com o Prisma Migrate, em back/prisma/migrations/.
   Uma migration por tarefa. RLS, policies e triggers entram como SQL
   bruto dentro dessa mesma migration (ADR-001 §5). O comando de geração
   ainda não foi verificado nesta máquina: diga qual vai usar e espere.
3. Toda tabela de domínio nasce com a coluna tenant_id (ADR-001 §6).
4. Toda tabela nova nasce com Row Level Security habilitada e pelo
   menos uma policy explícita na mesma migration. Tabela sem policy
   não entra no repositório.
5. Se a tabela já tem dado, diga o que acontece com as linhas
   existentes. Coluna obrigatória nova precisa de default ou de um
   passo de preenchimento.
6. A migration chega em produção antes do merge do PR: por um tempo, o
   código que está em main roda contra o schema novo. Ela não pode
   quebrar esse código. Remover ou renomear coluna ou tabela, só com
   pedido explícito meu.
7. Mostre o SQL gerado, inteiro, e PARE. Só aplique depois que eu
   aprovar.
8. Aplique com `prisma migrate deploy` (o comando do ADR-001 §9), usando
   o .env. Cole a saída.
9. Commite a migration na branch da tarefa.

## Verificação
A migration aplicada em produção é exatamente o SQL que eu aprovei, e
o mesmo arquivo está commitado na branch do PR.

## Não faça
- Não altere schema pelo Studio nem por SQL avulso. O que não está em
  migration não existe.
- Não edite migration que já foi aplicada. Escreva a próxima.
- Não rode `prisma migrate reset`: ele apaga o banco, e o banco é
  produção.
- Não rode `prisma db push`: ele altera o schema sem migration.
- Não rode `prisma migrate dev`: ele pode propor reset do banco.
- Não use a Supabase CLI para criar ou aplicar migration. O ADR-001
  descartou ter dois donos de schema.
- Não aplique migration sem a minha aprovação do SQL, mesmo que eu já
  tenha aprovado o DDL do passo 1.

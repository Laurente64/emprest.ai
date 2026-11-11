# 006 — Catálogo e solicitar empréstimo · Plano

Status: APROVADO em 2026-11-11 pelo agente, por delegação do usuário
(spec 004, "Aprovação"). Tarefas: tarefas.md.

## Back

| módulo | peças |
|---|---|
| `src/modules/autorizacao/` | `fabricaDeHabilidades(usuario)` com `@casl/ability` (ADR-001 §7): `read Equipment`, `create Loan`, `read Loan { borrowerMembershipId }`. |
| `src/modules/equipamentos/` | `GET /v1/equipamentos`: controller, service, repository, schemas Zod. Situação calculada no service a partir de `condition` e do empréstimo em andamento (spec 003, "situação do catálogo"). |
| `src/modules/emprestimos/` | `POST /v1/emprestimos` e `GET /v1/emprestimos/meus`. Regras puras em `regras.ts` (`motivoDaRecusa`), sem banco e sem relógio próprio: o `agora` vem de fora (skill de TypeScript). |
| `src/modules/eu/` | `limiteDeEmprestimos`. |
| `src/storage/url-publica.ts` | `${SUPABASE_URL}/storage/v1/object/public/equipamentos/<image_path>`. |

Pedido, dentro de uma transação do Prisma:
1. `SELECT … FOR UPDATE` na membership (serializa os pedidos da pessoa).
2. Equipamento do tenant com `FOR UPDATE`. Não existe ou `baixado`: 404.
3. Conta os empréstimos em andamento da pessoa e procura atraso
   (`status IN ('ativo','devolucao_solicitada') AND due_at < now()`).
4. `motivoDaRecusa(...)` decide, na ordem da spec.
5. Insere o empréstimo `ativo` e a linha `loan.request` na `audit_log`.
6. Violação do índice `loans_one_open_per_unit` (P2002) vira 409
   `/equipamento-indisponivel`.

Paginação por cursor: `take: limite + 1`, cursor = id do último item,
ordem estável (patrimônio no catálogo; `due_at` e id em "meus").

Dependência nova: `@casl/ability`, versão fixa (no ADR-001 §7).

### Testes
- `test/db/emprestimos.db-spec.ts`: a app Nest com supertest, o banco no
  Testcontainers (migrations + seed de teste próprio, com dois tenants)
  e a JWKS local da spec 004. CA-01 a CA-12. A concorrência usa o pool
  do Prisma do teste, que abre mais de uma conexão.
- `src/modules/emprestimos/regras.spec.ts`: ordem das recusas.

## Front

| peça | o que faz |
|---|---|
| `src/pages/Catalogo.tsx` | Cabeçalho, avisos, grade, crédito. |
| `src/componentes/CardDeEquipamento.tsx` | Foto, textos, pill e botão por situação (spec, item 10). |
| `src/componentes/DialogoSolicitar.tsx` | Diálogo de confirmação e erro com o `detail` da API. |
| `src/componentes/Aviso.tsx` | Aviso de confirmação (toast do layout.md §5.6, 2600 ms). |
| `src/pages/MeusEmprestimos.tsx` | Linhas e estado vazio. |
| `src/datas.ts` | DD/MM/AAAA e hoje + 14 dias. |
| `src/index.css` | Tokens de situação do ADR-002 §3. |
| `vercel.json` | `img-src` ganha a URL do Supabase. |

Estado de servidor no TanStack Query: consultas `eu`, `equipamentos` e
`meus`; o pedido invalida as três.

### Testes
Playwright, API interceptada com estado em memória por teste. CA-13 a
CA-20.

## Produção
Depois do merge e das variáveis na Vercel (usuário): o roteiro "Antes de
entregar" do enunciado com a conta de teste, e a conferência só de
leitura de que o professor tem zero empréstimos.

## Deploy
Merge na ordem: back 004 → back 005 → back 006 → front 004 → front 006.
Cada PR de back leva os commits do anterior até o anterior ser
mergeado; depois disso, o diff mostra só o que é dele.

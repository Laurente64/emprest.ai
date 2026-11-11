# 006 — Catálogo e solicitar empréstimo · Tarefas

Aprovações delegadas pelo usuário (spec 004, "Aprovação"). O resultado
de cada tarefa fica registrado no fim.

## T1 — Back: catálogo
- Branch `spec-006-solicitar-emprestimo` no back/, a partir de
  `spec-005-dados-iniciais`.
- `@casl/ability`, autorização, `GET /v1/equipamentos`, URL pública.
- Testes CA-01, CA-02 (catálogo), CA-12.

## T2 — Back: pedir e meus empréstimos
- `POST /v1/emprestimos`, `GET /v1/emprestimos/meus`, `limiteDeEmprestimos`.
- Testes CA-02 (pedido e meus), CA-03 a CA-11. `openapi.json` regerado.
- Pronto quando: `npm test`, `npm run test:e2e`, `npm run test:db`,
  build, lint e `openapi:check` passam.

## T3 — Front: catálogo e meus empréstimos
- Branch `spec-006-solicitar-emprestimo` no front/, a partir de
  `spec-004-login`. Cliente regerado.
- Testes CA-13 a CA-21.
- Pronto quando: `npm test`, build, lint e `api:check` passam.

## T4 — Verificação
- Verificador com back/ e front/ nas branches da spec. Rodada salva.
- Prova do verificador com erro plantado (enunciado, Entrega 1): um
  commit que quebra um CA, a rodada que reprova, o commit que desfaz e a
  rodada que aprova.

## T5 — Push, PR e produção
- Push e links de PR.
- Depois do merge e do painel da Vercel (usuário): roteiro "Antes de
  entregar" com a conta de teste.

## Registro
- T1 e T2 (2026-11-11) — back `spec-006-solicitar-emprestimo`, feitas
  juntas num worktree (o back/ estava preso à verificação da 004):
  276bb81 (catálogo, pedido, meus, CASL, `ZodValidationPipe` global),
  1e53e14 (testes da 004 aceitam o campo novo de `/v1/eu`), 7d6289c
  (busca da JWKS compartilhada, observação da rodada 2 da 004), 3430233
  (`min(1)` no lugar de `positive()`: o orval rejeitava o
  `exclusiveMinimum`). `npm test` 31 passed; `npm run test:e2e` 13
  passed; `npm run test:db` 124 passed; build e lint sem erro;
  `openapi:check` em dia. CASL: 6.8.1, não 7.1.2 (publicada um dia antes;
  a linha 7 tinha duas semanas).
- T3 (2026-11-11) — front `spec-006-solicitar-emprestimo` 74f0e54, feita
  por um subagente (fork) só no front/, com o cliente gerado do
  `openapi.json` do worktree. `npm test` 28 passed (10 novos, CA-13 a
  CA-21); build e lint sem erro. Detalhes de tela que a spec não dizia
  viraram o item 17 (emenda). Testes da 004 ajustados no branch: o CA-10
  espera o título "Catálogo" no lugar de "Bem-vindo"; helpers em
  `e2e/apoio.ts`. A lista dos 12 está copiada no teste: ela não
  acompanha mudanças no `equipamentos.json`.

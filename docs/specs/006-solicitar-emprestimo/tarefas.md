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
(preenchido durante a execução)

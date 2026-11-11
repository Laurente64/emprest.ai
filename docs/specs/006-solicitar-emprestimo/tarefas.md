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
- Ajustes antes da verificação (back): 83d9377 (`npm run lint` cobre
  `scripts/` e `prisma/`, observação da rodada 2 da 005), e1729eb
  (`.gitattributes` com `openapi.json` em LF: com `core.autocrlf`, o
  checkout no Windows punha CRLF e o `openapi:check` acusava divergência
  sem mudança de contrato), 4c28ad0 (documento OpenAPI 3.1: o Zod descreve
  nulo como `type: [..., "null"]`, inválido em 3.0; o orval avisava e
  gerava tipos sem `null`). Front 30dfcb0: cliente regerado.
- T4 (2026-11-11) — verificação.
  - Rodada 1 (back 4c28ad0, front 30dfcb0): NÃO PRONTA. CA-10: o 404 saía
    com `/problemas/equipamento-nao-encontrado`, e o item 2.1 exige
    `about:blank` (a spec vale; o código mudou). CA-01: o teste não
    conferia a categoria. CA-04: "não grava nada" não olhava a
    `audit_log`. Correção (agente principal), back 241a7ce; o CA-09 ganhou
    o caso `emAtraso` verdadeiro.
  - Prova com erro plantado (enunciado, Entrega 1, "Provado"): back
    7bcb40a muda o limite padrão de 3 para 5, o número do briefing que o
    PRD recusa. O verificador recebeu as mesmas entradas de sempre, sem
    pista. Rodada 2 (back 7bcb40a, front 30dfcb0): NÃO PRONTA, com CA-06,
    CA-08 e CA-11 REPROVADOS pela causa certa e os demais ATENDIDOS,
    inclusive as correções da rodada 1. A mensagem do commit diz que é
    erro plantado; a reprovação veio dos testes vermelhos, não da
    mensagem. Desfeito em back 9ec9d0e (git revert).
  - Rodada 3 (back 9ec9d0e, front 30dfcb0): PRONTA. O arquivo da rodada
    foi salvo primeiro como resumo, contra o passo 4 do AGENTS.md; em
    2026-11-11 foi trocado pela resposta inteira do verificador.
- Depois da rodada 3, SEM rodada nova do verificador (pendência):
  - back e3addb6 (em spec-004-login, levado por merge a 005 e 006): o
    build passa a rodar `prisma generate` antes do `nest build`, e o
    `schema.prisma` ganha `binaryTargets` com `rhel-openssl-3.0.x`. O
    build da Vercel falhava (TS2339 em `imagePath`) porque o Prisma
    Client não era gerado. Não é migration: não muda o banco.
  - front 62d9bc4 (branch ajuste-foto-meus-emprestimos): a foto da linha
    de "Meus empréstimos" ocupava a linha inteira (876 px), porque o
    componente aplicava `w-full` sempre. A largura de 96 px (`w-24`) foi
    decisão do subagente na T3 e não estava na spec: entra como emenda
    no item 17. O teste do CA-19 passa a medir a foto. Autorizado pelo
    usuário em 2026-11-11.

# 005 — Dados iniciais · Tarefas

Aprovações delegadas pelo usuário (spec 004, "Aprovação"). O resultado
de cada tarefa fica registrado no fim.

## T1 — Fotos (skill)
- Branch `spec-005-dados-iniciais` no back/, a partir de `spec-004-login`.
- `sharp`, scripts de preparar e enviar, `equipamentos.json`.
- Rodar `imagens:preparar`, revisão visual, repetir até as 12 aprovadas.
- Testes CA-08 a CA-13.
- Pronto quando: `npm test`, build e lint passam.

## T2 — Migration
- `image_path` no `schema.prisma`, `migrate diff`, revisão do SQL.
- Teste CA-01.
- Pronto quando: `npm run test:db` passa.

## T3 — Seed
- `semear`, `auth-admin`, `config-do-seed`, `seed`; CA-12 da spec 003
  passa a chamar a função.
- Testes CA-02 a CA-07.
- Pronto quando: `npm test`, `npm run test:db`, build e lint passam.

## T4 — Produção
- Os passos 1 a 4 de "Produção" no plano, com as saídas aqui.
- `git log -p --all -S<senha padrão>` nos três repositórios, lendo a
  senha do `back/.env`: saída vazia.
- O passo 5 (envio das fotos) e o login do professor na Vercel ficam com
  o usuário.

## T5 — Verificação, push e PR
- Verificador com back/ em `spec-005-dados-iniciais`. Rodada salva em
  verificacoes/. Push e link do PR.

## Registro
(preenchido durante a execução)

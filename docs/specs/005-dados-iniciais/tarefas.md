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
- T1 (2026-11-11) — back `spec-005-dados-iniciais` 9cf8015. `npm test` 17
  passed (CA-08 a CA-13); build e lint sem erro. Execução da skill:
  - Primeira rodada: nenhuma foto passou, porque o script só aceitava
    download de `cdn.pixabay.com` e o `largeImageURL` vem de
    `pixabay.com/get/…`. Host corrigido no script e no SKILL.md.
  - Revisão visual feita pelo agente, por delegação do usuário: 12
    aprovadas e 85 reprovadas (28 em tamanho final, 57 em folha de contato
    de pré-triagem). Motivos no `revisao.json`: pessoa, marca legível
    (Apple, Lenovo, Olympus, Canon, Fujifilm, Pentax, Rollei, iiyama…),
    tela com conteúdo, texto, e foto que não mostra o equipamento
    (caderno de papel no lugar de notebook, tablet no lugar de monitor).
  - Com o que as reprovações mostraram, o filtro do script ganhou tag
    exigida por categoria (“notebook” sozinho também é caderno), mais
    marcas bloqueadas, cache de 24 h em disco e até 3 páginas de busca com
    o mesmo termo da lista.
  - Três aprovações têm `observacao`: marcação genérica do produto (letras
    do teclado, "WEB CAMERA" na lente da webcam, "18-55 OIS" na lente da
    câmera). A regra foi escrita em references/revisao-visual.md antes da
    aprovação. É decisão do agente: o usuário confere.
- T2 (2026-11-11) — back d70fabd. `npm run test:db` 105 passed (CA-01).
  Comando de geração: `npx prisma migrate diff --from-schema-datamodel
  <schema da main> --to-schema-datamodel prisma/schema.prisma --script`;
  saída igual ao DDL do item 4.
- T3 (2026-11-11) — back 405edb2. `npm run test:db` 112 passed (CA-02 a
  CA-07 e o CA-12 da spec 003 chamando a função do seed); `npm test` 17
  passed; build e lint sem erro; `openapi:check` em dia.
- T4 (2026-11-11) — produção, rodada pelo agente com autorização do
  usuário (spec, "Quem escreve em produção"):
  - Revisão do SQL por subagente de contexto limpo: "Parecer: APROVADO"
    (6 itens de vibing/rules/migration.md conferidos).
  - `npx prisma migrate status` antes: "Following migration have not yet
    been applied: 20261111060000_imagem_do_equipamento".
  - `npx prisma migrate deploy`: "Applying migration
    `20261111060000_imagem_do_equipamento` … All migrations have been
    successfully applied." Depois, `migrate status`: "Database schema is
    up to date!"
  - Checksum em `_prisma_migrations` igual ao sha256 do arquivo commitado
    (3f93edd0…689c44): o aplicado é o revisado.
  - `npx prisma db seed`, duas vezes: "Seed concluído: 12 equipamentos e 2
    contas conferidos." nas duas.
  - Consulta só de leitura: 1 tenant `suporte_ti`; 12 equipamentos, 9
    ativos, manutenção TI-0004, TI-0007, TI-0010, 12 com `image_path`;
    2 memberships `colaborador` (Marcelo Grilo, Lucas Laurente) com conta
    confirmada e `tenant_id` no `app_metadata`; 0 empréstimos; 2 contas no
    Auth.
  - Login da conta de teste pela API do Auth: 200; token ES256 com `kid`,
    `aud` authenticated, `iss` do projeto e `tenant_id` no claim. Sessão
    revogada em seguida (204). A conta do professor não foi usada.
  - `git log -p --all -S<senha padrão>` em vibing/, back/ e front/: 0
    linhas nos três.
  - Pendente, do usuário: `npm run imagens:enviar` (a trava exige o
    terminal dele) e o login do professor pela Vercel depois do merge.

# 002 — Fundação da API e rota de saúde

Status: RASCUNHO — aguardando aprovação.
Origem: pedido direto — uma rota no back que possa ser testada pela
landing. Não faz parte da v1 do PRD (vibing/docs/PRD.md).
Repositório afetado: back/ apenas. O consumo na página é a spec 001.

## Objetivo
Fazer a API do EMPREST.AI existir na Vercel, em NestJS, e expor uma
rota pública de saúde que a landing consiga chamar.

## Comportamento
1. `GET /v1/health` responde 200 com `{"status":"ok"}`.
2. A rota é pública: não exige token.
3. O navegador só consegue chamar a API a partir da origem do front,
   que vem de variável de ambiente (ADR-001 §7, allowlist de CORS).
4. A API publica o documento OpenAPI gerado dos schemas Zod
   (ADR-001 §4). É dele que o front gera o cliente.
5. A configuração é validada no boot: faltando variável obrigatória, a
   aplicação não sobe (ADR-001 §9).

## O que sai do repositório
A function Python de teste `api/index.py`. Ela ocupa o caminho que a
API em Node vai ocupar, e o ADR-001 §3 escolhe NestJS.

## Critérios de aceitação
- CA-01 `GET /v1/health` responde 200 com corpo `{"status":"ok"}`.
- CA-02 A rota responde sem nenhum token de autenticação.
- CA-03 Requisição de navegador vinda de origem fora da allowlist é
  barrada por CORS; da origem do front, passa.
- CA-04 O documento OpenAPI gerado inclui `/v1/health` e o formato da
  resposta.
- CA-05 Subir a aplicação sem a variável de ambiente da origem do front
  falha no boot, com mensagem que diz qual variável falta.
- CA-06 O deploy na Vercel responde em `/v1/health`.
- CA-07 Nenhum segredo no repositório. O `.env.example` tem as chaves
  usadas, sem valor.

## Fora do escopo
- Banco, Prisma, schema, migrations e seed.
- Autenticação, multi-tenancy, CASL e RLS.
- Sentry, rate limit, auditoria, helmet e os demais itens do
  ADR-001 §10. Entram com o primeiro endpoint de negócio.
- Qualquer código em front/. É a spec 001.

## Decisões a confirmar antes do plano
1. Como o `openapi.json` chega ao front. O ADR-001 §4 diz "artefato do
   CI da API", o que exige montar o GitHub Actions do back/ nesta spec.
   A alternativa é a API servir o documento numa rota e o orval ler a
   URL do deploy — mais simples, mas é um acréscimo ao que o ADR
   decidiu.
2. Hoje o `.env.example` com `DATABASE_URL` e `DIRECT_URL` está em
   vibing/, mas essas variáveis são do back/. A regra de
   vibing/rules/secrets.md diz que o `.env.example` fica no
   repositório que usa a variável. Movo as duas para o back/ nesta
   spec, ou deixo onde estão?

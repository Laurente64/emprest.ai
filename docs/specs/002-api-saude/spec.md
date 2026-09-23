# 002 — Fundação da API e rota de saúde

Status: APROVADA em 2026-09-23.
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
4. Um comando do back/ gera o `openapi.json` a partir dos schemas Zod
   e o arquivo fica commitado no repositório. É dele que o front gera
   o cliente (ADR-001 §4, "Consumo"). Nenhuma rota de produção serve
   esse documento: ele existe para ferramenta, e ferramenta lê
   arquivo. O ADR previa distribuí-lo como artefato de CI; enquanto
   não há CI, ele viaja como arquivo versionado, e a mudança de
   contrato fica visível no diff do PR do próprio back/.
5. A configuração é validada no boot: faltando variável obrigatória, a
   aplicação não sobe (ADR-001 §9).

## O que sai do repositório
A function Python de teste `api/index.py`. Ela ocupa o caminho que a
API em Node vai ocupar, e o ADR-001 §3 escolhe NestJS.

## Arquivos de ambiente
O back/ não tem `.gitignore` hoje. Nesta ordem, sem exceção:
1. Criar o `.gitignore` do back/, ignorando `.env` e `.env.local`.
2. Só depois mover para o back/ o `.env.example` (com `DATABASE_URL` e
   `DIRECT_URL`, sem valor) e o `.env` local, que hoje estão em
   vibing/.

O `.env` guarda a credencial do banco de produção. Invertida a ordem,
ele fica exposto a um `git add`. As rules continuam valendo sobre ele
do mesmo jeito: o glob de vibing/rules/secrets.md é `**/.env*`, e o
controle vem de o agente ser aberto em emprest.ai/, não do repositório
em que o arquivo mora.

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
- CA-08 `git status` no back/ não mostra o `.env`, e um `git log` do
  repositório não tem nenhum commit com ele.
- CA-09 Rodar o comando de geração não muda o `openapi.json`
  commitado. Se mudar, é porque o arquivo no repositório não descreve
  mais a API que o código serve.

## Fora do escopo
- Banco, Prisma, schema, migrations e seed.
- Autenticação, multi-tenancy, CASL e RLS.
- Sentry, rate limit, auditoria, helmet e os demais itens do
  ADR-001 §10. Entram com o primeiro endpoint de negócio.
- Qualquer código em front/. É a spec 001.

## Decisões tomadas em 2026-09-23
1. O `openapi.json` é um arquivo gerado por comando e commitado no
   back/, sem nenhuma rota que o sirva. O front lê esse arquivo de
   `../back/`, porque os três repositórios ficam lado a lado
   (vibing/README.md). O artefato de CI da linha 73 do ADR fica para
   quando o CI existir: hoje não há GitHub Actions em nenhum dos dois
   repositórios.
2. O `.env.example` e o `.env` vão para o back/, depois do
   `.gitignore`.

## Lacuna conhecida
Enquanto não houver CI, a verificação de contrato do ADR-001 §8,
linha 167, não roda sozinha a cada PR. Ela existe como comando do
front/ (ver spec 001), rodado no fim de cada tarefa. A diferença é só
quem dispara: hoje é uma pessoa, depois é o GitHub. O CI dos dois
repositórios continua exigido pelo ADR-001, linha 195, e ainda não tem
spec.

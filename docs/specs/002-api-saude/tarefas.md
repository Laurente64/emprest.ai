# 002 — Fundação da API e rota de saúde · Tarefas

Uma por vez. Cada tarefa termina com os checks de
vibing/rules/checks.md e PARA até o próximo "pode implementar".

## T1 — Higiene do repositório
- Sincronizar a cópia local do back/ com o remoto e criar a branch
  `api-saude`.
- Criar o `.gitignore` (node_modules, dist, `.env`, `.env.local`).
  Isto vem antes de qualquer arquivo de ambiente entrar na pasta.
- Remover a function Python `api/index.py`.
- Pronto quando: `git status` limpo e o `.gitignore` commitado.

## T2 — NestJS de pé
- Gerar o scaffold do Nest numa pasta temporária e copiar para back/.
  Mostro a lista de arquivos antes de commitar.
- TypeScript com `strict`.
- Prefixo global `/v1`.
- Módulo `health` com controller e service; `GET /v1/health`
  respondendo `{"status":"ok"}` a partir de um schema Zod.
- Pronto quando: build passa e os testes de supertest cobrem CA-01 e
  CA-02.

## T3 — Configuração e CORS
- Variáveis de ambiente validadas com Zod no boot. Faltando
  `FRONT_ORIGIN`, a aplicação não sobe e diz qual variável falta.
- CORS por allowlist, com a origem vinda dessa variável.
- Pronto quando: testes cobrem CA-03 e CA-05.

## T4 — Documento OpenAPI
- O documento é gerado dos schemas Zod e servido numa rota.
- Pronto quando: um teste confirma que o documento traz `/v1/health` e
  o formato da resposta (CA-04).

## T5 — Adapter da Vercel
- Arquivo em `api/` criando a aplicação uma vez e guardando a
  instância fora do handler; `vercel.json` mandando todo caminho para
  lá.
- Pronto quando: o build passa. CA-06 e CA-09 só no deploy, depois do
  seu merge.

## T6 — Arquivos de ambiente
- Mover `.env.example` e `.env` de vibing/ para back/, sem exibir
  nenhum valor, e limpar o que sobrar em vibing/.
- Acrescentar `FRONT_ORIGIN` ao `.env.example`, sem valor.
- Pronto quando: CA-07 e CA-08 verificados, incluindo o `git log` sem
  nenhum commit do `.env`.

## T7 — Push e PR
- Push da branch `api-saude` e link do PR (não há `gh` nesta máquina).
- Depois do seu merge: conferir no deploy o CA-06 (`/v1/health`) e o
  CA-09 (documento acessível por URL).

# 002 — Fundação da API e rota de saúde · Plano

Status: RASCUNHO — aguardando aprovação.
Spec: spec.md (aprovada em 2026-09-23). Tarefas: tarefas.md.

## Ordem em relação à spec 001
Esta spec vem primeiro. O front só consegue gerar o cliente e rodar a
checagem de contrato depois que a API estiver no ar servindo o
`openapi.json`.

## Ponto de partida do back/
- Remoto: main com um commit ("Initial commit", 2026-09-21) que só tem
  a function Python `api/index.py`. A cópia local está vazia.
- O back/ não tem `.gitignore`. Ele é a primeira coisa a ser criada,
  antes de qualquer arquivo de ambiente entrar na pasta.
- A pasta não está vazia (.git e api/), então o scaffold do Nest é
  gerado numa pasta temporária e copiado para dentro.

## Stack (subconjunto do ADR-001)

| Item | Uso nesta spec |
|---|---|
| Node LTS + NestJS (§3) | a aplicação. Node 22.13.1 nesta máquina |
| Express com instância cacheada (§3) | `@vendia/serverless-express`, para a Vercel não pagar o bootstrap a cada request |
| Pastas por domínio (§3) | `src/modules/health/` |
| Controller → Service (§3) | sem Repository: esta spec não toca banco |
| Zod via `nestjs-zod` (§3, §4) | o schema da resposta e o OpenAPI saem dele |
| Prefixo `/v1` (§4) | `GET /v1/health` |
| Config validada no boot (§9) | Zod sobre as variáveis de ambiente |
| CORS por allowlist (§7) | origem do front vinda de variável de ambiente |
| Jest + supertest (§8) | testes sobre a app Nest |
| ESLint + Prettier + husky (§11) | qualidade, igual ao front |

Fora até haver endpoint de negócio, como a spec diz: Prisma, banco,
auth, multi-tenancy, CASL, RLS, pino, Sentry, helmet, rate limit e
auditoria.

## Variáveis de ambiente
- `FRONT_ORIGIN`: a origem do front na allowlist de CORS. Sem ela, a
  aplicação não sobe (CA-05).
- `DATABASE_URL` e `DIRECT_URL` vêm de vibing/ para o `.env.example`
  do back/, sem valor. Esta spec não as usa, mas é o repositório delas.
- Os valores no painel da Vercel são seus: a restrictions.md me proíbe
  de mexer lá.

## O documento OpenAPI
Não existe rota servindo o documento. Um comando do back/ monta a
aplicação em memória, gera o `openapi.json` dos schemas Zod e escreve
o arquivo na raiz do repositório, onde ele fica commitado.

Isso põe a checagem de divergência no lugar onde a mudança acontece: o
mesmo comando, rodado de novo, não pode mudar o arquivo (CA-09). Se
mudar, alguém alterou uma rota e não regerou o documento.

O front lê esse arquivo de `../back/`, como o vibing/README.md já
pressupõe. Quando houver CI, ele passa a vir do artefato, como o
ADR-001 §4 decidiu.

## Como isso roda na Vercel
O Nest não roda como processo na Vercel. O padrão é um arquivo em
`api/` que cria a aplicação uma vez, guarda a instância fora do
handler e responde a todas as rotas, com um `vercel.json` mandando
todo caminho para lá.

Esse desenho vem do ADR-001 §3, mas **eu nunca rodei isso nesta
máquina**. Na T5 eu mostro os arquivos antes de commitar, e o CA-06 só
é verificável depois do seu merge, no deploy.

## Deploy
O agente para no PR. Publicar é você fazer o merge na main do back/.

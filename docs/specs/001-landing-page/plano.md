# 001 — Landing page · Plano

Status: RASCUNHO — aguardando aprovação.
Spec: spec.md (aprovada em 2026-09-23, com o indicador de estado da
API). Tarefas: tarefas.md.

## Ordem em relação à spec 002
A 002 vem primeiro, porque é ela que produz o `openapi.json` commitado
no back/. Para gerar o cliente, a API não precisa estar no ar: basta o
arquivo existir em `../back/`. A API no ar só é necessária para o
indicador mostrar "no ar" (CA-05); sem ela, o comportamento correto é
o CA-06, com o resto da página inteiro.

## O que muda em cada repositório
- front/: tudo desta spec.
- back/: nada. É a spec 002.
- vibing/: a tabela de Comandos do AGENTS.md, na última tarefa.

## Ponto de partida do front/
- Remoto: main com um commit ("Initial commit", 2026-09-21) que só tem
  um index.html "Hello Vercel". A cópia local está vazia.
- A pasta não está vazia (.git e index.html), então o scaffold é
  gerado numa pasta temporária e copiado para dentro.
- O index.html do scaffold substitui o de teste.

## Stack (subconjunto do ADR-001)

| Item | Uso nesta spec |
|---|---|
| React + Vite + TypeScript (§2) | scaffold `react-ts` do create-vite |
| TypeScript `strict` (§2) | flag explícita no tsconfig.app.json; o scaffold não traz |
| React Router, modo data (§2) | uma rota, `/` |
| Tailwind CSS (§2) | tokens de cor e raio do layout.md |
| orval (§2, §4) | cliente gerado do `openapi.json`, commitado |
| Playwright (§8) | um teste por critério, CA-01 a CA-09 |
| ESLint + Prettier (§11) | no lugar do oxlint que o scaffold traz |
| husky + lint-staged (§11) | lint e formatação no pré-commit |

Fora até uma spec precisar: TanStack Query, Zustand, React Hook Form,
shadcn/ui, Vitest, Sentry.

## Fonte
Raleway pelo Google Fonts, pesos 400, 500, 600 e 700 (layout.md §2).

## Checagem de contrato
Um comando só, com dois passos: regenerar o cliente a partir de
`../back/openapi.json` e falhar se o resultado divergir do commitado.
Não há download nem rota de API no meio. A geração do check sai numa
pasta temporária, para conferir não sujar o repositório — quem escreve
é o comando de atualização, separado. É o mesmo par que o back/ já
tem: `openapi:emit` escreve, `openapi:check` só compara. Ele entra nos checks de fim
de tarefa. Quando houver CI, o mesmo comando roda em cada PR
(ADR-001 §8, linha 167), e aí o documento virá do artefato do back/,
não do disco.

## Cabeçalhos e build na Vercel (vercel.json)
- CSP restritiva, sem `unsafe-inline` (ADR-001 §10):
  `default-src 'self'`, `style-src 'self' https://fonts.googleapis.com`,
  `font-src https://fonts.gstatic.com`, `connect-src 'self'` mais o
  endereço da API.
- Framework Vite e saída `dist` fixados no arquivo, para o build não
  depender do preset que o projeto recebeu quando era só um index.html.

Ponto a resolver na T7: o `connect-src` precisa do endereço literal da
API, enquanto o código lê esse endereço de variável de ambiente. Os
dois têm que apontar para o mesmo lugar, senão o navegador bloqueia a
chamada e o indicador mostra falha com a API no ar.

Se você considera o vercel.json "alterar configuração da Vercel"
(restrictions.md), a T7 perde essa parte e os cabeçalhos ficam com
você.

## Deploy
O agente para no PR. Publicar é você fazer o merge na main do front/.

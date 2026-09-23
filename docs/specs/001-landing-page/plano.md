# 001 — Landing page · Plano

Status: DESATUALIZADO. Ele foi escrito antes de a spec ganhar o
indicador de estado da API e o cliente gerado com orval. Reescrevo
plano e tarefas depois que você aprovar a alteração da spec 001 e a
spec 002.
Spec: spec.md. Tarefas: tarefas.md.

## O que muda em cada repositório

- front/: tudo desta spec.
- back/: nada. A landing não chama a API (CA-04) e a spec deixa back/
  fora. O back/ entra na primeira spec que precisar dele.
- vibing/: este plano, as tarefas e, no fim, a tabela de Comandos do
  AGENTS.md.

## Ponto de partida do front/

- Remoto: main com um commit ("Initial commit", 2026-09-21) que só tem
  um index.html "Hello Vercel". A cópia local está vazia e ainda não
  foi sincronizada.
- O index.html do scaffold substitui esse arquivo.
- A pasta não está vazia (.git e index.html), então o scaffold é gerado
  numa pasta temporária e copiado para dentro de front/.

## Stack usada (subconjunto do ADR-001)

| Item | Uso nesta spec |
|---|---|
| React + Vite + TypeScript (§2) | scaffold `react-ts` do create-vite |
| TypeScript `strict` (§2) | `"strict": true` explícito no tsconfig.app.json; o scaffold atual não traz a flag |
| React Router, modo data (§2) | uma rota, `/` |
| Tailwind CSS (§2) | tokens de cor e raio do layout.md |
| Playwright (§8) | um teste por critério, CA-01 a CA-06 |
| ESLint + Prettier (§11) | substituem o oxlint que o scaffold traz |
| husky + lint-staged (§11) | só lint e formatação no pré-commit |

Ficam de fora até uma spec precisar deles: TanStack Query, Zustand,
React Hook Form, shadcn/ui, orval, Vitest, Sentry.

## Fonte
Raleway pelo Google Fonts, pesos 400, 500, 600 e 700 (layout.md §2).

## Cabeçalhos e build na Vercel (vercel.json)
- CSP restritiva, sem `unsafe-inline` (ADR-001 §10):
  `default-src 'self'; style-src 'self' https://fonts.googleapis.com;
  font-src https://fonts.gstatic.com`.
- Framework Vite e saída `dist` fixados no arquivo, para o build não
  depender do preset que o projeto recebeu quando era só um index.html.

O vercel.json é um arquivo do repositório, revisado no PR. Se você
considera isso "alterar configuração da Vercel" (restrictions.md), a
T6 perde essa parte e os cabeçalhos ficam com você.

## Deploy
O agente para no PR do front/. Publicar é você fazer o merge em main,
porque push em main publica. O agente não faz merge e não mexe no
painel da Vercel.

## Pontos em aberto fora desta spec
- back/ tem uma function Python (api/index.py). O ADR-001 §3 escolhe
  NestJS. Esta spec não toca nisso.

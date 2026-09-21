# 001 — Landing page · Tarefas

Uma por vez. Cada tarefa termina com os checks de
vibing/rules/checks.md e PARA até o próximo "pode implementar".

## T1 — Base do front/
- Trazer a main do remoto para a cópia local e criar a branch
  `landing-page` a partir dela.
- Gerar o scaffold `react-ts` numa pasta temporária e copiar para
  front/. Arquivos que ele cria (conferido em 2026-09-21, create-vite
  9.2.1): .gitignore, .oxlintrc.json, README.md, index.html,
  package.json, public/favicon.svg, public/icons.svg, src/App.css,
  src/App.tsx, src/assets/hero.png, src/assets/react.svg,
  src/assets/vite.svg, src/index.css, src/main.tsx, tsconfig.json,
  tsconfig.app.json, tsconfig.node.json, vite.config.ts.
- Remover o conteúdo de demonstração: src/App.css, src/assets/,
  public/icons.svg e o README do template.
- `"strict": true` no tsconfig.app.json.
- Pronto quando: `npm run build` passa.

## T2 — Lint e formatação
- Trocar o oxlint por ESLint + Prettier.
- husky + lint-staged rodando só lint e formatação no pré-commit.
- Pronto quando: lint e build passam, e o hook roda num commit.

## T3 — Rota e visual base
- React Router em modo data, com a rota `/`.
- Tailwind pelo plugin do Vite, com os tokens de cor e raio do
  layout.md. Raleway pelo Google Fonts.
- Pronto quando: o build passa e `/` mostra uma página vazia com fundo
  `#161826`.

## T4 — Testes dos critérios
- Playwright com Chromium, rodando contra o build servido localmente.
- Um teste por critério, CA-01 a CA-06.
- Pronto quando: os testes rodam e falham onde a landing ainda não
  existe. A saída vai na resposta.

## T5 — Landing
- Conteúdo e visual da spec.
- Pronto quando: os seis testes e o build passam.

## T6 — vercel.json, push e PR
- vercel.json com a CSP e o build do plano.
- Push da branch `landing-page` e link do PR (não há `gh` nesta
  máquina).
- Se a Vercel gerar preview da branch, conferir lá os CA-01 a CA-06 e
  se o console mostra alguma violação de CSP.
- Merge é deploy, e é seu.

## T7 — Comandos no AGENTS.md
- Registrar na tabela de Comandos de vibing/AGENTS.md os comandos do
  front/ que rodaram nesta máquina (dev, testes, build), depois que
  você confirmar.

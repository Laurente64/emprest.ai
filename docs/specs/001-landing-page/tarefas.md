# 001 — Landing page · Tarefas

Uma por vez. Cada tarefa termina com os checks de
vibing/rules/checks.md e PARA até o próximo "pode implementar".

## T1 — Base do front/
- Sincronizar a cópia local com o remoto e criar a branch
  `landing-page`.
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

## T4 — Cliente da API
- orval configurado; busca do `openapi.json` da API (spec 002), cliente
  gerado e commitado junto com a cópia do documento.
- O comando de checagem de contrato: buscar, regenerar, falhar se
  divergir.
- Endereço da API em variável de ambiente, e a chave no `.env.example`
  do front/, sem valor (vibing/rules/secrets.md).
- Pronto quando: o comando roda contra a API no ar e termina sem
  diferença (CA-09).

## T5 — Testes dos critérios
- Playwright com Chromium, rodando contra o build servido localmente.
- Um teste por critério, CA-01 a CA-08, incluindo os dois estados do
  indicador: resposta da API e ausência de resposta.
- Pronto quando: os testes rodam e falham onde a landing ainda não
  existe. A saída vai na resposta.

## T6 — Landing
- Conteúdo, visual e indicador de estado da API, como na spec.
- Pronto quando: os testes de CA-01 a CA-08 e o build passam.

## T7 — vercel.json, push e PR
- vercel.json com a CSP, o `connect-src` da API e o build do plano.
- Push da branch `landing-page` e link do PR.
- Se a Vercel gerar preview da branch, conferir lá os critérios e se o
  console acusa violação de CSP.

## T8 — Comandos no AGENTS.md
- Registrar na tabela de Comandos de vibing/AGENTS.md os comandos do
  front/ e do back/ que rodaram nesta máquina, depois que você
  confirmar.

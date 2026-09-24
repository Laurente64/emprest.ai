# 001 — Landing page

Status: APROVADA em 2026-09-23, na versão com o indicador de estado da
API. A v1, aprovada em 2026-09-21, era só desktop e não chamava a API.
Origem: pedido direto. Não faz parte da v1 do PRD (vibing/docs/PRD.md).
Repositório afetado: front/ apenas. A rota da API é a spec 002.

## Objetivo
Uma página pública de entrada que diga o que é o EMPREST.AI, para quem
é, quais são as regras de empréstimo, e mostre se a API está no ar.

## Rota
`/`, pública. É a única rota desta tarefa.

## Quem vê
Qualquer visitante, sem sessão.

## Conteúdo
Todo texto de regra sai do PRD, sem reescrever. Nenhuma regra nova.

1. Marca: EMPREST.AI — "EMPREST" em peso 700 e ".AI" no acento
   (layout.md §2).
2. O que é: controle de empréstimo de equipamentos internos
   (notebooks, monitores, cabos, câmeras), no lugar da planilha
   compartilhada (PRD — Problema).
3. Para quem (PRD — Quem usa):
   - Colaborador: vê o catálogo, pede um item emprestado, devolve.
   - Operações: cadastra equipamentos, vê quem está com o quê e
     registra devolução no balcão.
4. Regras (PRD — Regras que Operações já decidiu):
   - Cada pessoa pode estar com no máximo 3 itens ao mesmo tempo.
   - O prazo padrão de devolução é de 14 dias.
   - Quem tem item em atraso não pode pegar outro emprestado.
   - Equipamento em manutenção não aparece como disponível.
5. Indicador de estado da API, no rodapé: ao abrir, a página consulta
   a rota de saúde da API (spec 002) e mostra o resultado numa linha
   discreta. Sem botão.

## Como o front fala com a API
Pelo cliente gerado com orval a partir do `openapi.json` da API
(ADR-001 §4). Nenhum cliente HTTP escrito à mão. O endereço da API vem
de variável de ambiente, conforme vibing/rules/secrets.md.

A fonte é o `openapi.json` commitado no back/ (spec 002), lido de
`../back/`. Não há rota de API servindo esse documento. No front/ fica
commitado o código TypeScript gerado a partir dele, que ninguém edita
à mão.

## Checagem de contrato
Um comando do front/ faz, nesta ordem:
1. gera o cliente de novo a partir de `../back/openapi.json`;
2. falha se o resultado ficar diferente do que está commitado.

Diferença quer dizer que a API mudou e o front ainda está na versão
antiga. O comando não conserta nada: ele mostra o diff e falha. Quem
decide o que fazer é quem lê.

Quando roda: junto com testes e build, no fim de cada tarefa
(vibing/rules/checks.md). Quando existir CI, o mesmo comando roda em
cada PR, e aí ele vira a verificação de contrato do ADR-001 §8,
linha 167.

Limite: o comando precisa do back/ clonado ao lado do front/, que é
como o vibing/README.md manda trabalhar. Não precisa da API no ar.

## Visual
Segue o layout.md. Os pontos que a landing precisa respeitar:

- Fundo `#161826`, superfícies `#232532`, texto `#e9e9ed` (§1).
- Acento `#2fb8ac` como linha e brilho, nunca como preenchimento
  sólido (§1).
- Estado de falha em `#e88b8b`, o mesmo vermelho de "em atraso" (§1).
- Nunca preto puro nem branco puro (§1).
- Fonte única Raleway; títulos em peso 600, corpo em 400 (§2).
- Animação só `fadeUp` curta, nada de escala ou parallax (§8).
- Copy em português do Brasil, direto e operacional, sem exclamação e
  sem emoji (§10).

A implementação segue o ADR-001 §2 (React, Vite, React Router,
Tailwind). O layout.md dá os valores visuais; ele não decide a stack.

## Critérios de aceitação
- CA-01 Abrir `/` sem sessão mostra a página. Nada pede login.
- CA-02 A página mostra a marca, o que é o sistema e os dois perfis de
  uso (itens 1 a 3 de Conteúdo).
- CA-03 As quatro regras aparecem com o texto exato do PRD.
- CA-04 Ao abrir, a página faz exatamente uma requisição à API, na rota
  de saúde, pelo cliente gerado. Nenhuma outra.
- CA-05 Com a API respondendo, o rodapé mostra o estado de no ar, no
  acento `#2fb8ac`.
- CA-06 Com a API fora do ar ou respondendo erro, o rodapé mostra o
  estado de falha em `#e88b8b`, e o resto da página continua legível e
  completo.
- CA-07 Fundo `#161826`, acento `#2fb8ac` só em linha ou contorno,
  fonte Raleway.
- CA-08 Nenhum texto da página tem "!" ou emoji.
- CA-09 O comando de checagem de contrato termina sem diferença: o
  cliente commitado é igual ao que sai do `openapi.json` do back/.

## Dependência
O indicador depende da spec 002. Enquanto a rota de saúde não estiver
no ar, o CA-06 é o comportamento esperado: a landing entrega valor
sozinha e mostra a API como fora do ar.

## Fora do escopo
- Tela de login, autenticação e sessão — inclusive botão ou link de
  "Entrar". A landing não aponta para nenhuma outra rota.
- Qualquer código em back/. A API é a spec 002.
- Dado real (ex.: quantos itens estão disponíveis agora).
- O que o PRD já deixou fora da v1: reserva com data futura,
  notificação por e-mail, importação da planilha.
- Celular e tablet. A landing é só desktop, nas larguras de referência
  do layout.md (1360–1440px).

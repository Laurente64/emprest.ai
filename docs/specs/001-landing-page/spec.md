# 001 — Landing page

Status: APROVADA em 2026-09-21. Só desktop.
Origem: pedido direto. Não faz parte da v1 do PRD (vibing/docs/PRD.md).
Repositório afetado: front/ apenas. Nada em back/: o que é do front é
do front, o que é do back é do back.

## Objetivo
Uma página pública de entrada que diga o que é o EMPREST.AI, para quem
é e quais são as regras de empréstimo.

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

## Visual
Segue o layout.md. Os pontos que a landing precisa respeitar:

- Fundo `#161826`, superfícies `#232532`, texto `#e9e9ed` (§1).
- Acento `#2fb8ac` como linha e brilho, nunca como preenchimento
  sólido (§1).
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
- CA-04 Carregar a página não faz nenhuma requisição à API.
- CA-05 Fundo `#161826`, acento `#2fb8ac` só em linha ou contorno, fonte
  Raleway.
- CA-06 Nenhum texto da página tem "!" ou emoji.

## Fora do escopo
- Tela de login, autenticação e sessão — inclusive botão ou link de
  "Entrar". A landing não aponta para nenhuma outra rota.
- Qualquer coisa em back/.
- Dado real (ex.: quantos itens estão disponíveis agora).
- O que o PRD já deixou fora da v1: reserva com data futura,
  notificação por e-mail, importação da planilha.
- Celular e tablet. A landing é só desktop, nas larguras de referência
  do layout.md (1360–1440px).

# 006 — Catálogo e solicitar empréstimo

Status: APROVADA em 2026-11-11 pelo agente, por delegação do usuário
(ver spec 004, "Aprovação"). As decisões marcadas "usuário" foram
respondidas por ele em 2026-11-10; as do agente estão em "Para o usuário
confirmar".
Origem: briefing "solicitar empréstimo" da prática (pedido, não spec);
PRD v1, itens 2 e 3, e "Regras que Operações já decidiu".
Repositórios afetados: back/ e front/.
Depende de: specs 004 (sessão e contexto do request) e 005 (dados).

## Objetivo
A pessoa logada vê o catálogo com foto, nome, categoria e situação de
cada equipamento, pede um disponível e o empréstimo nasce na hora, com a
data de devolução. Ela vê os próprios empréstimos em aberto. As regras do
PRD (limite, atraso e manutenção) valem na API, e o motivo de uma recusa
aparece na tela.

## Comportamento

### API
1. `GET /v1/equipamentos?cursor=&limite=` lista os equipamentos do
   tenant, exceto `baixado`, por patrimônio e depois por id, com
   paginação por cursor (ADR-001 §4; `limite` de 1 a 100, padrão 50;
   fora disso, 400). Cada item: `id`, `patrimonio` (nulo em estoque),
   `nome`, `categoria`, `situacao`, `comigo` e `imagemUrl`.
   - `situacao` segue a spec 003 ("situação do catálogo"): `manutencao`
     se `condition = manutencao` (mesmo com empréstimo, P-11); em
     `unidade`, `emprestado` se há empréstimo em andamento; em `estoque`,
     `emprestado` se `quantity_total` − empréstimos em andamento for
     zero; senão `disponivel`. O nome `emprestado` é o "indisponível" da
     spec 003.
   - `comigo`: a pessoa do request tem empréstimo em andamento deste
     equipamento, qualquer que seja a situação.
   - `imagemUrl`: `<SUPABASE_URL>/storage/v1/object/public/equipamentos/<image_path>`,
     ou nulo sem `image_path`. `SUPABASE_URL` é variável de runtime da API
     (tabela da spec 004).
2. `POST /v1/emprestimos` com `{ "equipamentoId": <uuid> }` cria o
   empréstimo e responde 201 com ele (formato do item 5). Recusas, nesta
   ordem, em Problem Details com o `type` exato:
   1. equipamento que não existe no tenant, ou `baixado`: 404,
      `about:blank`;
   2. a pessoa tem empréstimo em atraso: 409,
      `/problemas/emprestimo-em-atraso`;
   3. a pessoa já tem empréstimos em andamento no limite
      (`max_open_loans`, ou 3 se nulo): 409,
      `/problemas/limite-de-emprestimos`, com o número no `detail`;
   4. equipamento em manutenção: 409, `/problemas/equipamento-em-manutencao`;
   5. sem peça livre (unidade com empréstimo em andamento, ou estoque sem
      sobra): 409, `/problemas/equipamento-indisponivel`.
   Corpo inválido: 400.
3. O empréstimo nasce `ativo`, com `equipment_tracking` copiado do
   equipamento, `requested_at`, `decided_at` e `handed_over_at` iguais ao
   instante do pedido, `decided_by` e `handed_over_by` = a própria
   membership, e `due_at` = instante do pedido + 14 dias. Na mesma
   transação, uma linha `loan.request` na `audit_log` com
   `payload.autoatendimento = true` e o `due_at`, para separar estes
   empréstimos das aprovações de Operações que vierem depois. Ver C-01 e
   C-02.
4. Dois pedidos ao mesmo tempo: a transação trava a membership e o
   equipamento (`SELECT … FOR UPDATE`) antes de contar; em `unidade`, o
   índice único parcial da spec 003 é a segunda barreira. Só um
   empréstimo é criado; o outro pedido recebe 409
   `/problemas/equipamento-indisponivel`.
5. `GET /v1/emprestimos/meus?cursor=&limite=` lista os empréstimos em
   andamento da pessoa do request, por data de devolução e depois por id.
   Cada item: `id`, `equipamento` (`id`, `patrimonio`, `nome`,
   `categoria`, `imagemUrl`), `retiradoEm`, `devolverAte` e `emAtraso`.
6. `GET /v1/eu` passa a trazer também `limiteDeEmprestimos`.
7. Autorização com CASL (ADR-001 §7): ler equipamento, criar empréstimo
   e ler empréstimo só o próprio. Vale para os dois papéis; nesta versão
   Operações vê só o que o colaborador vê.

### Tela
8. A área logada ganha as abas do colaborador (layout.md §6):
   "Catálogo" (`/app`) e "Meus empréstimos" (`/app/meus`). Operações vê
   as mesmas duas.
9. Catálogo: H1 "Catálogo", "X de Y equipamentos disponíveis agora" (X =
   itens `disponivel`; Y = itens do catálogo, inclusive em manutenção) e
   a grade de cards (layout.md §3 e §5.2). Card: foto 800×600 exibida em
   4:3 no topo, com o nome como texto alternativo; categoria; nome;
   patrimônio; pill de situação; linha de detalhe; botão de largura
   total. O front busca todas as páginas do catálogo.
10. Botão do card, na ordem (L = `limiteDeEmprestimos`):
    - com a própria pessoa (`comigo`): "Está com você", desabilitado,
      detalhe com a data de devolução;
    - em manutenção: "Indisponível", desabilitado, detalhe "Em
      manutenção";
    - emprestado para outra pessoa: "Emprestado", desabilitado, sem nome
      nem prazo de quem está com ele (L-03);
    - disponível, com a pessoa em atraso: "Bloqueado por atraso",
      desabilitado, com o aviso do item 13;
    - disponível, com a pessoa no limite: "Limite de L itens",
      desabilitado (layout.md §7.4), com o aviso do item 13;
    - disponível: "Solicitar".
11. "Solicitar" abre um diálogo com "Devolver até DD/MM/AAAA" (hoje + 14
    dias). Confirmar faz o pedido. Aceito: aviso "Empréstimo registrado.
    Devolva até DD/MM/AAAA." com o `devolverAte` da API, e o catálogo e
    "Meus empréstimos" se atualizam. Recusado: o `detail` da API aparece
    no diálogo.
12. Meus empréstimos: H1, "N de L itens em seu nome · prazo padrão de 14
    dias" e uma linha por empréstimo: foto, nome, patrimônio, "Retirada"
    e "Devolver até", pill "Em dia" ou "Em atraso". Sem empréstimos:
    "Nenhum empréstimo em aberto." e o botão "Ir ao catálogo". Sem botão
    "Devolver" (devolução fora do escopo).
13. Avisos no topo do catálogo: no limite, "Você está com L de L itens
    em aberto, o limite por pessoa. Novos pedidos voltam quando um item
    for devolvido."; em atraso, "Você tem item em atraso. Novos pedidos
    ficam bloqueados até a devolução." com atalho para "Meus empréstimos".
14. O catálogo mostra "Fotos: Pixabay" com link para https://pixabay.com/.
    Ver C-03.
15. Datas em DD/MM/AAAA no fuso America/Sao_Paulo, sem exclamação e sem
    emoji (layout.md §10).
16. A CSP do `vercel.json` libera, em `img-src`, exatamente a origem do
    projeto Supabase. Sem isso o navegador não carrega as fotos na
    Vercel, e o Playwright no preview não percebe.
17. Detalhes de tela decididos na execução (emenda de 2026-11-11):
    - linha de detalhe do card: "Pronto para retirada no balcão"
      (disponível), "Emprestado no momento" (com outra pessoa), "Devolver
      até DD/MM/AAAA" (com a própria pessoa, do `devolverAte` dela) e "Em
      manutenção";
    - diálogo: título "Solicitar empréstimo", texto "O empréstimo é
      registrado na hora. Retire o item no balcão de Operações.", resumo
      com Equipamento e Devolver até, botões "Cancelar" e "Confirmar";
      clicar fora fecha; falha que não vem da API: "Não foi possível
      registrar o pedido. Tente de novo.";
    - falha ao carregar: "Não foi possível carregar o catálogo. Tente de
      novo." e "Não foi possível carregar os seus empréstimos. Tente de
      novo.";
    - os dois avisos do item 13 podem aparecer juntos, atraso primeiro; o
      atalho é "Ver meus empréstimos";
    - o crédito fica abaixo da grade e abre em nova aba;
    - item sem `imagemUrl` mostra o espaço da foto com "Sem foto";
    - a tela do catálogo também busca "meus empréstimos", para limite,
      atraso e a data dos itens com a pessoa; a barra de 3 traços do
      layout.md §5.3 não entra.

## Conflitos com o que foi pedido (decisões do usuário, 2026-11-10)
- C-01 Fluxo: o briefing diz "solicitar cria o empréstimo na hora, sem
  aprovação de Operações"; a spec 003 (decisões 2 e 5, e "Regras que NÃO
  ficam no banco": aprovar e entregar "são de Operações") tem pedido,
  aprovação e entrega, com o prazo contando da entrega. Vale o briefing,
  nesta versão: sem tela de Operações, nenhum pedido sairia de
  `solicitado`. Sem migration: o empréstimo nasce `ativo` e as colunas
  de decisão e entrega, que os CHECKs da spec 003 exigem, recebem a
  própria membership (autoatendimento, marcado na `audit_log`). A spec
  003 ganha uma nota apontando para cá. A aprovação de Operações, quando
  existir, é spec nova.
- C-02 Prazo: o briefing diz "14 dias úteis"; o PRD e a spec 003 (P-06)
  dizem 14 dias. Valem 14 dias corridos.
- C-03 Crédito: o item 9 do briefing pede catálogo "sem rodapé nem texto
  de crédito"; o contrato da skill e a API do Pixabay exigem mostrar de
  onde vêm as fotos. O crédito fica, discreto.
- C-04 Limite: o briefing diz 5; o PRD diz 3, e a spec 003 (decisão 4)
  diz 3 por padrão, mais só com aprovação de Operações. Vale 3.
- C-05 Manutenção: TI-0004, TI-0007 e TI-0010 (spec 005, C-01).

## Lacunas decididas (agente, por delegação)
- L-01 Ordem das recusas (item 2): primeiro o que não existe, depois o
  que é da pessoa (atraso, limite), depois o que é do equipamento.
  Quem está no limite fica sabendo disso antes de tentar outro item.
- L-02 No limite e em atraso, o front desabilita o botão com o motivo
  (layout.md §7.4 e §7.5) e mostra o aviso do item 13. A API recusa de
  qualquer jeito, e o `detail` dela aparece se o pedido chegar lá (tela
  desatualizada, outra aba).
- L-03 O card não mostra quem está com o item (o layout.md §5.2 mostra
  "Com {pessoa}"). O PRD diz "cada pessoa vê os próprios empréstimos", e
  o layout.md §7.11 diz que a visão completa é de Operações.
- L-04 Sem busca e sem filtros por situação (layout.md §5.2 e §7.8–9):
  o briefing não pede, e são 12 itens.
- L-05 O card ganha a foto no topo: o layout.md não tem foto no card, e o
  briefing exige imagem.
- L-06 O "Devolver até" do diálogo é calculado no navegador só para
  mostrar; o que vale é o `due_at` gravado pela API, que aparece no
  aviso e em "Meus empréstimos".
- L-07 Estoque: nenhum dos 12 é de estoque, mas a spec 003 tem o tipo, e
  esta spec segue a regra dela em vez de tratar só `unidade`.
- L-08 Dados que sobram em produção: não há devolução nesta versão.
  Depois do roteiro "Antes de entregar" com a conta de teste, os itens
  que ela pediu ficam emprestados. Estado esperado na correção, se o
  roteiro chegar ao limite: 12 no catálogo, 3 em manutenção, 3
  emprestados para a conta de teste, 6 disponíveis. O professor ainda
  consegue pedir 3 e chegar ao limite. Só uma spec de devolução ou uma
  correção de dado aprovada muda isso.

## Para o usuário confirmar
L-01, L-03, L-04 e L-08.

## Critérios de aceitação
API: supertest sobre a app Nest, com o banco no Testcontainers e a JWKS
local da spec 004. Nunca o Supabase.
- CA-01 O catálogo lista os equipamentos do tenant, exceto `baixado`,
  com `imagemUrl`, nome, categoria e situação; equipamento em manutenção
  vem `manutencao`, nunca `disponivel`; estoque com sobra vem
  `disponivel` e sem sobra vem `emprestado`.
- CA-02 Isolamento (ADR-001 §8): o catálogo e "meus empréstimos" do
  tenant A não trazem nada do tenant B, e pedir um equipamento do tenant
  B responde 404.
- CA-03 Pedir um disponível responde 201 e grava o empréstimo `ativo`,
  com `due_at` = `handed_over_at` + 14 dias, decisão e entrega pela
  própria membership e uma linha `loan.request` com `autoatendimento` na
  `audit_log`. Depois, o item vem `emprestado`, com `comigo` verdadeiro
  para quem pediu e falso para outra pessoa.
- CA-04 Pedir um equipamento em manutenção responde 409
  `/problemas/equipamento-em-manutencao` e não grava nada.
- CA-05 Pedir uma unidade emprestada, ou um estoque sem sobra, responde
  409 `/problemas/equipamento-indisponivel`.
- CA-06 Com 3 empréstimos em andamento, o quarto pedido responde 409
  `/problemas/limite-de-emprestimos` com o número no `detail`. Com
  `max_open_loans = 4` aprovado, o quarto entra.
- CA-07 Com um empréstimo vencido, o pedido responde 409
  `/problemas/emprestimo-em-atraso`.
- CA-08 Dois pedidos simultâneos ao mesmo equipamento, de pessoas
  diferentes, criam um empréstimo só: um 201 e um 409. Dois pedidos
  simultâneos da mesma pessoa com uma vaga no limite criam um só.
- CA-09 "Meus empréstimos" traz só os empréstimos em andamento da
  pessoa, com `devolverAte` e `emAtraso`, e não traz os de outra pessoa
  nem os encerrados.
- CA-10 Corpo inválido e `limite` fora de 1 a 100 respondem 400, e todas
  as recusas são Problem Details com o `type` do item 2.
- CA-11 `GET /v1/eu` traz `limiteDeEmprestimos`: 3 sem limite aprovado,
  o valor aprovado com limite.
- CA-12 Paginação: com `limite=5`, o catálogo devolve 5 itens e um
  cursor; seguindo os cursores, chegam os 12, sem repetição.

Tela: Playwright, com Auth e API interceptados, relógio fixo e fuso
America/Sao_Paulo.
- CA-13 O catálogo mostra os 12 cards, cada um com a foto da `imagemUrl`,
  nome, categoria e situação, e "X de Y equipamentos disponíveis agora".
- CA-14 Em manutenção: "Indisponível" desabilitado. Emprestado para
  outra pessoa: "Emprestado" desabilitado, sem nome. Com a própria
  pessoa: "Está com você".
- CA-15 "Solicitar" abre o diálogo com a data de hoje + 14 dias;
  confirmar manda o `equipamentoId`, mostra o aviso e atualiza o card e
  "Meus empréstimos".
- CA-16 Recusa da API: o `detail` aparece na tela.
- CA-17 No limite: os disponíveis mostram "Limite de L itens"
  desabilitado, e o aviso explica o motivo.
- CA-18 Em atraso: os disponíveis mostram "Bloqueado por atraso"
  desabilitado, e o aviso aparece com atalho para "Meus empréstimos".
- CA-19 "Meus empréstimos" mostra nome, patrimônio e "Devolver até" em
  DD/MM/AAAA de cada empréstimo; sem empréstimos, o estado vazio.
- CA-20 O catálogo mostra "Fotos: Pixabay" com link para
  https://pixabay.com/.
- CA-21 A CSP do `vercel.json` tem em `img-src` a origem exata do projeto
  Supabase, sem curinga.

Os dois repositórios:
- CA-22 `npm run openapi:check` (back/) e `npm run api:check` (front/)
  terminam sem diferença.

Fora dos critérios, porque teste não toca produção: o roteiro "Antes de
entregar" do enunciado, no endereço da Vercel, com a conta de teste, por
quem tiver a sessão aberta no navegador (o usuário, depois do merge e do
painel). O professor em zero empréstimos é conferido por consulta só de
leitura no banco (agente) e, se o usuário quiser, entrando com a conta
dele e vendo o estado vazio.

## Fora do escopo
Devolução, tela de Operações, aprovação de pedido, reserva com data
futura, notificação por e-mail, renovação (briefing e PRD), busca e
filtros (L-04), alteração de limite pela interface.

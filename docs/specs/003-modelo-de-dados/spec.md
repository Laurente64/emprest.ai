# 003 — Modelo de dados

Status: APROVADA em 2026-10-22. Ajustes aprovados em 2026-11-05.
Origem: PRD v1 (vibing/docs/PRD.md), itens 1 a 5 e "Regras que
Operações já decidiu". O ADR-001 deixou a modelagem de domínio em
aberto ("O que este ADR não decide").
Repositório afetado: back/ apenas.

## Objetivo
Criar o schema do banco que sustenta a v1 do PRD: equipamentos,
empréstimos e o vínculo das pessoas com o tenant. Esta spec entrega
tabelas, constraints, RLS e seed. Nenhum endpoint e nenhuma tela: cada
item do PRD vira a sua própria spec depois, em cima deste schema.

## Decisões de 2026-10-19
Respostas do usuário:
1. P-01 — há dois tipos de equipamento: unidade (cada um com a sua
   linha, ex.: notebook) e estoque (uma linha com quantidade, ex.:
   "Cabo HDMI — 12").
2. P-04 — empréstimo precisa de aprovação de Operações antes.
3. P-05 — a devolução só fecha quando Operações confirma.
4. P-06 — o limite padrão é 3 itens; limite maior que 3 precisa de
   aprovação.
5. P-12 — aprovar e entregar são dois momentos. O prazo de 14 dias
   começa na entrega.
6. P-13 — enquanto a devolução espera a confirmação, o item continua
   contando como atrasado.
7. P-16 — pedido ainda não aprovado conta no limite.
8. Estoque: um pedido leva uma unidade. Quem precisa de duas faz dois
   pedidos.
9. Estoque: cada unidade emprestada conta no limite (2 cabos + 1
   notebook = 3).
10. P-14 — qualquer pessoa com papel `operacoes` aprova limite maior.
    Operações define o limite direto na membership; o colaborador pede
    fora do sistema.

**Divergência com o PRD:** o PRD diz "no máximo 3 itens ao mesmo
tempo". A decisão 4 transforma isso em "3 por padrão, mais com
aprovação". Esta spec segue a decisão 4. O PRD é do time de
Operações e continua dizendo outra coisa até alguém atualizá-lo.

## Decisões de 2026-11-05
A primeira verificação (vibing/agents/verificador-spec.md) achou
diferenças entre esta spec e o que foi construído. O usuário aceitou
ajustar a spec. O SQL não mudou.
1. `audit_log.id` é `bigserial`: o Prisma não gera identity, e o
   comportamento é o mesmo.
2. `loans` referencia `equipments` por uma única FK de três colunas, com
   a mesma proteção de (id, tracking) mais (id, tenant_id).
3. `tracking` não muda depois de criado: o banco impede quando há
   empréstimo, e o service impede antes disso.
4. Entram na spec, sem terem sido pedidos antes: RLS em
   `_prisma_migrations`; `recusado` exige decided_at e decided_by;
   `display_name` obrigatório; `memberships.user_id` com ON DELETE
   RESTRICT; `condition` com default `ativo`; nome "Suporte TI" no seed;
   `loans.status` com default `solicitado`; todas as FKs com ON DELETE
   RESTRICT e ON UPDATE RESTRICT; ids `uuid` com default
   `gen_random_uuid()`.
5. O antigo CA-13 deixa de ser critério de aceitação. Ele só se prova em
   produção, e teste não toca produção (vibing/rules/checks.md). A prova
   passa a ser a T8 e o passo 2 do checks.md.

## Modelo proposto

Todos os ids são `uuid`, exceto `audit_log.id`. Todas as datas são `timestamptz`. Toda tabela
de domínio tem `tenant_id` NOT NULL com FK para `tenants`
(ADR-001 §6).

### tenants
| Coluna | Tipo | Regra |
|---|---|---|
| id | uuid | PK |
| slug | text | único; o primeiro é `suporte_ti` (ADR-001 §5) |
| name | text | obrigatório |
| created_at | timestamptz | default now() |

### memberships
Vínculo pessoa–tenant com papel (ADR-001 §6).
| Coluna | Tipo | Regra |
|---|---|---|
| id | uuid | PK |
| tenant_id | uuid | FK tenants |
| user_id | uuid | id do usuário no Supabase Auth; FK `auth.users.id`, ON DELETE RESTRICT |
| role | enum `member_role` | `colaborador` ou `operacoes` |
| display_name | text | obrigatório; nome mostrado na tela de Operações (ver P-07) |
| max_open_loans | integer | nulo = usa o padrão (3); preenchido = limite aprovado para esta pessoa |
| max_open_loans_approved_by | uuid | FK memberships; quem aprovou o limite |
| max_open_loans_approved_at | timestamptz | quando aprovou |
| created_at | timestamptz | default now() |

- Único em (tenant_id, user_id).
- Login com membership não pode ser apagado do Auth, e membership com
  empréstimo também não pode ser apagada: isso preserva o histórico. O
  ADR-001 deixa a exclusão de dados (LGPD) em aberto; quando ela for
  decidida, esta regra precisa ser revista.
- CHECK max_open_loans IS NULL OR max_open_loans > 3. Limite menor
  que o padrão não é caso previsto; ver P-14b.
- CHECK: as três colunas `max_open_loans*` são todas nulas ou todas
  preenchidas.
- O padrão 3 é constante no código, não coluna. Decisão 4.
- Cada aprovação e cada alteração de limite gera uma linha em
  `audit_log`, que guarda o histórico. A coluna guarda só o valor
  vigente.

### equipments
Dois tipos (decisão 1), na mesma tabela:
- `unidade`: uma linha = uma peça física, ex.: "Notebook Dell 5440,
  patrimônio 0231".
- `estoque`: uma linha = um conjunto de peças iguais e
  intercambiáveis, ex.: "Cabo HDMI 2 m", quantidade 12.

| Coluna | Tipo | Regra |
|---|---|---|
| id | uuid | PK |
| tenant_id | uuid | FK tenants |
| tracking | enum `equipment_tracking` | `unidade` ou `estoque`; não muda depois de criado |
| name | text | obrigatório |
| category | enum `equipment_category` | `notebook`, `monitor`, `cabo`, `camera`, `outro` (ver P-02) |
| asset_tag | text | patrimônio/etiqueta; único por tenant; só em `unidade` (ver P-03) |
| quantity_total | integer | só em `estoque`: quantas peças utilizáveis existem |
| condition | enum `equipment_condition` | `ativo`, `manutencao`, `baixado`; default `ativo` |
| created_at | timestamptz | default now() |
| updated_at | timestamptz | atualizado a cada alteração |

- Único em (id, tenant_id, tracking), para a FK composta de `loans` (ver
  loans).
- `tracking` não muda depois de criado: com empréstimo, o ON UPDATE
  RESTRICT da FK de `loans` impede; sem empréstimo, quem impede é o
  service. Ver "Regras que NÃO ficam no banco".
- CHECK: `unidade` exige quantity_total nulo e asset_tag preenchido;
  `estoque` exige quantity_total >= 0 e asset_tag nulo.
- "Emprestado" NÃO é um valor de `condition`. Ele é derivado dos
  empréstimos em andamento. Guardar nos dois lugares é como a
  planilha de hoje diverge da realidade.
- Peça de estoque quebrada: Operações baixa `quantity_total`. A
  `condition` de um item de estoque vale para o conjunto inteiro. Ver
  P-17.
- A situação do catálogo (PRD item 2) é calculada assim:
  - `baixado` → não aparece no catálogo;
  - `manutencao` → aparece como "em manutenção", nunca como disponível
    (PRD, regras);
  - `unidade` `ativo` com empréstimo em andamento → "indisponível"
    (ver P-15);
  - `estoque` `ativo` → disponível = quantity_total − empréstimos em
    andamento; com zero, "indisponível";
  - nos demais casos, "disponível".
- Equipamento com histórico não é apagado: vai para `baixado` (FK
  `loans → equipments` com ON DELETE RESTRICT).

### loans
Um empréstimo nasce como pedido e passa por estes estados
(decisões 2 e 3):

```
solicitado ──aprova──▶ aprovado ──entrega no balcão──▶ ativo
    │  │                  │                              │
    │  └─recusa─▶ recusado└─cancela─▶ cancelado           ├─colaborador devolve─▶ devolucao_solicitada
    └─cancela─▶ cancelado                                │                              │
                                                         └──Operações confirma no balcão┴──▶ devolvido
```

- `solicitado`: o colaborador pediu.
- `aprovado`: Operações aprovou; o item ainda não foi retirado. Ver
  P-12b.
- `ativo`: o item foi entregue no balcão. O prazo começa aqui.
- `devolucao_solicitada`: o colaborador disse que devolveu. O
  empréstimo continua aberto.
- `devolvido`: Operações confirmou. Só Operações leva até aqui. Ela
  também pode confirmar direto a partir de `ativo`, quando a pessoa
  aparece no balcão sem ter marcado nada.
- `recusado` e `cancelado`: encerram o pedido sem entrega.

"Em andamento" = `solicitado`, `aprovado`, `ativo` ou
`devolucao_solicitada`.

| Coluna | Tipo | Regra |
|---|---|---|
| id | uuid | PK |
| tenant_id | uuid | FK tenants |
| equipment_id | uuid | FK equipments, ON DELETE RESTRICT |
| equipment_tracking | enum `equipment_tracking` | cópia do tipo do equipamento; FK composta (equipment_id, tenant_id, equipment_tracking) → equipments (id, tenant_id, tracking) garante que a cópia não diverge e que o equipamento é do mesmo tenant |
| borrower_membership_id | uuid | FK memberships: quem pediu e fica com o item |
| status | enum `loan_status` | os sete estados acima |
| requested_at | timestamptz | default now() |
| decided_at | timestamptz | quando Operações aprovou ou recusou |
| decided_by_membership_id | uuid | FK memberships; quem aprovou ou recusou |
| rejection_reason | text | opcional, só em `recusado` |
| handed_over_at | timestamptz | entrega no balcão |
| handed_over_by_membership_id | uuid | FK memberships; quem entregou |
| due_at | timestamptz | gravado na entrega: handed_over_at + 14 dias |
| return_requested_at | timestamptz | quando o colaborador marcou a devolução |
| returned_at | timestamptz | quando Operações confirmou |
| returned_by_membership_id | uuid | FK memberships; quem confirmou |
| canceled_at | timestamptz | quando foi cancelado |
| updated_at | timestamptz | atualizado a cada alteração |

- Um empréstimo é sempre de uma peça (decisão 8): não há coluna de
  quantidade.
- Índice único parcial em (equipment_id) WHERE equipment_tracking =
  'unidade' AND status IN ('solicitado', 'aprovado', 'ativo',
  'devolucao_solicitada'): uma unidade tem no máximo um empréstimo em
  andamento. É para isso que `equipment_tracking` existe em `loans`:
  índice parcial não enxerga coluna de outra tabela. Ver P-15.
- Para `estoque`, o banco não impede passar de `quantity_total`. O
  service trava a linha do equipamento (`SELECT … FOR UPDATE`) e conta
  os empréstimos em andamento antes de inserir. Ver "Regras que NÃO
  ficam no banco".
- CHECKs por estado, para o banco recusar linha incoerente:
  - `aprovado`, `recusado` e além exigem decided_at e decided_by;
  - `ativo` e além exigem handed_over_at, handed_over_by e due_at;
  - `devolucao_solicitada` exige return_requested_at;
  - `devolvido` exige returned_at e returned_by;
  - `cancelado` exige canceled_at;
  - due_at > handed_over_at; returned_at >= handed_over_at.
- O banco não garante a ordem das transições (ex.: `devolvido` voltar
  para `ativo`). Isso é do service. Ver "Regras que NÃO ficam no
  banco".
- `due_at` é gravado e não recalculado. Se o prazo padrão mudar, os
  empréstimos em andamento mantêm o prazo com que foram entregues.
- "Em atraso" é derivado: status IN ('ativo', 'devolucao_solicitada')
  AND due_at < now(). Não há coluna nem job para isso (ADR-001 §9:
  sem cron). Decisão 6.
- Índice em (tenant_id, borrower_membership_id) WHERE em andamento,
  para a regra do limite, a regra do atraso e a tela "meus
  empréstimos".
- Índice em (tenant_id, status), para a tela de Operações (PRD item 5)
  e as filas de "pedidos para aprovar" e "devoluções para confirmar".

### audit_log
Exigida pelo ADR-001 §10 ("retrofitar log de auditoria depois não
recupera o passado"). Ver P-08.
| Coluna | Tipo | Regra |
|---|---|---|
| id | bigserial | PK |
| tenant_id | uuid | FK tenants |
| actor_user_id | uuid | quem fez |
| action | text | ex.: `loan.approve`, `loan.return_confirm`, `membership.limit_approve` |
| resource_type | text | ex.: `loan` |
| resource_id | uuid | id do recurso |
| payload | jsonb | antes e depois, quando houver |
| created_at | timestamptz | default now() |

- Append-only: trigger rejeita UPDATE e DELETE.

### RLS
- RLS habilitado em todas as tabelas acima, sem exceção
  (vibing/rules/migration.md, item 4), e em `_prisma_migrations`, que o
  Prisma cria no schema `public`.
- A API conecta com o role dono das tabelas e bypassa RLS (ADR-001
  §7). As policies são defesa em profundidade: se alguém usar a anon
  key ou o token de um usuário direto na API REST do Supabase, não lê
  nem escreve nada.
- Policy proposta para cada tabela: RESTRICTIVE, FOR ALL, TO anon,
  authenticated, USING (false) WITH CHECK (false). Ver P-09.

### Regras que NÃO ficam no banco
Nota de 2026-11-11: a spec 006 (C-01), por decisão do usuário, faz o
pedido nascer `ativo`, sem aprovação nem entrega por Operações, com a
própria membership nas colunas de decisão e entrega e `autoatendimento`
na `audit_log`. As transições abaixo continuam valendo para quando a
aprovação de Operações existir.

Ficam no service das specs de funcionalidade, dentro de transação:
- Limite de empréstimos em andamento por pessoa: `max_open_loans` da
  membership, ou 3 se for nulo. Conta todo empréstimo em andamento,
  inclusive `solicitado`, e cada peça de estoque conta como um
  (decisões 7 e 9).
- Item de estoque só é pedido se quantity_total − empréstimos em
  andamento > 0.
- Só papel `operacoes` altera `max_open_loans` (decisão 10).
- Quem tem item em atraso não pede outro.
- Equipamento em `manutencao` ou `baixado` não pode ser pedido.
- `tracking` de equipamento sem empréstimo não muda.
- A ordem das transições de `loans`, e quem pode fazer cada uma:
  aprovar, recusar, entregar e confirmar devolução são de Operações;
  pedir, cancelar e marcar devolução são do colaborador.

O schema precisa permitir fazer isso sem corrida. A proposta é que o
service trave a linha da membership (`SELECT … FOR UPDATE`) antes de
contar os empréstimos em andamento, e que o índice único parcial de
`loans` segure dois pedidos simultâneos do mesmo equipamento. O
detalhe é do plano da spec de "solicitar empréstimo", não desta.

## Seed
Nota de 2026-11-11: a spec 005 muda esta seção. O seed passa a criar os
12 equipamentos da prática e as contas do professor e de teste.

`prisma/seed.ts`, idempotente, cria o tenant `suporte_ti` (ADR-001
§5), com o nome "Suporte TI". Não cria usuário nem equipamento. Ver
P-10.

## Critérios de aceitação
Todos verificados por teste contra Postgres em Testcontainers
(ADR-001 §8), nunca contra o Supabase.
- CA-01 A migration cria as tabelas `tenants`, `memberships`,
  `equipments`, `loans` e `audit_log`, e os enums descritos acima.
- CA-02 Toda tabela, exceto `tenants`, tem `tenant_id` NOT NULL com FK
  para `tenants`.
- CA-02b Uma linha que aponta para registro de outro tenant (ex.:
  empréstimo do tenant A com equipamento do tenant B) é rejeitada
  pelas FKs compostas com `tenant_id`. Incluído em 2026-10-26 (plano,
  D-03).
- CA-03 Todas as tabelas têm RLS habilitado e pelo menos uma policy;
  um SELECT feito com o role `authenticated` ou `anon` não retorna
  nenhuma linha, e um INSERT com esses roles falha.
- CA-04 Inserir um segundo empréstimo em andamento (qualquer um dos
  quatro estados) para o mesmo equipamento do tipo `unidade` falha por
  constraint. Depois que o primeiro vai para `devolvido`, `recusado`
  ou `cancelado`, o segundo entra. Para um equipamento do tipo
  `estoque`, dois empréstimos em andamento entram.
- CA-04b Empréstimo cujo `equipment_tracking` difere do `tracking` do
  equipamento é rejeitado pela FK composta.
- CA-04c Equipamento `unidade` com quantity_total, ou `estoque` sem
  quantity_total, com quantidade negativa ou com asset_tag, é
  rejeitado.
- CA-05 Para cada estado de `loans`, uma linha sem as colunas que o
  estado exige é rejeitada pelo banco. Um teste por estado.
- CA-06 Empréstimo com `due_at <= handed_over_at`, ou com
  `returned_at` anterior a `handed_over_at`, é rejeitado pelo banco.
- CA-07 Apagar um equipamento que tem empréstimo falha.
- CA-08 Uma segunda membership para o mesmo (tenant, user) falha.
- CA-09 Dois equipamentos com o mesmo `asset_tag` no mesmo tenant
  falham; em tenants diferentes, entram.
- CA-10 `max_open_loans` igual ou menor que 3 é rejeitado.
  `max_open_loans` preenchido sem quem aprovou e quando, ou o inverso,
  também é rejeitado.
- CA-11 UPDATE e DELETE em `audit_log` são rejeitados.
- CA-12 Rodar o seed duas vezes deixa exatamente um tenant com slug
  `suporte_ti`.

Fora destes critérios, porque teste não toca produção: a migration
aplicada em produção é exatamente o SQL aprovado, e `prisma migrate
status` não mostra pendência. A prova é a saída dos comandos na T8
(vibing/rules/migration.md e vibing/rules/checks.md, passo 2).

## Fora do escopo
- Endpoints, telas, CASL, guards e validação de JWT. Cada item do PRD
  é uma spec própria em cima deste schema. Isso inclui as telas de
  aprovação de pedido, de confirmação de devolução e de aprovação de
  limite.
- Convite e cadastro de usuário (ADR-001 §7, "Cadastro fechado").
- Tabela de rate limit (ADR-001 §10). Entra com o primeiro endpoint
  autenticado.
- Reserva com data futura, notificação por e-mail e importação da
  planilha (PRD, "O que NÃO entra"). Sem e-mail, Operações descobre
  pedido novo pela própria tela.
- Histórico de manutenção.

## Respostas por default (2026-10-19)
O usuário aceitou a proposta do rascunho para as perguntas que
restavam. Ficam assim:
- P-02 Categoria é lista fixa (enum `equipment_category`). Categoria
  nova exige migration.
- P-03 `asset_tag` é obrigatório em `unidade` e único por tenant. Sem
  número de série.
- P-06 O prazo de 14 dias é fixo no código, sem exceção por pessoa.
- P-07 O nome mostrado vem de `memberships.display_name`. O e-mail não
  é copiado para o nosso schema: fica só no Supabase Auth.
- P-08 A `audit_log` entra nesta spec.
- P-09 A policy de RLS nega tudo para `anon` e `authenticated`.
- P-10 O seed roda em produção uma vez, porque sem o tenant
  `suporte_ti` nada funciona. O comando entra no plano e só roda
  depois de aprovado, como a migration.
- P-11 Equipamento emprestado pode ir para `manutencao`. Ele deixa de
  estar disponível depois de devolvido.
- P-12b Pedido aprovado e nunca retirado não expira. Operações cancela
  à mão.
- P-14b Limite maior não tem teto nem data de validade. Operações volta
  a pessoa para o padrão à mão.
- P-15 Uma unidade com pedido `solicitado` ou `aprovado` aparece como
  indisponível para os outros.
- P-17 Peça de estoque quebrada sai de `quantity_total`. Não há coluna
  de quantidade em manutenção.

## Lacunas técnicas para o plano
- `memberships.user_id` aponta para `auth.users`, que fica em outro
  schema. O Prisma só modela isso com o recurso `multiSchema`; sem ele,
  a FK entra como SQL bruto na migration. Fica para o plano.
- Testcontainers precisa do daemon do Docker, e ele nunca foi
  conferido nesta máquina (AGENTS.md, "Ambiente"). Sem ele, nenhum
  CA desta spec pode ser verificado.
- Prisma ainda não está instalado no back/. A instalação e a leitura
  de `DATABASE_URL`/`DIRECT_URL` seguem vibing/rules/secrets.md.
- Divergência de região (us-west-2 no .env, sa-east-1 no ADR) segue
  aberta e não afeta o schema.

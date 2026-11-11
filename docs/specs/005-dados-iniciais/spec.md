# 005 — Dados iniciais

Status: APROVADA em 2026-11-11 pelo agente, por delegação do usuário
(ver spec 004, "Aprovação"). As decisões marcadas "usuário" foram
respondidas por ele em 2026-11-10; as do agente estão em "Para o usuário
confirmar". Emendada em 2026-11-11 depois da rodada 1 do verificador
(item 10 e CA-05): verificacoes/2026-11-11-rodada-1.md.
PRONTA em 2026-11-11: verificacoes/2026-11-11-rodada-2.md (back 4f61505).
Origem: briefing "dados iniciais" da prática (pedido, não spec) e a
lista de equipamentos do enunciado. PRD v1, item 2, precisa de
equipamentos para existir.
Repositório afetado: back/. Produção: banco, Auth e Storage do Supabase.
Depende de: spec 004 (login e D-01, o tenant no `app_metadata`).

## Objetivo
O sistema em produção tem os 12 equipamentos da prática, cada um com a
sua foto, a conta do professor para a correção e uma conta de teste do
usuário. Popular de novo não duplica nada.

## Comportamento
1. Os 12 equipamentos são exatamente os da lista do enunciado, copiada
   para back/prisma/dados/equipamentos.json (patrimônio, nome,
   categoria, situação e termo de busca da foto).
2. Todos são `tracking = unidade`, com `asset_tag` = patrimônio: cada
   um tem patrimônio próprio, e na spec 003 patrimônio só existe em
   `unidade`. `name` = nome, `category` = categoria.
3. Situação: `condition = manutencao` para TI-0004, TI-0007 e TI-0010;
   `ativo` para os outros 9. Ver conflito C-01.
4. Coluna nova `equipments.image_path` (text, nula): o caminho da foto
   no bucket público `equipamentos` do Supabase Storage. Os 12 recebem
   `<patrimonio>.jpg`. A API monta a URL pública (spec 006, item 1).
   DDL: `ALTER TABLE "equipments" ADD COLUMN "image_path" TEXT;`. Coluna
   nula e sem default: as linhas que existirem ficam com nulo, e o código
   da main não lê a coluna (vibing/rules/migration.md, itens 5 e 6).
5. Fotos: uma por equipamento, preparadas pela skill
   vibing/skills/imagens-equipamentos (Pixabay, 800×600, sem repetição,
   créditos, revisão visual), versionadas em back/prisma/dados/imagens/
   e enviadas ao bucket pelo script da skill.
6. Conta do professor: `marcelo.grilo@institutojef.org.br`, criada
   confirmada pela API de administração do Supabase Auth, com
   `app_metadata.tenant_id` do tenant `suporte_ti` (spec 004, D-01).
   Membership `colaborador`, `display_name` "Marcelo Grilo". A senha vem
   de `SEED_PROFESSOR_SENHA`. Ver conflito C-02.
7. Conta de teste do usuário: e-mail, senha e nome vêm de
   `SEED_TESTE_EMAIL`, `SEED_TESTE_SENHA` e `SEED_TESTE_NOME`. Sem
   e-mail, o seed não cria a conta de teste; com e-mail e sem senha, o
   seed falha. Mesmo papel e mesmo tenant. É a conta usada nos testes em
   produção: a do professor chega à correção com zero empréstimos.
8. O seed não cria empréstimo e não apaga nada.
9. Idempotência, por item:
   - equipamento, por (tenant, patrimônio): cria; se existe, atualiza só
     nome, categoria e `image_path`. Situação e `tracking` não são
     reescritos: situação é dado operacional depois que o sistema está
     no ar, e `tracking` não muda (spec 003).
   - conta, por e-mail: cria; se existe, junta `tenant_id` ao
     `app_metadata` que já existe (sem apagar `provider` e `providers`) e
     não troca a senha.
   - membership, por (tenant, conta): cria; se existe, não altera.
   O seed escreve no Auth e no banco em passos separados: se falhar no
   meio, rodar de novo completa o que faltou.
10. Faltando `SUPABASE_URL`, `SUPABASE_SERVICE_ROLE_KEY` ou
    `SEED_PROFESSOR_SENHA`, ou com `SEED_TESTE_EMAIL` sem
    `SEED_TESTE_SENHA`, o seed para antes de escrever qualquer coisa.
    Variável vazia (`""`, como no `.env.example`) conta como ausente
    (emenda de 2026-11-11, rodada 1 do verificador).
11. Variáveis: ver a tabela da spec 004. As do seed e do script ficam só
    no `back/.env` e no `.env.example`.

## Quem escreve em produção
- O enunciado diz: "O agente nunca escreve em produção. Tudo que escreve
  no banco ou no Storage do deploy é executado por você." A
  rules/migration.md (passo 8) e a spec 003 (P-10) dizem que o agente
  aplica a migration e roda o seed depois da aprovação.
- Decisão do usuário (2026-11-10): o agente roda a migration e o seed
  nesta prática, e o registro diz que foi o agente. A revisão do SQL, que
  seria do usuário, é feita por um subagente revisor de contexto limpo,
  por delegação.
- O envio das fotos ao Storage fica com o usuário: a skill exige que o
  agente não consiga enviar (trava de terminal interativo).
- Consequência registrada: enquanto a chave secreta estiver no
  `back/.env`, o agente tem como rodar o seed (e, escrevendo outro
  código, enviar fotos). A garantia completa é a chave fora do disco; o
  usuário a rotaciona depois da prática.

## Conflitos com o que foi pedido (decisões do usuário, 2026-11-10)
- C-01 Manutenção: a lista do enunciado ("use exatamente esta, dados
  iguais para todos") marca TI-0004, TI-0007 e TI-0010; o briefing diz
  "TI-0004 e TI-0007 em manutenção; os outros 10 disponíveis". Vale a
  lista: o próprio item 1 do briefing aponta para ela. 3 em manutenção,
  9 disponíveis.
- C-02 Senha: o briefing pede "deixe a senha escrita no próprio arquivo
  que popula os dados". Isso contraria a regra da prática ("Nenhuma
  senha, token ou chave em código ou no histórico do git") e
  vibing/rules/secrets.md. A senha fica em variável de ambiente
  (`back/.env`, fora do git; `.env.example` sem valor).
- C-03 sharp: o briefing manda instalar; não estava no ADR-001. O
  usuário aprovou, e o a decisão virou o ADR-003 (dependência de
  desenvolvimento, só no script de imagens).
- C-04 Fonte das fotos: o contrato da skill diz "só a API do Pexels"; o
  usuário decidiu usar o Pixabay. O crédito na tela passa a ser "Fotos:
  Pixabay".

## Conflito com as rules (agente)
- C-05 restrictions.md: "Não altere o Supabase fora de uma migration:
  nada de SQL avulso, de Studio, de alteração de schema ou de dado". O
  seed (já aceito na spec 003, P-10), a criação das contas pela API de
  administração do Auth e o envio das fotos ao Storage escrevem dado fora
  de migration. Nenhum deles cabe numa migration do Prisma: contas e
  arquivos não são tabelas nossas. Tratamento: são procedimentos
  versionados, revisados, idempotentes e com saída registrada, do mesmo
  tipo do seed da spec 003. A rule precisa dizer isso (ENTREGA.md, §7).

## Lacunas decididas (agente, por delegação)
- L-01 `tracking` dos cabos: `unidade`, não `estoque` (a spec 003 dá
  "Cabo HDMI — 12" como exemplo de estoque). A lista dá patrimônio
  próprio a cada cabo, e patrimônio só existe em `unidade`.
- L-02 Papel do professor: `colaborador`. A prática testa o fluxo de
  quem pede empréstimo; não há tela de Operações.
- L-03 Onde fica a foto: coluna `image_path` com o caminho, não a URL
  inteira. A URL depende do projeto Supabase e vem do ambiente.
  Alternativa descartada: derivar do patrimônio sem coluna (equipamento
  cadastrado depois sem foto ficaria com URL quebrada).
- L-04 Créditos: no `creditos.json` versionado ao lado das fotos, não em
  colunas do banco. O contrato pede que fique registrado; nenhuma tela
  mostra crédito por foto.
- L-05 O seed não reescreve a situação de equipamento que já existe
  (item 9), nem a senha de conta que já existe.
- L-06 Geração da migration sem banco: `prisma migrate diff
  --from-schema-datamodel <schema da main> --to-schema-datamodel
  prisma/schema.prisma --script`. Comparando dois `schema.prisma`, o
  Prisma não enxerga os objetos de SQL bruto da primeira migration (RLS,
  CHECKs, índices parciais) dos dois lados, e por isso não gera `DROP`
  para eles, que é o risco registrado no plano da spec 003. Alternativa:
  shadow database no Testcontainers (`--from-migrations`), que exige
  recriar a imitação do Supabase antes de cada geração. O comando é novo
  nesta máquina (migration.md, passo 2): o SQL gerado passa pela revisão.

## Para o usuário confirmar
L-01, L-02, L-05, L-06 e C-05.

## Critérios de aceitação
Banco: Jest contra Postgres no Testcontainers (ADR-001 §8). A API de
administração do Auth é substituída por uma imitação que grava em
`auth.users` do banco de teste. Nunca o Supabase.
- CA-01 A migration nova adiciona `equipments.image_path` (text, nula).
  Aplicada sobre o banco da spec 003, as linhas existentes ficam com
  nulo.
- CA-02 O seed cria exatamente os 12 equipamentos da lista, com
  patrimônio, nome, categoria e `tracking = unidade`.
- CA-03 TI-0004, TI-0007 e TI-0010 ficam em `manutencao`; os outros 9,
  em `ativo`.
- CA-04 Cada equipamento tem `image_path = <patrimonio>.jpg`, e o arquivo
  existe em back/prisma/dados/imagens/.
- CA-05 Rodar o seed duas vezes com a conta de teste configurada deixa 1
  tenant, 12 equipamentos, 2 memberships e 0 empréstimos, e cria cada
  conta uma vez só. Sem `SEED_TESTE_EMAIL`, ausente ou vazia, deixa 1
  membership.
- CA-06 A conta do professor é criada com o e-mail do briefing,
  confirmada, com `app_metadata.tenant_id` do `suporte_ti` (somado ao
  `app_metadata` existente), membership `colaborador` "Marcelo Grilo" e
  zero empréstimos.
- CA-07 Sem `SEED_PROFESSOR_SENHA`, sem `SUPABASE_URL`, sem
  `SUPABASE_SERVICE_ROLE_KEY`, ou com `SEED_TESTE_EMAIL` sem
  `SEED_TESTE_SENHA`, o seed falha e não escreve nada; a senha da conta
  criada é a do ambiente.

Fotos: Jest, lendo os arquivos versionados.
- CA-08 Há 12 fotos JPEG, uma por patrimônio, e cada uma tem exatamente
  800×600.
- CA-09 Nenhuma foto se repete: ids do Pixabay e sha256 únicos.
- CA-10 O `creditos.json` tem as 12 fotos com id e página no Pixabay,
  autor, licença e link da licença, termo de busca, e o sha256 bate com
  o arquivo.
- CA-11 Toda página e todo download registrados são do Pixabay, e a
  verificação de origem do script recusa outros hosts.
- CA-12 Toda foto tem aprovação no `revisao.json` com o mesmo id do
  `creditos.json`; sem isso, a conferência que o envio faz antes de
  qualquer chamada de rede recusa.
- CA-13 O envio recusa rodar fora de um terminal interativo, antes de
  qualquer chamada de rede.

Fora dos critérios, porque teste não toca produção (provas coladas na
tarefa de produção, como na spec 003):
- a migration aplicada é a revisada, e `prisma migrate status` não
  mostra pendência;
- o seed rodou duas vezes sem duplicar; os 12 existem no banco com a
  situação decidida; as duas contas existem, confirmadas, com o
  `tenant_id`; o professor tem zero empréstimos (consulta só de leitura);
- as 12 URLs públicas abrem sem login e têm 800×600 (depois do envio,
  que é do usuário);
- `git log -p --all -S<senha padrão>` nos três repositórios não encontra
  nada (o comando lê a senha do `back/.env`, sem escrevê-la);
- o professor entra pelo endereço da Vercel com a senha padrão (usuário,
  depois do merge e do painel).

## Fora do escopo
Tela de catálogo (spec 006), cadastro de equipamento pela interface,
importação da planilha (briefing e PRD) e empréstimos.

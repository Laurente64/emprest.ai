# Entrega da prática

Turma REST: três repositórios. Este (vibing/) tem documentação, specs,
rules, skills e o subagente; o código está em back/ e front/. A tag
`entrega-pratica` vai nos três.

Como ler: toda afirmação aponta para arquivo, commit ou rodada do
verificador. Commits sem repositório indicado são do vibing/; os de
código dizem `back` ou `front`.

## Links
- Repositório de documentação e decisões: https://github.com/Laurente64/emprest.ai
- API (back): https://github.com/Laurente64/emprest.ai-back
- SPA (front): https://github.com/Laurente64/emprest.ai-front
- Produção (Vercel): API — https://emprest-ai-back-sigma.vercel.app · front — endereço do projeto front na Vercel (preencher antes da tag)

## Quem fez o quê nesta entrega
Fui eu (aluno) quem respondeu às perguntas de negócio e de conflito. O
agente (Claude Code) escreveu specs, código e testes, e com autorização
minha registrada também:
- aprovou as specs 004, 005 e 006 por delegação, enquanto eu estava
  ausente, depois de um subagente revisor de contexto limpo (spec 004,
  "Aprovação"; tarefas da 004, "Revisão");
- aplicou a migration e rodou o seed em produção (spec 005, "Quem escreve
  em produção"; registro em 005/tarefas.md, T4);
- fez a revisão visual das fotos (`revisao.json`, campo `por`).

O envio das fotos ao Storage não foi feito pelo agente: o script trava
sem terminal interativo, e o envio fica comigo (seção 4). As decisões que
o agente tomou sozinho estão listadas em "Para o usuário confirmar" de
cada spec, e eu as confiro antes da tag.

## 1. Conflitos: o que foi pedido contra o que já existia

| pedido | contrariava o quê | o que você fez | por quê | onde ficou registrado | commit |
|--------|-------------------|----------------|---------|-----------------------|--------|
| Enunciado: "já está no ar com login" | AGENTS.md, "Ainda NÃO existe": autenticação, guards, contexto de tenant | Spec nova (004) antes das outras duas | Sem login, nem o professor entra nem existe "colaborador logado" | specs/004-login/spec.md, "Origem" | be36206 |
| Briefing de empréstimo: "solicitar cria o empréstimo na hora" | Spec 003, decisões 2 e 5 (aprovação e entrega por Operações) | Nasce `ativo`, com a própria membership na decisão e na entrega e `autoatendimento` na `audit_log`; sem migration | Sem tela de Operações, nenhum pedido sairia de `solicitado`; os CHECKs da 003 continuam valendo | 006 C-01; nota na spec 003 | be36206 · back 1cc310c |
| Briefing de empréstimo: limite 5 | PRD ("no máximo 3") e spec 003, decisão 4 | Limite 3 | Regra de negócio é do PRD; o briefing é pedido | 006 C-04 | be36206 |
| Briefing de empréstimo: "14 dias úteis" | PRD e spec 003 (P-06): 14 dias | 14 dias corridos | Mesmo motivo; e "úteis" não diz quais feriados | 006 C-02 | be36206 |
| Briefing de empréstimo, item 9: catálogo "sem rodapé nem texto de crédito" | Contrato da skill e regra da API de imagens (mostrar a origem) | Crédito "Fotos: Pixabay" mantido, discreto | Exigência da fonte da imagem vale mais que preferência visual | 006 C-03, item 14, CA-20 | be36206 |
| Briefing de dados: "deixe a senha escrita no próprio arquivo" | Regra da prática e rules/secrets.md | Senha em `SEED_PROFESSOR_SENHA` (`back/.env`, fora do git) | Senha no git não sai mais do histórico | 005 C-02; CA-07; `git log -S` vazio nos três repositórios (005/tarefas.md, T4) | back 4eafb0e |
| Briefing de dados: "instale a biblioteca sharp" | rules/restrictions.md e ADR-001 (biblioteca fora do ADR) | Eu aprovei; o ADR-001 ganhou o §12 (devDependency, só no script) | 800×600 precisa ser garantido por código | adr/001-stack.md §12 | 5ca5c67 |
| Briefing de dados: "TI-0004 e TI-0007 em manutenção; os outros 10" | A lista do próprio enunciado ("use exatamente esta") marca também TI-0010 | Vale a lista: 3 em manutenção, 9 disponíveis | O item 1 do briefing aponta para a lista | 005 C-01; CA-03 | be36206 |
| Contrato da skill: "só a API do Pexels" | O Pexels não funcionou para mim | Pixabay, com crédito "Fotos: Pixabay" | Mesma garantia (fonte única, licença, crédito) com outro provedor | 005 C-04; skills/imagens-equipamentos/SKILL.md | 710fc5a |
| Tech lead: "o verificador pode corrigir o teste quebrado" | Requisito "Independência" do enunciado; AGENTS.md, passo 4 | Recusado | Quem verifica e corrige perde o olhar independente; teste quebrado é REPROVADO e volta | agents/verificador-spec.md, "Proibições"; AGENTS.md passo 4 | 608cc80 |
| Enunciado: "o agente nunca escreve em produção" | Minha autorização para o agente rodar migration e seed nesta prática | O agente rodou e o registro diz isso; o envio das fotos ficou comigo | Eu não estaria disponível; a skill exige que o agente não consiga enviar | 005, "Quem escreve em produção" | be36206 |
| Spec 001: "Nada pede login" e "sem link Entrar" | O professor entra pelo endereço da Vercel | Emenda da spec 001: link "Entrar" | Sem o link, não se acha o login | 004 D-04; spec 001 emendada; CA-17 | be36206 · front 734c5fd |
| layout.md: card sem foto, "Com {pessoa}" no card, "Entrar como Operações" | Briefing (imagem obrigatória) e PRD ("cada pessoa vê os próprios empréstimos"); ADR-001 §6 (papel da membership) | Foto no card; card não mostra quem está com o item; sem botão de Operações | — | 006 L-03 e L-05; 004 D-03 | be36206 |
| Spec 003: "rate limit entra com o primeiro endpoint autenticado" | Escopo da prática | Adiado | Decisão do agente; pendência na seção 8 | 004 D-08 | be36206 |
| rules/secrets.md: variável "em três lugares ou em nenhum" | Variáveis só de script (chave secreta, senhas do seed, chave do Pixabay) | Ficam no `.env` e no `.env.example`, não na Vercel | Chave secreta num deploy que não a usa é exposição sem uso | Tabela de variáveis da spec 004 | be36206 |
| rules/restrictions.md: "nada de dado fora de migration" | Contas do Auth e arquivos do Storage não cabem em migration do Prisma | Só por procedimento versionado e idempotente: seed e script da skill | — | 005 C-05 | be36206 |

## 1b. Lacunas: o que foi pedido de forma incompleta

| spec | o que faltava | alternativas | o que decidiu | por quê | commit |
|------|---------------|--------------|---------------|---------|--------|
| 004 | De onde vem o tenant no token | Custom Access Token Hook; buscar a membership só pelo `sub` | `app_metadata.tenant_id`, gravado pelo seed com a chave secreta | Cumpre o ADR-001 §6 (claim do JWT) sem migration nem painel; o usuário não altera `app_metadata`. Limitação: um tenant por conta | back 24e400b |
| 004 | Como validar o token na API | `jose` (fora do ADR); WebCrypto à mão | `supabase-js` com a chave escolhida pela API, sem cair no `getUser` | Biblioteca já no ADR-001 §7; criptografia à mão é risco | back 24e400b · b3733ce |
| 004 | Idioma de módulo novo | inglês, como `health` | Português (`autenticacao`, `eu`, `emprestimos`) | Idioma do domínio no PRD; models do Prisma seguem em inglês | back 24e400b |
| 004 | Comportamento de deploy e de falha que a spec não dizia | — | Rewrite de SPA no `vercel.json`; "Sair" só neste aparelho; mensagens de falha do Auth e de `/v1/eu` | Achados na execução e na rodada 1 do verificador; levados para a spec | front 734c5fd · bfe7458 |
| 005 | `tracking` dos cabos | `estoque` (exemplo da spec 003) | `unidade` | Cada cabo tem patrimônio, e patrimônio só existe em `unidade` | back 4eafb0e |
| 005 | Papel do professor | `operacoes` | `colaborador` | A prática testa o fluxo de quem pede | back 4eafb0e |
| 005 | Onde fica a foto e o crédito | URL inteira na coluna; colunas de crédito | Caminho em `image_path`; crédito em `creditos.json` versionado | A URL depende do projeto; nenhuma tela mostra crédito por foto | back 996e724 · 8a0f66c |
| 005 | Seed de novo sobre dado existente | Reescrever tudo | Não reescreve situação nem senha | Situação é dado operacional depois que o sistema está no ar | back 4eafb0e |
| 005 | Comando para gerar a 2ª migration | Shadow database no Testcontainers | `migrate diff` entre duas versões do `schema.prisma` | Não toca banco e não gera `DROP` do SQL bruto | back 996e724 |
| 006 | Ordem das recusas | Equipamento antes da pessoa | 404, atraso, limite, manutenção, indisponível | Quem está no limite sabe disso antes de tentar outro item | back 1cc310c |
| 006 | Como a tela mostra o limite | Deixar pedir e mostrar a recusa | Botão desabilitado com o motivo + aviso; a API recusa de qualquer jeito | layout.md §7.4–7.5; o `detail` aparece se o pedido chegar | front (spec 006) |
| 006 | Estoque | Tratar só `unidade` | Regra da spec 003 (sobra de `quantity_total`) | A spec 003 tem o tipo; nenhum dos 12 é de estoque | back 1cc310c |
| 006 | Busca e filtros do layout.md | Implementar | Fora | O briefing não pede; são 12 itens | — |
| 006 | Dados que sobram em produção depois do roteiro | Corrigir dado à mão | Registrar o estado esperado na correção | Não há devolução nesta versão | 006 L-08 |
| skill | O que é "texto" na revisão de imagem | Reprovar qualquer letra | Marcação genérica do objeto (teclas, "WEB CAMERA", "18-55 OIS") passa com `observacao`; marca, papel e tela não | Sem isso nenhuma foto de notebook ou câmera passaria | references/revisao-visual.md · 2de7aa7 |
| 006 | Versão do CASL | 7.1.2 (publicada um dia antes) | 6.8.1 | Linha 7 com duas semanas; 6.8.1 madura | back 1cc310c |

## 2. Onde o agente tentou decidir por mim
- **Host do download de fotos.** O agente escreveu o script aceitando só
  `cdn.pixabay.com`, sem conferir a resposta da API. A primeira rodada
  não achou nenhuma foto; ao inspecionar a resposta, o `largeImageURL`
  vinha de `pixabay.com/get/…`. Corrigido no script e no SKILL.md
  (005/tarefas.md, T1).
- **Comportamento de produção não escrito.** O subagente que fez o front
  da 004 decidiu sozinho o rewrite de SPA, as mensagens de falha e o
  escopo do "Sair". Ele registrou, e a rodada 1 do verificador apontou
  como "fora de escopo". O agente principal levou tudo para a spec como
  emenda, e o "Sair" global virou local (004/verificacoes/rodada-1).
- **Aprovação.** O primeiro rascunho das specs marcava como "decisão"
  coisas que eram escolha do agente (tenant no `app_metadata`, idioma dos
  módulos, adiar o rate limit). O revisor de contexto limpo apontou que
  isso atribuiria a mim decisões do agente; cada spec ganhou "Para o
  usuário confirmar".
- **Instruções do próprio agente.** Ao fim, o agente tentou afrouxar
  rules/restrictions.md e rules/secrets.md por conta própria; o
  classificador do harness bloqueou como automodificação. As mudanças
  ficaram como proposta (seção 7), para eu decidir.

## 3. Uma decisão que merece (ou não) um ADR
**sharp virou ADR (ADR-001 §12).** Biblioteca fora do ADR é proibida por
rules/restrictions.md; a única forma de permitir é mudar o ADR, e a
decisão (ferramenta de máquina, fora do bundle) vale para qualquer
script futuro. Alternativas no próprio ADR: Pillow, tamanho pela URL do
provedor.

**Tenant no `app_metadata` não virou ADR, e deveria quando houver o
segundo tenant.** Ela interpreta o ADR-001 §6 ("claim do JWT") de um jeito
que limita uma conta a um tenant, o oposto do que o §6 prevê para
memberships. Hoje, com um tenant só, a spec 004 (D-01) registra a
limitação; o ADR fica para quando a escolha passar a custar algo.

## 4. A skill imagens-equipamentos
- **SKILL.md:** o contrato e quem garante cada item, o procedimento e o
  que é proibido. **references/:** o que a API do Pixabay permite e exige
  (lido, não decorado) e o roteiro da revisão visual. **Script (back/scripts/imagens/):**
  tudo que precisa sair igual toda vez: busca, filtro, 800×600, sha256,
  créditos, envio. Critério: o que é repetível vira código; o que é
  consulta vira referência; o que é julgamento vira revisão humana com
  registro.
- **Garantido por código e teste:** fonte única (`origemPermitida`,
  CA-11), 800×600 (sharp + CA-08), sem repetição (CA-09), créditos
  completos (CA-10), envio só com aprovação (CA-12), upsert (repetir não
  duplica). **Depende de revisão:** direito de imagem. O filtro de tags é
  pré-filtro: pessoa, marca e tela com conteúdo só se veem na foto.
- **Como o agente fica impedido de enviar:** `imagens:enviar` recusa sem
  terminal interativo (CA-13) e pede o ref do projeto digitado. Limite
  honesto: a chave secreta esteve no `back/.env` durante a prática, porque
  o seed precisou dela; com a chave no disco, um agente poderia escrever
  outro código para enviar. A garantia completa é a chave fora do disco
  (seção 8).
- **Fotos reprovadas: 85.** 28 em tamanho final e 57 em folha de contato
  de pré-triagem. Motivos: marca legível (Apple, Lenovo, Olympus, Canon,
  Fujifilm, Pentax, Rollei, iiyama…), pessoa, tela com conteúdo, texto e
  foto que não mostra o equipamento (caderno de papel no lugar de
  notebook, tablet no lugar de monitor). Cada uma, com o motivo, está em
  `back/prisma/dados/imagens/revisao.json`. A revisão foi do agente por
  delegação, e eu confiro as 12 aprovadas.

## 5. O subagente verificador
- **Gatilho: regra, não lembrança.** AGENTS.md (passo 4) manda delegar
  antes de declarar a spec pronta, e rules/checks.md, carregada em toda
  sessão, diz que a spec só está pronta com a última rodada salva em
  `verificacoes/` com "Veredito: PRONTA" (608cc80). Gatilho pela
  `description` é probabilístico, e pedido explícito depende de alguém
  lembrar.
- **O que ele lê:** AGENTS.md, restrictions.md, checks.md e spec, plano e
  tarefas da spec. Ele não herda a conversa, e precisa da régua do
  projeto (checks.md), não de um resumo meu.
- **O que ele não consegue fazer:** o arquivo proíbe editar, commitar,
  trocar de branch, rodar migration, seed, envio de fotos e deploy, e ler
  `.env`. No harness, ele é disparado como general-purpose lendo
  vibing/agents/, porque eu não quis nada na raiz de emprest.ai/. Por
  isso as proibições valem por instrução, não por permissão do harness.
  É a maior fraqueza desta entrega (seção 8).
- **Back e front:** a entrada é a spec mais, por repositório afetado,
  branch e intervalo de commits. Ele confere a branch, roda os comandos
  de cada repositório e responde uma tabela com coluna de repositório. As
  rodadas são comparáveis porque o formato é fixo (uma linha por CA, na
  ordem da spec) e cada uma fica em um arquivo.

## 6. O verificador em ação
Tabelas finais: specs/00X/verificacoes/ (a última rodada de cada spec).

- **004, rodada 1 (NÃO PRONTA).** CA-02: um token RS256 de chave RSA com
  o mesmo `kid` voltava 500. Causa: exceção do `getClaims` fora do tipo
  que o guard converte em 401. Correção: back b3733ce, front bfe7458.
  Arquivo: specs/004-login/verificacoes/2026-11-11-rodada-1.md.
- **004, rodada 2 (PRONTA).** specs/004-login/verificacoes/2026-11-11-rodada-2.md.
- **005, rodada 1 (NÃO PRONTA).** CA-06: "confirmada" era constante da imitação e o e-mail vinha da constante do código; CA-05: variável vazia derrubava o seed. Correção back 4f61505. **Rodada 2: PRONTA** (005/verificacoes/).
- **006, rodada 1 (NÃO PRONTA).** CA-10: 404 com type diferente de `about:blank` (a spec vale; código mudou); CA-01 sem conferir categoria; CA-04 sem olhar a audit_log. Correção back 241a7ce.
- **Prova com erro plantado.** back 7bcb40a: limite padrão 3 → 5 (o número do briefing). Rodada 2 (006/verificacoes/2026-11-11-rodada-2.md): NÃO PRONTA, CA-06, CA-08 e CA-11 REPROVADOS pela causa certa (regras.ts:4), mais 3 testes vermelhos. Erro desfeito em back 9ec9d0e (git revert). Rodada 3: **PRONTA** (rodada-3.md).

## 7. Uma regra minha que esta prática pôs à prova
**rules/checks.md, "teste nunca toca produção".** Os testes unitários do
back só passavam porque o Prisma Client carrega o `back/.env`, que aponta
para produção. Ninguém viu, porque a máquina tinha o `.env`. Apareceu
quando o agente rodou os testes num worktree sem `.env`. Agora os
testes carregam um ambiente de teste antes de tudo (back b3733ce), e o
teste do seed da spec 003 deixou de usar o CLI, que carregaria o `.env` e
criaria contas de verdade (back 4eafb0e).

Propostas que ficaram para mim (o harness bloqueou a edição pelo agente):
- rules/secrets.md, item 3: exceção para variável só de script de
  máquina (dois lugares, não três).
- rules/restrictions.md: dado que não cabe em migration (contas do Auth,
  arquivos do Storage) só por seed ou script versionado, idempotente e
  aprovado em spec, com saída colada.
- rules/migration.md, passo 2, e AGENTS.md, "Comandos": registrar o
  `migrate diff` entre dois schemas, `migrate status/deploy`, `db seed` e
  os comandos da skill, todos rodados em 2026-11-11.

## 8. O que ficou de fora e o que faria diferente
- **Verificador sem trava técnica:** proibições só por instrução. Eu faria
  diferente: o arquivo em `vibing/.claude/agents/`, com `tools` e hook
  `PreToolUse` no frontmatter, e a sessão aberta com `--add-dir vibing`.
- **Chave secreta no disco durante a prática:** rotacionar e tirá-la do
  `back/.env`; o seed e o envio leem do terminal de quem roda.
- **Revisão das fotos feita pelo agente:** eu confiro antes da tag.
- **Rate limit, helmet, pino, Sentry e CI** (ADR-001 §9 e §10): fora,
  sem spec. O `POST /v1/emprestimos` não tem limite por pessoa além das
  regras de negócio.
- **Testes da landing (spec 001, CA-02 a CA-08)** citam o CA só em
  comentário, o que rules/checks.md não aceita.
- **Região do Supabase** (us-west-2 no projeto, sa-east-1 no ADR-001 §9):
  divergência aberta, que é minha.
- **Estado de produção depois do roteiro:** sem devolução, os itens
  pedidos pela conta de teste ficam emprestados (006 L-08).

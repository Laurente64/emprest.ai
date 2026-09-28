---
description: Como salvar e retomar o handoff de uma sessão
globs: ["vibing/handoffs/**"]
alwaysApply: false
---

# Handoff de sessão
> leitor: agente

O handoff guarda o que uma sessão precisa passar para a próxima. Quem
retoma não viu a conversa: só tem o arquivo e os repositórios.

## Quando
- Salvar: só quando eu pedir ("salve o handoff", "salva a sessão" ou
  parecido). Não salve por conta própria.
- Retomar: quando eu pedir para retomar ("retome o último handoff",
  "retome o handoff de <assunto>").

## Onde
vibing/handoffs/AAAA-MM-DD-nome-curto.md

- Data: o dia em que o handoff foi salvo.
- nome-curto: 2 a 4 palavras em kebab-case, sem acento, que digam do
  que a sessão tratou. Ex.: 2026-09-28-regra-handoff.md,
  2026-09-30-spec-003-cadastro.md.
- Se o arquivo já existir (mesma data e mesmo assunto), atualize-o em
  vez de criar outro. Assunto diferente no mesmo dia vira outro arquivo.
- A pasta está no .gitignore do vibing/: handoff é contexto, como o
  layout.md, e não entra em commit.

## Salvar
1. Escreva o arquivo com o modelo abaixo. Preencha cada seção. Se uma
   seção não tiver conteúdo, escreva "nenhum", não a apague.
2. Rode `git status --short` e `git branch --show-current` em cada
   repositório tocado na sessão e cole a saída em "Estado dos
   repositórios". Não escreva de memória.
3. Mostre o caminho do arquivo e o "Próximo passo" na resposta.

O que entra:
- Só o que aconteceu na sessão. O que não foi conferido fica marcado
  como "não conferido".
- Aprovação só entra se eu a dei, com o que foi aprovado. Pedido meu
  não é aprovação; rascunho não é aprovação.
- Aponte arquivos por caminho em vez de copiar o conteúdo deles. O que
  já está na spec, no plano, no ADR ou no código não se repete aqui.
- Nada de segredo: sem valor de variável de ambiente, token, senha ou
  connection string (ver rules/secrets.md). Nome da variável pode.

## Retomar
1. Sem assunto no pedido, leia o handoff mais recente de
   vibing/handoffs/ (a data do nome manda; empate, o "Salvo em" do
   cabeçalho). Com assunto, o mais recente cujo nome bate. Se houver
   mais de um candidato, liste e pergunte.
2. Leia os arquivos de "Ler antes de continuar".
3. Rode `git status --short` e `git branch --show-current` nos
   repositórios listados e compare com "Estado dos repositórios". Se
   mudou, diga o que mudou.
4. Resuma em poucas linhas: onde paramos, o que espera aprovação e qual
   é o próximo passo. PARE e espere eu confirmar antes de agir.

O handoff não autoriza nada. Uma aprovação registrada nele vale para o
artefato aprovado, e só. A próxima tarefa ainda precisa do meu "pode
implementar" (rules/restrictions.md).

## Modelo

```markdown
# Handoff — <assunto em poucas palavras>

- Salvo em: AAAA-MM-DD HH:MM
- Spec em andamento: <NNN-funcionalidade, ou "nenhuma">
- Etapa: <spec | plano e tarefas | tarefa N de M | fora de feature>

## Objetivo da sessão
<O que eu pedi, em uma ou duas frases.>

## O que foi feito
- <ação> — <arquivo(s) tocado(s)>

## Decisões
- <decisão> — <quem decidiu e por quê>

## Esperando minha aprovação
- <artefato ou pergunta> — <onde está>

## Perguntas em aberto
- <pergunta que ninguém respondeu ainda>

## Checks
- <comando> → <última linha da saída>, ou "não rodado"

## Estado dos repositórios
- vibing/ — branch <x>
  <saída do git status --short>
- back/ — branch <x>
  <saída do git status --short>
- front/ — branch <x>
  <saída do git status --short>

## Próximo passo
<A ação exata que a próxima sessão deve fazer primeiro, e se precisa
de aprovação antes.>

## Ler antes de continuar
- <caminho> — <por quê>
```

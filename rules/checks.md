---
description: O que precisa passar antes de declarar uma tarefa pronta
globs: []
alwaysApply: true
---

# Verificação de fim de tarefa
> leitor: agente

## Quando
Sempre que você for dizer "pronto", "implementado" ou "funcionando".

## Procedimento
1. Rode os testes do repositório que você alterou (back/ ou front/),
   com os comandos da tabela Comandos do AGENTS.md. Cole a última linha
   da saída na resposta. Teste de banco usa Testcontainers (ADR-001 §8),
   nunca o Supabase. Se o comando que a tarefa precisa não estiver na
   tabela, diga que a tarefa NÃO foi testada.
2. Se a tarefa tocou em migration, confirme que o que foi aplicado em
   produção é exatamente o SQL que eu aprovei (vibing/rules/migration.md)
   e cole a saída do comando que aplicou.
3. Rode o build do repositório alterado antes de qualquer push, com o
   comando da mesma tabela. Build que quebra na Vercel é o feedback mais
   lento e mais caro deste projeto.
4. Rode `git status --short` em cada repositório que você tocou
   (vibing/, back/ e front/ são repositórios separados). Só podem
   aparecer arquivos do escopo da tarefa.
5. Diga qual critério de aceitação (CA-xx) da spec esta tarefa atende.
   O teste que prova um CA cita o ID no nome, por exemplo
   `it('CA-01 responde 200')`. Comentário não conta.

## Verificação
Pronto = testes e build terminaram sem falha, a migration aplicada (se
houve) é a aprovada, E o git status não trouxe surpresa. Tudo isso, não
parte. Tarefa sem teste ou sem build rodado não está pronta: está "não
testada".

## Não faça
- Não relate sucesso parcial. Teste vermelho é tarefa não terminada,
  mesmo que o código "esteja certo".
- Não tente consertar a mesma falha duas vezes seguidas sem me mostrar
  a saída do erro.
- Não rode teste contra o Supabase: ele é produção.

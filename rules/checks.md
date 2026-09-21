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
1. Rode os testes do repositório que você alterou (back/ ou front/).
   Cole a última linha da saída na resposta. Teste de banco usa
   Testcontainers (ADR-001 §8), nunca o Supabase. Ainda não há comando
   de teste verificado (ver Comandos no AGENTS.md): enquanto não houver,
   diga que a tarefa NÃO foi testada.
2. Se a tarefa tocou em migration, confirme que o que foi aplicado em
   produção é exatamente o SQL que eu aprovei (vibing/rules/migration.md)
   e cole a saída do comando que aplicou.
3. Rode o build do repositório alterado antes de qualquer push. Build
   que quebra na Vercel é o feedback mais lento e mais caro deste
   projeto. Ainda não há comando de build verificado: enquanto não
   houver, diga que o build não foi rodado.
4. Rode `git status --short` em cada repositório que você tocou
   (vibing/, back/ e front/ são repositórios separados). Só podem
   aparecer arquivos do escopo da tarefa.
5. Diga qual critério de aceitação (CA-xx) da spec esta tarefa atende.

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

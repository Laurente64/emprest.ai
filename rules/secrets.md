---
description: Onde cada variável de ambiente vive e qual chave usar
globs: ["**/.env*", "back/src/**", "front/src/**"]
alwaysApply: false
---

# Variáveis de ambiente e chaves
> leitor: agente

## Quando
Ao criar ou usar qualquer variável de ambiente, ao instanciar o client
do banco, ou ao escrever código que lê configuração.

## Procedimento
1. Chave pública (anon): só ela pode aparecer em código que roda no
   navegador (front/). A proteção dela são as políticas de acesso do
   banco.
2. Chave de serviço (service_role): ignora todas as políticas. Só em
   back/, nunca em front/, nunca em variável com prefixo público
   (ADR-001 §10).
3. Variável nova existe em TRÊS lugares ou em nenhum, sempre no
   repositório que a usa:
   `.env` / `.env.local` (sua máquina, fora do git) · `.env.example`
   (versionado, sem valor) · painel do projeto Vercel desse repositório
   (preview e production).
4. O painel da Vercel é sempre meu (restrictions.md). Ao criar uma
   variável, diga na resposta em quais lugares você já a colocou e o
   que falta eu fazer à mão.

## Verificação
Uma busca por "service" em front/ não retorna nada. O `.env.example`
de cada repositório tem todas as chaves que ele usa, sem nenhum valor
real.

## Não faça
- Não escreva valor de chave em resposta, commit, log ou comentário.
- Não crie variável só no `.env` / `.env.local`. Ela quebra o deploy e
  o erro só aparece no build da Vercel.
- Não contorne uma política de acesso trocando de chave. Se a política
  atrapalha, a política está errada — pare e me avise.

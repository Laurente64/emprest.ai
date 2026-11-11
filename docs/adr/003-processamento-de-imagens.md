# ADR-003 — Processamento de imagens nos scripts de dados

Status: ACEITO em 2026-11-10.
Origem: decisão do usuário em 2026-11-10; texto redigido pelo agente,
por exceção pedida pelo usuário à regra "Não escreva ADR"
(rules/restrictions.md). O briefing de dados iniciais da prática pedia
"instale a biblioteca sharp", e o contrato da skill imagens-equipamentos
exige todas as imagens em 800×600, garantido.

Este ADR acrescenta uma decisão que o ADR-001 não tinha. Ele não muda
nem supera nenhuma decisão do ADR-001: por isso é um ADR novo, e o
ADR-001 fica como estava. Em 2026-11-11 esta decisão chegou a ser
escrita como §12 dentro do ADR-001; foi desfeito, porque ADR aceito não
se edita.

## Decisão

| Item | Escolha |
|---|---|
| Processamento de imagem | `sharp`, dependência de desenvolvimento do back/, versão fixa |
| Onde roda | Só nos scripts de back/scripts/imagens/, na máquina de quem prepara as fotos |

- **Processamento de imagem** — "todas em 800×600" precisa ser garantido por código e conferido por teste; pedir o tamanho à API de imagens não garante, e redimensionar à mão não se repete igual.
- **Onde roda** — fora do bundle da API: a function da Vercel não carrega binário nativo que ninguém usa em runtime.

## Alternativas descartadas

| Alternativa | Motivo do descarte |
|---|---|
| Redimensionar com Python (Pillow) ou ferramenta do sistema | Linguagem e instalação fora da stack; o script deixaria de rodar em quem só tem Node |
| Pedir o tamanho pela URL da API de imagens | Depende do provedor (o Pixabay não corta em 800×600) e não é conferível por teste |

## Consequências

- O back/ ganha uma dependência com binário nativo, só de desenvolvimento.
- Biblioteca nova em script continua exigindo ADR (rules/restrictions.md).

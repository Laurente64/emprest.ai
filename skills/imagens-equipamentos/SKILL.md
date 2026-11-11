---
name: imagens-equipamentos
description: Obtém, revisa, padroniza em 800×600 e publica as fotos dos equipamentos do EMPREST.AI, com o crédito de cada uma registrado. Use ao preparar, trocar ou conferir foto de equipamento, ou ao mexer em back/scripts/imagens/ ou back/prisma/dados/imagens/. Não use para imagens da landing, logo ou ilustrações.
---

# Imagens de equipamentos
> leitor: agente

Toda foto de equipamento do sistema sai desta skill. Foto de equipamento
que tenha outra origem é substituída pela saída dela.

O código está em back/scripts/imagens/ (precisa do `sharp` e do `.env`
do back/, por isso não mora aqui). A saída fica versionada em
back/prisma/dados/imagens/: os arquivos `.jpg`, o `creditos.json` e o
`revisao.json`. A lista de equipamentos é back/prisma/dados/equipamentos.json.

## O contrato e quem garante cada item

| requisito | garantido por | como |
|---|---|---|
| Fonte única | código e teste | O script só chama `https://pixabay.com/api/` e só baixa de `https://pixabay.com/get/…` (o `largeImageURL`). Qualquer outro host é recusado (`origemPermitida`). |
| Direito de imagem | código e revisão humana | O código filtra (`safesearch=true`, `image_type=photo`, tags bloqueadas). Filtro de tag não enxerga a foto: a aprovação é humana, registrada no `revisao.json` (references/revisao-visual.md). O envio recusa foto sem aprovação. |
| 800×600 | código e teste | O `sharp` corta e redimensiona para 800×600 (`fit: cover`). Um teste lê cada arquivo commitado e falha se algum não tiver exatamente 800×600. |
| Sem repetição | código e teste | A escolha pula foto já usada por outro equipamento. Um teste falha se dois equipamentos tiverem o mesmo id do Pixabay ou o mesmo arquivo (sha256). |
| Créditos | código e teste | O `creditos.json` guarda, por equipamento: id e página no Pixabay, autor, licença com link, busca usada e sha256. Um teste confere os 12, completos. |
| Armazenamento | código | Bucket público `equipamentos` no Supabase Storage. Caminho fixo `<patrimonio>.jpg`, envio com upsert: repetir substitui, não duplica. O seed grava o caminho em `equipments.image_path`. |
| Produção | código | O envio exige terminal interativo e o ref do projeto digitado. O shell do agente não é interativo. Ver "Limite conhecido". |
| Segredos e dependências | regra e revisão | `PIXABAY_API_KEY` e `SUPABASE_SERVICE_ROLE_KEY` só no `back/.env` (fora do git). `sharp` é dependência de desenvolvimento, aprovada no ADR-001 §12, e não entra no bundle da API. |
| Crédito na tela | teste do front | O catálogo mostra "Fotos: Pixabay" com link para https://pixabay.com/ (spec 006). A API pede que o usuário veja de onde vêm as imagens. |

## Procedimento

1. Em back/, rode `npm run imagens:preparar`. Pode ser o agente. O
   script, para cada equipamento sem foto aprovada:
   busca no Pixabay com o termo `busca` da lista, descarta o que o
   filtro reprova e o que está em `revisao.json` como reprovado ou já
   usado, baixa a primeira candidata, gera o 800×600 e atualiza o
   `creditos.json`. Equipamento com foto aprovada não é buscado de
   novo: a API pede cache de 24 h e proíbe download em massa.
2. Revisão visual: abra cada `.jpg` novo e siga
   references/revisao-visual.md. Registre o resultado no
   `revisao.json`: aprovada, ou reprovada com o motivo.
3. Houve reprovação: volte ao passo 1. O script pega a próxima
   candidata daquele equipamento.
4. Rode `npm test` em back/. Os testes das imagens (tamanho, repetição,
   créditos e aprovação) precisam passar.
5. Commite os `.jpg`, o `creditos.json` e o `revisao.json` na branch da
   tarefa.
6. Envio para produção: uma pessoa, no próprio terminal, roda
   `npm run imagens:enviar` em back/. O script recusa se alguma foto não
   estiver aprovada, se o tamanho não bater ou se o terminal não for
   interativo, cria o bucket se faltar, envia com upsert e confere a URL
   pública de cada foto.
7. O seed (`prisma db seed`, spec 005) grava o caminho de cada foto. A
   API monta a URL pública a partir de `SUPABASE_URL`.

## Limite conhecido
A trava do envio impede o agente de rodar `imagens:enviar`. Ela não
impede um agente de escrever outro código que use a chave secreta, se a
chave estiver no `back/.env`. A garantia completa é a chave fora do
disco: quem envia define `SUPABASE_SERVICE_ROLE_KEY` só no próprio
terminal, na hora do envio.

## Não faça
- Não use outra origem de imagem: Pexels, Unsplash, busca na web, banco
  de imagens do próprio Supabase, imagem gerada por IA.
- Não aponte a tela para URL do Pixabay. A API proíbe hotlink permanente:
  a foto é baixada e servida pelo nosso Storage.
- Não aprove foto com pessoa (mão inclusive), marca ou logo legível,
  texto legível ou tela ligada com conteúdo.
- Não redimensione à mão nem com outra ferramenta. O tamanho sai do
  script, e o teste confere.
- Não troque a foto de um equipamento sem o `creditos.json` e o
  `revisao.json` acompanharem.
- Agente: não rode `imagens:enviar`, não contorne a trava e não escreva
  outro jeito de enviar.

Referências: references/pixabay-api.md (o que a API permite e exige) e
references/revisao-visual.md (o que reprova uma foto).

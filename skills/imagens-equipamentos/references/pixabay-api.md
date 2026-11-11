# API do Pixabay — o que importa para esta skill
> leitor: agente

Lido em 2026-11-11 em https://pixabay.com/api/docs/ e
https://pixabay.com/service/license-summary/. A licença completa, a
única juridicamente vinculante, está em https://pixabay.com/service/terms/.

## Busca de imagens
`GET https://pixabay.com/api/?key=<PIXABAY_API_KEY>&q=<termo>`

| parâmetro | uso aqui | valores |
|---|---|---|
| `q` | o `busca` da lista, URL-encoded | até 100 caracteres |
| `image_type` | `photo` | all, photo, illustration, vector |
| `orientation` | `horizontal` | all, horizontal, vertical |
| `min_width` / `min_height` | `800` / `600` | inteiro |
| `safesearch` | `true` | true, false |
| `order` | `popular` (padrão) | popular, latest |
| `per_page` | `30` | 3 a 200 |
| `lang` | `en` (os termos da lista estão em inglês) | — |

Campos usados de cada resultado (`hits[]`): `id`, `pageURL`, `tags`
(texto separado por vírgula), `largeImageURL` (até 1280 px), `imageWidth`,
`imageHeight`, `user`, `user_id`. Sem acesso completo à API, o maior
arquivo disponível é o `largeImageURL`. O `webformatURL` vale só 24 h.

## Regras da API
- Limite: "By default, you can make up to 100 requests per 60 seconds."
  Passou disso, HTTP 429. Cabeçalhos `X-RateLimit-Limit`,
  `X-RateLimit-Remaining`, `X-RateLimit-Reset`.
- Cache: "requests must be cached for 24 hours." E: "do not send lots of
  automated queries." / "Systematic mass downloads are not allowed."
- Hotlink: "permanent hotlinking of images (using Pixabay URLs in your
  app) is not allowed." / "If you intend to use the images, please
  download them to your server first."
- Atribuição: "If you make use of the API, show your users where the
  images and videos are from, whenever search results are displayed."
  É o motivo do "Fotos: Pixabay" no catálogo.

## Licença (Pixabay Content License, resumo)
- Pode: usar de graça, sem atribuir o autor, modificar e adaptar (o
  corte para 800×600 é adaptação).
- Não pode: vender ou distribuir o conteúdo sem alteração; usar conteúdo
  com marca, logo ou marca registrada reconhecível para fins comerciais;
  usar "in any immoral or illegal way, especially Content which features
  recognisable people"; usar de forma enganosa.
- Outros direitos (marca, direito de imagem, privacidade) podem valer
  além da licença. Por isso a revisão visual reprova pessoa e marca
  mesmo quando o Pixabay aceita a foto.

O `creditos.json` registra a licença como "Pixabay Content License",
com o link https://pixabay.com/service/license-summary/.

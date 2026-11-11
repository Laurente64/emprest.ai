# Revisão visual das fotos
> leitor: quem revisa (pessoa ou agente com delegação registrada)

O filtro do script lê as tags que o autor escreveu, não a foto. Esta
revisão é o que garante o direito de imagem. Abra cada `.jpg` em
back/prisma/dados/imagens/ no tamanho final (800×600) e reprove se
houver qualquer um destes:

| reprova | exemplos |
|---|---|
| Pessoa | rosto, corpo, mão, braço, reflexo de pessoa na tela |
| Marca ou logo legível | maçã da Apple, "DELL", "HP", logo do Windows, marca no cabo ou na câmera |
| Texto legível | teclado com letras nítidas em close não reprova; papel, post-it, embalagem ou adesivo com texto legível reprova |
| Tela com conteúdo | site, código, planilha, foto ou papel de parede figurativo na tela. Tela desligada, preta ou de uma cor só passa |
| Não é o equipamento | a foto não mostra o que o nome diz (um cabo USB no lugar de um HDMI, um tablet no lugar de notebook) |

Na dúvida, reprove: há outras candidatas.

## Como registrar
No `back/prisma/dados/imagens/revisao.json`:

```json
{
  "aprovadas": {
    "TI-0001": { "pixabayId": 123, "por": "<quem>", "em": "AAAA-MM-DD" }
  },
  "reprovadas": [
    { "patrimonio": "TI-0001", "pixabayId": 456, "motivo": "logo legível na tampa", "por": "<quem>", "em": "AAAA-MM-DD" }
  ]
}
```

- `por` diz quem revisou. Se for um agente, escreva isso; a revisão de
  agente vale até uma pessoa conferir.
- Reprovada não volta: o script nunca escolhe de novo um `pixabayId`
  que esteja em `reprovadas`.
- O id aprovado precisa ser o mesmo do `creditos.json`. Se o script
  trocar a foto, a aprovação antiga deixa de valer e o envio recusa.

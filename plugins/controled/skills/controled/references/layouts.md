# Layouts

Um layout é uma tela desenhada que o player monta ao vivo: textos, imagens, notícias de um feed
RSS, relógio, clima, cotações e áreas que tocam mídias. As notícias giram sozinhas: o layout
mostra `item_count` notícias, cada uma por `seconds_per_item` segundos.

## Passo a passo

1. **Fonte das notícias** (só se o layout tiver notícias). `GET /fontes-conteudo` lista os feeds
   já cadastrados. Para um feed novo, `POST /fontes-conteudo` com `{"url": "<URL do RSS>"}`:
   devolve 201 quando cria e 200 quando a organização já tinha a mesma URL. Use a URL do RSS
   oficial do veículo. Se não souber a URL, procure (busca na web, se houver) ou pergunte ao
   usuário; não invente.
2. **Tamanho do canvas**. Use a resolução da tela onde o layout vai rodar (`GET /telas/{id}`,
   campos `width` e `height`): 1920×1080 para painel horizontal, 1080×1920 para totem vertical.
   Se for para várias telas com resoluções diferentes, pergunte ou faça um layout por formato.
3. **Criar**: `POST /layouts` com `{"name", "canvas_width", "canvas_height"}` (32 a 7680 px).
   Nasce vazio e devolve o `id`.
4. **Salvar o documento**: `PUT /layouts/{id}` com o documento inteiro (modelo abaixo). O PUT
   substitui tudo; para editar um layout existente, leia antes com `GET /layouts/{id}` e mande o
   documento completo de volta com a mudança.
5. **Conferir**: `GET /layouts/{id}`.
6. **Pôr na tela**: a API não faz este passo. O usuário aplica o layout no painel do Controled
   (menu Layouts), como peça da programação ou como moldura da tela. Diga isso ao entregar.

Outras ações: `PATCH /layouts/{id}` renomeia, `POST /layouts/{id}/duplicar` copia,
`DELETE /layouts/{id}` exclui (409 `layout_in_use` enquanto alguma programação o usa).

## Documento (`PUT /layouts/{id}`)

| Campo | Regra |
|---|---|
| `name` | 1 a 120 caracteres |
| `item_count` | 1 a 6 notícias por volta |
| `seconds_per_item` | 3 a 50 segundos por notícia |
| `content_source_id` | uuid da fonte RSS, ou `null` sem notícias |
| `background` | cor `#RRGGBB` ou `#RRGGBBAA` |
| `elements` | até 20 elementos |

Todo elemento tem `id` (único no layout, até 64 caracteres), `type` e a caixa `x`, `y`, `w`, `h`
em **% do canvas** (0 a 100, origem no canto superior esquerdo). Texto mede a fonte em
`fontSizePct`, % da altura do canvas (1 a 30): num canvas de 1080 px, 4 dá cerca de 43 px.

Estilo de texto (`style`, obrigatório nos elementos de texto): `color` (padrão `#ffffff`),
`fontSizePct` (padrão 4), `align` (`left`, `center`, `right`), `maxLines` (0 a 5; 0 = sem
limite), `background` (cor ou `null`), `fontFamily` (`geist`, `inter`, `roboto`, `montserrat`,
`poppins`, `oswald`, `bebas`, `playfair`, `lora`, `source-serif`, `jetbrains-mono`,
`roboto-mono`), `bold`.

Animação de entrada opcional (`enter`): `{"kind": "fade" | "slide_left" | "slide_right" |
"slide_up" | "slide_down", "duration": 0.2-2, "delay": 0-3}`. No máximo 8 elementos animados.

### Tipos de elemento

| `type` | Campos próprios | Uso |
|---|---|---|
| `rect` | `fill` (cor) | faixas, fundos de bloco |
| `text` | `text` (até 500), `style` | texto fixo |
| `image` | `midiaId`, `fit` (`cover`/`contain`) | logo ou foto da biblioteca; só imagem, vídeo é recusado |
| `feed_title` | `itemOffset`, `style`, `field` | título da notícia |
| `feed_body` | `itemOffset`, `style`, `field` | resumo da notícia |
| `feed_image` | `itemOffset`, `imageIndex` (0-3), `fit`, `pan`, `fallbackMidiaId`, `field` | foto da notícia |
| `feed_date` | `itemOffset`, `style`, `format` (`date`/`datetime`) | data da notícia |
| `feed_qr` | `itemOffset` | QR code do link da notícia |
| `clock` | `style`, `format` (`time`, `date`, `datetime`, `weekday`) | relógio no fuso da tela |
| `weather` | `style`, `field` (obrigatório: `temperature`, `condition`, `icon`, `min_max`, `city`), `city`, `state` | clima; sem `city`/`state` usa a cidade da tela; `state` é a UF com 2 letras |
| `market` | `style`, `symbols` (1 a 6 de `IBOV`, `IFIX`, `PETR4`, `VALE3`, `ITUB4`, `AMBEV`) | cotações |
| `area` | `role` (`ads`, `playlist_1`, `playlist_2`), `items` | região que toca mídias |

`itemOffset` (0 a 5) escolhe qual notícia da volta o elemento mostra: 0 é a notícia da vez, 1 a
seguinte. Título, foto e data da mesma notícia usam o mesmo `itemOffset`.

Regras que o servidor recusa com 400 ou 422:
- `pan` (foto em movimento contínuo) em no máximo um `feed_image`, com `fit: "cover"` e sem
  `enter`.
- Áreas não se sobrepõem, cada `role` aparece uma vez, não têm `enter`. `items`
  (`[{"midiaId", "durationSeconds": 1-300}]`, até 50) só em `playlist_1` e `playlist_2`; a área
  `ads` toca os anúncios contratados da tela.
- Ids repetidos.

## Exemplo: notícias com faixa de clima embaixo (1920×1080)

```json
{
  "name": "Notícias com clima",
  "item_count": 5,
  "seconds_per_item": 10,
  "content_source_id": "<uuid da fonte>",
  "background": "#0b1220",
  "elements": [
    {"id": "foto", "type": "feed_image", "x": 0, "y": 0, "w": 55, "h": 82, "itemOffset": 0, "fit": "cover"},
    {"id": "titulo", "type": "feed_title", "x": 58, "y": 8, "w": 39, "h": 40, "itemOffset": 0,
     "style": {"fontSizePct": 6, "bold": true, "maxLines": 4, "fontFamily": "montserrat"},
     "enter": {"kind": "fade", "duration": 0.6}},
    {"id": "resumo", "type": "feed_body", "x": 58, "y": 50, "w": 39, "h": 28, "itemOffset": 0,
     "style": {"fontSizePct": 3.2, "maxLines": 4, "color": "#cbd5e1"}},
    {"id": "faixa", "type": "rect", "x": 0, "y": 82, "w": 100, "h": 18, "fill": "#111827"},
    {"id": "clima-icone", "type": "weather", "field": "icon", "x": 3, "y": 85, "w": 7, "h": 12,
     "style": {}},
    {"id": "clima-temp", "type": "weather", "field": "temperature", "x": 11, "y": 85, "w": 12, "h": 12,
     "style": {"fontSizePct": 7, "bold": true}},
    {"id": "clima-cond", "type": "weather", "field": "condition", "x": 24, "y": 86, "w": 30, "h": 10,
     "style": {"fontSizePct": 3.5, "color": "#cbd5e1"}},
    {"id": "hora", "type": "clock", "format": "time", "x": 80, "y": 85, "w": 17, "h": 12,
     "style": {"fontSizePct": 7, "bold": true, "align": "right"}}
  ]
}
```

Para totem vertical (1080×1920), empilhe: foto em cima (`h` ~45), título e resumo no meio,
faixa de clima e hora embaixo. Ajuste `fontSizePct` pensando em leitura a distância: título
entre 4 e 7, corpo entre 2,5 e 4.

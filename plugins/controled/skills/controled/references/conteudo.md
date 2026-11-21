# Conteúdo: anunciantes, contratos, campanhas, institucional, mídias e vídeo

## Como o conteúdo chega à tela

Cada tela tem `slots` (posições no loop) e `tempo_slot` (segundos de cada posição). Um
**contrato** reserva slots de uma tela para um **anunciante** num período. A **campanha** do
anunciante diz quais **mídias** dele tocam, em quais telas, datas, dias e horários. Slots sem
anunciante recebem a **programação institucional** da empresa. `GET /telas/{id}/slots` mostra a
sequência montada (anunciantes e inserções institucionais) da tela.

## Anunciantes e contratos

- `GET /clientes?q=<nome>` acha o anunciante; `GET /clientes/{id}` traz os contratos.
- `GET /contratos` filtra por `cliente_id` ou `tela_ref` (tela_code ou uuid): é o caminho para
  "em quais telas a farmácia está".
- Novo anunciante: `POST /clientes`. Contrato: `POST /contratos` com `cliente_id`, `tela_id`,
  `slots_contratados`, `contract_start_date` e, opcionais, `contract_end_date`,
  `exhibition_limit` e `contract_number` (sem ele, o banco gera; `GET /contratos/proximo-numero`
  mostra o próximo). Vários contratos do mesmo anunciante: `{"cliente_id", "contratos": [...]}`,
  até 100.
- `PATCH /contratos/{id}` muda slots, datas e limite de exibições; o número é imutável.
- **Tirar o anunciante de uma tela** (cancelamento): `POST /contratos/{id}/desvincular`. Sem
  `confirmar` devolve a prévia: as campanhas e mídias que deixam de existir. Mostre a prévia,
  peça o ok e chame de novo com `params: {"id": ..., "confirmar": "true"}`. Se a prévia apaga
  campanhas ou mídias, a conexão precisa também de `campanhas:delete` ou `midias:delete`.
- Alternativas mais leves, quando o cliente só quer parar por um tempo: pausar a campanha
  (`POST /campanhas/{id}/pausar`) ou tirar aquela tela das faixas da campanha (`PUT
  /campanhas/{id}`). Se não ficar claro se é cancelamento ou pausa, pergunte.
- `PUT /telas/{id}/slots` só reordena o loop: uma entrada por slot contratado, cada anunciante
  repetido exatamente o número de slots dele (422 se o total não bater).

## Campanhas

Campanha = programação de um anunciante. Corpo do `POST /campanhas`:

```json
{
  "cliente_id": "<uuid>",
  "name": "Verão 2027",
  "active": true,
  "blocks": [{
    "name": "Janeiro e fevereiro",
    "start_date": "2027-01-01", "end_date": "2027-02-28",
    "weekdays": [0, 1, 2, 3, 4, 5, 6],
    "all_day": true, "start_time": null, "end_time": null,
    "target_tela_ids": ["<uuid da tela>"],
    "item_midia_ids": ["<uuid da mídia>"]
  }]
}
```

- `weekdays`: 0 = domingo até 6 = sábado. Com `all_day: false`, informe `start_time` e
  `end_time` (`HH:MM`).
- As telas precisam ter contrato com o anunciante, e as mídias precisam ser dele (`GET
  /midias?cliente_id=`). Leia os dois antes de montar.
- `PUT /campanhas/{id}` substitui todas as faixas (o anunciante não muda): leia a campanha,
  altere e mande inteira.
- Estado: `ativar`, `pausar`, `duplicar`, `DELETE`. Campanha criada pelo anunciante no painel
  dele espera aprovação: `aprovar`, ou `recusar` com `rejection_reason` (o anunciante vê o texto).

## Programação institucional

Conteúdo da própria empresa (avisos, datas comemorativas, autopromoção) que entra nos slots sem
anunciante. Mesmo formato de `blocks` da campanha, sem `cliente_id`, usando mídias da biblioteca
institucional. `layout_area` (`playlist_1`/`playlist_2`) só importa quando a tela usa um layout
com áreas de playlist. Ações: `GET`, `POST`, `PUT` (inteira), `ativar`, `pausar`, `duplicar`,
`DELETE` em `/programacao-institucional`.

Para saber quais telas têm espaço sobrando: `get_network_summary` (ocupação) e `GET
/telas/{id}/slots` por tela.

## Mídias

- `GET /midias` (filtros `cliente_id`, `tipo` `video`/`image`), `GET /midias/{id}`.
- **Upload em três passos**:
  1. `POST /midias/uploads` com `cliente_id`, `content_type` (`image/jpeg`, `image/png`,
     `image/webp`, `video/mp4`, `video/quicktime`, `video/webm`) e `size` em bytes (até 500 MB).
     Devolve `upload_url`, `storage_key` e `headers`.
  2. O arquivo vai para `upload_url` por HTTP PUT, com exatamente aqueles `headers`, em até 2 h.
     Esse passo acontece fora do conector: no chat, sem acesso ao arquivo nem a um terminal, o
     caminho é o usuário subir pelo painel. Num ambiente com terminal, um `curl -X PUT` resolve.
  3. `POST /midias` com `storage_key`, `cliente_id`, `filename` e, se souber, `tela_id`,
     `duracao_segundos`, `width`, `height`. O tipo é conferido pelo conteúdo do arquivo (422
     `invalid_media_content` se não bater). Vídeo nasce `processing`: consulte `GET /midias/{id}`
     até `transcode_status` ser `ready` ou `failed`.
- **Vídeo cortando no fim**: o vídeo é mais longo que o `tempo_slot` da tela. Confira a duração
  (`duracao_segundos`) e o slot da tela, e use `POST /midias/{id}/ajuste-slot` com
  `target_seconds` igual ao slot. Cria uma cópia acelerada (o original fica), que nasce
  `processing`; depois troque a mídia na campanha pela cópia.
- `DELETE /midias/{id}` exclui.

## Criador de vídeo (arte e vídeo a partir de modelo)

É o caminho quando o usuário pede "uma arte", "um vídeo", "uma peça" para as telas. O vídeo sai
na resolução e na duração do slot da tela escolhida e cai na biblioteca de mídias.

1. `GET /videos/modelos`: modelos (`aviso`, `oferta`, `livre`, `canvas`) e os campos de cada um.
   `livre` e `canvas` trazem `document_schema` (JSON Schema do roteiro ou do canvas). `livre`
   só aparece quando está liberado para a organização.
2. `POST /videos/projetos` com `cliente_id` (uuid do anunciante, ou `"institucional"` para peça
   da empresa), `template_id`, `tela` (tela_code ou uuid; a tela precisa ter resolução e
   `tempo_slot`) e `campos` (textos; imagens e vídeos são ids de mídias do dono). Salva como
   rascunho e não gasta crédito; erro de campo volta em `details`. `PUT /videos/projetos/{id}`
   edita.
3. `GET /videos/saldo`: saldo e custo de um vídeo. Mostre ao usuário e peça o ok.
4. `POST /videos/projetos/{id}/gerar` (risco, gasta créditos, `idempotency_key` obrigatório)
   responde 202 com a geração na fila. A mesma chave devolve sempre a mesma geração; projeto que
   já tem geração na fila não cobra de novo. 402 `insufficient_credits` sem saldo.
5. `GET /videos/geracoes/{id}` até ficar pronta: traz `midia_id`. Cancelar só na fila
   (`POST /videos/geracoes/{id}/cancelar`).
6. Coloque o vídeo no ar: campanha do anunciante, ou programação institucional para peça da
   empresa.

Exemplo, "arte de natal para as telas sem anunciante": veja as telas com slots livres, crie o
projeto com `cliente_id: "institucional"`, modelo `aviso` ou `canvas`, mostre saldo e custo e
peça o ok para gerar. São duas decisões: gastar o crédito e pôr no ar. Proponha junto as telas e
o período (de dezembro até o Natal) e só crie a programação institucional, que nasce ativa,
depois do ok para a veiculação. Sem esse ok, não crie a programação: o vídeo fica na biblioteca
e você diz como pôr no ar depois.

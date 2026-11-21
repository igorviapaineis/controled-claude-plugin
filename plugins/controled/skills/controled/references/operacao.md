# Operação: telas, player, diagnósticos, exibições, relatórios, audiência e equipe

## Telas

- `list_screens` / `GET /telas` e `GET /telas/{id}`: cadastro (endereço, resolução, slots,
  tempo de slot, horário).
- `get_screen_status` / `GET /telas/{id}/status`: estado ao vivo, último sinal do player, URL do
  último screenshot e se há programação ou comandos esperando o player.
- `PATCH /telas/{id}` muda só os campos enviados: `nome`, endereço (`endereco`, `bairro`,
  `cidade`, `estado`, `cep`, `google_maps_url`), `slots`, `tempo_slot`, `width`, `height`,
  `horario_ligar` e `horario_desligar` (`HH:MM`), `horario_semana` (7 dias
  `[{"dow": 0-6, "closed", "on", "off"}]`, 0 = domingo), `blackout` (tela preta),
  `offline_alerts_enabled`, `timezone`, `pos_x`, `pos_y`, `fullscreen`, `always_on_top`.
- `PATCH /telas/lote` (risco): até 100 telas de uma vez. O corpo é `{"tela_refs": [...],
  "patch": {...}, "brilho": [...]}`: `patch` é obrigatório (`{}` quando só muda o brilho) e aceita
  só `slots`, `tempo_slot`, `width`, `height`, `horario_ligar`, `horario_desligar`, `pos_x`,
  `pos_y`, `fullscreen` e `always_on_top`; `brilho`, opcional, fica fora do `patch`. Os outros
  campos (nome, endereço, `blackout`, `horario_semana`, `timezone`) só tela por tela.
- `POST /telas` cria; `DELETE /telas/{id}` exclui tela sem contratos. Vincular o player:
  `POST /telas/pareamentos` com o código do QR que o player mostra.

## Tela parada ou com problema

1. `get_screen_status`: quando foi o último sinal e o screenshot.
2. `list_diagnostics` filtrado por `tela_code`: incidentes abertos e o código
   (`GET /diagnostico-codigos` explica cada código).
3. Várias telas caíram no mesmo horário costuma ser rede ou energia na região, não o player.
4. Comando ao player, se a tela ainda responde (confirme antes):
   `POST /telas/{id}/comandos` com `command`:
   - `SCREENSHOT_CAPTURE`: tira um print agora (o link sai em `GET /comandos/{id}`).
   - `PLAYLIST_RELOAD`: recarrega a programação.
   - `CACHE_CLEAR_AND_RESYNC`: apaga o cache e baixa as mídias de novo.
   - `WINDOW_RESTART`: reinicia a janela do player.
   - `MACHINE_REBOOT`: reinicia o computador da tela; exige também o escopo `comandos:reboot`.
   Responde 202; para tela offline o comando fica `pending`, com aviso.
   `POST /comandos` manda o mesmo comando a até 100 telas (tudo ou nada).
   `GET /comandos/{id}` mostra o estado reportado pelo player.

## Brilho

`GET /telas/{id}/brilho` e `PUT /telas/{id}/brilho` com
`{"slots": [{"start_time": "HH:MM", "brightness": 0-100}]}`: cada faixa vale do seu horário até a
próxima. O PUT substitui a agenda inteira.

## Diagnósticos

`GET /diagnosticos` (filtros `status`, `severidade`, `codigo`, `tela_code`, `desde`),
`GET /diagnosticos/{id}`. Resolver: `PATCH /diagnosticos/{id}` com `{"status": "resolvido"}`.
`POST /telas/{id}/diagnosticos` abre um incidente; é o caminho dos apps de monitoramento, raramente
de uma pessoa.

## Exibições e relatórios

- `GET /telas/{id}/exibicoes` com `desde` e `ate` (data e hora ISO, ex.
  `2026-11-20T00:00:00-03:00`): exibições recentes de uma tela, janela de até 31 dias e só dentro
  da retenção do log bruto. Para "quantas vezes o anúncio X passou ontem": ache as telas do
  anunciante (`GET /contratos?cliente_id=`) e a mídia pedida (`GET /midias?cliente_id=`), e conte
  pela mídia (`midia_id`), não pelo anunciante, que pode ter várias. Para um número oficial, use
  o relatório `by_media`. Tela offline no período pode ter exibido sem conexão e mandar os
  registros quando voltar: sem registro da mídia, diga "nenhuma exibição registrada até agora",
  não "não passou"; com registros, dê o total registrado e avise que pode estar incompleto.
- Relatório em PDF: `POST /relatorios` com `report_type` (`proof_of_play`, `by_screen`,
  `by_media`, `by_campaign`, `summary`, `audience`), `cliente_id`, `period_start` e `period_end`
  (`AAAA-MM-DD`, até 31 dias). Responde 202 `queued`; consulte `GET /relatorios/{id}` até
  `completed` ou `failed`. Pronto, pegue o link em `GET /relatorios/{id}/download` (vale 5
  minutos; vencido, peça outro). `failed` não é refeito sozinho: avise e, se o usuário quiser,
  peça um relatório novo.
- Relatório mensal automático por e-mail: `GET`/`PUT /relatorios/agendamentos/{id}`, com o uuid
  do anunciante em `params.id`. Ligar exige ao menos um e-mail em `recipient_emails` e respeita a
  cota de automações da organização (422); desligar sempre funciona. O relatório sai todo mês
  para esses e-mails: confirme os endereços que o usuário não disse antes de ligar.

## Audiência

- `GET /telas/{id}/contagens` (com `desde`): contagem de pessoas e veículos pela câmera, em
  blocos de 15 minutos. Bloco sem medição não é zero.
- `GET`/`PUT /telas/{id}/validacao-contagem`: validação manual da contagem; registrar (`PUT`)
  exige cargo 50+, ler não.
- `GET`/`POST /audiencia/fatores`: fator de ocupação por classe usado nos relatórios de
  audiência. Nova versão vale de hoje em diante (retroativa dá 422), uma por classe por dia
  (409), cargo 50+.
- `POST /telas/{id}/contagens` é o envio do app da câmera; não é uma ação para o chat.

## Equipe e acesso do anunciante

- `GET /eu`: quem está conectado, organização, cargo e escopos. Útil quando algo dá 403.
- Administradores: `GET /equipe`, `GET /equipe/cargos`, `POST /equipe` (convite),
  `PATCH /equipe/{id}` (cargo), `DELETE /equipe/{id}`, `POST /equipe/{id}/reenviar-convite`.
  Ninguém atribui cargo acima do próprio.
- Painel do anunciante: `POST`/`PATCH /clientes/{id}/acesso` (cria o acesso ou troca o e-mail),
  `GET`/`POST /clientes/{id}/membros`, `DELETE /clientes/{id}/membros/{membro}`,
  `POST /clientes/{id}/membros/{membro}/reenviar-convite`. Convites mandam e-mail: confirme o
  endereço antes.

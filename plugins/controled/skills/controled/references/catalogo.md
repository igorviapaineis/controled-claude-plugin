# Catálogo de ações do Controled

Gerado do código da API por `pnpm plugin:catalog`; não edite à mão. São 102 ações. O id da ação é `MÉTODO /caminho`, igual ao que `search_actions` devolve.
Para o schema exato de params e body, chame `search_actions` com o id exato (ex.: `search_actions({query: "PUT /layouts/{id}", limit: 1})`).

Legenda: `[escopo]`; **risco** = ação destrutiva ou privilegiada, a mesma marca `risk` do `search_actions` (confirmar antes); **Idempotency-Key** = a ação exige `idempotency_key`; **cargo N+** = pelo conector, só usuário com cargo de nível N ou mais (abaixo disso, 403 `insufficient_role`). Todo POST pede `idempotency_key` no `write_action`.

## Rede e telas

- `GET /resumo` [telas:read]: Resumo da operação e da ocupação da rede. Ocupação e contratos a vencer exigem contratos:read; anunciantes sem campanha exige clientes:read. Sem o escopo, o bloco é null.
- `GET /telas` [telas:read]: Lista telas.
- `GET /telas/{id}` [telas:read]: Detalha uma tela.
- `GET /telas/{id}/slots` [telas:read]: Sequência de anunciantes e inserções institucionais da tela.
- `PATCH /telas/lote` [telas:write] **risco**: Aplica os mesmos campos a até 100 telas. Valida todas as referências e regras antes de escrever. A escrita usa o rollback compensatório do painel; se o desfazer falhar, parte das alterações pode ter ficado aplicada.
- `POST /telas` [telas:write]: Cria uma tela. O serial de ativação só é incluído para o OAuth do app Controled; a entrega é auditada.
- `PATCH /telas/{id}` [telas:write]: Atualiza uma tela (só os campos enviados).
- `DELETE /telas/{id}` [telas:delete] **risco**: Exclui uma tela sem contratos de anunciante.
- `GET /telas/{id}/status` [telas:read]: Estado ao vivo da tela, com o último screenshot.
- `GET /telas/{id}/exibicoes` [telas:read]: Exibições recentes da tela. Janela de até 31 dias, das mais recentes, só dentro da retenção do log bruto (dias recentes). Histórico mais antigo: /relatorios.
- `PUT /telas/{id}/slots` [telas:write]: Define a sequência de anunciantes nos slots da tela.
- `GET /telas/{id}/brilho` [telas:read]: Agenda de brilho da tela.
- `PUT /telas/{id}/brilho` [telas:write]: Substitui a agenda de brilho da tela.
- `POST /telas/pareamentos` [telas:write]: Vincula um player à tela pelo QR que ele exibe.
- `GET /telas/pareamentos/{id}` [telas:write]: Estado do último pareamento da tela (`{id}` é a tela).
- `GET /telas/{id}/arquivos` [telas:write]: Lista os arquivos de configuração da tela.
- `DELETE /telas/{id}/arquivos` [telas:write] **risco**: Apaga um arquivo de configuração da tela.

## Diagnósticos

- `POST /telas/{id}/diagnosticos` [diagnosticos:write] **cargo 50+**: Abre (ou reafirma) um diagnóstico.
- `GET /diagnosticos` [diagnosticos:read]: Lista diagnósticos.
- `GET /diagnosticos/{id}` [diagnosticos:read]: Detalha um diagnóstico.
- `PATCH /diagnosticos/{id}` [diagnosticos:write] **cargo 50+**: Resolve um diagnóstico.
- `GET /diagnostico-codigos` [diagnosticos:read]: Lista o catálogo de códigos.

## Comandos ao player

- `POST /telas/{id}/comandos` [comandos:write] **risco**: Enfileira um comando para a tela. Responde 202. `MACHINE_REBOOT` exige também `comandos:reboot`. Tela offline fica `pending` com `warning`.
- `POST /comandos` [comandos:write] **risco**: Enfileira um comando para várias telas. Até 100 telas, um comando por máquina. Tudo ou nada: uma tela fora da organização recusa o lote inteiro. `MACHINE_REBOOT` exige também `comandos:reboot`.
- `GET /comandos/{id}` [comandos:read] **cargo 30+**: Estado de um comando. Como o player reportou; no screenshot, o link da imagem quando ela chega.

## Anunciantes e contratos

- `GET /clientes` [clientes:read]: Lista anunciantes.
- `GET /clientes/{id}` [clientes:read]: Detalha um anunciante.
- `POST /clientes` [clientes:write]: Cria um anunciante (contratos vão por POST /contratos).
- `PATCH /clientes/{id}` [clientes:write]: Atualiza um anunciante (só os campos enviados).
- `GET /contratos` [contratos:read]: Lista contratos (filtros `cliente_id` e `tela_ref`).
- `POST /contratos` [contratos:write]: Cria um contrato, ou vários de um anunciante (`contratos[]`).
- `GET /contratos/proximo-numero` [contratos:read]: Próximo número de contrato que o banco vai gerar.
- `PATCH /contratos/{id}` [contratos:write]: Atualiza um contrato. Slots, datas e limite de exibições. O número do contrato é imutável.
- `POST /contratos/{id}/desvincular` [contratos:write] **risco**: Desvincula a tela do anunciante. Sem `confirmar=true` devolve só a prévia do impacto. Com `confirmar=true`, se a prévia apagar campanhas exige também `campanhas:delete`, e se apagar mídias exige `midias:delete` (403 sem eles).

## Campanhas

- `GET /campanhas` [campanhas:read] **cargo 30+**: Lista campanhas.
- `GET /campanhas/{id}` [campanhas:read] **cargo 30+**: Detalha uma campanha.
- `POST /campanhas` [campanhas:write]: Cria uma campanha. Com faixas, telas e mídias, pelas mesmas regras do editor do painel.
- `PUT /campanhas/{id}` [campanhas:write]: Substitui uma campanha. Troca todas as faixas; o anunciante não muda.
- `DELETE /campanhas/{id}` [campanhas:delete] **risco**: Exclui uma campanha.
- `POST /campanhas/{id}/duplicar` [campanhas:write]: Duplica a campanha como "<nome> (cópia)".
- `POST /campanhas/{id}/ativar` [campanhas:write]: Ativa a campanha.
- `POST /campanhas/{id}/pausar` [campanhas:write]: Pausa a campanha.
- `POST /campanhas/{id}/aprovar` [campanhas:write]: Aprova a campanha (o aprovador registrado é o criador da chave).
- `POST /campanhas/{id}/recusar` [campanhas:write]: Recusa a campanha com a observação que o anunciante vê.

## Programação institucional

- `GET /programacao-institucional` [campanhas:read] **cargo 30+**: Lista as programações institucionais da organização.
- `POST /programacao-institucional` [campanhas:write]: Cria uma programação institucional. Usa mídias da biblioteca institucional.
- `GET /programacao-institucional/{id}` [campanhas:read] **cargo 30+**: Detalha uma programação institucional. Traz faixas, telas e mídias.
- `PUT /programacao-institucional/{id}` [campanhas:write]: Substitui a programação institucional inteira (todas as faixas).
- `DELETE /programacao-institucional/{id}` [campanhas:delete] **risco**: Exclui uma programação institucional.
- `POST /programacao-institucional/{id}/duplicar` [campanhas:write]: Duplica a programação como "<nome> (cópia)".
- `POST /programacao-institucional/{id}/ativar` [campanhas:write]: Ativa a programação institucional.
- `POST /programacao-institucional/{id}/pausar` [campanhas:write]: Pausa a programação institucional.

## Mídias

- `GET /midias` [midias:read] **cargo 30+**: Lista as mídias (imagens e vídeos) da organização.
- `POST /midias/uploads` [midias:write]: Gera a URL assinada de envio de mídia. Envie o arquivo com PUT e exatamente os `headers` devolvidos, dentro de 2 h. Depois registre com POST /midias.
- `POST /midias` [midias:write]: Registra o arquivo enviado. O tipo é conferido pelo conteúdo (assinatura do arquivo): se não bater, o arquivo é apagado e a resposta é 422 `invalid_media_content`. Vídeo nasce `processing`.
- `GET /midias/{id}` [midias:read] **cargo 30+**: Detalha uma mídia. Traz `transcode_status`: consulte até `ready` ou `failed`.
- `DELETE /midias/{id}` [midias:delete] **risco**: Exclui uma mídia.
- `POST /midias/{id}/ajuste-slot` [midias:write]: Acelera um vídeo para uma duração. Cria uma cópia do vídeo acelerada para caber em `target_seconds`. A cópia nasce `processing`.

## Layouts e fontes de notícia

- `GET /layouts` [layouts:read]: Lista os layouts da organização.
- `POST /layouts` [layouts:write]: Cria um layout vazio (o documento entra com `PUT /layouts/{id}`).
- `GET /layouts/{id}` [layouts:read]: Detalha um layout. Traz fundo e elementos, com posições em % do canvas.
- `PUT /layouts/{id}` [layouts:write]: Salva o documento do layout. Elementos, fundo, fonte e notícias × segundos por notícia. Re-renderiza nas telas que o usam.
- `PATCH /layouts/{id}` [layouts:write]: Renomeia um layout.
- `DELETE /layouts/{id}` [layouts:delete] **risco**: Exclui um layout. 409 `layout_in_use` enquanto alguma mídia de programação o aplica.
- `POST /layouts/{id}/duplicar` [layouts:write]: Duplica um layout como "<nome> (cópia)".
- `GET /fontes-conteudo` [layouts:read]: Lista as fontes de conteúdo (feeds RSS) da organização.
- `POST /fontes-conteudo` [layouts:write]: Resolve uma URL de RSS em fonte de conteúdo. 201 quando cria, 200 quando a organização já tinha a mesma URL.

## Criador de vídeo

- `GET /videos/saldo` [videos:read] **cargo 30+**: Saldo de créditos de vídeo da organização e custo de um vídeo.
- `GET /videos/modelos` [videos:read] **cargo 30+**: Modelos de vídeo com os campos que cada um aceita. Para `livre` e `canvas`, `document_schema` traz o JSON Schema do roteiro ou do documento do canvas. O roteiro livre só aparece enquanto o super admin o mantém ligado.
- `GET /videos/projetos` [videos:read] **cargo 30+**: Projetos de vídeo de um anunciante.
- `POST /videos/projetos` [videos:write] **risco** **cargo 30+**: Cria um projeto de vídeo para uma tela. Valida como rascunho, com as regras do painel: a tela precisa de resolução e tempo de slot (422), e o campo inválido volta em `details` (400). Não gasta créditos; gerar é POST /videos/projetos/{id}/gerar.
- `GET /videos/projetos/{id}` [videos:read] **cargo 30+**: Detalha um projeto de vídeo, com a última geração.
- `PUT /videos/projetos/{id}` [videos:write] **risco** **cargo 30+**: Substitui modelo, tela e campos de um projeto de vídeo.
- `POST /videos/projetos/{id}/gerar` [videos:write] **risco** **Idempotency-Key** **cargo 30+**: Gera o vídeo do projeto, debitando créditos da organização. Responde 202 com a geração na fila; acompanhe em GET /videos/geracoes/{id}. A mesma `Idempotency-Key` devolve sempre a primeira geração, mesmo depois de pronta. Projeto que já tem geração na fila ou renderizando devolve essa geração,…
- `GET /videos/geracoes/{id}` [videos:read] **cargo 30+**: Estado de uma geração de vídeo: fila, renderizando, pronta ou falha. Pronta, traz `midia_id`, o vídeo na biblioteca do anunciante (GET /midias/{id}).
- `POST /videos/geracoes/{id}/cancelar` [videos:write] **risco** **cargo 30+**: Cancela uma geração que ainda está na fila. Geração já renderizando ou concluída responde `{ cancelled: false }`.

## Relatórios

- `GET /relatorios` [relatorios:read]: Lista os relatórios gerados da organização, em ordem de ID.
- `POST /relatorios` [relatorios:write]: Pede um relatório em PDF. Anunciante e período de até 31 dias. Responde 202 com o relatório `queued`; consulte GET /relatorios/{id} até `completed` ou `failed`.
- `GET /relatorios/{id}` [relatorios:read]: Detalha um relatório. Traz `status` (queued, processing, completed, failed). Falhou: peça outro.
- `GET /relatorios/{id}/download` [relatorios:read]: Link temporário do PDF do relatório. Válido por 5 min; vencido, peça outro aqui. 409 `report_not_ready` enquanto aguarda geração; 409 `report_failed` se a geração falhou (`failed`).
- `GET /relatorios/agendamentos` [agendamentos:read]: Lista os agendamentos do relatório mensal dos anunciantes. Só anunciantes com agendamento salvo, em ordem de cliente_id.
- `GET /relatorios/agendamentos/{id}` [agendamentos:read]: Agendamento do relatório mensal de um anunciante. `{id}` é o cliente_id do anunciante. 404 quando não há agendamento salvo.
- `PUT /relatorios/agendamentos/{id}` [agendamentos:write] **risco** **cargo 30+**: Cria ou substitui o agendamento do relatório mensal de um anunciante. `{id}` é o cliente_id. Mesmas regras do painel: cargo 30 ou mais; ligar exige ao menos um e-mail e respeita a cota de automações da organização (422); desligar sempre é aceito. O relatório sai todo mês por e-mail para recipient_emails.

## Audiência

- `POST /telas/{id}/contagens` [contagens:write] **cargo 50+**: Envia a contagem de fluxo de uma tela.
- `GET /telas/{id}/contagens` [contagens:read]: Lista a contagem de fluxo de uma tela.
- `GET /audiencia/fatores` [audiencia:read]: Fatores de ocupação por classe: o vigente hoje e os futuros. O fator da organização vence o padrão global (`origem`). É o mesmo fator que os relatórios de audiência usam.
- `POST /audiencia/fatores` [audiencia:write] **cargo 50+**: Registra uma nova versão do fator de ocupação de uma classe. Mesmas regras do painel: cargo 50 ou mais; vigência hoje ou depois (422 se retroativa, porque mudaria relatórios já emitidos); uma versão por classe e dia (409). Não há edição nem exclusão: um fator novo substitui o anterior a partir da…
- `GET /telas/{id}/validacao-contagem` [audiencia:read]: Validação da contagem da câmera de uma tela. `{id}` aceita tela_code ou uuid. Campos null quando a tela nunca foi validada.
- `PUT /telas/{id}/validacao-contagem` [audiencia:write] **cargo 50+**: Registra a validação da contagem da câmera de uma tela. `{id}` aceita tela_code ou uuid. Substitui data, erro e observação. Cargo 50 ou mais, como no painel.

## Equipe e acessos

- `GET /equipe` [equipe:read]: Lista os administradores. Com cargo e estado do convite.
- `POST /equipe` [equipe:write] **risco**: Convida um administrador. Mesmo e-mail do painel. 201 quando cria o convite, 200 quando o e-mail já tinha conta ou convite. Cargo acima do nível do criador da chave: 403.
- `GET /equipe/cargos` [equipe:read]: Cargos que a chave pode atribuir. Até o nível do criador da chave.
- `PATCH /equipe/{id}` [equipe:write] **risco**: Muda o cargo de um administrador. Membro ou cargo acima do nível do criador da chave: 403.
- `DELETE /equipe/{id}` [equipe:write] **risco**: Remove um administrador da organização (mesmas regras do painel).
- `POST /equipe/{id}/reenviar-convite` [equipe:write] **risco**: Reenvia o convite pendente. Convite já aceito devolve `already_member`.
- `POST /clientes/{id}/acesso` [equipe:write] **risco**: Cria o acesso do anunciante ao painel. Envia o convite. 201 quando cria.
- `PATCH /clientes/{id}/acesso` [equipe:write] **risco**: Muda o e-mail de login do acesso do anunciante.
- `GET /clientes/{id}/membros` [equipe:read]: Lista os membros do painel do anunciante. Com cargo e estado do convite.
- `POST /clientes/{id}/membros` [equipe:write] **risco**: Convida um membro do painel do anunciante. `cargo_id` é um cargo de anunciante. 201 quando cria o convite, 200 quando vincula uma conta existente.
- `DELETE /clientes/{id}/membros/{membro}` [equipe:write] **risco**: Remove o membro do painel do anunciante.
- `POST /clientes/{id}/membros/{membro}/reenviar-convite` [equipe:write] **risco**: Reenvia o convite pendente do membro do anunciante.

## Identidade

- `GET /eu` [qualquer]: Identidade do principal: usuário, organização, cargo e escopos. Usuário (app ou conector) traz usuário e cargo; chave de API, só organização e escopos. `permissoes` é o JSONB do cargo, sem transformação.

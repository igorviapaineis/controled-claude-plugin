# Controled para o Claude

Opere a rede de telas de LED e totens da sua empresa no Controled conversando com o Claude: veja
quais telas estão paradas, monte um layout com notícias e clima, crie uma arte para os horários
sem anunciante, tire o anúncio de um cliente que cancelou, confira quantas vezes um anúncio
passou e peça o relatório do anunciante.

O plugin junta duas peças:

- **O conector do Controled**, que dá ao Claude acesso à sua organização no Controled, com as
  mesmas permissões que você tem no painel.
- **A skill `controled`**, que ensina o Claude a reconhecer pedidos sobre as telas mesmo quando
  você não cita o Controled, a achar a ação certa e a confirmar com você antes de qualquer
  alteração que apague dados, mande comando a uma tela ou gaste créditos.

## Instalar

- **Claude.ai, app do Claude e Cowork**: em Personalizar › Plugins › Adicionar › Adicionar
  marketplace, informe `igorviapaineis/controled-claude-plugin` e instale o plugin Controled.
- **Claude Code**:

  ```
  /plugin install controled --marketplace igorviapaineis/controled-claude-plugin
  ```

- **Arquivo do plugin**: compacte a pasta `plugins/controled` do repositório e envie em
  Personalizar › Plugins › Adicionar › Fazer upload de plugin.

## Como usar

1. Com o plugin instalado, conecte o conector do Controled pela aba de conectores do plugin.
2. Entre com a sua conta do Controled. A conexão exige ser membro ativo do painel de uma
   organização ativa, com autenticação em dois fatores, e o interruptor "Conectores de IA" ligado em
   Perfil, no painel do Controled; ligar o interruptor é de um administrador com cargo de nível
   50 ou mais.
3. Peça em português, do jeito que falaria com alguém da equipe. Exemplos:
   - "quais telas estão offline agora?"
   - "cria um layout com uma faixa de clima embaixo e notícias no resto"
   - "faz uma arte de natal pras telas sem anunciante"
   - "o vídeo do cliente está cortando no final, dá pra arrumar?"

Colocar um layout numa tela e enviar o arquivo de uma mídia são feitos no painel do Controled.

## Dados

O plugin não guarda nada e não chama nenhum serviço além do Controled. Quando você pede algo, o
Claude chama o servidor do Controled em `https://controled.com.br/api/mcp`, autenticado pela sua
conta (OAuth), e envia os dados da ação pedida: ids de telas e anunciantes, campos de cadastro,
textos de campanhas e layouts e, quando for o caso, endereços de e-mail. O servidor devolve só os
dados da sua organização que o seu cargo permite ver e registra os acessos e as alterações na
auditoria do Controled.

Algumas ações fazem o próprio Controled agir fora da conversa: o relatório mensal agendado e os
convites de acesso saem por e-mail para os endereços informados, e o envio de uma mídia vai para o
armazenamento do Controled por um link temporário. Você pode revogar a conexão a qualquer momento
em Perfil, no painel do Controled.

Documentação da API: https://controled.com.br/docs/api

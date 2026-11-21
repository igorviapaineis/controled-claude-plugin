# Controled para o Claude

Plugin do [Controled](https://controled.com.br) para o Claude (claude.ai, app do Claude, Cowork e
Claude Code). Ele junta o conector do Controled a uma skill que ensina o Claude a operar a rede de
telas de LED e totens da sua empresa: telas e players, anunciantes, contratos, campanhas,
programação institucional, mídias, layouts com notícias e clima, criador de vídeo, relatórios e
diagnósticos.

O plugin fica em [`plugins/controled`](plugins/controled), com o que ele faz, como usar e que dados
envia.

## Instalar

- **Claude.ai, app do Claude e Cowork**: em Personalizar › Plugins › Adicionar › Adicionar
  marketplace, informe `igorviapaineis/controled-claude-plugin` e instale o plugin Controled.
- **Claude Code**:

  ```
  /plugin install controled --marketplace igorviapaineis/controled-claude-plugin
  ```

Depois, conecte o conector do Controled pela aba de conectores do plugin, com a sua conta do
Controled.

Este repositório é uma cópia publicada; o plugin é mantido junto com o código do Controled.
Licença: [LICENSE](plugins/controled/LICENSE).

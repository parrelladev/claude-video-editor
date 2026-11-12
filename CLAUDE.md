# Editor de Vídeo com IA

Esta pasta transforma o Claude Code num editor de vídeo. A pessoa grava, coloca o arquivo em `01-bruto` e pede a edição com `/editar`. O estilo vem da folha criada com `/estilo`.

## Pastas

| Pasta ou arquivo | Função | Quem mexe |
| --- | --- | --- |
| `01-bruto/` | Vídeos gravados, do jeito que saíram da câmera | A pessoa. **O Claude só lê, nunca altera, move ou apaga.** |
| `02-referencias/` | Vídeos de referência baixados pelo `/estilo` | O Claude |
| `03-prontos/` | Vídeos finalizados (`<nome>-reel.mp4`) | O Claude |
| `folha-de-estilo.md` | Regras de estilo de edição | O Claude escreve, a pessoa aprova |
| `trabalho/` | Arquivos temporários de cada edição (transcrição, cortes, projeto HyperFrames, storyboard) | O Claude. Crie a pasta quando precisar. Pode ser apagada a qualquer momento. |

Se alguma das três pastas numeradas não existir, crie antes de começar.

## Ferramentas

- **FFmpeg / ffprobe**: cortes, junções, análise e mixagem.
- **Whisper** (`whisper`, ou `python -m whisper` se o primeiro não estiver no PATH): transcrição. Modelo padrão `small`, idioma `pt`. Se a transcrição sair ruim, use `medium`.
- **yt-dlp**: download dos vídeos de referência.
- **HyperFrames** (skills `/hyperframes`, `/hyperframes-cli`, `/hyperframes-registry`, `/media-use`): legendas, motion, efeitos e render. Leia a skill `hyperframes` antes de montar qualquer projeto.

No Windows, rode os comandos pelo terminal disponível e use caminhos entre aspas.

## Regras

1. **Nunca altere `01-bruto`.** Todo processamento gera arquivos novos em `trabalho/`.
2. **Não use material das referências no vídeo final.** Os vídeos de `02-referencias` servem só para estudar ritmo, enquadramento, legenda e efeitos. Nenhum trecho, música, imagem, logo ou identidade visual deles entra no vídeo.
3. **Grafismos, trilhas e efeitos sonoros** são criados no próprio projeto ou vêm de bibliotecas livres pelas skills do HyperFrames. Anote a origem e a licença de cada trilha e efeito em `trabalho/<nome>/creditos.md`.
4. **Nada de voz sintética ou clone de voz/rosto** de pessoas reais, a não ser que a pessoa peça e confirme que tem autorização do dono da voz ou da imagem.
5. **Não envie vídeos, áudios ou transcrições para serviços externos** (HeyGen, render na nuvem, APIs) sem perguntar antes. As gravações podem conter dados pessoais (LGPD). Prefira sempre as opções locais.
6. **Só edite com a folha de estilo aprovada.** O `/editar` não começa se `folha-de-estilo.md` não existir ou não estiver com `Status: APROVADA`.
7. **Mostre o storyboard e espere aprovação antes do render final.**
8. **Revisão humana obrigatória.** Ao entregar, lembre que o vídeo precisa ser assistido inteiro por uma pessoa antes de publicar. Em temas sensíveis (eleições, segurança pública, saúde, denúncias, reputação), o vídeo passa pelo editor-chefe antes da publicação.
9. **Transparência.** Aponte nomes próprios, números e trechos em que a transcrição teve baixa confiança. Nunca invente falas nem altere o sentido do que foi dito.

---
name: editar
description: Edita um vídeo de 01-bruto como reel vertical seguindo a folha de estilo aprovada. Corta pausas, coloca legendas, efeitos e motion, mostra um storyboard para aprovação e exporta para 03-prontos. Use quando a pessoa digitar /editar ou pedir para editar um vídeo gravado.
argument-hint: [arquivo em 01-bruto]
disable-model-invocation: true
---

# /editar: editar um vídeo

Arquivo pedido: $ARGUMENTS

Siga as regras do `CLAUDE.md`. Trabalhe na raiz do projeto. **Nunca altere nada em `01-bruto/`.**

## 1. Conferir antes de começar

1. Leia `folha-de-estilo.md`. Se o arquivo não existir ou não tiver `Status: APROVADA`, pare e diga para rodar `/estilo` e aprovar a folha.
2. Escolha o vídeo: o arquivo pedido acima, procurado em `01-bruto/`. Se nada foi pedido, use o vídeo mais recente da pasta. Se a pasta estiver vazia, diga para colocar o vídeo lá e pare.
3. Defina `<nome>` como o nome do arquivo sem extensão e crie `trabalho/<nome>/`. Todos os arquivos desta edição ficam ali.
4. Confira o vídeo com `ffprobe` (duração, resolução, rotação, áudio). Se não houver áudio, avise.

## 2. Transcrever

```bash
python -m whisper "01-bruto/<arquivo>" --model small --language pt --word_timestamps True --output_format json --output_dir "trabalho/<nome>"
```

No macOS e no Linux, use `whisper` no lugar de `python -m whisper`. Se um não funcionar, tente o outro com os mesmos argumentos. Leia a transcrição e descarte palavras que o Whisper inventa em silêncio (por exemplo "Obrigado." no fim).

## 3. Cortar

1. A partir dos tempos de cada palavra, monte a lista de trechos a manter em `trabalho/<nome>/cortes.json`:
   - corte toda pausa entre palavras maior que a da folha de estilo (padrão 0,15 s), deixando uma folga de 0,05 s antes e depois da fala;
   - corte erros, repetições e falsos começos, ficando com a última tentativa completa da frase;
   - nunca corte no meio de uma palavra nem mude o sentido do que foi dito.
2. Gere `trabalho/<nome>/cortado.mp4` com o FFmpeg (filtros `trim`/`atrim` + `concat`, ou `select`/`aselect`), mantendo áudio e vídeo sincronizados.
3. Recalcule os tempos das palavras para o vídeo cortado e salve em `trabalho/<nome>/palavras.json` no formato `[{ "id": "w0", "text": "...", "start": 0.0, "end": 0.4 }]`.
4. Confira: a duração de `cortado.mp4` bate com a soma dos trechos, e o som está em sincronia (olhe alguns quadros com boca aberta).

## 4. Montar a edição no HyperFrames

1. Leia a skill `hyperframes` e, dentro dela, `references/types/vertical.md`, `references/captions.md`, `references/overlays.md`, `references/motion.md` e `references/sound.md`.
2. Crie o projeto e coloque o vídeo cortado em `assets/`:
   ```bash
   npx hyperframes init "trabalho/<nome>/projeto" --skip-skills --skill=hyperframes
   ```
3. Monte o reel em 1080x1920, 30 fps, aplicando a folha de estilo: gancho no primeiro segundo, legendas palavra a palavra a partir de `palavras.json`, zooms, cards e transições nos pontos que a folha indica.
4. Antes de criar um efeito do zero, procure pronto no registro (`/hyperframes-registry`).
5. Trilha e efeitos sonoros: use `/media-use`, só com opções locais ou bibliotecas livres. Pergunte antes de usar qualquer serviço que exija enviar o vídeo ou a voz para fora. Anote origem e licença de cada som em `trabalho/<nome>/creditos.md`.
6. Rode `npx hyperframes check --json` dentro do projeto e corrija todos os erros.

## 5. Storyboard e aprovação

1. Escreva `trabalho/<nome>/storyboard.md` com uma tabela: cena, início–fim, fala, legenda, efeito visual, som.
2. Mostre no chat a tabela e a duração final. Abra a pré-visualização (`npx hyperframes preview --background`) e passe o endereço.
3. Pergunte o que ajustar. Aplique os pedidos ("tira o zoom da cena 3", "legenda menor"), rode o `check` de novo e mostre o storyboard atualizado.
4. Só siga para o render quando a pessoa disser que aprova.

## 6. Exportar

1. Crie `03-prontos/` se não existir e renderize:
   ```bash
   npx hyperframes render --quality delivery -o "<caminho absoluto do projeto>/03-prontos/<nome>-reel.mp4"
   ```
2. Confira o arquivo: não está vazio, a duração no `ffprobe` bate com a do projeto, e alguns quadros extraídos estão corretos.

## 7. Entregar

Diga onde está o vídeo e a duração, e liste os pontos que merecem atenção na revisão: nomes próprios, números, palavras em que a transcrição teve dúvida e cortes mais arriscados.

Lembre que o vídeo precisa ser assistido inteiro por uma pessoa antes de publicar e que, em temas sensíveis (eleições, segurança pública, saúde, denúncias, reputação), ele passa pelo editor-chefe antes.

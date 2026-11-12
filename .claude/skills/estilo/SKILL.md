---
name: estilo
description: Estuda de 3 a 5 vídeos de referência e escreve a folha de estilo de edição (folha-de-estilo.md) para a pessoa aprovar. Use quando a pessoa digitar /estilo ou pedir para criar ou refazer a folha de estilo.
argument-hint: <link-1> <link-2> <link-3> [...]
disable-model-invocation: true
---

# /estilo: criar a folha de estilo

Links recebidos: $ARGUMENTS

Siga as regras do `CLAUDE.md`. Trabalhe na raiz do projeto.

## 1. Conferir a entrada

- Se não vieram links, veja se já há vídeos em `02-referencias/`. Se houver, use esses. Se não houver, peça de 3 a 5 links e pare.
- Com menos de 3 vídeos, avise que a análise fica mais fraca e pergunte se quer seguir.
- Se `folha-de-estilo.md` já existir, avise que ela será substituída e pergunte antes de continuar.

## 2. Baixar as referências

Crie as pastas `02-referencias/` e `trabalho/estilo/` se não existirem. Para cada link:

```bash
yt-dlp -f "bv*[height<=1080]+ba/b" --merge-output-format mp4 -o "02-referencias/%(extractor)s-%(id)s.%(ext)s" "<link>"
```

Se um download falhar (vídeo privado, login exigido, bloqueio da plataforma), siga com os outros. No fim, diga quais falharam e explique que a pessoa pode baixar o vídeo por conta própria e colocar em `02-referencias/`.

## 3. Analisar cada vídeo

Para cada arquivo em `02-referencias/`, salve os resultados em `trabalho/estilo/<arquivo>/`:

1. **Formato:** `ffprobe -v error -show_entries stream=width,height,r_frame_rate -show_entries format=duration -of json "<arquivo>"`.
2. **Ritmo de corte:** conte as trocas de plano e calcule a duração média de cada plano.
   ```bash
   ffmpeg -i "<arquivo>" -vf "select='gt(scene,0.3)',showinfo" -an -f null - 2> "trabalho/estilo/<arquivo>/cortes.txt"
   ```
3. **Quadros:** extraia um quadro por segundo e olhe pelo menos 8 deles, espalhados pelo vídeo, incluindo o primeiro segundo.
   ```bash
   ffmpeg -i "<arquivo>" -vf "fps=1,scale=540:-1" "trabalho/estilo/<arquivo>/q%03d.jpg"
   ```
   Observe: posição, tamanho, peso e caixa da legenda; palavras por linha; cor de destaque; contorno ou fundo; zooms e punch-ins; textos e cards na tela; enquadramento; o que aparece no primeiro quadro (gancho).
4. **Fala:** transcreva com Whisper (`--model small --language pt --word_timestamps True --output_format json`) e calcule palavras por segundo e o tamanho das pausas que sobraram na edição.
5. **Som:** veja se há trilha de fundo, efeitos sonoros em cortes ou textos e o volume da trilha em relação à voz (`ffmpeg -i "<arquivo>" -af volumedetect -f null -`).

## 4. Escrever a folha

Escreva `folha-de-estilo.md` na raiz com o que os vídeos têm **em comum**, sempre com números (pixels, segundos, cores em hex, palavras por linha). Não copie a identidade visual dos criadores: escolha fonte, cores e grafismos próprios que tenham o mesmo efeito. Use fontes que o HyperFrames já traz (veja `references/format.md` na skill `hyperframes`).

Use esta estrutura:

```markdown
# Folha de estilo

Status: RASCUNHO
Referências: <arquivos analisados>
Atualizada em: <data>

## Formato
- Saída: reel vertical 1080x1920, 30 fps
- Duração alvo: ...

## Gancho (primeiros 2 segundos)
- ...

## Cortes e ritmo
- Pausa máxima mantida: 0,15 s
- Duração média de plano: ... s
- Cortar: erros, repetições, falsos começos, "é...", "tipo..." quando sobrarem

## Zoom e enquadramento
- Punch-in: ... % em ..., duração ... s
- ...

## Legendas
- Fonte, peso, tamanho (px), cor, contorno ou fundo
- Posição (faixa vertical em px), palavras por linha, máximo de linhas
- Destaque da palavra falada: cor ..., animação ...

## Textos e cards na tela
- ...

## Motion e transições
- ...

## Som
- Trilha: estilo, volume em relação à voz (dB)
- Efeitos sonoros: onde entram (cortes, textos, zooms)

## Evitar
- ...
```

## 5. Aprovação

Mostre no chat um resumo curto (até 10 itens) e pergunte o que ajustar. Aplique os ajustes que a pessoa pedir.

Quando ela disser que aprova, troque a linha `Status: RASCUNHO` por `Status: APROVADA` e atualize a data. Avise que agora é só colocar um vídeo em `01-bruto/` e digitar `/editar`.

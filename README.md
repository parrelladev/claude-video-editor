# Editor de Vídeo com IA

Transforme o Claude Code num editor de vídeo. Você grava do seu jeito, joga o arquivo numa pasta e pede a edição. O Claude corta as pausas, coloca legenda, efeitos sonoros e motion seguindo o seu estilo, mostra um storyboard para você aprovar e exporta o vídeo pronto.

> Projeto independente, sem vínculo com a Anthropic, a HeyGen ou os mantenedores das ferramentas usadas.

---

## Como funciona

Você só mexe em três pastas:

| Pasta | O que vai nela |
| --- | --- |
| `01-bruto/` | Os vídeos que você gravou. O Claude nunca altera esses arquivos. |
| `02-referencias/` | Vídeos de referência que o Claude baixa para estudar o estilo. |
| `03-prontos/` | Os vídeos editados, prontos para revisar e publicar. |

E usa dois comandos no Claude Code:

- **`/estilo`** estuda vídeos de referência e escreve a sua folha de estilo (`folha-de-estilo.md`). Você faz isso uma vez.
- **`/editar`** edita um vídeo de `01-bruto` como reel do Instagram seguindo essa folha.

O resto o Claude cuida sozinho: ele cria uma pasta `trabalho/` para os arquivos temporários de cada edição e segue as regras do `CLAUDE.md`, que lê toda vez que você abre o projeto.

```mermaid
flowchart LR
    A[Grava o vídeo] --> B[Coloca em 01-bruto]
    B --> C["/editar"]
    C --> D[Claude transcreve, corta e planeja]
    D --> E[Storyboard para revisão]
    E -->|ajustes| D
    E -->|aprovado| F[Render em 03-prontos]
    F --> G[Revisão humana e publicação]
```

Para editar, o Claude usa quatro ferramentas que você instala no computador:

| Ferramenta | Para que serve |
| --- | --- |
| [FFmpeg](https://github.com/FFmpeg/FFmpeg) | Corta, junta, converte e mixa áudio |
| [Whisper](https://github.com/openai/whisper) | Transcreve a fala e gera as legendas |
| [yt-dlp](https://github.com/yt-dlp/yt-dlp) | Baixa os vídeos de referência |
| [HyperFrames](https://github.com/heygen-com/hyperframes) | Legendas animadas, motion, efeitos e render |

---

## Requisitos

- Windows 10/11, macOS ou Linux
- [App desktop do Claude](https://claude.ai/download) com acesso ao Claude Code
- [Git](https://git-scm.com/downloads)
- [Node.js](https://nodejs.org) 22 ou mais recente
- [Python](https://www.python.org/downloads/) 3
- Cerca de 3 GB livres em disco para as ferramentas e o modelo de transcrição

---

## Tutorial

### 1. Baixe o projeto

Clone o repositório:

```bash
git clone https://github.com/parrelladev/claude-video-editor.git
cd claude-video-editor
```

Se preferir, clique em **Code > Download ZIP**, extraia e renomeie a pasta para `claude-video-editor` (o ZIP vem como `claude-video-editor-main`).

Confira se a pasta tem o arquivo `CLAUDE.md` e a pasta `.claude/skills/` com `estilo` e `editar` dentro. No Windows, a `.claude` pode estar oculta: no Explorador de Arquivos, ative **Exibir > Mostrar > Itens ocultos**.

### 2. Crie as pastas

Abra um terminal dentro da pasta do projeto e rode o comando do seu sistema.

**Windows (PowerShell):**

```powershell
"01-bruto","02-referencias","03-prontos" | ForEach-Object { New-Item -ItemType Directory -Force -Path $_ | Out-Null }
```

**macOS e Linux:**

```bash
mkdir -p 01-bruto 02-referencias 03-prontos
```

A pasta fica assim:

```
claude-video-editor/
├─ 01-bruto/           seus vídeos gravados
├─ 02-referencias/     vídeos de referência
├─ 03-prontos/         vídeos editados
├─ .claude/skills/     comandos /estilo e /editar (e as skills do HyperFrames, no passo 3)
└─ CLAUDE.md           regras que o Claude segue neste projeto
```

### 3. Instale as ferramentas

Instale cada ferramenta com o gerenciador de pacotes do seu sistema. No Windows, use o [winget](https://learn.microsoft.com/windows/package-manager/winget/), que já vem no Windows 10 e 11. No macOS, use o [Homebrew](https://brew.sh).

Depois de instalar, **feche e abra o terminal de novo** para o sistema reconhecer os novos comandos.

#### Node.js e Python

Pule se já tiver os dois instalados.

| Sistema | Comando |
| --- | --- |
| Windows | `winget install OpenJS.NodeJS.LTS`; Python pelo [site oficial](https://www.python.org/downloads/), marcando **Add python.exe to PATH** na instalação |
| macOS | `brew install node python` |
| Linux | Python pelo gerenciador da distribuição; Node.js 22+ pelo [nvm](https://github.com/nvm-sh/nvm) ou pelo [site oficial](https://nodejs.org) (o pacote do `apt` costuma ser antigo) |

#### FFmpeg

| Sistema | Comando |
| --- | --- |
| Windows | `winget install Gyan.FFmpeg` |
| macOS | `brew install ffmpeg` |
| Linux (Debian/Ubuntu) | `sudo apt install ffmpeg` |

#### Whisper

| Sistema | Comando |
| --- | --- |
| Windows | `python -m pip install -U openai-whisper` |
| macOS | `brew install openai-whisper` |
| Linux | `pipx install openai-whisper` |

O Whisper baixa o modelo de transcrição (cerca de 460 MB) na primeira edição. Não precisa fazer nada além de esperar.

#### yt-dlp

| Sistema | Comando |
| --- | --- |
| Windows | `winget install yt-dlp.yt-dlp` |
| macOS | `brew install yt-dlp` |
| Linux | `pipx install yt-dlp` |

#### HyperFrames

O HyperFrames entra no Claude Code como um conjunto de skills. Rode este comando **dentro da pasta do projeto**:

```bash
npx skills add heygen-com/hyperframes -s hyperframes hyperframes-cli hyperframes-registry media-use -a claude-code -y
```

Ele instala as skills em `.claude/skills/`, ao lado de `estilo` e `editar`.

O HyperFrames coleta dados de uso anônimos. Para desligar:

```bash
npx hyperframes telemetry disable
```

### 4. Confira a instalação

Rode cada comando abaixo. Todos devem responder com um número de versão ou um texto de ajuda, sem erro.

```bash
node --version
ffmpeg -version
yt-dlp --version
npx hyperframes --version
```

Para o Whisper, o comando muda conforme o sistema:

| Sistema | Comando |
| --- | --- |
| Windows | `python -m whisper --help` |
| macOS e Linux | `whisper --help` |

No Windows, o comando `whisper` sozinho costuma não ser encontrado, e isso é normal. O que vale é o `python -m whisper`, e é ele que o Claude usa.

### 5. Abra a pasta no Claude Code

1. Abra o app desktop do Claude e faça login.
2. Vá em **Code**, escolha **Local** e selecione a pasta `claude-video-editor`.
3. No chat, digite `/` e confira se aparecem **`/estilo`** e **`/editar`** na lista.

Se não aparecerem, veja [Problemas comuns](#problemas-comuns).

### 6. Crie a sua folha de estilo

Faça isso uma vez só. Digite `/estilo` seguido de 3 a 5 links de vídeos com o estilo que você quer seguir. Podem ser vídeos seus ou de outro criador.

```
/estilo https://link-1 https://link-2 https://link-3
```

O Claude vai:

1. Baixar os vídeos para `02-referencias`.
2. Estudar o que eles têm em comum: legenda (fonte, cor, tamanho, posição), ritmo de corte, zooms, textos na tela, música e efeitos sonoros.
3. Escrever as regras em `folha-de-estilo.md`, com números (tamanhos, tempos, cores) e com fonte e cores próprias, sem copiar a identidade visual dos criadores.
4. Mostrar um resumo e perguntar o que ajustar.

Peça as mudanças que quiser ("legenda amarela em vez de verde", "menos zoom"). Quando estiver bom, escreva **"aprovo"**. A folha passa de `RASCUNHO` para `APROVADA`, e a partir daí o `/editar` funciona.

Se algum link não baixar (vídeo privado ou que exige login), baixe o vídeo por conta própria, coloque em `02-referencias` e rode `/estilo` sem links: o Claude usa o que estiver na pasta.

### 7. Edite um vídeo

Grave do jeito que você sempre grava e coloque o arquivo em `01-bruto`. No Claude Code, digite:

```
/editar meu-video.mp4
```

Sem nome de arquivo, o Claude usa o vídeo mais recente da pasta.

O Claude vai:

1. Transcrever o áudio.
2. Cortar as pausas maiores que 0,15 segundo, além de erros e repetições.
3. Montar legenda, efeitos sonoros e motion conforme a folha de estilo, no formato de reel (vertical, 1080x1920).
4. Mostrar um storyboard com cada cena, o tempo, a fala e os efeitos, e um link de pré-visualização.

### 8. Revise e exporte

Leia o storyboard, assista à pré-visualização e peça os ajustes como pediria a um editor: "tira o zoom da cena 3", "a legenda está grande demais", "corta a parte em que eu gaguejo no começo". Quando estiver bom, escreva **"aprovo"**. O Claude renderiza e salva o vídeo em `03-prontos` com o nome `<arquivo>-reel.mp4`.

Assista ao vídeo inteiro antes de publicar. O Claude lista os pontos que merecem atenção, como nomes próprios, números e trechos em que a transcrição ficou em dúvida. Em temas sensíveis (eleições, segurança pública, saúde, denúncias, reputação), o vídeo passa pelo editor-chefe antes da publicação.

---

## Personalização

| O que mudar | Onde |
| --- | --- |
| Pausa máxima, legenda, zoom, trilha | Peça ao Claude para ajustar `folha-de-estilo.md` |
| Passo a passo da edição e formato de saída | `.claude/skills/editar/SKILL.md` |
| O que o Claude analisa nas referências | `.claude/skills/estilo/SKILL.md` |
| Regras gerais e modelo do Whisper | `CLAUDE.md` |

O modelo `small` do Whisper é rápido e funciona bem na maioria das gravações. Se as legendas em português saírem com muitos erros, peça para usar o `medium`, que é mais preciso e mais lento.

Pausas de 0,15 s deixam o ritmo bem acelerado. Se o vídeo ficar picotado, peça 0,25 ou 0,3 s na folha de estilo.

---

## Problemas comuns

**`/estilo` e `/editar` não aparecem no Claude Code.** Confira se abriu exatamente a pasta `claude-video-editor`, e não a pasta acima dela, e se existem os arquivos `.claude/skills/estilo/SKILL.md` e `.claude/skills/editar/SKILL.md`. Se abriu o app antes de baixar o projeto, feche e abra de novo.

**"Comando não encontrado" logo depois de instalar.** Feche e abra o terminal de novo. Se o Claude Code já estava aberto, feche e abra o app também, porque ele só enxerga os comandos que existiam quando foi iniciado.

**O `pip install` falha no macOS ou no Linux com a mensagem "externally-managed-environment".** O sistema protege o Python principal. Use o Homebrew (`brew install openai-whisper`) ou o `pipx`, como na tabela do passo 3.

**Erro ao instalar as skills do HyperFrames.** Confira se o Git está instalado (`git --version`) e se o Node.js é 22 ou mais recente (`node --version`).

**O `/editar` não começa.** A folha de estilo não existe ou ainda está como rascunho. Rode `/estilo`, revise e escreva "aprovo".

**A primeira edição demora.** Na primeira vez, o Whisper baixa o modelo de transcrição e o HyperFrames prepara o navegador de render. As próximas são mais rápidas.

---

## Direitos autorais e dados

Os vídeos de referência servem só para estudar ritmo, enquadramento, legenda e efeitos. O `CLAUDE.md` proíbe o Claude de reaproveitar trechos, músicas, imagens ou a identidade visual deles. Grafismos, trilhas e efeitos sonoros são criados no próprio projeto ou vêm de bibliotecas livres, e o Claude anota a origem de cada som.

Baixar vídeos de terceiros pode violar os termos de uso das plataformas. Use as referências com responsabilidade.

As gravações podem conter dados pessoais. O Claude trabalha com as ferramentas locais e pergunta antes de enviar qualquer arquivo para um serviço externo.

---

## Créditos

Este projeto não redistribui nenhuma das ferramentas abaixo. Cada uma tem a própria licença:

- [HyperFrames](https://github.com/heygen-com/hyperframes) (Apache 2.0)
- [FFmpeg](https://ffmpeg.org/legal.html) (LGPL/GPL)
- [Whisper](https://github.com/openai/whisper) (MIT)
- [yt-dlp](https://github.com/yt-dlp/yt-dlp) (Unlicense)

## Licença

Defina aqui a licença deste repositório.

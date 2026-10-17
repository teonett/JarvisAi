# J.A.R.V.I.S. — Local AI Voice Assistant

## Instalação e primeiro uso no Windows (PT-BR)

Este é o guia principal de instalação deste fork. Execute os comandos no **PowerShell**, na pasta do projeto, e avance somente quando cada etapa terminar sem erro. Para esta instalação, utilize **Python 3.11 de 64 bits** e o ambiente `.venv311`.


### Atualizar uma instalação existente para PT-BR

Encerre o Jarvis com **Ctrl+C** e execute `git status` antes de atualizar. Se houver alterações locais (por exemplo, no `config.yaml`), preserve-as antes de fazer `git pull`; não descarte suas configurações ou credenciais.

```powershell
git pull --ff-only
```

Confira estes campos nos blocos existentes do seu `config.yaml`, preservando os demais valores:

```yaml
stt:
  model: small
  language: pt
  device: cpu
  compute_type: int8
tts:
  voice: pm_alex
  lang_code: p
  speed: 1.0
```

Baixe a nova voz (mesmo que já tenha baixado uma voz inglesa) e teste a reprodução:

```powershell
.\.venv311\Scripts\python.exe -c "import yaml; from huggingface_hub import hf_hub_download; cfg=yaml.safe_load(open('config.yaml', encoding='utf-8')); voz=cfg['tts']['voice']; print(hf_hub_download(repo_id='hexgrad/Kokoro-82M', filename=f'voices/{voz}.pt', local_files_only=False))"
.\.venv311\Scripts\python.exe -c "from jarvis.tts import speak; speak('Olá! Agora estou falando em português brasileiro.')"
.\.venv311\Scripts\python.exe -m jarvis.main
```

Outras vozes brasileiras: `pf_dora` (feminina) e `pm_santa` (masculina). Mantenha `lang_code: p`, baixe a voz selecionada e reinicie após alterar. Referência: [vozes oficiais do Kokoro](https://huggingface.co/hexgrad/Kokoro-82M/blob/main/VOICES.md).

Se o teste indicar ausência de `phonemizer` ou `espeakng_loader`, instale-os no mesmo ambiente:

```powershell
.\.venv311\Scripts\python.exe -m pip install phonemizer espeakng-loader
```

Diga **“Hey Jarvis”**, espere **“Pode falar.”** e converse em português. Comandos reconhecidos após a transcrição incluem **“pare”**, **“cancelar”** e **“silêncio”**, além dos comandos ingleses existentes. O modelo de ativação não muda de idioma ao renomear uma configuração: **“Olá, Jarvis”** requer um modelo apropriado. A interface e as mensagens internas das ferramentas ainda não foram integralmente traduzidas.

### 1. Pré-requisitos

| Requisito | Orientação |
|---|---|
| Sistema | Windows 10/11, 64 bits, com sessão gráfica |
| Python | **3.11 de 64 bits**, selecionado explicitamente com `py -3.11` |
| Git | Necessário para clonar; alternativamente, use **Code → Download ZIP** no GitHub |
| Ollama | Necessário para o provedor local padrão |
| Áudio | Microfone e saída de áudio configurados no Windows |
| Internet | Necessária para instalar dependências e baixar os modelos no primeiro uso |
| Hardware | 16 GB de RAM como referência inicial; GPU NVIDIA é opcional e o consumo depende dos modelos |

O [Kokoro 0.9.4 publicado no PyPI](https://pypi.org/project/kokoro/0.9.4/) exige Python `>=3.10,<3.13`. Este guia padroniza em **3.11**; não use Python 3.13 ou superior para instalar as dependências atuais.

Instale Python e Git, se necessário:

```powershell
winget install --id Python.Python.3.11 -e --source winget
winget install --id Git.Git -e --source winget
```

Sem Winget, use os instaladores oficiais: [Python para Windows](https://www.python.org/downloads/windows/) (**3.11, Windows installer 64-bit**) e [Git para Windows](https://git-scm.com/downloads/win).

**Feche e reabra o terminal após instalar.** Confira:

```powershell
py -0p
py -3.11 --version
git --version
```

Se aparecer `No suitable Python runtime found`, o Python 3.11 não está instalado ou ainda não foi localizado pelo launcher. Conclua a instalação e abra outro terminal antes de continuar.

### 2. Obter o projeto

```powershell
git clone https://github.com/teonett/JarvisAi.git
cd JarvisAi
```

Se já tem o projeto, apenas entre na pasta existente. Exemplo:

```powershell
cd C:\source\Trash\JarvisAi
```

Ao usar ZIP, extraia o arquivo e entre na pasta que contém `requirements.txt` e `config.yaml`.

### 3. Criar o ambiente e instalar as dependências

```powershell
py -3.11 -m venv .venv311
.\.venv311\Scripts\python.exe -m pip install --upgrade pip setuptools wheel
.\.venv311\Scripts\python.exe -m pip install -r requirements.txt
.\.venv311\Scripts\python.exe -m pip check
```

Verifique o interpretador usado:

```powershell
.\.venv311\Scripts\python.exe -c "import sys, struct; print(sys.executable); print(sys.version); print('Bits:', struct.calcsize('P') * 8)"
```

O caminho deve apontar para `.venv311\Scripts\python.exe`, a versão deve ser **3.11.x** e a arquitetura deve ser **64**.

Os comandos deste guia usam diretamente o Python do ambiente; não é necessário ativá-lo nem alterar a política de execução do PowerShell. Se desejar ativar, use `.\.venv311\Scripts\Activate.ps1`.

> **Atenção aos arquivos BAT:** os scripts atuais `install.bat` e `start.bat` usam `.venv`, enquanto este guia usa `.venv311`. Para seguir esta instalação, use os comandos explícitos abaixo; os BAT não selecionam automaticamente este ambiente.

### 4. Instalar e preparar o Ollama

Instale o [Ollama para Windows](https://ollama.com/download/windows), abra o aplicativo e mantenha-o em execução. Reabra o terminal se o comando ainda não estiver disponível.

```powershell
ollama --version
ollama pull qwen3:8b
ollama list
```

Teste a resposta do modelo:

```powershell
ollama run qwen3:8b "Reply with a short greeting."
```

Se houver erro de conexão, inicie o serviço em **outro terminal**, mantendo-o aberto:

```powershell
ollama serve
```

Se a porta já estiver em uso pelo Ollama, não é necessário iniciar uma segunda instância.

O `config.yaml` já aponta para `qwen3:8b`. Modelos menores, como `qwen3:4b`, podem reduzir o consumo: baixe o modelo escolhido e ajuste `llm.providers.ollama.model`. A ferramenta `delegate_task` ainda possui `qwen3:8b` fixado em `jarvis/tools/subagent.py`; mantenha esse modelo baixado se usar a delegação. O modelo principal precisa suportar chamadas de ferramentas.

### 5. Baixar a voz do Kokoro e preparar a síntese

A voz padrão é `pm_alex`, em português brasileiro. O código procura o arquivo de voz apenas no cache local, por isso o download deve ocorrer **antes de iniciar o Jarvis**:

```powershell
.\.venv311\Scripts\python.exe -c "import yaml; from huggingface_hub import hf_hub_download; cfg=yaml.safe_load(open('config.yaml', encoding='utf-8')); voz=cfg['tts']['voice']; print(hf_hub_download(repo_id='hexgrad/Kokoro-82M', filename=f'voices/{voz}.pt', local_files_only=False))"
```

Prepare e teste os demais recursos da síntese, ainda com internet:

```powershell
.\.venv311\Scripts\python.exe -c "from jarvis.tts import speak_to_bytes; audio=speak_to_bytes('Olá! Jarvis está pronto.'); assert len(audio) > 0, 'Nenhum audio gerado'; print('Kokoro pronto. Bytes de audio:', len(audio))"
```

Esse teste gera áudio sem reproduzi-lo. O primeiro carregamento pode baixar pesos e recursos linguísticos. Se alterar a voz em `config.yaml`, repita o download e reinicie a aplicação.

### 6. Baixar os modelos de ativação “Hey Jarvis”

O pacote Python não garante que os arquivos ONNX já estejam presentes. Baixe o modelo e seus recursos auxiliares:

```powershell
.\.venv311\Scripts\python.exe -c "from openwakeword.utils import download_models; download_models(model_names=['hey_jarvis'])"
```

Valide o carregamento:

```powershell
.\.venv311\Scripts\python.exe -c "from openwakeword.model import Model; Model(wakeword_models=['hey_jarvis'], inference_framework='onnx'); print('Modelo de ativacao carregado com sucesso!')"
```

Os arquivos ficam em `.venv311\Lib\site-packages\openwakeword\resources\models`, incluindo `hey_jarvis_v0.1.onnx`, `melspectrogram.onnx` e `embedding_model.onnx`. Referência: [download de modelos do openWakeWord](https://github.com/dscripka/openWakeWord/blob/main/openwakeword/utils.py).

### 7. Configurar e preparar a transcrição

Para começar sem depender da configuração CUDA, altere **somente o bloco `stt` existente** em `config.yaml`, preservando as outras seções:

```yaml
stt:
  model: small
  device: cpu
  compute_type: int8
```

Prepare o Whisper antes do primeiro comando de voz:

```powershell
.\.venv311\Scripts\python.exe -c "from jarvis.stt import _get_model; _get_model(); print('Whisper pronto!')"
```

Depois de validar o funcionamento, você pode testar `device: cuda` com GPU NVIDIA e bibliotecas CUDA compatíveis. O projeto tenta voltar para CPU se a inicialização ou inferência CUDA falhar.

**Idioma atual:** a conversa por voz usa português brasileiro: `stt.language: pt`, `tts.lang_code: p` e `tts.voice: pm_alex`. A IA recebe instruções para responder em PT-BR; falas fixas, comandos de parada e gatilhos de memória também foram adaptados. A ativação continua sendo **“Hey Jarvis”**; a interface e mensagens técnicas das ferramentas ainda podem aparecer em inglês.

### 8. Iniciar e usar

Configure o microfone e a saída de som padrão no Windows. Em **Configurações → Privacidade e segurança → Microfone** (o nome varia conforme o Windows), permita acesso ao microfone para aplicativos da área de trabalho.

Na pasta do projeto:

```powershell
.\.venv311\Scripts\python.exe -m jarvis.main
```

A interface deve abrir em **http://localhost:7860**. O aplicativo tenta falar “Olá! Jarvis está pronto para ajudar.” e, após carregar a ativação, exibe `Listening for wake word...`.

| Entrada | Como usar |
|---|---|
| Voz | Diga **“Hey Jarvis”**, espere **“Pode falar.”** e fale, por exemplo, **“Que horas são?”** |
| Interface web | Digite uma pergunta na barra de conversa e envie |
| Teclado | Com o terminal em foco, pressione **F2**, digite e pressione Enter |
| Silenciar voz | Com o terminal em foco, pressione **Insert** |
| Solicitar interrupção | **Esc** no terminal ou **ABORT** na interface; chamadas em andamento podem demorar a encerrar |
| Encerrar | Pressione **Ctrl+C** no terminal que iniciou o Jarvis |

Nos próximos usos, basta manter o Ollama em execução e repetir o comando de inicialização. Não é necessário reinstalar os pacotes ou baixar os modelos novamente, desde que os ambientes e caches sejam preservados.

### 9. Internet, caches e primeiro uso offline

Voz, transcrição e LLM local podem funcionar sem nuvem depois de preparados. Faça primeiro um teste completo conectado: transcreva um comando, obtenha uma resposta, ouça a voz e teste uma preferência pessoal para inicializar os embeddings da memória. O ChromaDB também pode baixar seu modelo de embeddings no primeiro armazenamento de um fato.

Busca web, previsão do tempo, leitura de páginas e provedores de IA na nuvem continuam exigindo internet. A interface também referencia fontes do Google. Portanto, “local” não significa que todas as funcionalidades estejam disponíveis offline.

### 10. Solução dos erros mais comuns

| Mensagem ou sintoma | Causa e ação |
|---|---|
| `No suitable Python runtime found` | Instale Python 3.11, reabra o terminal e confira `py -3.11 --version`. |
| `No matching distribution found for kokoro>=0.9.4` | Confira o interpretador: use o Python 3.11 de `.venv311`, não o `python` global. |
| `Failed to build spacy/thinc` | Confirme Python 3.11 de 64 bits e atualize pip, setuptools e wheel. Se persistir, tente `.\.venv311\Scripts\python.exe -m pip install --only-binary=:all: spacy thinc` e depois repita a instalação dos requisitos. Se não houver distribuição compatível, examine as linhas anteriores do erro antes de instalar compiladores. |
| `LocalEntryNotFoundError` em `_get_voice` | Execute o download da voz na etapa 5 com internet; o Jarvis usa `local_files_only=True`. |
| `hey_jarvis_v0.1.onnx ... File doesn't exist` | Execute o download e a validação da etapa 6 no mesmo ambiente. |
| Aviso sobre `HF_TOKEN` | Downloads públicos podem funcionar sem token; ele é opcional para limites maiores. Se houver erro de limite ou autenticação, trate esse erro específico. |
| Avisos `dropout`, `weight_norm` ou `torch.jit.script` | Esses avisos, isoladamente, não encerram a aplicação. Procure o traceback final se ela parar. |
| DLL CUDA ausente ou falta de VRAM | Configure STT com `device: cpu` e `compute_type: int8`; considere um LLM menor. |
| Ollama não responde | Abra o aplicativo Ollama, confira `ollama list` e teste `ollama run`. Use `ollama serve` apenas se o serviço não estiver ativo. |
| Microfone não funciona | Confira permissões, dispositivo padrão e uso por outros aplicativos. |
| Voz de ativação não detectada | Confira o microfone e, se necessário, reduza `wake_word.threshold` de `0.5` para `0.3` em `config.yaml`; valores menores podem aumentar falsas ativações. |
| Interface não abre | Confira o terminal e tente http://localhost:7860 manualmente; erro de porta em uso pode indicar outra instância aberta. |

Após editar `config.yaml`, reinicie o Jarvis para garantir o recarregamento dos componentes.

---


A fully local, agentic AI voice assistant with wake word detection, screen vision, desktop control, and an Iron Man-inspired web UI. Runs entirely on your machine — no cloud required.

> Say **"Hey Jarvis"** → ask anything → Jarvis sees your screen, controls your apps, searches the web, and speaks back.

![Python 3.11](https://img.shields.io/badge/python-3.11-blue)
![Windows](https://img.shields.io/badge/platform-Windows-0078D6)
![License](https://img.shields.io/badge/license-MIT-green)

---

## Features

- **Voice-first interaction** — wake word detection ("Hey Jarvis"), natural speech input, streaming TTS responses
- **31 built-in tools** — desktop automation, screen reading (OCR), app control, web search, file ops, system commands
- **Agentic tool chaining** — up to 15 sequential tool calls per request (click → verify → scroll → click → done)
- **Screen vision** — reads your screen via OCR, finds UI elements by text, clicks buttons by coordinates
- **Multiple LLM providers** — Ollama (local), LM Studio, OpenAI, NVIDIA NIM, or any OpenAI-compatible API
- **Iron Man HUD web UI** — real-time chat, tool execution logs, provider management, settings, quick actions
- **Persistent memory** — remembers facts across sessions using SQLite + ChromaDB semantic search
- **Context management** — sliding window with auto-summarization to stay within token limits
- **One-click install** — `install.bat` sets up everything, `start.bat` launches

---
Coffe helps: https://buymeacoffee.com/azzren
---

## Quick Start (legacy scripts)

> For this fork, follow the Portuguese installation guide at the beginning of this README. The legacy scripts below use `.venv` and do not select Python 3.11 explicitly.

### Prerequisites

- **Windows 10/11**
- **Python 3.11 (64-bit)** — [python.org/downloads](https://www.python.org/downloads/)
- **Ollama** — [ollama.com/download](https://ollama.com/download) (for local LLM)

### 1. Install

```bash
git clone https://github.com/teonett/JarvisAi.git
cd JarvisAi
install.bat
```

This will:
1. Create a Python virtual environment (`.venv`)
2. Install all 18 dependencies
3. Download wake word ONNX models

### 2. Pull an LLM model

```bash
ollama pull qwen3:8b
```

Any Ollama model works — `qwen3:8b` is a good balance of speed and quality. Smaller options: `qwen3:4b`, `llama3.2:3b`. Larger: `qwen3:14b`, `llama3.1:8b`.

### 3. Launch

```bash
start.bat
```

Jarvis will:
- Start the voice listener (wake word detection)
- Open the web UI at **http://localhost:7860**
- Speak "Good morning. Jarvis online."

### 4. Use it

| Method | How |
|--------|-----|
| **Voice** | Say **"Hey Jarvis"** → wait for "Yes?" → speak your command |
| **Web UI** | Type in the chat bar at the bottom → click Send |
| **Keyboard** | Press **F2** → type command in terminal → Enter |

---

## Architecture

```
┌─────────────┐     ┌──────────────┐     ┌──────────────┐
│  Wake Word  │────▶│   STT        │────▶│    LLM       │
│  (pyaudio)  │     │ (Whisper)    │     │ (Ollama/API) │
└─────────────┘     └──────────────┘     └──────┬───────┘
                                                │
┌─────────────┐     ┌──────────────┐     ┌──────▼───────┐
│   Web UI    │◀───▶│  Event Bus   │◀────│  Tool Router │
│  (FastAPI)  │     │ (WebSocket)  │     │  (31 tools)  │
└─────────────┘     └──────────────┘     └──────┬───────┘
                                                │
                    ┌──────────────┐     ┌──────▼───────┐
                    │   Memory     │     │    TTS       │
                    │(SQLite+Chroma│     │  (Kokoro)    │
                    └──────────────┘     └──────────────┘
```

**Pipeline:** Wake word → Record speech → Transcribe (Whisper) → LLM with tools → Speak response (Kokoro)

**All components are local.** No data leaves your machine unless you configure a cloud LLM provider.

---

## Tools (31)

### Desktop Automation & Vision
| Tool | Description |
|------|-------------|
| `read_screen` | Screenshot + OCR — returns all visible text with coordinates |
| `find_on_screen` | Find text/button on screen, returns (x, y) coordinates |
| `click_at` | Click at screen coordinates (left/right/double click) |
| `type_text` | Type text at current cursor position |
| `press_key` | Keyboard shortcuts (ctrl+c, alt+tab, win+d, etc.) |
| `scroll_screen` | Scroll up/down |
| `move_mouse` | Move cursor to coordinates |
| `focus_window` | Bring window to foreground by title |
| `get_open_windows` | List all open window titles |
| `media_control` | Play/pause/next/previous/mute media |
| `screenshot` | Save screenshot to Desktop |

### Apps & Web
| Tool | Description |
|------|-------------|
| `open_app` | Launch apps by name (30+ mapped: Chrome, Discord, VS Code, Spotify...) |
| `open_url` | Open URL in default browser |
| `kill_process` | Close a running process |
| `web_search` | Search DuckDuckGo, return top results |
| `fetch_page` | Fetch and extract text from a URL |
| `get_weather` | Current weather for any location |

### System Control
| Tool | Description |
|------|-------------|
| `set_volume` / `get_volume` | Control system volume (0-100%) |
| `set_brightness` | Screen brightness (laptops) |
| `get_system_info` | CPU, RAM, disk usage |
| `get_clipboard` / `set_clipboard` | Read/write clipboard |
| `show_notification` | Windows toast notification |
| `set_timer` | Countdown timer with notification |
| `lock_screen` | Lock workstation |
| `power_command` | Shutdown, restart, or sleep |

### Files & Code
| Tool | Description |
|------|-------------|
| `read_file` / `write_file` | Read/write files (sandboxed to allowed paths) |
| `list_files` | List directory contents |
| `run_python` | Execute Python code in sandbox (10s timeout) |
| `delegate_task` | Send subtask to local LLM for parallel processing |

---

## Web UI

The web interface runs at `http://localhost:7860` and provides:

- **Real-time chat** — voice and typed conversations displayed together
- **Arc Reactor status** — animated indicator (listening / thinking / speaking / idle)
- **Tool execution panel** — see every tool call and result as it happens
- **Live event feed** — full timeline of all events
- **Quick actions** — one-click buttons for weather, screenshot, system info, etc.
- **Provider management** — add, edit, delete, and switch LLM providers
- **Settings** — configure TTS voice/speed, STT model, wake word threshold, LLM temperature

---

## Configuration

All settings are in `config.yaml`. You can also change most of them from the web UI's Config tab.

### Adding a Cloud LLM Provider

Open the web UI → Config tab → **Add Provider**, or edit `config.yaml`:

```yaml
llm:
  active_provider: nvidia   # Switch to this provider
  providers:
    ollama:
      type: ollama
      label: Ollama (Local)
      model: qwen3:8b
    nvidia:
      type: openai
      label: NVIDIA NIM
      model: meta/llama-3.1-70b-instruct
      base_url: https://integrate.api.nvidia.com/v1
      api_key: nvapi-xxxx
    lmstudio:
      type: openai
      label: LM Studio
      model: local-model
      base_url: http://localhost:1234/v1
```

Any OpenAI-compatible API works (LM Studio, vLLM, Together AI, Groq, etc.)

### STT Options

```yaml
stt:
  model: small      # tiny (fastest) | base | small (default) | medium | large-v3 (best)
  device: cuda      # cuda (GPU) or cpu — auto-falls back to CPU if CUDA fails
  compute_type: int8
```

### TTS Options

```yaml
tts:
  voice: pm_alex    # Brazilian Portuguese voice
  lang_code: p      # Brazilian Portuguese
  speed: 1.0        # Speech speed (0.5 = slow, 2.0 = fast)
```

---

## Controls

| Input | Action |
|-------|--------|
| **"Hey Jarvis"** (idle) | Wake up — starts listening for your command |
| **"Hey Jarvis"** (busy) | Stop — interrupts current task/speech |
| **"Stop"** / **"Cancel"** / **"Shut up"** | Voice stop command (after wake word) |
| **Esc** | Keyboard abort — stops everything instantly |
| **F2** | Type a command in the terminal |
| **Insert** | Toggle mute (TTS on/off) |
| **ABORT button** (web UI) | Stop current task |
| **"stop"** (typed in web chat) | Stop current task |

---

## Project Structure

```
jarvis/
├── config.yaml          # All configuration
├── install.bat          # One-click installer
├── start.bat            # Launcher
├── requirements.txt     # Python dependencies
│
├── jarvis/
│   ├── main.py          # Core orchestrator — pipeline, abort, events
│   ├── wake.py          # Wake word detection (openWakeWord)
│   ├── stt.py           # Speech-to-text (faster-whisper)
│   ├── tts.py           # Text-to-speech (Kokoro)
│   ├── web.py           # Web UI server (FastAPI + WebSocket)
│   ├── context.py       # Sliding window context manager
│   ├── memory.py        # Long-term memory (SQLite + ChromaDB)
│   ├── llm.py           # Internal LLM calls (summarization)
│   ├── static/
│   │   └── index.html   # Iron Man HUD web interface
│   └── tools/
│       ├── router.py    # Tool registry and dispatch
│       ├── desktop.py   # Screen/mouse/keyboard automation
│       ├── app_control.py  # App launching, URL opening
│       ├── web_search.py   # DuckDuckGo, weather, page fetch
│       ├── system.py    # Volume, brightness, clipboard, power
│       ├── file_ops.py  # Sandboxed file read/write/list
│       ├── code_exec.py # Python code execution sandbox
│       └── subagent.py  # Task delegation to local LLM
│
└── tests/               # Unit tests (pytest)
```

---

## Troubleshooting

| Issue | Fix |
|-------|-----|
| `cublas64_12.dll not found` | Normal on systems without CUDA toolkit. STT auto-falls back to CPU. |
| Wake word not detecting | Lower `threshold` in config (try 0.3). Check your mic is set as default input. |
| No sound from Jarvis | Check `is_muted` state (Insert key toggles). Check system audio output. |
| STT recording hangs | Mic conflict resolved — wake word mic auto-pauses during recording. |
| Ollama connection error | Make sure Ollama is running (`ollama serve`). Check `base_url` in config. |
| Web UI not updating | Open browser console (F12) — check for `[WS]` log messages. Refresh page. |

---

## Tech Stack

| Component | Technology |
|-----------|------------|
| Wake Word | [openWakeWord](https://github.com/dscripka/openWakeWord) (ONNX) |
| Speech-to-Text | [faster-whisper](https://github.com/SYSTRAN/faster-whisper) |
| Text-to-Speech | [Kokoro-82M](https://huggingface.co/hexgrad/Kokoro-82M) |
| LLM | [Ollama](https://ollama.com/) / OpenAI-compatible APIs |
| Memory | SQLite + [ChromaDB](https://www.trychroma.com/) |
| Web UI | [FastAPI](https://fastapi.tiangolo.com/) + WebSocket |
| Desktop Control | [PyAutoGUI](https://pyautogui.readthedocs.io/) + PowerShell OCR |

---

## License

MIT

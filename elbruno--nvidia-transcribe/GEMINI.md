## nvidia-transcribe

> This toolkit provides local audio transcription using NVIDIA ASR models via the NeMo framework. It offers five scenarios with shared patterns for different use cases.

# Copilot Instructions for NVIDIA ASR Transcription Toolkit

## Project Overview

This toolkit provides local audio transcription using NVIDIA ASR models via the NeMo framework. It offers five scenarios with shared patterns for different use cases.

## Repository Structure

```
/
├── README.md                    # Main readme (only doc in root)
├── requirements.txt             # Shared dependencies
├── fix_lhotse.py               # Post-install compatibility fix
├── transcribe.py               # Legacy script (backward compatibility)
├── docs/                       # All documentation except main README
│   ├── plans/               # Implementation plans (plan_YYMMDD_HHMM.md)
│   ├── PLAN.md
│   ├── QUICKREF.md
│   ├── USAGE_EXAMPLES.md
│   └── IMPLEMENTATION_SUMMARY.md
├── utils/                      # Environment validation tools
│   ├── check_environment.py   # Validates Python, PyTorch, CUDA, dependencies
│   ├── check_models.py        # Shows model download status
│   └── README.md
├── scenario1/                  # Simple CLI transcription
│   ├── transcribe.py
│   ├── simple-transcribe.py   # Simplified transcription script
│   └── README.md
├── scenario2/                  # Interactive menu transcription
│   ├── transcribe.py
│   └── README.md
├── scenario3/                  # Multilingual transcription
│   ├── transcribe.py
│   └── README.md
├── scenario4/                  # Client-server architecture
│   ├── server/                # Python FastAPI server
│   │   ├── app.py
│   │   ├── nvidia_asr_monitor.py
│   │   ├── requirements.txt
│   │   ├── requirements-windows.txt
│   │   ├── Dockerfile
│   │   ├── setup-venv.ps1/.sh
│   │   └── README.md
│   ├── clients/
│   │   ├── console/          # C# console client
│   │   └── webapp/           # Blazor web app client
│   ├── AppHost/              # .NET Aspire orchestration
│   ├── ServiceDefaults/      # Shared Aspire service defaults
│   ├── Directory.Build.props # Shared MSBuild properties
│   ├── NvidiaTranscribe.slnx # Solution file
│   ├── test_server.py        # Server integration test
│   ├── README.md             # Only doc in scenario root
│   └── docs/                 # All other scenario docs
│       ├── QUICKREF.md
│       ├── USAGE_EXAMPLES.md
│       ├── ARCHITECTURE.md
│       ├── AZURE_DEPLOYMENT.md
│       ├── GPU_SETUP_GUIDE.md
│       ├── STRUCTURE.md
│       ├── TESTING_CHECKLIST.md
│       ├── APPINSIGHTS_QUICKSTART.md
│       ├── APPLICATION_INSIGHTS.md
│       ├── IMPLEMENTATION_COMPLETE.md
│       ├── IMPLEMENTATION_NOTES.md
│       ├── IMPLEMENTATION_SUMMARY.md
│       ├── IMPLEMENTATION_SUMMARY_SCENARIO4_ENHANCEMENTS.md
│       └── OPTION2_IMPLEMENTATION.md
├── scenario5/                  # Voice Agent (ASR + TTS + LLM)
│   ├── app.py                 # FastAPI server with WebSocket
│   ├── requirements.txt       # Scenario-specific dependencies
│   ├── README.md
│   ├── static/
│   │   └── index.html         # Browser-based voice UI
│   └── pynini_stub/           # Windows TTS compatibility
│       ├── setup.py
│       └── pynini/__init__.py
└── output/                     # Shared output directory
```

### Structure Conventions

- **Root README only**: Main `README.md` stays in repo root; all other docs go in `docs/`
- **Scenario folders**: Each scenario has its own folder with `transcribe.py` and `README.md` at the root; all other documentation goes in a `docs/` subfolder within the scenario
- **Shared resources**: `requirements.txt`, `fix_lhotse.py`, and `output/` remain in root (shared across scenarios)
- **Utils folder**: `utils/` contains environment validation scripts (`check_environment.py`, `check_models.py`)
- **Legacy support**: Root `transcribe.py` maintained for backward compatibility
- **No audio files in repo**: Audio files (`.mp3`, `.wav`, `.flac`) are gitignored; users provide their own
- **Plans**: All implementation plans are saved in `docs/plans/` with the naming convention `plan_YYMMDD_HHMM.md` (e.g., `plan_260209_1008.md` for a plan created on 2026-02-09 at 10:08). Plans capture feature proposals, architecture decisions, and implementation steps before work begins.

## Architecture & Scenarios

| Folder | Model | Use Case |
|--------|-------|----------|
| `scenario1/` | Parakeet (English) | CLI for single file transcription |
| `scenario2/` | Parakeet (English) | Interactive menu to select from local audio files |
| `scenario3/` | Canary-1B (Multilingual) | Language-specific transcription (es, en, de, fr) |
| `scenario4/` | Parakeet + Canary | Client-server architecture with REST API, .NET Aspire orchestration |
| `scenario5/` | Parakeet + FastPitch + HiFiGAN + TinyLlama | Real-time voice agent with ASR, TTS, and Smart Mode LLM |

Root `transcribe.py` is the original script (same as scenario2) - kept for backward compatibility.

## Critical Constraints

- **Python 3.10-3.12 only** - Python 3.13 is NOT supported due to NeMo/lhotse incompatibility
- **Install PyTorch with CUDA first** - Before `pip install -r requirements.txt` to ensure GPU support
- **Run `python fix_lhotse.py` after installation** - Patches lhotse for PyTorch 2.10+ compatibility
- **Audio max length: 24 minutes** - Model limitation per file
- Canary-1B license is **non-commercial (CC-BY-NC-4.0)** - Parakeet allows commercial use

## Code Patterns

### Shared Function Pattern (all scenarios follow this)

```python
# 1. Audio conversion (MP3/FLAC → 16kHz mono WAV)
temp_wav = convert_to_wav(audio_path)  # Uses librosa + soundfile

# 2. Model loading
import nemo.collections.asr as nemo_asr
asr_model = nemo_asr.models.ASRModel.from_pretrained(MODEL_NAME)

# 3. Transcription with timestamp fallback
output = asr_model.transcribe([str(audio_path)], timestamps=True)
text = output[0].text
segments = output[0].timestamp.get('segment', [])

# 4. Output generation
save_outputs(text, segments, audio_file, output_dir)  # Creates .txt and .srt
```

### Output File Naming

Files are saved to `output/` with format: `{YYYYMMDD_HHMMSS}_{original_name}.{txt|srt}`
For multilingual (scenario3): `{YYYYMMDD_HHMMSS}_{original_name}_{lang}.{txt|srt}`

### SRT Timestamp Format

```python
def seconds_to_srt_time(seconds: float) -> str:
    """Format: HH:MM:SS,mmm (comma before milliseconds per SRT spec)"""
```

## Adding New Features

When extending functionality:
1. Keep the 4-step pattern: convert → load → transcribe → save
2. Support the same audio formats: `.wav`, `.flac`, `.mp3` (via `AUDIO_EXTENSIONS`)
3. Include timestamp fallback (some models/configs may fail timestamp extraction)
4. Clean up temp WAV files in a `finally` block
5. Use `Path(__file__).parent.resolve() / "output"` for output directory

### Scenario 4 (Client-Server) Patterns

When extending the client-server architecture:
1. **Server**: Use FastAPI patterns with async/await
2. **Clients**: Follow REST API conventions with proper error handling
3. **CORS**: Enable for web clients, configure appropriately for production
4. **Docker**: Maintain multi-stage builds with CUDA support
5. **Azure**: Use Container Apps for serverless deployment

### Scenario 5 (Voice Agent) Patterns

When extending the voice agent:
1. **Server**: FastAPI + WebSocket for real-time audio streaming
2. **Models**: Lazy-loaded via `get_asr_model()`, `get_tts_models()`, `get_llm()` singletons
3. **Pipeline**: Audio in → ASR → (optional LLM) → TTS → Audio out
4. **Logs**: Broadcast via `/ws/logs` WebSocket for real-time monitoring
5. **Windows**: Use `pynini_stub/` to bypass pynini C++ dependency for NeMo TTS
6. **Dependencies**: Scenario 5 has its own `requirements.txt` (needs `nemo_toolkit[asr,tts]`, `transformers`, `bitsandbytes`)

**Server Pattern:**
```python
@app.post("/endpoint")
async def endpoint(background_tasks: BackgroundTasks, file: UploadFile = File(...)):
    # 1. Validate input
    # 2. Process file
    # 3. Schedule cleanup
    # 4. Return response
```

**Client Pattern (C#):**
```csharp
using var form = new MultipartFormDataContent();
using var fileContent = new StreamContent(fileStream);
form.Add(fileContent, "file", fileName);
var response = await client.PostAsync(url, form);
```

## Development Setup

```bash
py -3.12 -m venv venv              # Must use Python 3.12 (not 3.13)
venv\Scripts\activate               # Windows
pip install torch --index-url https://download.pytorch.org/whl/cu121  # GPU support
pip install -r requirements.txt
python fix_lhotse.py                # Required post-install fix
python utils/check_environment.py  # Validate setup
```

## Testing

No formal test suite. Manual testing:
```bash
python scenario1/transcribe.py test_short.mp3
python scenario2/transcribe.py  # Select from menu
python scenario3/transcribe.py test_short.mp3 en
```

Scenario 5 testing:
```bash
cd scenario5
python app.py  # Open http://localhost:8000 and use the voice UI
```

Outputs appear in `output/` directory with `.txt` and `.srt` files.

## Model Constants

```python
MODEL_NAME = "nvidia/parakeet-tdt-0.6b-v2"  # Scenarios 1 & 2 (English)
MODEL_NAME = "nvidia/canary-1b-v2"           # Scenario 3 default (Multilingual, supports canary-1b, canary-1b-v2, parakeet-1.1b)
TARGET_SAMPLE_RATE = 16000                   # Required for all models
```

---
> Source: [elbruno/nvidia-transcribe](https://github.com/elbruno/nvidia-transcribe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:gemini_md:2026-05-04 -->

# Local Multimodal Search with Embedding Gemma 2

Search your own PC by meaning: Obsidian notes, code and unnamed screenshots, fully local, no OCR and no cloud.

This repo is a **blueprint, not a codebase**. Paste [MASTER_PROMPT.md](MASTER_PROMPT.md) into your coding AI (Claude, Codex,
Gemini or a local model) and it builds a version that fits your folders, Python/Conda setup, chat server and GPU.

Video walkthrough: [YouTube @AlexFP16](https://www.youtube.com/@AlexFP16)

## What it builds

| Block | What it does |
|---|---|
| 1. Persistent daemon | FastAPI on `127.0.0.1:8766` keeps Embedding Gemma 2 warm in bfloat16, with a GPU lock and CUDA cache release |
| 2. Multimodal router | Notes, code and raw image pixels go into one 256-dimension (Matryoshka) ChromaDB collection |
| 3. Folder watchdog | Watches only the folders you choose, with a 1 s debounce and a float-tolerant mtime check (no re-index loops) |
| 4. Image bypass + citation gate | Image hits return instantly without an LLM. Text answers from your local chat model must cite sources `[1]`, `[2]` or get rejected |
| 5. PowerShell CLI | `ask "query" --filter image -o` one-liners, plus an interactive shell with a `/` command menu |

## What it used on my machine

- Windows, Conda, Embedding Gemma 2 (safetensors from Hugging Face; a GGUF for llama.cpp also exists)
- Gemma 4 E4B QAT (Q4_0, Q8_0 KV cache, 32K context) in `llama-server` for cited answers (optional)
- About 6 GB of VRAM for the whole stack (about 8 GB with Faster Whisper also loaded), and 500–700 MB of system RAM for the daemon
- Embedding Gemma 2 is small enough to run the search daemon on CPU only

This reflects how the setup worked on my machine. Your coding AI should adapt the paths, ports and GPU settings to yours.

# Master Prompt: Local Multimodal Search with Embedding Gemma 2

Paste everything below the line into your coding AI (Claude, Codex, Gemini or a local coding model). Before you send it, add:

- the folders you want indexed and watched
- your chat server: llama.cpp `llama-server`, LM Studio or Ollama (and its port)
- your GPU and VRAM, or "CPU only"

---

> Build a local, zero-cloud multimodal semantic search and RAG daemon in Python + PowerShell for Windows powered by `google/embeddinggemma-2` (`sentence-transformers`) and `chromadb`, paired with an optional local OpenAI-compatible chat server (`llama-server` / LM Studio / Ollama) for grounded answers.
>
> Create 4 modular files (`semdaemon.py`, `local_answer.py`, `ask.py`, and `ask_shell.py`) plus `start-semdaemon.ps1` and `stop-semdaemon.ps1` using these exact 5 architectural rules:
>
> 1. **Persistent FastAPI Daemon & GPU Lock (`semdaemon.py` on `127.0.0.1:8766`):** Load `google/embeddinggemma-2` in `bfloat16` (if CUDA supports it, else `float32`) with `config_kwargs={"audio_config": None}`. Wrap all model encoding and ChromaDB writes inside a single `threading.Lock()` (`gpu_lock`) and call `torch.cuda.empty_cache()` after indexing and search calls so VRAM is immediately returned to Windows.
>
> 2. **256-Dim Matryoshka Multimodal Indexing (`local_multimodal_vault` in ChromaDB with cosine distance):** Truncate all embeddings to `truncate_dim=256` with `normalize_embeddings=True`. Support three modalities:
>
>    * `note` (`.md`, `.txt`, `.json`, `.yaml`, `.csv`): chunk text (6,000 chars, 600 overlap) and encode as `f"title: {filepath.name} | text: {chunk}"`.
>
>    * `code` (`.py`, `.ps1`, `.js`, `.ts`, `.cpp`, `.rs`, `.go`, `.sql`): encode the same way, and use `prompt_name="CodeRetrieval"` during code queries (`prompt_name="SearchQuery"` for general queries).
>
>    * `image` (`.png`, `.jpg`, `.jpeg`, `.webp`, `.bmp`): encode raw pixels directly via `model.encode({"image": canonical_filepath}, truncate_dim=256, normalize_embeddings=True)`.
>
> 3. **Debounced Real-Time Watchdog & Windows Edge-Case Guards:** Use `watchdog` with a `1.0s` debounce timer per file so rapid saves in Obsidian or IDEs don't spam the GPU. **Critical fixes to include:**
>
>    * Compare stored ChromaDB `mtime` floats using `math.isclose(stored_mtime, current_mtime, rel_tol=0, abs_tol=1e-6)` so float serialization shifts (\~0.24 microseconds) never cause infinite re-indexing loops.
>
>    * Paginate ChromaDB metadata scans in batches of `2000` (`limit=2000, offset=offset`) so large vaults never hit SQLite's parameter limit.
>
>    * Normalize Windows `\\?\` extended-length paths so delete events cleanly purge matching vectors.
>
> 4. **Smart Image Bypass & Regex Citation Gate (`local_answer.py`):** In `/answer` mode, if `modality == "image"` or the #1 result is an image, bypass the chat LLM completely and return the image matches directly in milliseconds. For text/code passages, query the local chat server (`/v1/chat/completions?autoload=false` at `temperature=0`) and validate with regex (`r"\[(\d+(?:\s*,\s*\d+)*)\]"`) that every answer cites valid source numbers `[1]`, `[2]` from the retrieved chunks (or states `"I couldn't find that in the retrieved notes."`).
>
> 5. **Fast PowerShell CLI & Interactive `/` Shell (`ask.py` + `ask_shell.py`):** Provide a zero-dependency CLI supporting `ask "query"`, `--filter [all|image|note|code]`, `-o` / `--open` (using `os.startfile` on Windows to immediately open the #1 match), `--index "FOLDER"`, `--purge "FOLDER" --dry-run`, and an interactive `prompt_toolkit` shell (`ask` or `ask /`) with a persistent bottom filter toolbar and a `/` popup command completer (`/images`, `/notes`, `/code`, `/open`, `/status`, `/exit`).


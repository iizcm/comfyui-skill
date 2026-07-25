# comfyui-skill

<div align="center">
  <img src="https://img.shields.io/badge/License-MIT-blue.svg" alt="License: MIT">
  <img src="https://img.shields.io/badge/Universal_AI_Agent_Skill-orange?style=for-the-badge" alt="Universal AI Agent Skill">
  <img src="https://img.shields.io/badge/Category-ai-art-purple?style=for-the-badge" alt="Category: ai-art">
</div>

## English

### Overview
The `comfyui` skill enables Hermes Agent to generate images, video, audio, and 3D content using ComfyUI. It leverages the official `comfy-cli` for setup and lifecycle management (installing, launching, managing nodes/models) and directly interacts with ComfyUI's REST/WebSocket API for executing workflows with parameter injection and monitoring. This skill provides a robust pipeline for generative AI tasks, supporting various Stable Diffusion models (1.5, XL, Flux, SD3), advanced pipelines like ControlNet and inpainting, and multimedia generation.

### When to Use
Use this skill when:
*   Generating images with Stable Diffusion, SDXL, Flux, SD3, etc.
*   Running a specific ComfyUI workflow file.
*   Chaining generative steps (e.g., text-to-image → upscale → face restore).
*   Performing advanced tasks such as ControlNet, inpainting, or image-to-image transformations.
*   Managing the ComfyUI queue, checking models, or installing custom nodes.
*   Generating video, audio, or 3D content using tools like AnimateDiff, Hunyuan, or Wan T2V.

### Quick Start

1.  **Detect Environment**:
    ```bash
    command -v comfy >/dev/null 2>&1 && echo "comfy-cli: installed"
    curl -s http://127.0.0.1:8188/system_stats 2>/dev/null && echo "server: running"
    python3 scripts/hardware_check.py # Check GPU/VRAM/disk
    ```
2.  **One-line Health Check**:
    ```bash
    python3 scripts/health_check.py
    # Returns JSON indicating comfy_cli path, server reachability, checkpoint status, and smoke test results.
    ```
3.  **Core Workflow**:
    *   **Get Workflow JSON in API Format**: Ensure workflows are in API format (each node has `class_type`). Export from ComfyUI web UI or use examples from `workflows/` directory.
    *   **See Controllable Parameters**:
        ```bash
        python3 scripts/extract_schema.py workflow_api.json --summary-only
        # Returns: {"parameter_count": 12, "has_negative_prompt": true, "has_seed": true, ...}
        ```
    *   **Run with Parameters**:
        ```bash
        # Local
        python3 scripts/run_workflow.py \
          --workflow workflow_api.json \
          --args '{"prompt": "a beautiful sunset over mountains", "seed": -1, "steps": 30}' \
          --output-dir ./outputs

        # Cloud (requires COMFY_CLOUD_API_KEY environment variable)
        export COMFY_CLOUD_API_KEY="comfyui-..."
        python3 scripts/run_workflow.py \
          --workflow workflow_api.json \
          --args '{"prompt": "..."}' \
          --host https://cloud.comfy.org \
          --output-dir ./outputs
        ```
    *   **Present Results**: Scripts emit JSON to stdout detailing output files.

### Key Features and Workflow

#### Architecture
The skill operates in two layers:
1.  **comfy-cli**: For setup, server lifecycle, custom node and model management (`comfy install`, `launch`, `stop`, `node`, `model`).
2.  **REST/WebSocket API + Skill Scripts**: For workflow execution, parameter injection, monitoring, and output download (`run_workflow.py`, `run_batch.py`, `ws_monitor.py`).

#### Setup & Onboarding
**Always ask first: Local vs Comfy Cloud?**
*   **Comfy Cloud**: Hosted, zero install, API key required (paid subscription for execution). Ideal for users without capable GPUs.
*   **Local**: Free, but requires specific hardware (NVIDIA GPU ≥6GB VRAM, AMD GPU with ROCm, or Apple Silicon M1+ with ≥16GB unified memory).

**Hardware Check (for local installation)**:
```bash
python3 scripts/hardware_check.py --json [--check-pytorch]
```
The script provides a `verdict` (`ok`, `marginal`, `cloud`) and `notes` to guide installation.

**Installation Paths**:
*   **Path A: Comfy Cloud**: Sign up, get API key, set `COMFY_CLOUD_API_KEY` env var, then use `--host https://cloud.comfy.org` with `run_workflow.py`.
*   **Path B: ComfyUI Desktop**: One-click installer for Windows/macOS (Beta).
*   **Path C: ComfyUI Portable**: Windows only, download and run `run_nvidia_gpu.bat`.
*   **Path D: comfy-cli (Recommended for Agents)**: Install `pipx install comfy-cli`, then `comfy install --nvidia/--amd/--m-series/--cpu`, then `comfy launch --background`.
*   **Path E: Manual Install**: For unsupported hardware.

**Post-Install**:
*   **Download Models**: `comfy model download --url <url> --relative-path models/checkpoints`
*   **Install Custom Nodes**: `comfy node install <name>`
*   **Verify**: `python3 scripts/health_check.py` and `python3 scripts/check_deps.py my_workflow.json`

#### Image Upload
Use `--input-image image=./photo.png` with `run_workflow.py` to automatically upload and reference images in workflows (e.g., for img2img, inpainting).

#### Cloud Specifics
*   **Base URL**: `https://cloud.comfy.org`
*   **Auth**: `X-API-Key` header.
*   **Output Download**: Scripts handle 302 redirects for signed URLs.
*   **Endpoint Differences**: Some endpoints (`/api/object_info`, `/api/queue`, `/api/userdata`) may be 403 on free tiers.
*   **Concurrent Jobs**: Free/Standard: 1, Creator: 3, Pro: 5. Use `run_batch.py --parallel N`.

#### Queue & System Management
*   **Local**: Use `curl` commands for `/queue`, `/interrupt`, `/free`.
*   **Cloud**: `python3 scripts/fetch_logs.py --tail-queue --host https://cloud.comfy.org`

### Pitfalls
1.  **API Format Required**: Workflows must be in API format. Scripts will warn if editor format is detected.
2.  **Server Must Be Running**: Ensure `comfy launch --background` is active.
3.  **Exact Model Names**: Model names are case-sensitive and include extensions. Use `comfy model list`.
4.  **Missing Custom Nodes**: `check_deps.py` and `auto_fix_deps.py` help identify and install missing nodes.
5.  **Working Directory**: `comfy-cli` needs to auto-detect the workspace or be explicitly configured with `--workspace`.
6.  **Cloud Free-Tier API Limits**: Be aware of 403 errors on certain endpoints.
7.  **Video/Audio Workflow Timeout**: Default timeout increases for multimedia workflows.
8.  **Path Traversal in Output Filenames**: Scripts protect against arbitrary paths.
9.  **Workflow JSON is Arbitrary Code**: Inspect workflows from untrusted sources.
10. **Auto-Randomized Seed**: Use `seed: -1` or `--randomize-seed`.
11. **`tracking` Prompt**: Use `comfy --skip-prompt tracking disable` to avoid interactive prompts.

## Bahasa Indonesia

### Gambaran Umum
Skill `comfyui` memungkinkan Hermes Agent untuk menghasilkan gambar, video, audio, dan konten 3D menggunakan ComfyUI. Skill ini memanfaatkan `comfy-cli` resmi untuk pengaturan dan manajemen siklus hidup (menginstal, meluncurkan, mengelola node/model) dan berinteraksi langsung dengan REST/WebSocket API ComfyUI untuk mengeksekusi alur kerja dengan injeksi parameter dan pemantauan. Skill ini menyediakan pipeline yang kuat untuk tugas AI generatif, mendukung berbagai model Stable Diffusion (1.5, XL, Flux, SD3), pipeline canggih seperti ControlNet dan inpainting, serta generasi multimedia.

### Kapan Menggunakan
Gunakan skill ini ketika:
*   Menghasilkan gambar dengan Stable Diffusion, SDXL, Flux, SD3, dll.
*   Menjalankan file alur kerja ComfyUI tertentu.
*   Merangkai langkah-langkah generatif (misalnya, teks-ke-gambar → upscale → pemulihan wajah).
*   Melakukan tugas-tugas canggih seperti ControlNet, inpainting, atau transformasi gambar-ke-gambar.
*   Mengelola antrean ComfyUI, memeriksa model, atau menginstal node kustom.
*   Menghasilkan konten video, audio, atau 3D menggunakan alat seperti AnimateDiff, Hunyuan, atau Wan T2V.

### Mulai Cepat

1.  **Deteksi Lingkungan**:
    ```bash
    command -v comfy >/dev/null 2>&1 && echo "comfy-cli: terinstal"
    curl -s http://127.0.0.1:8188/system_stats 2>/dev/null && echo "server: berjalan"
    python3 scripts/hardware_check.py # Periksa GPU/VRAM/disk
    ```
2.  **Pemeriksaan Kesehatan Satu Baris**:
    ```bash
    python3 scripts/health_check.py
    # Mengembalikan JSON yang menunjukkan jalur comfy_cli, jangkauan server, status checkpoint, dan hasil uji asap.
    ```
3.  **Alur Kerja Inti**:
    *   **Dapatkan Workflow JSON dalam Format API**: Pastikan alur kerja dalam format API (setiap node memiliki `class_type`). Ekspor dari UI web ComfyUI atau gunakan contoh dari direktori `workflows/`.
    *   **Lihat Parameter yang Dapat Dikontrol**:
        ```bash
        python3 scripts/extract_schema.py workflow_api.json --summary-only
        # Mengembalikan: {"parameter_count": 12, "has_negative_prompt": true, "has_seed": true, ...}
        ```
    *   **Jalankan dengan Parameter**:
        ```bash
        # Lokal
        python3 scripts/run_workflow.py \
          --workflow workflow_api.json \
          --args '{"prompt": "pemandangan matahari terbenam yang indah di atas gunung", "seed": -1, "steps": 30}' \
          --output-dir ./outputs

        # Cloud (membutuhkan variabel lingkungan COMFY_CLOUD_API_KEY)
        export COMFY_CLOUD_API_KEY="comfyui-..."
        python3 scripts/run_workflow.py \
          --workflow workflow_api.json \
          --args '{"prompt": "..."}' \
          --host https://cloud.comfy.org \
          --output-dir ./outputs
        ```
    *   **Sajikan Hasil**: Skrip mengeluarkan JSON ke stdout yang merinci file output.

### Fitur Utama dan Alur Kerja

#### Arsitektur
Skill ini beroperasi dalam dua lapisan:
1.  **comfy-cli**: Untuk pengaturan, siklus hidup server, manajemen node dan model kustom (`comfy install`, `launch`, `stop`, `node`, `model`).
2.  **REST/WebSocket API + Skrip Skill**: Untuk eksekusi alur kerja, injeksi parameter, pemantauan, dan unduhan output (`run_workflow.py`, `run_batch.py`, `ws_monitor.py`).

#### Pengaturan & Onboarding
**Selalu tanyakan terlebih dahulu: Lokal vs Comfy Cloud?**
*   **Comfy Cloud**: Dihosting, tanpa instalasi, kunci API diperlukan (langganan berbayar untuk eksekusi). Ideal untuk pengguna tanpa GPU yang mumpuni.
*   **Lokal**: Gratis, tetapi membutuhkan perangkat keras khusus (GPU NVIDIA ≥6GB VRAM, GPU AMD dengan ROCm, atau Apple Silicon M1+ dengan memori terpadu ≥16GB).

**Pemeriksaan Perangkat Keras (untuk instalasi lokal)**:
```bash
python3 scripts/hardware_check.py --json [--check-pytorch]
```
Skrip ini memberikan `verdict` (`ok`, `marginal`, `cloud`) dan `notes` untuk memandu instalasi.

**Jalur Instalasi**:
*   **Jalur A: Comfy Cloud**: Daftar, dapatkan kunci API, atur variabel lingkungan `COMFY_CLOUD_API_KEY`, lalu gunakan `--host https://cloud.comfy.org` dengan `run_workflow.py`.
*   **Jalur B: ComfyUI Desktop**: Penginstal satu klik untuk Windows/macOS (Beta).
*   **Jalur C: ComfyUI Portable**: Hanya Windows, unduh dan jalankan `run_nvidia_gpu.bat`.
*   **Jalur D: comfy-cli (Direkomendasikan untuk Agen)**: Instal `pipx install comfy-cli`, lalu `comfy install --nvidia/--amd/--m-series/--cpu`, lalu `comfy launch --background`.
*   **Jalur E: Instalasi Manual**: Untuk perangkat keras yang tidak didukung.

**Pasca Instalasi**:
*   **Unduh Model**: `comfy model download --url <url> --relative-path models/checkpoints`
*   **Instal Node Kustom**: `comfy node install <nama>`
*   **Verifikasi**: `python3 scripts/health_check.py` dan `python3 scripts/check_deps.py my_workflow.json`

#### Unggah Gambar
Gunakan `--input-image image=./photo.png` dengan `run_workflow.py` untuk secara otomatis mengunggah dan mereferensikan gambar dalam alur kerja (misalnya, untuk img2img, inpainting).

#### Spesifik Cloud
*   **URL Dasar**: `https://cloud.comfy.org`
*   **Otentikasi**: `X-API-Key` header.
*   **Unduhan Output**: Skrip menangani pengalihan 302 untuk URL yang ditandatangani.
*   **Perbedaan Endpoint**: Beberapa endpoint (`/api/object_info`, `/api/queue`, `/api/userdata`) mungkin mengembalikan 403 pada tingkatan gratis.
*   **Pekerjaan Bersamaan**: Gratis/Standar: 1, Pembuat: 3, Pro: 5. Gunakan `run_batch.py --parallel N`.

#### Manajemen Antrean & Sistem
*   **Lokal**: Gunakan perintah `curl` untuk `/queue`, `/interrupt`, `/free`.
*   **Cloud**: `python3 scripts/fetch_logs.py --tail-queue --host https://cloud.comfy.org`

### Jebakan
1.  **Format API Diperlukan**: Alur kerja harus dalam format API. Skrip akan memperingatkan jika format editor terdeteksi.
2.  **Server Harus Berjalan**: Pastikan `comfy launch --background` aktif.
3.  **Nama Model yang Tepat**: Nama model peka huruf besar/kecil dan termasuk ekstensi. Gunakan `comfy model list`.
4.  **Node Kustom yang Hilang**: `check_deps.py` dan `auto_fix_deps.py` membantu mengidentifikasi dan menginstal node yang hilang.
5.  **Direktori Kerja**: `comfy-cli` perlu mendeteksi ruang kerja secara otomatis atau dikonfigurasi secara eksplisit dengan `--workspace`.
6.  **Batas API Tingkat Gratis Cloud**: Waspadai kesalahan 403 pada endpoint tertentu.
7.  **Batas Waktu Alur Kerja Video/Audio**: Batas waktu default meningkat untuk alur kerja multimedia.
8.  **Penelusuran Jalur di Nama File Output**: Skrip melindungi dari jalur arbitrer.
9.  **Workflow JSON adalah Kode Arbitrer**: Periksa alur kerja dari sumber yang tidak tepercaya.
10. **Seed yang Diacak Otomatis**: Gunakan `seed: -1` atau `--randomize-seed`.
11. **Prompt `tracking`**: Penggunaan `comfy` pertama kali mungkin meminta analitik. Gunakan `comfy --skip-prompt tracking disable` untuk menghindari prompt interaktif.

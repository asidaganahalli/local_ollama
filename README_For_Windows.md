## local_ollama (Windows 10 Pro)
    A complete guide to set up a lightweight, local AI assistant on Windows 10 Pro using Ollama and Llama 3.2 1B (4‑bit quantized) with optimized system parameters for 8 GB RAM PCs.

## Prerequisites
    Hardware: Windows 10 Pro, Intel CPU, Atleast 8 GB RAM
    Pre‑installed: PowerShell 7+, Administrator access
    Goal: Keep Ollama + model under 3 GB RAM
    Model: Llama 3.2 1B Instruct (4‑bit quantized, ~600 MB download)

## 1. Install Ollama on Windows
    Navigate to https://ollama.com/download.

    Download and run the Windows installer (OllamaSetup.exe).

    After installation, open PowerShell and verify:

    Open powershell & Run:
      ollama --version

    You should see something like: Ollama version 0.x.x.

    Ollama runs as a background service automatically. 
    
    To confirm:
      Open powershell & Run:
      Get-Service Ollama

    To manually control:

    Open powershell & Run:
      Start-Service Ollama
      Stop-Service Ollama
      Restart-Service Ollama

## 2. Configure Environment Variables for Memory Optimization
    Windows uses persistent system or user environment variables (instead of plist files on macOS). You can set them permanently via PowerShell.
    Run PowerShell as Administrator and execute:
        setx OLLAMA_FLASH_ATTENTION "1"
        setx OLLAMA_KV_CACHE_TYPE "q4_0"
        setx OLLAMA_NUM_THREAD "4"
        setx OLLAMA_CONTEXT_LENGTH "512"
        setx OLLAMA_BATCH "8"
        setx OLLAMA_GPU_LAYERS "0"
        setx OLLAMA_MAX_LOADED_MODELS "1"
        setx OLLAMA_KEEP_ALIVE "60"
        setx OLLAMA_MMAP "1"
        setx OLLAMA_NUM_PARALLEL "1"

    Then, restart your computer or restart the service:

    Open powershell & Run:
      Restart-Service Ollama

    To confirm they are loaded:
    Open powershell & Run:
      $env:OLLAMA_CONTEXT_LENGTH
      $env:OLLAMA_KV_CACHE_TYPE

    Expected values:
        Variable	Expected Value
        OLLAMA_CONTEXT_LENGTH	512
        OLLAMA_KV_CACHE_TYPE	q4_0
        OLLAMA_MAX_LOADED_MODELS	1
        OLLAMA_NUM_PARALLEL	1

## 3. What Each Parameter Does
    Variable	                    Purpose
    OLLAMA_FLASH_ATTENTION=1	  Enables flash attention for memory‑efficient performance.
    OLLAMA_KV_CACHE_TYPE=q4_0	  4‑bit quantized cache; lower RAM use with small quality trade‑off.
    OLLAMA_NUM_THREAD=4	        Caps CPU threads for smoother multitasking.
    OLLAMA_CONTEXT_LENGTH=512	  Reduces maximum tokens to control memory footprint.
    OLLAMA_BATCH=8	            Smaller batches keep memory steady under 3 GB.
    OLLAMA_GPU_LAYERS=0	        CPU‑only inference (safer for Intel integrated graphics).
    OLLAMA_MAX_LOADED_MODELS=1	Keeps only one model resident in memory.
    OLLAMA_KEEP_ALIVE=60	      Frees RAM after 60 seconds idle.
    OLLAMA_MMAP=1	              Loads model weights via memory‑mapped files for RAM savings.
    OLLAMA_NUM_PARALLEL=1	      Handles one request at a time; avoids CPU spikes.

## 4. Pull and Run Llama 3.2 1B
Download the model:
    Open powershell & Run:
      ollama pull llama3.2:1b

Test it:
    Open powershell & Run:
      ollama run llama3.2:1b "Explain recursion in two sentences."

## 5. Manage Ollama Service on Windows
Usmg PowerShell (Admin):

  # Start service (runs on boot)
    Start-Service Ollama
  
  # Stop to free memory
    Stop-Service Ollama
  
  # Restart after config changes
    Restart-Service Ollama
  
  # View status
    Get-Service Ollama

To make Ollama auto‑start on boot if it doesn’t already:
Open powershell & Run:
  Set-Service -Name "Ollama" -StartupType Automatic

#  6. Monitor Memory Usage
Open Task Manager → Details tab → ollama.exe.
While chatting with llama3.2:1b, expect:
    2 – 2.5 GB RAM usage peak
    CPU stable under 60–70%

If your system slows down:
    Lower OLLAMA_CONTEXT_LENGTH → 256
    Lower OLLAMA_BATCH → 4
    Then rerun the setx commands and restart the service.

## 7. End Result
You now have:
    Ollama running as a Windows service
    A 4‑bit Llama 3.2 1B model fully offline
    Optimized environment variables for 8 GB setups
    Consistent performance under 3 GB RAM
    Easy start/stop/restart controls with PowerShell

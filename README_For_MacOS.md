# local_ollama
A complete guide to set up a modern, local AI assistant on resource-constrained Intel Macs using Ollama and Llama 3.2 1B with custom memory optimizations.

# Prerequisites
MacBook : Intel CPU, 8GB RAM, macOS Ventura+
Pre Installed Apps: Homebrew, Ollama
Goal: Keep Ollama + model under 3GB RAM (leaves 5GB+ for other apps)
Model: Llama 3.2 1B Instruct (4-bit quantized, ~600MB download)

# 1. Install Ollama via Homebrew

    brew install ollama
    
    # Start service (runs in background)
    brew services start ollama
    
    # Verify running
    brew services list | grep ollama

# 2. Locate the Homebrew Ollama plist
    Homebrew manages Ollama as a LaunchAgent using a plist file. On Intel Macs, the “source” plist is usually here:
    
    ls /usr/local/opt/ollama/homebrew.mxcl.ollama.plist
    
    Homebrew also copies or symlinks this into your user LaunchAgents directory:
    
    ls ~/Library/LaunchAgents/homebrew.mxcl.ollama.plist
    
    The file in /usr/local/opt/ollama/ is the template Homebrew regenerates.
    
    The one in ~/Library/LaunchAgents/ is the active per‑user plist that launchd actually uses.
    
    You can edit either, but the one under /usr/local/opt/ollama/ may get overwritten when Ollama is upgraded via Homebrew. 
    
    That’s why we will add a separate env plist later for a more robust solution.

# 3. (Optional) Editing the Homebrew plist directly
    If you want to see or adjust the default plist that Homebrew uses for Ollama:
    
    Edit the user LaunchAgent plist at ~/Library/LaunchAgents/homebrew.mxcl.ollama.plist
    
    Inside, you’ll see a top‑level <dict> containing keys like Label, ProgramArguments, etc. 
    If you see an existing EnvironmentVariables block like:

    <key>EnvironmentVariables</key>
    <dict>
        <key>OLLAMA_FLASH_ATTENTION</key>
        <string>1</string>
        <key>OLLAMA_KV_CACHE_TYPE</key>
        <string>q8_0</string>
    </dict>

    you can extend or change it (for example, to use q4_0). However, any changes here can be lost when Homebrew regenerates the plist on upgrades or reinstall.
    
    That’s why the recommended approach is to keep your tuning in a separate local.ollama.env.plist that sets environment variables globally before Ollama starts.

# 5. Create an upgrade-proof LaunchAgent that sets Ollama environment variables before Homebrew starts the service.
    
    cd ~/Library/LaunchAgents (If it does not exist, create one mkdir -p ~/Library/LaunchAgents)

    # Paste this complete configuration (optimized for 8GB Intel Mac) into a file named "local.ollama.env.plist":
    
    <?xml version="1.0" encoding="UTF-8"?>
    <!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
    <plist version="1.0">
    <dict>
        <key>Label</key>
        <string>local.ollama.env</string>
        
        <key>ProgramArguments</key>
        <array>
            <string>/bin/sh</string>
            <string>-c</string>
            <string>
                launchctl setenv OLLAMA_FLASH_ATTENTION 1 &&
                launchctl setenv OLLAMA_KV_CACHE_TYPE q4_0 &&
                launchctl setenv OLLAMA_NUM_THREAD 4 &&
                launchctl setenv OLLAMA_CONTEXT_LENGTH 512 &&
                launchctl setenv OLLAMA_BATCH 8 &&
                launchctl setenv OLLAMA_GPU_LAYERS 0 &&
                launchctl setenv OLLAMA_MAX_LOADED_MODELS 1 &&
                launchctl setenv OLLAMA_KEEP_ALIVE 60 &&
                launchctl setenv OLLAMA_MMAP 1 &&
                launchctl setenv OLLAMA_NUM_PARALLEL 1
            </string>
        </array>
        
        <key>RunAtLoad</key>
        <true/>
        <key>KeepAlive</key>
        <false/>
    </dict>
    </plist>
    
    # Restart the Ollama Service
    brew services restart ollama
    
    # Confirm if the parameters are loaded in the environment 
    launchctl getenv OLLAMA_CONTEXT_LENGTH    # Should show: 512
    launchctl getenv OLLAMA_KV_CACHE_TYPE     # Should show: q4_0
    launchctl getenv OLLAMA_MAX_LOADED_MODELS # Should show: 1
    
    # Expected values:
    OLLAMA_CONTEXT_LENGTH → 512
    OLLAMA_KV_CACHE_TYPE → q4_0
    OLLAMA_MAX_LOADED_MODELS → 1
    OLLAMA_NUM_PARALLEL → 1

# 7. What each parameter does
    OLLAMA_FLASH_ATTENTION=1
    Enables flash attention for performance and memory efficiency.
    
    OLLAMA_KV_CACHE_TYPE=q4_0
    4‑bit quantized key/value cache; minimizes RAM usage at slight quality cost for very long contexts.
    
    OLLAMA_NUM_THREAD=4
    Limits CPU threads used by the model; reduces CPU contention and heat.
    
    OLLAMA_CONTEXT_LENGTH=512
    Maximum tokens of context; 512 is conservative and reduces memory versus 4k+ defaults.
    
    OLLAMA_BATCH=8
    Fewer tokens processed at once, lowering peak memory usage.
    
    OLLAMA_GPU_LAYERS=0
    Forces CPU‑only inference (good on Intel Macs with no useful GPU).
    
    OLLAMA_MAX_LOADED_MODELS=1
    Only one model can be in memory at a time.
    
    OLLAMA_KEEP_ALIVE=60
    Unloads idle models after 60 seconds to free RAM.
    
    OLLAMA_MMAP=1
    Uses memory‑mapped files for model weights; reduces RAM at the cost of some disk I/O.
    
    OLLAMA_NUM_PARALLEL=1
    Only one request processed at a time; avoids spikes in memory and CPU.


# 8. Pull and run Llama 3.2 1B (offline)

    # One-time download
    ollama pull llama3.2:1b
    
    # Run a quick test
    ollama run llama3.2:1b "Explain what recursion is in 2 sentences."

# 9. Start/stop Ollama service
      # To Start service on boot / login
      brew services start ollama
      
      #To Stop service (frees RAM)
      brew services stop ollama
      
      # Restart (after config changes)
      brew services restart ollama
      
      # Check status
      brew services list | grep ollama

# 10. Monitoring RAM usage
      Open Activity Monitor → Memory:
      Look for the ollama process.
      While chatting with llama3.2:1b using the above settings, expect:
      Roughly ~2–2.5 GB RAM usage for ollama at peak
      macOS memory pressure indicator should stay green
      If memory pressure goes yellow or red, you can further reduce:
      OLLAMA_CONTEXT_LENGTH from 512 → 256
      OLLAMA_BATCH from 8 → 4
      Update the local.ollama.env.plist, reload it, and restart the service.

# 11. End result
You now have:
Ollama installed via Homebrew
A 4‑bit quantized Llama 3.2 1B model running fully offline
Aggressive RAM control tuned for an 8GB Intel MacBook Air
Clean start/stop service commands
A reusable local.ollama.env.plist that survives Homebrew upgrades

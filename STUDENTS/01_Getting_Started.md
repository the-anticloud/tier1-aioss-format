# AIOSS Format — Student Getting Started Guide

## What You'll Build

A tamper-evident ledger for your own AI experiments. Every LLM call you make will be logged with a cryptographic hash chain — you can prove exactly what was asked, when, and that no one changed the record.

## Prerequisites

- Python 3.10+
- Rust toolchain (for building from source) OR pre-built binary

## Option A: Pre-Built Binary (fastest)

```bash
# Linux/macOS
curl -L https://github.com/kleinnner/Anticloud/releases/latest/download/aioss-linux-x64.tar.gz | tar xz
./aioss --version
```

```powershell
# Windows
Invoke-WebRequest -Uri "https://github.com/kleinnner/Anticloud/releases/latest/download/aioss-windows-x64.zip" -OutFile aioss.zip
Expand-Archive aioss.zip
.\aioss\aioss.exe --version
```

## Option B: Build from Source

```bash
git clone https://github.com/kleinnner/Anticloud
cd Anticloud/04-aioss-format
cargo build --release
./target/release/aioss --version
```

## Your First Ledger (5 minutes)

```bash
# Create a new ledger
aioss init ./my-first-ledger --user "student"

# Log something
aioss append ./my-first-ledger/ledger.aioss \
  --type study_note \
  --actor me \
  --content '{"topic": "hash chains", "confidence": 0.7}'

# Log another entry
aioss append ./my-first-ledger/ledger.aioss \
  --type study_note \
  --actor me \
  --content '{"topic": "binary formats", "confidence": 0.9}'

# Verify the chain is intact
aioss verify ./my-first-ledger/ledger.aioss
# → Verified: 2 entries, 0 tampered

# See what's in it
aioss export ./my-first-ledger/ledger.aioss --format json
```

## Python Integration (10 minutes)

```python
# Install the Python binding
pip install aioss  # or build from bindings/python/

import aioss
import hashlib

def sha3_256(text: str) -> str:
    return hashlib.sha3_256(text.encode()).hexdigest()

ledger = aioss.open("./my-llm-log.aioss")

# Simulate an LLM call (replace with your actual model)
prompt = "What is a hash chain?"
response = "A hash chain links each block to the previous via its hash..."

ledger.append(
    entry_type="llm_response",
    actor="my-model",
    content={
        "prompt_hash": sha3_256(prompt),       # never store the raw prompt if sensitive
        "response_hash": sha3_256(response),
        "tokens_in": len(prompt.split()),
        "tokens_out": len(response.split()),
        "cost_if_cloud_microcents": 0,         # local = free
    }
)

result = ledger.verify()
print(f"Chain valid: {result.verified}, entries: {result.total}")
```

## Try on Kaggle (no GPU needed)

1. Go to https://kaggle.com and sign in as `loiskleinner`
2. Create a new notebook
3. Paste:
```python
!pip install aioss  # if available, else build from source

import subprocess, json

# Build from source on Kaggle
subprocess.run(["git", "clone", "https://github.com/kleinnner/Anticloud"], check=True)
subprocess.run(["cargo", "build", "--release", "--manifest-path", 
                "Anticloud/04-aioss-format/Cargo.toml"], check=True)

print("AIOSS built successfully on Kaggle T4!")
```

## What's Next

- Read `TECHNICAL/01_Architecture.md` for the binary format spec
- Integrate with PAX inference (see TIER_2/PAX_INFERENCE_CORE)
- Try K5 hashing (see TIER_1/KANTOR_K5) for post-quantum upgrade

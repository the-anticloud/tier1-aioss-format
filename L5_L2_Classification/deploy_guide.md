# Deploy Guide — AIOSS_FORMAT
**Air-gap capable. No external dependencies. stdlib only.**

## Prerequisites
- Python 3.11+ (stdlib only, no pip installs required)
- Write access to chain storage path (local SSD recommended for throughput)

## Environment
- CPU-only, 2MB RAM minimum
- Runs on Linux/Windows/macOS/embedded ARM
- No GPU required

## Install
```bash
pip install anticloud-aioss  # or copy aioss_format.py directly for embedded
```

## Air-Gap Deployment
No network required at any point. Copy the single file aioss_format.py to target machine.

## AIOSS Integration (this project is the chain)
```bash
aioss init --module AIOSS_FORMAT --output ./audit.aioss
aioss append --chain ./audit.aioss --payload ./output.bin --module MY_MODULE
aioss verify --chain ./audit.aioss
```

## PAX Harness Wiring
```python
from anticloud_pax import PAXHarness
harness = PAXHarness(model_path="./pax-27b-q4.gguf", module="AIOSS_FORMAT",
                     aioss_chain="./audit.aioss")
result = harness.process(input_data)
```

## Verification
```bash
aioss verify --chain ./audit.aioss --verbose
python -c "from aioss_format import AIOSSChain; print(AIOSSChain('audit.aioss').verify_all().valid)"
```

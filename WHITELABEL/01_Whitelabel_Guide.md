# AIOSS Format — Whitelabel & OEM Guide

## License

Apache 2.0. You can rebrand, redistribute, embed, and sell AIOSS commercially. No royalties. No attribution required (though appreciated).

## What You Can Customize

| Component | Whitelabel Option |
|-----------|------------------|
| Magic bytes | Change `AIOSS` to your product name (5 bytes) |
| CLI binary name | Rename to `yourproduct-audit`, `myaudit`, etc. |
| Python package name | Rename `aioss` → `yourproduct_audit` |
| Compliance frameworks | Add/remove from the 8 defaults |
| Hash engine | Swap SHA3 for K5 or your proprietary hash |

## Rebranding Steps

### 1. Magic Bytes

```rust
// In crates/aioss-core/src/types.rs
// Change:
pub const AIOSS_MAGIC: &[u8; 5] = b"AIOSS";
// To:
pub const AIOSS_MAGIC: &[u8; 5] = b"YOURB";  // 5 chars
```

Note: ledgers created with your magic bytes will not be readable by vanilla AIOSS CLI. This is intentional for product differentiation.

### 2. CLI Binary Name

```toml
# In crates/aioss-cli/Cargo.toml
[[bin]]
name = "yourproduct-audit"  # was "aioss"
path = "src/main.rs"
```

### 3. Python Package

```toml
# In bindings/python/setup.py
name = "yourproduct-audit"  # was "aioss"
```

### 4. Custom Compliance Framework

```rust
// In crates/aioss-core/src/compliance.rs
pub enum ComplianceFramework {
    // ... existing ...
    YourIndustryStandard,
}

impl ComplianceFramework {
    pub fn check_your_industry_standard(entries: &[LedgerEntry]) -> FrameworkResult {
        // Your compliance logic
    }
}
```

## Embedding in Your Product

### Python SDK Embedding

```python
# In your product's SDK
from yourproduct_audit import Ledger  # rebranded aioss

class YourProductAI:
    def __init__(self, model_path: str):
        self.ledger = Ledger.open("./audit.yourproduct")
        self.model = load_model(model_path)
    
    def generate(self, prompt: str) -> str:
        response = self.model.generate(prompt)
        self.ledger.append(
            entry_type="generation",
            actor=self.model.name,
            content={"tokens_in": ..., "tokens_out": ..., "cost_if_cloud_microcents": 0}
        )
        return response
```

### Rust Crate Embedding

```toml
# Cargo.toml
[dependencies]
aioss-core = { git = "https://github.com/kleinnner/Anticloud", subdirectory = "04-aioss-format/crates/aioss-core" }
```

## Pricing Reference for OEM

If selling AIOSS-based compliance as a service:

| Tier | Suggested Price | Includes |
|------|----------------|---------|
| Basic | $1,000/year | Core ledger, verify |
| Standard | $5,000/year | + compliance reports (8 frameworks) |
| Enterprise | $25,000/year | + custom frameworks, SLA, air-gap installer |
| Government | $50,000/year | + FedRAMP audit package, classified-compatible |

## Attribution (Optional but Appreciated)

```
Powered by AIOSS — AI Open Signed Storage
https://github.com/kleinnner/Anticloud
Lois-Kleinner Alpasan, The Anticloud 2026
```

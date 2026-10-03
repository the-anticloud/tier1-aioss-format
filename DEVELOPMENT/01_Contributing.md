# AIOSS Format — Developer Guide

## Repository Structure

```
aioss-format/
  Cargo.toml              — workspace manifest
  crates/
    aioss-core/           — core library
      src/
        types.rs          — AiossHeader, AiossEntry structs (C-repr packed)
        hash_chain.rs     — compute_entry_hash, verify_chain
        hash_engine.rs    — HashEngine trait (swappable: SHA3, K5)
        binary.rs         — 256-byte fixed-width codec
        json.rs           — JSON ledger codec  
        compliance.rs     — 8-framework compliance checks
        analyzer.rs       — statistics, cost aggregation
        event_store.rs    — SQLite high-frequency events
        health.rs         — parallel .health diagnostic ledger
    aioss-cli/            — clap-based CLI
    aioss-health/         — health ledger writer
    aioss-logger/         — structured log → ledger
    aioss-installer/      — cross-platform installer
  bindings/
    python/               — PyO3 native extension
    go/                   — Go wrapper
    js/                   — NAPI-RS Node.js binding
  .github/
    workflows/
      ci.yml              — cargo test, clippy, fmt
      audit.yml           — cargo deny check
      release.yml         — binary builds for Linux/macOS/Windows
```

## Building

```bash
cargo build --release --workspace
```

## Testing

```bash
# Integration tests
cargo test -p aioss-core --test integration_tests

# Property tests (proptest)
cargo test -p aioss-core --test property_tests

# All tests
cargo test --workspace
```

## Adding a New Compliance Framework

1. Open `crates/aioss-core/src/compliance.rs`
2. Add a new variant to the `ComplianceFramework` enum
3. Implement `check_framework(&self, entries: &[LedgerEntry]) -> FrameworkResult`
4. Add it to the `ALL_FRAMEWORKS` slice
5. Write a test in `#[cfg(test)]` block

## Adding a New Hash Engine

The `HashEngine` trait allows swapping SHA3 for K5 (post-quantum) or any other:

```rust
// In hash_engine.rs
pub trait HashEngine: Send + Sync {
    fn hash(&self, data: &[u8]) -> Vec<u8>;
    fn hash_hex(&self, data: &[u8]) -> String {
        hex::encode(self.hash(data))
    }
}

pub struct Sha3Engine;
impl HashEngine for Sha3Engine {
    fn hash(&self, data: &[u8]) -> Vec<u8> {
        use sha3::{Digest, Sha3_256};
        Sha3_256::digest(data).to_vec()
    }
}

// K5 engine would implement the same trait using Poseidon/Goldilocks
// See KANTOR_K5 project for K5Engine implementation
```

Use `compute_entry_hash_with(entry, &K5Engine)` to hash with K5.

## No Frontier API Keys Policy

AIOSS has zero network dependencies. No API keys anywhere. Verify before PRs:

```bash
grep -r "OPENAI_API_KEY\|ANTHROPIC_API_KEY\|GEMINI_API_KEY" . --include="*.rs" --include="*.py" --include="*.go"
# Must return empty
```

## CI/CD

- `ci.yml`: runs on every push — `cargo test`, `cargo clippy`, `cargo fmt --check`
- `audit.yml`: weekly `cargo deny check` for supply chain vulnerabilities
- `release.yml`: builds cross-platform binaries on tag push

## Code Standards

- **No unsafe** in aioss-core except where required for C FFI
- **All public functions must have `#[doc]` comments**
- **Error types must implement `std::error::Error`**
- **Binary format changes require a version bump** (`AIOSS_VERSION += 1`)

## AIOSS Ledger in CI

The CI pipeline itself logs to a ledger:

```yaml
# .github/actions/verify-ledger/action.yml
- name: Verify ledger integrity
  run: ./aioss verify ./test_ledger.aioss
```

Every test run appends build metadata to a ledger that's archived as a CI artifact.

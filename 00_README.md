# AIOSS — AI Open Signed Storage

**Status:** Production-ready | **Version:** 1.0.0 | **Author:** Lois-Kleinner Alpasan

---

## What Is AIOSS?

AIOSS is a tamper-evident, SHA3-256 hash-chained ledger format for auditing AI interactions, compliance records, and system diagnostics. It provides cryptographic proof that data hasn't been altered, with support for 8 compliance frameworks (SOC2, FedRAMP, ISO27001, GDPR, HIPAA, EU AI Act, UAE AI Act, SPASA).

AIOSS stands alone as an audit format but is strengthened by integration with cryptographic projects (K5, MF+SO) and AI systems (Miirai, Inte11ect).

---

## Key Features

- **Dual format:** Binary (compact, fixed-size) and JSON (human-readable)
- **Hash chain:** SHA3-256 chaining with Ed25519 state proofs
- **8 compliance frameworks:** SOC2, FedRAMP, ISO27001, GDPR, HIPAA, EU AI Act, UAE AI Act, SPASA
- **Health ledger:** Parallel `.health` diagnostic logs with hash chain
- **Event store:** SQLite-backed high-frequency event capture
- **Cross-platform:** Linux, macOS, Windows
- **Language bindings:** Python (native PyO3), Go (CLI), JavaScript/Node.js (NAPI-RS)
- **Post-quantum ready:** K5 integration path for SHA3-256 → K5-512 migration

---

## Quick Start

```bash
# Create a new ledger
aioss init ./my-ledger --user "me"

# Append entries
aioss append ./my-ledger/ledger.aioss --type user_message --actor user1 --content '{"msg":"hello"}'

# Verify the hash chain
aioss verify ./my-ledger/ledger.aioss

# Analyze with compliance
aioss analyze ./my-ledger/ledger.aioss

# Export to different format
aioss export ./my-ledger/ledger.aioss --format txt
```

---

## Architecture

```
User/App → [Ledger Entry Generator] 
        → [SHA3-256 Hasher]
        → [Binary/JSON Serializer]
        → [SQLite Event Store]
        → [File System (.aioss, .health)]
        → [Compliance Analyzer]
```

---

## Ledger Entry Structure

Each entry contains:
```json
{
  "timestamp": "2026-09-28T12:34:56Z",
  "type": "ai_inference|user_message|system_event|compliance_marker",
  "actor": "user_id|system|admin",
  "content": { /* arbitrary JSON */ },
  "prev_hash": "a3b2c1d0...",
  "current_hash": "SHA3-256(prev_hash || content || timestamp)",
  "signature": "Ed25519(current_hash, private_key)",
  "compliance_tags": ["GDPR", "SOC2"]
}
```

---

## Compliance Frameworks Supported

| Framework | Use Case | Status |
|-----------|----------|--------|
| **SOC2** | Enterprise service organization | Full |
| **FedRAMP** | US government cloud services | Full |
| **ISO27001** | Information security management | Full |
| **GDPR** | EU data protection | Full |
| **HIPAA** | US healthcare data | Full |
| **EU AI Act** | European AI regulation | Full |
| **UAE AI Act** | Emirates AI regulation | Full |
| **SPASA** | Privacy and security assessments | Full |

---

## Use Cases

### 1. AI Inference Audit Trail
Record every model input, output, confidence score, and latency for accountability.

**Example:**
```
Miirai → User asks medical question
      → Input hash: K5-512(question)
      → Output: Model response
      → Confidence: 92%
      → AIOSS entry: (input_hash || output_hash || confidence || timestamp)
      → Hash chain proves output wasn't tampered with
```

### 2. Compliance Recording
Automatically tag entries with compliance frameworks and generate audit reports.

**Example:**
```
HIPAA audit: "Show me all medical inferences with patient data"
AIOSS analyze --filter "compliance_tags:HIPAA" --filter "actor:doctor"
→ Compliance report: 247 HIPAA-tagged entries, all signatures valid
```

### 3. System Diagnostics
Log all system events (errors, warnings, state changes) in tamper-evident format.

**Example:**
```
aioss append ledger.aioss --type system_event --actor "system" --content '{
  "event": "inference_timeout",
  "service": "miirai-api",
  "duration_ms": 5000,
  "error_code": "TIMEOUT_5S"
}'
```

### 4. Supply Chain Provenance
Track product provenance through custody chain with cryptographic signatures.

**Example:**
```
Factory → K5-512(component_id || cert_hash)
Warehouse → Verify: Re-hash → Same? Authentic
Retailer → Same verification → Unbreakable supply chain
```

---

## Integration with Other Projects

### K5 (Post-Quantum Hash)
Replace SHA3-256 with K5-512 for quantum-resistant ledger:
```yaml
# aioss config
hash_function: "k5-512"  # was: "sha3-256"
quantum_resistant: true
```

### MF+SO (Identity Vault)
Sign AIOSS entries with MF+SO-generated Ed25519 keys:
```python
aioss.append(
  type="user_message",
  actor=mfso.get_identity(),
  content=message,
  signature=mfso.sign(hash)
)
```

### Miirai (AI Companion)
Wrap every Miirai inference in AIOSS ledger:
```python
prompt_hash = k5_512(prompt)
output = miirai.generate(prompt)
output_hash = k5_512(output)
aioss.append(type="ai_inference", content={
  "prompt_hash": prompt_hash,
  "output_hash": output_hash,
  "model": "miirai-0.5b"
})
```

---

## Security Model

- **Hash chain integrity:** SHA3-256 cryptographically binds each entry to previous
- **Signature verification:** Ed25519 proves author authenticity
- **Tamper detection:** Any modification breaks hash chain immediately
- **Compliance auditing:** Tags enable framework-specific analysis
- **Post-quantum path:** K5 integration ready for quantum-era migration

---

## Performance

| Operation | Latency |
|-----------|---------|
| Append entry | ~1ms |
| Verify chain (10K entries) | ~15ms |
| Compliance analysis | ~50ms |
| Export to JSON | ~100ms |

---

## Project Structure

```
aioss-format/
├── crates/
│   ├── aioss-core/        # Hash chain engine
│   ├── aioss-cli/         # Command-line interface
│   ├── aioss-python/      # Python bindings (PyO3)
│   ├── aioss-go/          # Go bindings
│   └── aioss-js/          # JavaScript bindings (NAPI-RS)
├── docs/                  # Technical documentation
├── tests/                 # Integration and property tests
└── README.md
```

---

## License

MIT — Lois-Kleinner Alpasan

---

## References

1. Lois-Kleinner Zenodo: https://doi.org/10.5281/zenodo.20781790
2. GitHub: https://github.com/kleinnner/Anticloud
3. ORCID: https://orcid.org/0009-0009-2233-6107

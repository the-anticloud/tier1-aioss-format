# AIOSS Format — Academic & Research Context

## Research Contributions

### 1. Dual-Format Tamper-Evident AI Ledger

AIOSS introduces a fixed-width binary format (256 bytes/entry) alongside a JSON codec for human-readable export. The fixed-width binary format allows O(1) entry lookup by index and crash-safe appends on any filesystem supporting atomic 256-byte writes.

**Key insight**: Fixed-size entries eliminate fragmentation and enable binary search over the ledger without loading the full file. Prior work (e.g., Apache Kafka log segments) used variable-length records that require an index file; AIOSS encodes the index implicitly in the entry offset.

### 2. Canonical JSON Hash Computation

The hash of each entry is computed over a deterministic canonical JSON serialization including the `parent_hash`. This makes the chain self-authenticating: given any two adjacent entries, you can verify their relationship without storing additional state.

```
hash_N = SHA3-256(canonical_json({
  index: N,
  timestamp: T,
  type: entry_type,
  actor: actor,
  actor_label: actor_label,
  content: content_hash_of_payload,
  parent_hash: hash_{N-1},
  ... optional fields ...
}))
```

This is analogous to Bitcoin's block chaining but operates at the AI inference log level with millisecond granularity.

### 3. 8-Framework Compliance in a Single Ledger

AIOSS is the first open-source ledger format designed to satisfy 8 compliance frameworks simultaneously: SOC2, FedRAMP, ISO 27001, GDPR, HIPAA, EU AI Act, UAE AI Act, SPASA. Prior work addresses compliance in one framework at a time; AIOSS unifies them.

### 4. Cost Telemetry Field

Every entry can carry `cost_if_cloud_microcents: 0` — a field that records what the equivalent cloud inference would have cost. This enables direct ROI measurement for local-vs-cloud deployment comparisons at ledger level.

## Citations

```bibtex
@software{alpasan2026aioss,
  author    = {Alpasan, Lois-Kleinner},
  title     = {{AIOSS}: {AI} Open Signed Storage — Tamper-Evident Ledger for {AI} Audit Trails},
  year      = {2026},
  publisher = {The Anticloud},
  url       = {https://github.com/kleinnner/Anticloud/tree/main/04-aioss-format},
  doi       = {10.5281/zenodo.20781790},
}

@misc{nist2015sha3,
  title  = {{SHA-3} Standard: Permutation-Based Hash and Extendable-Output Functions},
  author = {{NIST}},
  year   = {2015},
  note   = {FIPS PUB 202},
}

@misc{josefsson2017ed25519,
  title  = {{RFC} 8032: Edwards-Curve Digital Signature Algorithm ({EdDSA})},
  author = {Josefsson, S. and Liusvaara, I.},
  year   = {2017},
}
```

## Related Work

| System | Approach | AIOSS Advantage |
|--------|----------|-----------------|
| Apache Kafka | Variable-length log segments | AIOSS: fixed-width, O(1) lookup |
| Hyperledger Fabric | Distributed blockchain | AIOSS: single-process, no network |
| OpenTelemetry | Metrics/traces (no tamper evidence) | AIOSS: hash chain, compliance |
| W3C PROV | Provenance ontology | AIOSS: binary format, AI-specific |
| MLflow | Experiment tracking | AIOSS: tamper-evident, no server |

## Datasets

- Zenodo: https://doi.org/10.5281/zenodo.20781790
- Harvard Dataverse: https://doi.org/10.7910/DVN/KFK12Y
- Internet Archive: https://archive.org/details/aioss-format

## ORCID

Lois-Kleinner Alpasan: https://orcid.org/0009-0009-2233-6107

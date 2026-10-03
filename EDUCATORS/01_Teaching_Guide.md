# AIOSS Format — Educator's Teaching Guide

## Course Fit

AIOSS is appropriate for:
- **CS/SE courses**: data structures (hash chains), file formats, cryptography
- **AI ethics courses**: audit trails, accountability, transparency requirements
- **Compliance engineering**: satisfying SOC2, GDPR, HIPAA requirements

## 3-Week Module: Tamper-Evident Systems

### Week 1: Hash Chains

**Lecture topics**:
1. SHA3-256: construction, security properties (FIPS 202)
2. Hash chains: how each block depends on the previous
3. Genesis block: why `parent_hash = 0x00…`
4. Canonical JSON: why key ordering matters for determinism

**Lab exercise**: students implement a minimal hash chain in Python

```python
import hashlib, json, time

def canonical(entry: dict) -> str:
    return json.dumps(entry, sort_keys=True)

def chain_hash(entry: dict) -> str:
    return hashlib.sha3_256(canonical(entry).encode()).hexdigest()

entries = []
prev_hash = "0" * 64  # genesis

for i, content in enumerate(["hello", "world", "tamper-me"]):
    entry = {
        "index": i,
        "timestamp": time.time_ns(),
        "content": content,
        "parent_hash": prev_hash,
    }
    entry["hash"] = chain_hash(entry)
    entries.append(entry)
    prev_hash = entry["hash"]

# Verify — student task: what happens if entries[1]["content"] is modified?
```

**Assignment**: modify one entry and show how verification catches it.

### Week 2: Binary Formats and Fixed-Width Records

**Lecture topics**:
1. C-repr packed structs in Rust/C
2. Fixed-width records: O(1) lookup vs variable-length
3. Magic bytes and version fields
4. CRC checksums

**Lab exercise**: parse a real AIOSS binary ledger file in Python

```python
import struct

MAGIC = b"AIOSS"
HEADER_SIZE = 155
ENTRY_SIZE = 256

with open("sample.aioss", "rb") as f:
    header = f.read(HEADER_SIZE)
    magic = header[:5]
    version = struct.unpack_from("<H", header, 5)[0]
    entry_count = struct.unpack_from("<I", header, 13)[0]
    print(f"Magic: {magic}, Version: {version}, Entries: {entry_count}")
    
    for i in range(entry_count):
        entry = f.read(ENTRY_SIZE)
        index = struct.unpack_from("<I", entry, 0)[0]
        ts_ms = struct.unpack_from("<Q", entry, 4)[0]
        entry_type = entry[12:32].rstrip(b"\x00").decode()
        print(f"  [{index}] @{ts_ms}ms type={entry_type}")
```

### Week 3: Compliance Engineering

**Lecture topics**:
1. SOC2 Trust Service Criteria
2. GDPR Article 17 (right to erasure) vs audit trail requirements
3. EU AI Act Article 13 (transparency)
4. Tension: immutable logs vs right to be forgotten → solution: hash content, don't store PII

**Discussion questions**:
- How does storing a SHA3-256 hash of PII satisfy GDPR's right to erasure?
- Why does AIOSS use canonical JSON rather than the raw struct bytes for hashing?
- What attack does the genesis `parent_hash = 0x00…` prevent?

## Assessment

| Assessment | Weight | Task |
|------------|--------|------|
| Lab 1 | 25% | Python hash chain with tamper detection |
| Lab 2 | 25% | Parse real AIOSS binary file |
| Lab 3 | 25% | Map AIOSS fields to 3 compliance requirements |
| Final project | 25% | Integrate AIOSS into a toy LLM chatbot |

## Sample Exam Questions

1. An AIOSS entry has `parent_hash` = X. After appending it, entry N+1 must reference X as its `parent_hash`. If an attacker changes entry N's content, what changes? *(Answer: entry N's `hash` changes, making entry N+1's `parent_hash` a lie — verify detects this)*

2. Why does AIOSS use `canonical_json` with sorted keys rather than raw bytes for entry hashing? *(Answer: JSON serialization is not unique — same data can serialize differently; canonical form ensures determinism across implementations)*

3. A healthcare company stores patient queries in AIOSS. Does this violate GDPR? *(Answer: Only if PII is stored directly. AIOSS design: hash the query, store the hash. SHA3-256 is one-way — no PII recoverable)*

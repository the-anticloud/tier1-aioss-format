# Developer Cookbook — AIOSS_FORMAT
**Stack:** Python 3.11 stdlib | SHA3-256 | Binary I/O

## Initialize a chain
```python
from aioss_format import AIOSSChain
chain = AIOSSChain.init("./audit.aioss", module_id="AIOSS_FORMAT")
print(chain.root_hash)
```

## Append inference result
```python
import json
payload = json.dumps({"query": "...", "response": "...", "tokens": 142}).encode()
entry = chain.append(payload)
print(f"Entry #{entry.index}: {entry.chain_hash}")
```

## Verify entire chain
```python
report = chain.verify_all()
for e in report.entries:
    print(f"[{e.index}] {'OK' if e.valid else 'FAIL'} {e.chain_hash[:16]}...")
assert report.valid
```

## Export for audit submission
```python
chain.export_json("./audit_export.json")
```

## High-throughput batch append
```python
with chain.batch_writer() as bw:
    for result in inference_stream:
        bw.append(result.to_bytes())
```

## AIOSS append (raw Python, no deps)
```python
import hashlib, struct, time

def aioss_append(chain_path, payload: bytes, module_id: str):
    entry_hash = hashlib.sha3_256(payload).digest()
    ts = int(time.time_ns()).to_bytes(8, 'big')
    with open(chain_path, 'rb') as f:
        f.seek(-32, 2); prev_hash = f.read(32)
    new_hash = hashlib.sha3_256(prev_hash + entry_hash + ts).digest()
    with open(chain_path, 'ab') as f:
        f.write(ts + entry_hash + new_hash)
    return new_hash.hex()
```

## Performance
SHA3-256 throughput: ~500MB/s on modern CPU. Use batch_writer() to amortize fsync.
Pre-allocate: `chain.preallocate(n_entries=100000)` to avoid fragmentation.

## Integration
All 123 Anticloud projects call AIOSS_FORMAT. It is the universal audit substrate.

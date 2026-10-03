# AIOSS Format — Ethics Assessment

## EU AI Act Classification

AIOSS is an **infrastructure tool**, not an AI system making decisions. It logs AI decisions made by other systems. Classification: **Minimal Risk** (Article 52 — transparency obligations met via immutable logs).

AIOSS *enables* high-risk AI systems to meet their EU AI Act Article 13 transparency obligations.

## Privacy-by-Design Analysis

| Principle | Implementation |
|-----------|---------------|
| Data minimization | Content stored as hash only — no raw text in ledger |
| Purpose limitation | Ledger entries are append-only audit records, not analytics data |
| Storage limitation | Ledger is local-only, no cloud sync |
| Integrity | SHA3-256 hash chain ensures no silent modification |
| Confidentiality | File system permissions; no network exposure |

## Fairness

AIOSS logs **all** inferences uniformly. No demographic filtering. The `actor` field captures model identity, not user identity. Users are never identified in ledger entries.

## Dual-Use Risk

**Risk**: An oppressive government could use AIOSS to build surveillance logs of citizen queries.

**Mitigation**: 
1. AIOSS logs **hashes**, not content. You cannot reconstruct what was asked from the hash.
2. Apache 2.0 license cannot prevent misuse (like any open-source tool).
3. By storing hashes, AIOSS gives users plausible deniability: the log proves an interaction happened but not its content.

## Environmental Impact

| Resource | AIOSS Usage |
|----------|------------|
| Compute | Negligible — SHA3-256 is ~50 ns/entry on modern CPUs |
| Storage | 256 bytes/entry binary; 1M entries = 256 MB |
| Network | Zero — fully local |
| Carbon | Effectively zero |

Compared to cloud logging services (CloudWatch, Datadog): AIOSS eliminates data transfer, remote storage, and always-on servers.

## Transparency

AIOSS is fully open-source (Apache 2.0). The hash algorithm (SHA3-256 FIPS 202), entry format, and chain rule are all public. There are no hidden backdoors or proprietary components.

The binary format magic bytes (`AIOSS`) are documented. Any tool can verify ledger integrity by reimplementing the chain rule from `hash_chain.rs`.

## No Frontier API Keys

AIOSS has zero runtime network dependencies. It cannot phone home. No telemetry, no API calls, no license checks.

```bash
# Verify no network calls — run under strace on Linux
strace -e trace=network ./aioss verify ./ledger.aioss 2>&1 | grep -c "connect\|sendto\|recvfrom"
# → 0
```

## Accountability

Author: Lois-Kleinner Alpasan  
Contact: quazakeido@gmail.com  
ORCID: https://orcid.org/0009-0009-2233-6107  
Published: Zenodo https://doi.org/10.5281/zenodo.20781790  

# AIOSS Format — Enterprise Use Cases

## Use Case 1: AI Audit Trail for Regulated Industries

**Problem**: Healthcare, finance, and government deployments of LLMs require immutable audit logs proving what the model was asked and what it responded.

**Solution**: Every PAX inference routes through AIOSS. The ledger records:
- SHA3-256 of the prompt (not the prompt itself — PII-safe)
- SHA3-256 of the response
- Token counts, latency, model version
- `cost_if_cloud_microcents: 0` showing zero cloud spend

```python
# Drop-in audit wrapper for any LLM call
from aioss import Ledger

ledger = Ledger.open("./hospital_ai_audit.aioss")

def audited_inference(prompt: str, model) -> str:
    response = model.generate(prompt)
    ledger.append(
        entry_type="inference_result",
        actor="pax-27b",
        content={
            "prompt_hash": sha3_256(prompt),
            "response_hash": sha3_256(response),
            "tokens_in": count_tokens(prompt),
            "tokens_out": count_tokens(response),
            "cost_if_cloud_microcents": 0,
        }
    )
    return response
```

**Outcome**: HIPAA + EU AI Act compliant audit trail. Auditors run `aioss verify` to confirm no tampering.

## Use Case 2: Multi-Agent Workflow Provenance

**Problem**: In a multi-agent pipeline (planner → coder → reviewer), tracking which agent produced which output is critical for debugging and accountability.

**Solution**: Each agent appends to a shared AIOSS ledger with its `actor` ID. The hash chain links planner output to coder input to reviewer verdict.

```
entry 0: actor=planner,    type=task_plan,      content_hash=abc...
entry 1: actor=coder,      type=code_output,    parent_hash=abc...
entry 2: actor=reviewer,   type=review_verdict, parent_hash=def...
```

**Outcome**: Any step in the pipeline can be traced back to its originating plan. Disputes resolved by hash chain verification.

## Use Case 3: Compliance Evidence for Procurement

**Problem**: Enterprise procurement requires evidence that AI tools meet SOC2, ISO 27001, and GDPR.

**Solution**: Run `aioss analyze ./ledger.aioss` to generate a compliance report. All 8 frameworks emit PASS/FAIL with evidence.

```bash
aioss analyze ./production_ledger.aioss --format pdf > compliance_report.pdf
```

**Outcome**: Compliance report auto-generated from production ledger. Zero manual evidence collection.

## Use Case 4: Air-Gap Deployment

**Problem**: Government and defense customers cannot use cloud AI APIs. They need local AI with audit capabilities.

**Solution**: AIOSS runs with zero network dependencies. The binary format works on any POSIX/Windows filesystem.

```bash
# Completely offline — no network required
aioss init /air-gap/ledger --user "classified-system"
aioss append /air-gap/ledger/ledger.aioss \
  --type inference_result \
  --actor classified-llm \
  --content '{"tokens_in": 200, "tokens_out": 100}'
aioss verify /air-gap/ledger/ledger.aioss
```

**Outcome**: Full audit capability with zero cloud exposure.

## Use Case 5: Cost Accounting for Local-vs-Cloud ROI

**Problem**: Finance teams want to quantify savings from running models locally instead of via OpenAI/Anthropic.

**Solution**: Every AIOSS entry carries `cost_if_cloud_microcents` — the equivalent cloud cost. Aggregate across all entries.

```bash
# Total cloud cost avoided
aioss analyze ./ledger.aioss --metric cost_savings
# → Local cost: $0.00
# → Equivalent OpenAI (GPT-4-Turbo) cost: $4,230.50
# → Savings: $4,230.50
```

**Outcome**: Direct ROI figure for finance teams, generated from production ledger data.

## Deployment Architecture

```
┌─────────────────────────────────────────────┐
│  Production Server (air-gap optional)       │
│                                             │
│  PAX Inference Engine                       │
│    └─► AIOSS Logger ──► ledger.aioss        │
│                                             │
│  Compliance Scanner (scheduled)             │
│    └─► aioss verify ledger.aioss            │
│    └─► aioss analyze --format pdf           │
│                                             │
│  Audit Export (on-demand)                   │
│    └─► aioss export --format json           │
└─────────────────────────────────────────────┘
```

No external services. No API keys. Runs on any Linux/macOS/Windows server.

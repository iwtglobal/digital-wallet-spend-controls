# Architecture — Digital Wallet Spend Controls

High-level reference for evaluating **digital wallet spend controls**, transaction limits, and wallet velocity controls. Educational only; vendor designs vary.

## Logical Components

```
┌──────────────────┐     ┌──────────────────┐     ┌─────────────────┐
│ Wallet app / API │────►│ Intent gateway   │────►│ Policy resolver │
│ pay / send UX    │     │ (pre-ledger)     │     │ KYC + product   │
└──────────────────┘     └────────┬─────────┘     └────────┬────────┘
                                  │                        │
                                  ▼                        ▼
                         ┌──────────────────┐     ┌─────────────────┐
                         │ Limit & velocity │────►│ Decision log    │
                         │ engine           │     │ + reason codes  │
                         └────────┬─────────┘     └─────────────────┘
                                  │ allow only
                                  ▼
                         ┌──────────────────┐
                         │ Ledger authorize │
                         │ / hold           │
                         └──────────────────┘
```

## Suggested Decision Metadata

| Field | Meaning |
|-------|---------|
| `intent_id` | Durable ID for the attempted spend or transfer |
| `wallet_id` | Subject wallet |
| `rule_set_id` | Active policy pack (KYC tier + product) |
| `limit_type` | `per_tx` / `daily_amount` / `velocity_count` / `contextual` |
| `requested_amount` | Amount on the intent |
| `remaining_headroom` | Snapshot after evaluation (when allowed or soft-declined) |
| `decision` | `allow` / `soft_decline` / `hard_decline` |
| `reason_code` | Stable code for UX and support |

## Audit Events Worth Keeping

- Policy resolution (which rule set bound)  
- Each rule evaluation outcome in the chain  
- Headroom before and after allow  
- Soft vs hard decline with customer-safe reason  
- Rule-set changes (KYC upgrade, temporary hold)  
- Fail-closed / fail-open incidents when the engine is unavailable  

## Failure Modes to Design For

- Concurrent intents racing the same daily headroom (serialize or optimistic consume)  
- Clock skew across window boundaries (document timezone and window type)  
- Stale rule cache after KYC upgrade  
- Generic “failed” UX that hides an actionable limit  

## Related Reading

Live product reading: [evdsystem.com](https://evdsystem.com/), [About MoboGage](https://evdsystem.com/about-mobogage/).

Platforms in the MoboGage / EVD System family keep wallet policy decisions auditable so support can reconstruct why a spend was allowed or blocked without rewriting ledger history.

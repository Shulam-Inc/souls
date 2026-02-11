# Shulam Master Project Plan

## Overview

This document outlines the complete development roadmap for Shulam's x402 payment infrastructure. It maps dependencies across all 7 repositories and sequences work for the 18 apostle agents.

---

## Repository Dependency Map

```
┌─────────────────────────────────────────────────────────────────────────┐
│                           DEPENDENCY FLOW                                │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  ┌──────────┐                                                           │
│  │ contracts│ ←── Must deploy first (escrow, cashback)                  │
│  └────┬─────┘                                                           │
│       │                                                                  │
│       ▼                                                                  │
│  ┌──────────┐     ┌────────────┐                                        │
│  │facilitator│────▶│ compliance │  (screens before settlement)          │
│  └────┬─────┘     └────────────┘                                        │
│       │                                                                  │
│       ├───────────────────┐                                             │
│       ▼                   ▼                                             │
│  ┌──────────┐     ┌───────────┐                                         │
│  │buyer-sdk │     │ dashboard │  (both consume facilitator API)         │
│  └────┬─────┘     └─────┬─────┘                                         │
│       │                 │                                                │
│       └────────┬────────┘                                               │
│                ▼                                                         │
│          ┌──────────┐                                                   │
│          │   docs   │  (documents all other repos)                      │
│          └──────────┘                                                   │
│                                                                          │
│  ┌──────────┐                                                           │
│  │  souls   │  (orchestrates agents across all repos)                   │
│  └──────────┘                                                           │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Development Phases

### Phase 1: Foundation (Weeks 1-2)
| Repo | Milestone | Dependency |
|------|-----------|------------|
| contracts | Deploy USDC mock to Sepolia | None |
| facilitator | Health endpoint + config | None |
| compliance | OFAC list loader | None |
| souls | Agent harness scaffold | None |

### Phase 2: Core Infrastructure (Weeks 3-6)
| Repo | Milestone | Dependency |
|------|-----------|------------|
| contracts | Escrow contract deployed | Phase 1 |
| facilitator | EIP-3009 verification | contracts |
| facilitator | CDP wallet integration | contracts |
| compliance | KYT screening endpoint | facilitator |

### Phase 3: Settlement (Weeks 7-10)
| Repo | Milestone | Dependency |
|------|-----------|------------|
| facilitator | Settlement engine | compliance |
| contracts | Cashback vault deployed | facilitator |
| buyer-sdk | Wallet connection | facilitator |
| dashboard | Merchant onboarding | facilitator |

### Phase 4: Integration (Weeks 11-14)
| Repo | Milestone | Dependency |
|------|-----------|------------|
| buyer-sdk | Payment widget | facilitator |
| dashboard | Transaction view | facilitator |
| docs | API reference | all |
| compliance | SAR generation | facilitator |

### Phase 5: Mainnet (Weeks 15-18)
| Repo | Milestone | Dependency |
|------|-----------|------------|
| contracts | Mainnet deployment | audit |
| facilitator | Production config | contracts |
| compliance | Production KYT | facilitator |
| dashboard | Production deploy | all |

---

## Cross-Repo Integration Points

### facilitator ↔ compliance
```
POST /verify → calls compliance/screen before approval
POST /settle → calls compliance/check before execution
```

### facilitator ↔ contracts
```
Settlement calls Escrow.release()
Cashback calls CashbackVault.distribute()
```

### buyer-sdk ↔ facilitator
```
SDK calls POST /verify with signed authorization
SDK polls GET /status/:txId for confirmation
```

### dashboard ↔ facilitator
```
Dashboard calls GET /transactions for merchant
Dashboard calls POST /webhooks/configure
```

---

## Agent Assignments

| Repo | Primary Agent | Supporting Agents |
|------|---------------|-------------------|
| facilitator | Andrew (Developer) | James (Oracle), Thomas (QA) |
| contracts | Andrew (Developer) | Thomas (QA) |
| compliance | Timothy (Compliance) | Andrew (Developer) |
| buyer-sdk | Andrew (Developer) | Thaddaeus (Designer) |
| dashboard | Thaddaeus (Designer) | Andrew (Developer) |
| docs | Matthew (Content) | Luke (Scribe) |
| souls | Peter (Lead) | Simon (DevOps) |

---

## Success Metrics

| Phase | Metric | Target |
|-------|--------|--------|
| Phase 2 | Test transactions verified | 100 |
| Phase 3 | End-to-end settlement | 10 tx |
| Phase 4 | Design partner integrations | 2 |
| Phase 5 | Mainnet transactions | 1,000 |
| Phase 5 | Volume processed | $100K |

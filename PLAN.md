# Souls Project Plan

## Overview

21 specialized AI agents ("Apostles") organized into 6 teams for autonomous operation of the Shulam x402 payment facilitator. Agents execute in two modes — **heartbeat** (minute-level cycle for periodic work) and **streaming** (continuous event-driven for the payment critical path) — using Slack as the communication backbone and Claude as the reasoning engine.

The system runs as a single Node.js process on macOS with Slack Bolt (Socket Mode), BullMQ + Redis for streaming agents, SQLite for state (migrated to PostgreSQL in production), and the Claude API for agent reasoning.

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        MacBook (Local)                          │
│                                                                 │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                   SOUL ENGINE (Node.js)                  │   │
│  │                                                          │   │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐      │   │
│  │  │ Heartbeat   │  │  Streaming  │  │  Human      │      │   │
│  │  │ Scheduler   │  │  Event Loop │  │  Command    │      │   │
│  │  │ (node-cron) │  │  (BullMQ)   │  │  Handler    │      │   │
│  │  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘      │   │
│  │         │                │                │              │   │
│  │         └────────────────┼────────────────┘              │   │
│  │                          │                               │   │
│  │                    ┌─────┴─────┐                         │   │
│  │                    │  SOUL     │                         │   │
│  │                    │  ROUTER   │                         │   │
│  │                    └─────┬─────┘                         │   │
│  │                          │                               │   │
│  │         ┌────────────────┼────────────────┐              │   │
│  │         │                │                │              │   │
│  │    ┌────┴────┐     ┌────┴────┐     ┌─────┴────┐         │   │
│  │    │ PETER   │     │ JAMES   │     │ SAMUEL   │  ...×21 │   │
│  │    │ Soul    │     │ Soul    │     │ Soul     │         │   │
│  │    │ Module  │     │ Module  │     │ Module   │         │   │
│  │    └─────────┘     └─────────┘     └──────────┘         │   │
│  │                                                          │   │
│  └──────────────────────────┬───────────────────────────────┘   │
│                             │                                   │
│         ┌───────────────────┼───────────────────┐               │
│         │                   │                   │               │
│    ┌────┴────┐        ┌────┴────┐         ┌────┴────┐          │
│    │  Slack  │        │  Redis  │         │ SQLite  │          │
│    │  Bolt   │        │ (Docker)│         │  (D1)   │          │
│    │ Socket  │        │         │         │         │          │
│    └────┬────┘        └─────────┘         └─────────┘          │
│         │                                                       │
└─────────┼───────────────────────────────────────────────────────┘
          │ WebSocket
          │
┌─────────┴──────────┐     ┌─────────────────┐
│   Slack Workspace  │     │   Claude API    │
│                    │     │   (Anthropic)   │
│  #leadership       │     └─────────────────┘
│  #protocol-eng     │
│  #risk-compliance  │     ┌─────────────────┐
│  #data-intel       │     │   Base RPC      │
│  #merchant-growth  │     │   (Alchemy)     │
│  #brand-comms      │     └─────────────────┘
│  #war-room         │
│  #settlement-ops   │     ┌─────────────────┐
│  #compliance-alerts│     │  Coinbase CDP   │
│  #human-override   │     │  Facilitator    │
└────────────────────┘     └─────────────────┘
```

---

## Dependencies

| Dependency | Type | Purpose |
|------------|------|---------|
| All Shulam repos | Internal | Agents work across repos |
| facilitator | Internal | Settlement data, verification pipeline |
| contracts | Internal | Base contract patterns (OpenZeppelin) |
| compliance | Internal | OFAC/KYT screening service |
| Claude API | External | Agent reasoning (Anthropic) |
| Slack Bolt | External | Communication backbone (Socket Mode) |
| BullMQ + Redis | External | Streaming agent event queues |
| SQLite / PostgreSQL | External | State management, audit log |
| node-cron | External | Heartbeat scheduler |
| GitHub API | External | PR management |
| OpenZeppelin | External | ERC-5192 (SBT), access control |
| Foundry | External | Compile, test, deploy identity contracts |
| Coinbase CDP SDK | External | Settlement path integration |

---

## The 21 Apostles

### Leadership Team

| # | Name | Role | Mode | Heartbeat |
|---|------|------|------|-----------|
| 1 | Peter | Squad Lead | Heartbeat | :00 |
| 2 | Barnabas | Scrum Master | Heartbeat | :15 |
| 3 | Paul | Project Manager | Heartbeat | :13 |

### Risk & Compliance Team

| # | Name | Role | Mode | Heartbeat |
|---|------|------|------|-----------|
| 4 | Samuel | Chief Compliance Officer | **Streaming** | :18 |
| 5 | Gabriel | Chief Security Officer | **Streaming** | :19 |
| 6 | Levi | Treasury & Settlement | **Streaming** | :20 |

### Protocol Engineering Team

| # | Name | Role | Mode | Heartbeat |
|---|------|------|------|-----------|
| 7 | Andrew | Lead Protocol Engineer | Heartbeat | :01 |
| 8 | James | Verification Engine | **Streaming** | :02 |
| 9 | Thaddaeus | Scheme Architect | Heartbeat | :08 |
| 10 | Judas | Network Operations | **Streaming** | :12 |

### Data & Intelligence Team

| # | Name | Role | Mode | Heartbeat |
|---|------|------|------|-----------|
| 11 | Ezra | Data Science / ML / Fraud | **Streaming** | :16 |
| 12 | Matthias | Analytics & Reporting | Heartbeat | :11 |
| 13 | Bartholomew | Market Research | Heartbeat | :05 |

### Merchant & Growth Team

| # | Name | Role | Mode | Heartbeat |
|---|------|------|------|-----------|
| 14 | Simon | Merchant Acquisition | Heartbeat | :09 |
| 15 | John | Ecosystem Intelligence | Heartbeat | :03 |
| 16 | Matthew | Developer Relations | Heartbeat | :06 |
| 17 | Philip | Merchant Support | Heartbeat | :04 |
| 18 | Zacchaeus | Enterprise Sales | Heartbeat | :14 |

### Brand & Communications Team

| # | Name | Role | Mode | Heartbeat |
|---|------|------|------|-----------|
| 19 | Mary | Creative Director | Heartbeat | :17 |
| 20 | Thomas | Community Manager | Heartbeat | :07 |

### Operations

| # | Name | Role | Mode | Heartbeat |
|---|------|------|------|-----------|
| 21 | Luke | Scribe / Changelog | Heartbeat | :10 |

**6 Streaming Agents** (continuous event-driven, critical payment path): James, Samuel, Gabriel, Levi, Judas, Ezra
**15 Heartbeat Agents** (minute-level cycle, periodic work): All others

---

## Milestones

### Phase 1: Foundation (Weeks 1-2)

#### M1: Soul Engine Core
- [ ] Soul Engine orchestrator (`core/engine.ts`)
- [ ] Soul Router — routes messages/events to correct soul
- [ ] BaseSoul abstract class + shared types
- [ ] 21 soul module stubs organized by team
- [ ] Soul configuration registry with all agent attributes
- [ ] Entry point boots everything

#### M2: Slack Communication Layer
- [ ] Slack Bolt app initialization (Socket Mode)
- [ ] Channel management — create 10 team/special channels
- [ ] Soul-to-Slack messenger abstraction (per-soul identity)
- [ ] Slash command handlers (`/souls-status`, `/souls-override`, `/souls-escalate`)
- [ ] Button/modal interaction handlers

#### M3: Heartbeat System
- [ ] node-cron heartbeat scheduler (21 minute-slots)
- [ ] Health monitoring + failure detection per soul
- [ ] Heartbeat agent scan/triage/action loop
- [ ] Agent status tracking (healthy/degraded/failed)
- [ ] Missed-heartbeat alerting (2 min → alert, 6 min → activate backup)

#### M4: Claude Reasoning Integration
- [ ] Claude API client wrapper
- [ ] Per-soul system prompts (personality, key belief, communication style)
- [ ] Reasoning chain management (context accumulation)
- [ ] Working/short-term/long-term memory tiers
- [ ] SQLite state management + migrations + audit logging

### Phase 2: Streaming & Protocol (Weeks 3-4)

#### M5: Streaming Engine
- [ ] BullMQ + Redis streaming workers
- [ ] Streaming agent event loop for 6 agents
- [ ] Backpressure handling (configurable queue depth threshold)
- [ ] Circuit breaker pattern for downstream dependency failures
- [ ] P99 latency monitoring with automatic alerting
- [ ] Auto-scaling based on queue depth

#### M6: Payment Protocol Agents
- [ ] JAMES: x402 verification pipeline (EIP-3009 signature validation, nonce, balance)
- [ ] SAMUEL: OFAC/SDN screening integration + KYB workflow
- [ ] LEVI: Multi-path settlement routing (Coinbase Business → on-chain → bank rails)
- [ ] THADDAEUS: Scheme-specific verification/settlement logic (exact, upto, escrow)
- [ ] Inter-soul messaging protocol (Signal interface, pub/sub)

#### M7: Agent Identity (SBT)
- [ ] Soul-bound token (SBT) contract for agent identity attestations
- [ ] `AgentRegistry` contract — register agent address, owner, capabilities
- [ ] Agent attestation: link on-chain address to verifiable identity
- [ ] `isRegisteredAgent(address)` view function for trust verification
- [ ] Agent capability claims: what services the agent provides
- [ ] Revocation: owner can revoke an agent's SBT
- [ ] Off-chain metadata URI: agent description, supported endpoints, pricing

### Phase 3: Intelligence (Weeks 5-6)

#### M8: Reputation Scoring
- [ ] `ReputationOracle` contract — tracks agent transaction history on-chain
- [ ] Reputation score derived from: payment count, volume, success rate, age
- [ ] `getReputation(address)` returns score (0-1000) and history summary
- [ ] Score updates on each facilitator settlement
- [ ] Decay function: reputation degrades if agent is inactive
- [ ] Dispute impact: failed escrows and chargebacks reduce reputation
- [ ] Integration with buyer-sdk/agent `compareProviders()`

#### M9: Intelligence & Swarms
- [ ] Cross-soul signal amplification (3+ agents + confidence >0.8 → auto-escalate)
- [ ] Swarm protocol activation (9 defined swarm types)
- [ ] EZRA: Fraud detection ML model integration (Fraud Scorer, Merchant Risk, Anomaly Detector, Structuring Detector)
- [ ] MATTHIAS: Acquisition readiness dashboard with real metrics
- [ ] JOHN: Coinbase relationship health scoring (0-100, 7-factor weighted)
- [ ] JOHN: Coinbase sensitivity classification (GREEN/YELLOW/AMBER/RED + sequence dependency)

### Phase 4: Production Hardening (Weeks 7-8)

#### M10: Resilience & Recovery
- [ ] SAMUEL degraded mode (cached compliance, 4h max)
- [ ] JAMES degraded mode (exact-only verification, 2h max)
- [ ] Backup agent activation for all 21 agents
- [ ] Escalation + war room protocol (swarm coordination)
- [ ] Human override interface (`/souls-override` → Slack modal)
- [ ] Chaos testing (kill souls, verify recovery)

#### M11: GitHub Operations
- [ ] PR creation + automated code review
- [ ] Auto-merge with guardrails (1 approval, checks pass, <500 LOC, no protected files)
- [ ] Branch management + conflict resolution
- [ ] Protected file enforcement (contracts/, .env, etc.)

#### M12: Production Readiness
- [ ] Migrate SQLite → PostgreSQL
- [ ] Redis persistence configuration
- [ ] Full integration test suite with behavior scenarios
- [ ] Load testing (verification throughput under stress)
- [ ] Compliance audit: all agent actions logged immutably

---

## User Stories (Gherkin)

### Epic 1: Agent Lifecycle

```gherkin
Feature: Agent Lifecycle Management
  As an operator
  I want to manage agent lifecycles
  So that agents run reliably

  Background:
    Given the Soul Engine is running
    And Slack workspace is connected

  Scenario: Start the Soul Engine
    When I run `npm start`
    Then the Soul Engine boots
    And all 21 soul modules are loaded
    And the heartbeat scheduler starts
    And streaming workers connect to Redis
    And a startup message posts to #leadership

  Scenario: View all agent status
    When I type `/souls-status` in Slack
    Then I see a table of all 21 agents:
      | agent       | team       | mode      | status  | heartbeat |
      | Peter       | Leadership | heartbeat | healthy | :00       |
      | James       | Protocol   | streaming | healthy | :02       |
      | Samuel      | Risk       | streaming | healthy | :18       |
      | ...         | ...        | ...       | ...     | ...       |

  Scenario: Detect failed heartbeat agent
    Given agent "Andrew" misses 3 consecutive heartbeats
    When the health monitor detects the failure
    Then Andrew's status changes to "failed"
    And an alert posts to #protocol-eng
    And Gabriel is activated as backup (infrastructure only)

  Scenario: Detect failed streaming agent
    Given streaming agent "James" stops processing events
    When the health monitor detects queue depth exceeding threshold
    Then James's status changes to "degraded"
    And an alert posts to #settlement-ops
    And ANDREW enters degraded verification mode

  Scenario: Emergency stop all agents
    When I type `/souls-override halt-all` in #human-override
    Then all agents stop immediately
    And in-progress tasks are preserved
    And "EMERGENCY HALT" posts to all channels
```

### Epic 2: Task Management

```gherkin
Feature: Task Queue Management
  As Peter (Squad Lead)
  I want to manage the task queue
  So that work is distributed effectively

  Scenario: Create a new task
    Given a feature request comes in
    When Peter creates a task:
      """
      {
        type: "code_write",
        title: "Implement /verify endpoint",
        assignee: "Andrew",
        priority: "high",
        repo: "facilitator"
      }
      """
    Then the task is added to the queue
    And Andrew is notified in #protocol-eng
    And the task status is "pending"

  Scenario: Agent picks up task
    Given Andrew is idle
    And there is a high-priority task for Andrew
    When Andrew's heartbeat runs at :01
    Then Andrew picks up the task
    And task status changes to "in_progress"
    And Andrew begins working

  Scenario: Complete a task
    Given Andrew is working on task "task_123"
    When Andrew finishes the work
    Then task status changes to "completed"
    And results are recorded
    And Peter is notified

  Scenario: Delegate task to another agent
    Given Andrew has a design task
    When Andrew delegates to Thaddaeus
    Then the task assignee changes
    And Thaddaeus is notified in #protocol-eng
```

### Epic 3: GitHub Operations

```gherkin
Feature: GitHub PR Management
  As Andrew (Developer)
  I want to create and manage PRs
  So that code changes are reviewed and merged

  Background:
    Given Andrew is running
    And GitHub credentials are configured

  Scenario: Create a pull request
    Given Andrew has completed code changes
    When Andrew creates a PR
    Then a PR is created on GitHub
    And Thomas (QA) is assigned as reviewer
    And PR link posts to #protocol-eng

  Scenario: Auto-merge approved PR
    Given a PR has:
      | condition              | met   |
      | 1 approval             | yes   |
      | all checks pass        | yes   |
      | no protected files     | yes   |
      | < 500 lines changed    | yes   |
    When auto-merge criteria are checked
    Then the PR is automatically merged
    And merge is logged to audit
    And Slack is notified

  Scenario: Block auto-merge for protected files
    Given a PR modifies contracts/ShulamEscrow.sol
    When auto-merge criteria are checked
    Then auto-merge is blocked
    And PR is flagged for human review in #human-override

  Scenario: Handle merge conflicts
    Given a PR has merge conflicts
    When Andrew detects the conflict
    Then Andrew rebases the branch
    And resolves conflicts
    And re-requests review if needed
```

### Epic 4: Scheduled Operations

```gherkin
Feature: Scheduled Agent Tasks
  As an operator
  I want agents to run scheduled tasks
  So that recurring work happens automatically

  Scenario: Peter daily morning briefing
    Given it is 8:00 AM (Peter's :00 heartbeat)
    When Peter gathers status from all agents
    Then Peter posts to #leadership:
      """
      Daily Briefing:

      Metrics:
      - Transactions yesterday: 47
      - Volume: $12,340 USDC
      - Uptime: 99.9%

      Completed:
      - Andrew: Implemented /verify endpoint
      - Thomas: Reviewed 3 PRs

      Today's Priorities:
      1. Complete settlement engine
      2. Fix webhook retry logic
      """

  Scenario: Luke end-of-day changelog
    Given it is 6:00 PM (Luke's :10 heartbeat)
    When Luke compiles all merged PRs
    Then Luke writes changelog entry
    And Luke commits to docs repo
    And Luke posts summary to #leadership

  Scenario: Matthias daily metrics report
    Given it is 6:00 AM (Matthias's :11 heartbeat)
    When Matthias pulls metrics from all sources
    Then Matthias refreshes the acquisition dashboard
    And Matthias posts KPI summary to #data-intel
```

### Epic 5: Inter-Agent Communication

```gherkin
Feature: Agent Collaboration
  As Peter (Squad Lead)
  I want agents to collaborate
  So that complex tasks are completed

  Scenario: Peter delegates multi-part task
    Given a new feature requires API, UI, tests, and docs
    When Peter creates a project task
    Then Peter creates subtasks:
      | component | agent     | channel         |
      | API       | Andrew    | #protocol-eng   |
      | Tests     | Thomas    | #brand-comms    |
      | Docs      | Matthew   | #merchant-growth|
    And dependencies are tracked

  Scenario: Cross-soul signal sharing
    Given James detects verification failures increasing for one merchant
    And Ezra detects transaction pattern anomaly from same merchant
    And Samuel detects merchant's wallet in sanctions news
    When combined signal confidence exceeds 0.8
    Then Peter triggers compliance war room
    And Samuel initiates enhanced due diligence
    And Levi holds pending settlements
    And war room posts to #war-room

  Scenario: Escalate blocker to Peter
    Given Andrew is blocked on a task
    When Andrew escalates to Peter
    Then Peter receives the escalation in #leadership
    And Peter assesses the blocker
    And Peter either resolves or escalates to human in #human-override
```

### Epic 6: Safety & Guardrails

```gherkin
Feature: Agent Safety Guardrails
  As an operator
  I want safety guardrails
  So that agents don't cause harm

  Scenario: Prevent modification of secrets
    Given Andrew tries to modify .env file
    When the guardrail checks the file
    Then the modification is blocked
    And Andrew receives error "Protected file"
    And the attempt is logged to audit

  Scenario: Rate limit API calls
    Given Andrew has made 99 API calls this hour
    And the limit is 100 calls/hour
    When Andrew tries call #101
    Then the call is delayed until cooldown

  Scenario: Limit auto-merges per day
    Given 10 PRs have been auto-merged today
    When PR #11 meets auto-merge criteria
    Then auto-merge is deferred to tomorrow
    And alert notes "Daily auto-merge limit reached"

  Scenario: Audit all agent actions
    Given any agent performs an action
    Then the action is logged to immutable audit:
      | field      | recorded |
      | agent      | yes      |
      | action     | yes      |
      | target     | yes      |
      | timestamp  | yes      |
      | result     | yes      |
    And logs are searchable via SQLite
```

### Epic 7: Agent Identity

```gherkin
Feature: Agent Identity via Soul-Bound Tokens
  As an agent operator
  I want to register my agent's identity on-chain
  So that other agents and merchants can verify trustworthiness

  Scenario: Register a new agent
    Given I own a wallet with an agent private key
    When I call AgentRegistry.register(agentAddress, metadataUri)
    Then a non-transferable SBT is minted to the agent address
    And the metadata URI points to agent capabilities JSON
    And isRegisteredAgent(agentAddress) returns true

  Scenario: Verify agent identity before accepting payment
    Given a merchant receives a payment with an agent identity header
    When the merchant calls isRegisteredAgent(agentAddress)
    Then the merchant confirms the agent has a valid SBT
    And can read the agent's metadata (capabilities, owner, domain)

  Scenario: Revoke a compromised agent
    Given an agent's private key has been compromised
    When the owner calls AgentRegistry.revoke(agentAddress)
    Then the agent's SBT is burned
    And isRegisteredAgent(agentAddress) returns false

  Scenario: Query agent capabilities
    Given an agent is registered with capabilities ["data-provider", "compute"]
    When I call AgentRegistry.getCapabilities(agentAddress)
    Then I receive the list of declared capabilities
```

### Epic 8: Reputation Scoring

```gherkin
Feature: On-Chain Reputation for Agents
  As a buyer agent
  I want to check a provider's reputation before paying
  So that I can avoid unreliable or fraudulent providers

  Scenario: Build reputation through successful transactions
    Given an agent has completed 100 successful payments
    When I call ReputationOracle.getReputation(agentAddress)
    Then I receive a score of 850/1000
    And a summary: { txCount: 100, volume: "5000.00", successRate: 0.99 }

  Scenario: Reputation degrades on disputes
    Given an agent has a reputation of 850
    When 3 escrow disputes are filed against the agent
    Then the reputation drops to 720

  Scenario: Reputation decays with inactivity
    Given an agent has been inactive for 90 days
    When I check the agent's reputation
    Then the score includes a decay penalty

  Scenario: Buyer agent uses reputation for provider selection
    Given I have discovered 3 providers for /api/weather
    When I call compareProviders(manifests) with reputation weighting
    Then providers are ranked by: (price * 0.4) + (reputation * 0.6)
```

### Epic 9: Streaming Engine

```gherkin
Feature: Real-Time Streaming Agent Processing
  As a streaming agent (James, Samuel, Gabriel, Levi, Judas, Ezra)
  I want to process events continuously
  So that payment-critical operations have sub-second latency

  Background:
    Given BullMQ workers are connected to Redis
    And streaming agents are registered

  Scenario: Process verification event in real-time
    Given a payment verification request arrives
    When the event is added to the James verification queue
    Then James processes the verification within 500ms (p99)
    And the result is published to the settlement queue

  Scenario: Handle backpressure under load
    Given the James verification queue exceeds 1000 events
    When backpressure threshold is breached
    Then James enables backpressure mode
    And P0 events are prioritized
    And P3+ events are deferred
    And an alert posts to #settlement-ops

  Scenario: Circuit breaker activates on downstream failure
    Given Samuel's OFAC screening API is unresponsive
    When 5 consecutive failures occur within 30 seconds
    Then the circuit breaker opens
    And Samuel enters degraded mode (cached compliance)
    And an alert posts to #risk-compliance

  Scenario: Streaming agent reports health at heartbeat
    Given James is processing verifications continuously
    When James's :02 heartbeat fires
    Then James reports health metrics:
      | metric              | value   |
      | queue_depth         | 42      |
      | p99_latency_ms      | 287     |
      | events_processed_1m | 1,203   |
      | error_rate          | 0.001   |
    And health is posted to #protocol-eng
```

### Epic 10: Payment Verification (JAMES)

```gherkin
Feature: x402 Payment Verification Pipeline
  As JAMES (Verification Engine)
  I want to verify x402 payment signatures
  So that only valid payments proceed to settlement

  Background:
    Given JAMES streaming worker is running
    And the facilitator is connected

  Scenario: Verify valid EIP-3009 TransferWithAuthorization
    Given a buyer submits a payment with a valid EIP-3009 signature
    When JAMES receives the verification request
    Then JAMES decodes the payment payload from base64
    And validates the scheme (exact)
    And verifies the EIP-3009 signature via EIP-712 typed data recovery
    And checks nonce uniqueness (replay protection)
    And confirms USDC balance >= payment amount
    And routes to SAMUEL for compliance screening
    And authorizes settlement to LEVI
    And returns verification result within 500ms

  Scenario: Reject replayed nonce
    Given a buyer submits a payment with a previously-used nonce
    When JAMES checks nonce uniqueness
    Then JAMES rejects the payment immediately
    And returns error code "NONCE_ALREADY_USED"
    And logs a replay attack attempt
    And alerts GABRIEL (security)

  Scenario: Reject insufficient balance
    Given a buyer submits a payment for 100 USDC
    And the buyer's wallet holds 50 USDC
    When JAMES checks USDC balance
    Then JAMES rejects the payment
    And returns error code "INSUFFICIENT_BALANCE"

  Scenario: Handle upto scheme verification
    Given a buyer submits an upto-scheme payment with maxAmount=100 USDC
    When JAMES verifies the payment
    Then JAMES validates the upto scheme constraints
    And confirms actualAmount <= maxAmount
    And passes scheme-specific logic to THADDAEUS
    And proceeds with standard verification pipeline

  Scenario: JAMES degraded mode (exact-only)
    Given JAMES has failed and ANDREW is activated as backup
    When a payment verification request arrives
    Then ANDREW processes exact-scheme payments only
    And rejects upto, escrow, recurring, deferred schemes
    And verification latency SLA relaxes to 2000ms
    And all verifications are logged with degraded-mode flag
    And after 2 hours without JAMES recovery, all verifications halt
```

### Epic 11: Compliance Screening (SAMUEL)

```gherkin
Feature: OFAC Sanctions Screening and Compliance
  As SAMUEL (Chief Compliance Officer)
  I want to screen all wallets and merchants
  So that Shulam never processes sanctioned transactions

  Background:
    Given SAMUEL streaming worker is running
    And OFAC SDN list is loaded

  Scenario: Screen wallet with no match
    Given a buyer wallet "0xABC..." is submitted for screening
    When SAMUEL screens against OFAC SDN, EU, UN sanctions lists
    And match confidence is <70%
    Then SAMUEL returns CLEAR status
    And logs the screening result
    And transaction proceeds

  Scenario: Screen wallet with partial match
    Given a buyer wallet matches a sanctioned entity at 80% confidence
    When SAMUEL screens the wallet
    Then SAMUEL returns REVIEW status
    And settlement is placed on HOLD
    And an alert posts to #compliance-alerts
    And manual review must complete within 4 hours

  Scenario: Screen wallet with exact match
    Given a buyer wallet matches an SDN entry at 97% confidence
    When SAMUEL screens the wallet
    Then SAMUEL returns BLOCK status
    And settlement is immediately halted
    And escalation posts to #human-override
    And SAR filing is initiated
    And all associated transactions are frozen

  Scenario: KYB screening for new merchant
    Given a new merchant submits onboarding request
    When SAMUEL initiates KYB screening
    Then SAMUEL verifies business registration
    And screens merchant principals against PEP lists
    And assesses geographic risk
    And returns KYB status (CLEARED / REVIEW / REJECTED)
    And logs all screening decisions to audit

  Scenario: SAMUEL degraded mode (cached compliance)
    Given SAMUEL has failed
    When transactions continue arriving
    Then only merchants with CLEARED KYB (last 30 days) are processed
    And only wallets with clean OFAC history (last 24 hours) proceed
    And all new merchants are HALTED
    And all new/unseen wallets are HALTED
    And all degraded-mode transactions are flagged for re-screening
    And after 4 hours without recovery, all settlements halt
```

### Epic 12: Settlement Operations (LEVI)

```gherkin
Feature: Multi-Path Settlement Execution
  As LEVI (Treasury & Settlement Officer)
  I want to execute settlements through the optimal path
  So that merchants receive funds reliably and efficiently

  Background:
    Given LEVI streaming worker is running
    And Coinbase Business API is configured

  Scenario: Settle via Coinbase Business (default path)
    Given a verified payment of 100 USDC needs settlement
    And Coinbase Business API is available
    When LEVI executes settlement
    Then LEVI routes through Coinbase Business API
    And merchant receives 99.25 USDC (after 0.75% facilitator fee)
    And settlement is logged with path "coinbase_business"
    And reconciliation entry is created

  Scenario: Failover to direct on-chain settlement
    Given Coinbase Business API is unavailable
    When LEVI attempts settlement
    Then LEVI detects Coinbase failure
    And fails over to direct on-chain settlement
    And executes EIP-3009 transferWithAuthorization (buyer → facilitator)
    And executes USDC.transfer (facilitator → merchant, netAmount)
    And alerts JOHN about Coinbase service degradation
    And settlement is logged with path "on_chain"

  Scenario: Enterprise bank rail settlement
    Given an enterprise merchant has bank rail settlement configured
    When LEVI executes settlement
    Then LEVI requires L1 approval (human)
    And SAMUEL runs full compliance suite
    And settlement is logged with path "bank_rails"

  Scenario: Daily reconciliation
    Given it is LEVI's :20 heartbeat
    When LEVI runs reconciliation
    Then LEVI compares all settlement records against on-chain state
    And flags any discrepancies
    And reports cash position across all accounts
    And posts reconciliation summary to #settlement-ops
```

### Epic 13: Intelligence & Swarms

```gherkin
Feature: Cross-Soul Intelligence and Swarm Protocols
  As the Souls collective
  I want to share signals and form swarms
  So that complex threats are detected and resolved collaboratively

  Background:
    Given all 21 souls are running
    And the signal bus is active

  Scenario: Swarm activation on settlement incident
    Given JAMES detects verification failures increasing
    And LEVI detects settlement reconciliation drift
    When combined signal confidence exceeds 0.8
    Then PETER activates the settlement incident swarm
    And swarm members (James, Levi, Gabriel, Andrew) join #war-room
    And all pending settlements are held
    And incident is investigated collaboratively

  Scenario: EZRA detects fraud pattern
    Given EZRA's fraud scoring model flags a transaction cluster
    When fraud probability exceeds 0.85
    Then EZRA publishes a high-severity signal to SAMUEL
    And SAMUEL initiates enhanced due diligence
    And GABRIEL reviews API access patterns
    And transactions from flagged wallets are held

  Scenario: MATTHIAS updates acquisition dashboard
    Given it is Monday morning (Peter's weekly review)
    When MATTHIAS presents top-line metrics
    Then acquisition readiness scores are updated:
      | metric                    | current | target  |
      | Monthly transactions      | 50K     | 100K    |
      | Monthly settlement volume | $250K   | $500K   |
      | Active merchants          | 30      | 50      |
      | Coinbase health score     | 62      | 70      |
    And off-track metrics are flagged for action

  Scenario: JOHN assesses Coinbase ecosystem shift
    Given Coinbase announces a new CDP feature
    When JOHN detects the announcement within 2 hours
    Then JOHN publishes an ecosystem signal
    And JOHN classifies the impact (GREEN/YELLOW/AMBER/RED)
    And JOHN assesses impact on Shulam positioning
    And PETER + MARY are notified if impact is AMBER or RED
    And strategic response is planned
```

### Epic 14: Coinbase Relationship Management

```gherkin
Feature: Coinbase Sensitivity Classification
  As JOHN (Ecosystem Intelligence Lead)
  I want to classify all Shulam actions for Coinbase sensitivity
  So that we maintain optimal positioning for acquisition

  Scenario: Classify GREEN action (complementary)
    Given Shulam plans to ship a new payment scheme
    When JOHN classifies the action
    Then sensitivity level is GREEN (complementary to Coinbase)
    And only JOHN advisory is required
    And action proceeds

  Scenario: Classify YELLOW action (ambiguous)
    Given Shulam plans to add Ethereum mainnet support
    And formal Coinbase partnership has not been established
    When JOHN classifies the action
    Then sensitivity level is YELLOW (ambiguous framing)
    And JOHN + MARY review is required
    And messaging must frame as "extending x402 reach"

  Scenario: Classify RED action (competitive signal)
    Given Shulam plans to announce multi-facilitator routing
    And formal Coinbase partnership has not been established
    When JOHN classifies the action
    Then sensitivity level is RED (competitive alternative)
    And JOHN + PETER + Human approval is required
    And action is deferred to Phase 4 (acquisition window)

  Scenario: Sequence dependency reclassification
    Given adding Ethereum mainnet was YELLOW before partnership
    When formal Coinbase CDP partnership is established
    Then JOHN reclassifies Ethereum mainnet to GREEN
    And action is approved for immediate execution

  Scenario: Weekly relationship health score update
    Given JOHN maintains the Coinbase Relationship Health Score
    When JOHN computes the weekly score
    Then the 7-factor weighted score (0-100) is updated:
      | factor                    | weight | score |
      | Partnership Status        | 20%    | 40    |
      | Open-Source Contributions  | 15%    | 25    |
      | Settlement Volume via CB   | 15%    | 90    |
      | CDP Team Engagement        | 15%    | 30    |
      | Competitive Perception     | 15%    | 85    |
      | Ecosystem Visibility       | 10%    | 20    |
      | Talent Network Overlap     | 10%    | 10    |
    And score is posted to #leadership
    And PETER reviews if score drops below 60
```

### Epic 15: Resilience & Recovery

```gherkin
Feature: Failure Detection and Graceful Recovery
  As an operator
  I want resilient failure handling
  So that payment operations degrade gracefully instead of halting

  Background:
    Given all 21 souls are running
    And backup agent mappings are configured

  Scenario: Heartbeat agent fails — activate backup
    Given Andrew (Protocol Engineer) has missed 3 heartbeats
    When the health monitor triggers backup activation
    Then Gabriel is activated as Andrew's backup (infrastructure only)
    And an alert posts to #protocol-eng
    And Andrew's tasks are preserved for recovery

  Scenario: Streaming agent queue overload
    Given James's verification queue depth exceeds auto-scale threshold
    When the streaming engine detects overload
    Then additional James worker instances are spawned
    And queue depth begins to decrease
    And P99 latency stays below 500ms

  Scenario: War room activation on P0 incident
    Given a settlement failure is detected (P0 severity)
    When PETER activates the war room
    Then swarm members (James, Levi, Gabriel, Andrew) are assembled
    And #war-room channel is activated
    And all pending settlements are held
    And incident timeline starts logging
    And human is notified within 15 minutes

  Scenario: Recovery audit after degraded mode
    Given SAMUEL operated in degraded mode for 2 hours
    When SAMUEL recovers
    Then all wallets transacted during degraded mode are re-screened
    And transaction monitoring is re-run on degraded-mode transactions
    And a compliance incident report is generated
    And human reviews and signs off on degraded-mode operations

  Scenario: Human override via Slack
    Given SAMUEL has blocked a transaction
    And human review determines false positive
    When human uses `/souls-override unblock TX_123` in #human-override
    Then the override is logged with human identity
    And SAMUEL releases the hold
    And transaction proceeds to settlement
    And override is recorded in immutable audit log
```

### Epic 16: Heartbeat & Health Monitoring

```gherkin
Feature: Minute-Level Heartbeat System
  As the Soul Engine
  I want each agent to report health every minute
  So that failures are detected within 2 minutes

  Background:
    Given the heartbeat scheduler is running
    And 21 minute-slots are assigned

  Scenario: Heartbeat agent executes work cycle
    Given it is minute :01 (Andrew's slot)
    When the heartbeat scheduler fires Andrew
    Then Andrew performs his scan/triage/action:
      | step   | action                                    |
      | scan   | Check build health, deployment queue       |
      | triage | Prioritize by P0 > P1 > P2                |
      | action | Process highest-priority item              |
      | report | Post status to #protocol-eng               |

  Scenario: Streaming agent reports health at heartbeat
    Given it is minute :02 (James's health report slot)
    When the heartbeat scheduler fires James
    Then James reports health metrics (not operational work):
      | metric              | value   |
      | verification_p99_ms | 287     |
      | nonce_registry_size | 12,405  |
      | queue_depth         | 42      |
      | error_rate_1m       | 0.001   |

  Scenario: Judas dual heartbeat slots
    Given Judas has slots at :10 and :12
    When minute :10 fires
    Then Judas performs cross-network analysis (gas arbitrage, failover validation)
    When minute :12 fires
    Then Judas reports standard health (network status, RPC latency, gas prices)

  Scenario: Detect and alert on missed heartbeat
    Given Andrew misses his :01 heartbeat
    When 2 minutes pass without response
    Then Barnabas is alerted (impediment detected)
    When 6 minutes pass without response
    Then Gabriel is activated as Andrew's backup
    And alert posts to #protocol-eng with recovery instructions
```

---

## Directory Structure

```
souls/
├── package.json
├── tsconfig.json
├── .env
│
├── data/
│   └── shulam.db                  # SQLite database (auto-created)
│
├── src/
│   ├── index.ts                   # Entry point — boots everything
│   │
│   ├── core/
│   │   ├── engine.ts              # Soul Engine — orchestrates all souls
│   │   ├── router.ts              # Routes messages/events to correct soul
│   │   ├── heartbeat.ts           # Heartbeat scheduler (node-cron)
│   │   ├── streaming.ts           # Streaming agent event loop (BullMQ)
│   │   ├── escalation.ts          # Escalation & war room logic
│   │   └── health.ts              # Health monitoring & failure detection
│   │
│   ├── slack/
│   │   ├── app.ts                 # Slack Bolt app initialization
│   │   ├── channels.ts            # Channel management & setup
│   │   ├── messenger.ts           # Soul-to-Slack messaging abstraction
│   │   ├── commands.ts            # Slash command handlers
│   │   └── interactions.ts        # Button/modal interaction handlers
│   │
│   ├── llm/
│   │   ├── claude.ts              # Claude API client wrapper
│   │   ├── prompts.ts             # System prompts per soul
│   │   └── reasoning.ts           # Reasoning chain management
│   │
│   ├── state/
│   │   ├── database.ts            # SQLite setup & migrations
│   │   ├── memory.ts              # Working / short-term / long-term memory
│   │   ├── redis.ts               # Redis client & pub/sub
│   │   └── audit.ts               # Immutable audit log
│   │
│   ├── souls/
│   │   ├── _base.ts               # BaseSoul abstract class
│   │   ├── _types.ts              # Shared soul types & interfaces
│   │   │
│   │   ├── leadership/
│   │   │   ├── peter.ts           # Squad Lead
│   │   │   ├── barnabas.ts        # Scrum Master
│   │   │   └── paul.ts            # Project Manager
│   │   │
│   │   ├── risk/
│   │   │   ├── samuel.ts          # Compliance + OFAC (STREAMING)
│   │   │   ├── gabriel.ts         # Security (STREAMING)
│   │   │   └── levi.ts            # Treasury & Settlement (STREAMING)
│   │   │
│   │   ├── protocol/
│   │   │   ├── andrew.ts          # Lead Protocol Engineer
│   │   │   ├── james.ts           # Verification Engine (STREAMING)
│   │   │   ├── thaddaeus.ts       # Scheme Architect
│   │   │   └── judas.ts           # Network Ops (STREAMING)
│   │   │
│   │   ├── data/
│   │   │   ├── ezra.ts            # ML / Fraud Detection (STREAMING)
│   │   │   ├── matthias.ts        # Analytics & Dashboards
│   │   │   └── bartholomew.ts     # Research
│   │   │
│   │   ├── merchant/
│   │   │   ├── simon.ts           # Merchant Acquisition
│   │   │   ├── john.ts            # Ecosystem Intelligence
│   │   │   ├── matthew.ts         # Developer Relations
│   │   │   ├── philip.ts          # Merchant Support
│   │   │   └── zacchaeus.ts       # Enterprise Sales
│   │   │
│   │   ├── brand/
│   │   │   ├── mary.ts            # Creative Director
│   │   │   └── thomas.ts          # Community Manager
│   │   │
│   │   └── operations/
│   │       └── luke.ts            # Scribe / Changelog
│   │
│   ├── protocols/
│   │   ├── compliance.ts          # OFAC screening, KYB, SAR
│   │   ├── verification.ts        # x402 payment verification pipeline
│   │   ├── settlement.ts          # Multi-path settlement routing
│   │   ├── coinbase.ts            # Coinbase sensitivity scoring
│   │   └── acquisition.ts         # Acquisition readiness metrics
│   │
│   └── config/
│       ├── souls.ts               # Soul configuration registry
│       ├── channels.ts            # Channel-to-team mapping
│       ├── soul-identities.ts     # Slack display config per soul
│       └── constants.ts           # System-wide constants
│
├── contracts/                     # On-chain identity & reputation
│   ├── src/
│   │   ├── AgentRegistry.sol      # SBT-based agent identity
│   │   └── ReputationOracle.sol   # Transaction-derived reputation scores
│   ├── test/
│   ├── script/
│   │   └── Deploy.s.sol
│   └── foundry.toml
│
├── docs/
│   ├── shulam-souls-framework.md
│   └── shulam-souls-implementation-guide.md
│
├── scripts/
│   ├── setup-channels.ts          # Creates Slack channels
│   ├── seed-db.ts                 # Seeds initial database state
│   └── health-check.ts            # CLI health check
│
└── tests/
    ├── souls/                     # Per-soul unit tests
    ├── integration/               # Cross-soul integration tests
    └── scenarios/                 # Behavior scenario tests
```

---

## Environment Variables

```bash
# AI
ANTHROPIC_API_KEY=

# Slack
SLACK_BOT_TOKEN=
SLACK_APP_TOKEN=
SLACK_SIGNING_SECRET=

# Redis (BullMQ streaming)
REDIS_URL=redis://localhost:6379

# GitHub
GITHUB_TOKEN=
GITHUB_ORG=Shulam-Inc

# Coinbase
COINBASE_API_KEY=
COINBASE_API_SECRET=

# Blockchain
BASE_RPC_URL=
ALCHEMY_API_KEY=

# Operations
NODE_ENV=development
LOG_LEVEL=info
```

---

## Agent Instructions

### For Peter (Squad Lead)
1. You orchestrate all 21 agents across 6 teams
2. Prioritize tasks based on business impact and acquisition readiness
3. Escalate blockers you cannot resolve to human in #human-override
4. Send daily briefings at 8 AM
5. Cannot be overridden except by human; cannot override SAMUEL on compliance

### For James (Verification Engine — Streaming)
1. Verify every EIP-3009 TransferWithAuthorization signature
2. Validate nonce uniqueness — zero tolerance for replay
3. Confirm USDC balance before authorizing settlement
4. Route to SAMUEL for compliance screening before LEVI settles
5. Target: <500ms p99 verification latency

### For Samuel (Compliance — Streaming)
1. Screen ALL wallets against OFAC/SDN lists before settlement
2. Exact match (>95%) → BLOCK + escalate + file SAR
3. Partial match (70-95%) → HOLD + manual review within 4 hours
4. Override authority: can block ANY agent, ANY transaction, at ANY time
5. Only human can override SAMUEL

### For Levi (Treasury & Settlement — Streaming)
1. Default settlement path: Coinbase Business API
2. Failover: Direct on-chain (EIP-3009 transferWithAuthorization)
3. Enterprise: Bank rails (L1 approval required)
4. Reconcile all settlements daily — no exceptions
5. Every satoshi is accounted for

### For John (Ecosystem Intelligence)
1. Monitor all x402 Foundation activity and Coinbase CDP announcements
2. Classify every Shulam action for Coinbase sensitivity (GREEN/YELLOW/AMBER/RED)
3. Maintain Coinbase Relationship Health Score (0-100)
4. Balance partnership positioning with strategic independence
5. Assess Coinbase CDP announcements within 2 hours

### For Luke (Scribe / Changelog)
1. Compile all merged PRs into daily changelog at 6 PM
2. Maintain records of all agent actions and decisions
3. Commit changelog entries to docs repo
4. Track decision history for audit and institutional memory

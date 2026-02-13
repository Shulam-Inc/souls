# Souls Project Plan

## Overview

Autonomous AI agent orchestration system for Shulam. 18 specialized agents ("Apostles") that operate proactively on schedules, scan for work, write code, and auto-merge PRs with safety guardrails.

---

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        SOULS                                 │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌─────────────────────────────────────────────────┐        │
│  │              Mac Mini Harness                    │        │
│  │                                                  │        │
│  │  ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐   │        │
│  │  │ Peter  │ │ James  │ │ Andrew │ │  ...   │   │        │
│  │  │ (Lead) │ │(Oracle)│ │ (Dev)  │ │  x18   │   │        │
│  │  └───┬────┘ └───┬────┘ └───┬────┘ └───┬────┘   │        │
│  │      │          │          │          │         │        │
│  │      └──────────┴──────────┴──────────┘         │        │
│  │                     │                            │        │
│  │           Mission Control (Convex)               │        │
│  └─────────────────────────────────────────────────┘        │
│                          │                                   │
│       ┌──────────────────┼──────────────────┐               │
│       ▼                  ▼                  ▼               │
│   ┌────────┐        ┌────────┐        ┌────────┐           │
│   │ GitHub │        │ Slack  │        │  APIs  │           │
│   │  PRs   │        │ Alerts │        │Services│           │
│   └────────┘        └────────┘        └────────┘           │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

## Dependencies

| Dependency | Type | Purpose |
|------------|------|---------|
| All repos | Internal | Agents work across repos |
| facilitator | Internal | Settlement data for reputation scoring |
| contracts | Internal | Base contract patterns (OpenZeppelin) |
| Claude API | External | Agent intelligence |
| GitHub API | External | PR management |
| Slack API | External | Notifications |
| Convex | External | Mission Control database |
| Redis | External | Task queue |
| OpenZeppelin | External | ERC-5192 (SBT), access control |
| Foundry | External | Compile, test, deploy identity contracts |

---

## The 18 Apostles

| # | Name | Role | Focus |
|---|------|------|-------|
| 1 | Peter | Lead | Orchestration, delegation |
| 2 | James | Oracle | Verification, blockchain |
| 3 | Andrew | Developer | Core code |
| 4 | John | Analyst | Strategy, intel |
| 5 | Philip | Support | Merchant help |
| 6 | Bartholomew | Researcher | Market research |
| 7 | Matthew | Content | Docs, blog |
| 8 | Thomas | QA | Testing, security |
| 9 | Thaddaeus | Designer | UI/UX |
| 10 | Simon | DevOps | CI/CD, infra |
| 11 | James-Lesser | Ops | Onboarding, KYB |
| 12 | Matthias | Data | Analytics |
| 13 | Judas | Growth | Outreach |
| 14 | Paul | Partnerships | BD, integrations |
| 15 | Timothy | Compliance | KYT, AML |
| 16 | Titus | Finance | Treasury |
| 17 | Barnabas | Community | Discord, Twitter |
| 18 | Luke | Scribe | Records, changelog |

---

## Milestones

### M1: Foundation
- [ ] Harness scaffold
- [ ] PM2 process management
- [ ] Session management
- [ ] Basic agent loop

### M2: Mission Control
- [ ] Convex schema
- [ ] Task queue
- [ ] Agent status tracking
- [ ] Inter-agent messaging

### M3: GitHub Integration
- [ ] PR creation
- [ ] Code review
- [ ] Auto-merge logic
- [ ] Branch management

### M4: Slack Integration
- [ ] Alert channels
- [ ] Daily briefings
- [ ] Interactive commands
- [ ] Escalation flows

### M5: Scheduling
- [ ] Cron-based tasks
- [ ] Agent-specific schedules
- [ ] Priority queue
- [ ] Backoff logic

### M6: Safety & Monitoring
- [ ] Guardrails enforcement
- [ ] Audit logging
- [ ] Rate limiting
- [ ] Kill switch

### M7: Agent Identity (SBT)
- [ ] Soul-bound token (SBT) contract for agent identity attestations
- [ ] `AgentRegistry` contract — register agent address, owner, capabilities
- [ ] Agent attestation: link on-chain address to verifiable identity (KYC, org, domain)
- [ ] `isRegisteredAgent(address)` view function for trust verification
- [ ] Agent capability claims: what services the agent provides (data, compute, etc.)
- [ ] Revocation: owner can revoke an agent's SBT (compromised key, decommissioned)
- [ ] Off-chain metadata URI: agent description, supported endpoints, pricing

### M8: Reputation Scoring
- [ ] `ReputationOracle` contract — tracks agent transaction history on-chain
- [ ] Reputation score derived from: payment count, volume, success rate, age
- [ ] `getReputation(address)` returns score (0-1000) and history summary
- [ ] Score updates on each facilitator settlement (facilitator calls `recordTransaction`)
- [ ] Decay function: reputation degrades if agent is inactive
- [ ] Dispute impact: failed escrows and chargebacks reduce reputation
- [ ] Leaderboard: top agents by reputation (for discovery ranking)
- [ ] Integration with buyer-sdk/agent `compareProviders()` for trust-weighted selection

---

## User Stories (Gherkin)

### Epic 1: Agent Lifecycle

```gherkin
Feature: Agent Lifecycle Management
  As an operator
  I want to manage agent lifecycles
  So that agents run reliably

  Background:
    Given the harness server is running
    And Mission Control is connected

  Scenario: Start an agent
    Given agent "Andrew" is not running
    When I run `npm run agent:start andrew`
    Then Andrew's process starts via PM2
    And Andrew connects to Mission Control
    And Andrew's status shows "online"
    And a startup message posts to Slack

  Scenario: Stop an agent
    Given agent "Andrew" is running
    When I run `npm run agent:stop andrew`
    Then Andrew's process stops gracefully
    And current tasks are saved
    And Andrew's status shows "offline"

  Scenario: Restart crashed agent
    Given agent "Andrew" crashed unexpectedly
    When PM2 detects the crash
    Then Andrew is automatically restarted
    And the crash is logged
    And an alert is sent to Slack
    And restart count is incremented

  Scenario: View all agent status
    When I run `npm run status`
    Then I see a table of all 18 agents:
      | agent    | status  | tasks | uptime |
      | Peter    | online  | 3     | 2h     |
      | James    | online  | 1     | 2h     |
      | Andrew   | online  | 5     | 2h     |
      | ...      | ...     | ...   | ...    |

  Scenario: Emergency stop all agents
    Given multiple agents are running
    When I run `npm run harness:stop`
    Then all agents stop immediately
    And in-progress tasks are preserved
    And "EMERGENCY STOP" posts to Slack
```

### Epic 2: Task Management

```gherkin
Feature: Task Queue Management
  As Peter (Lead)
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
    And Andrew is notified
    And the task status is "pending"

  Scenario: Agent picks up task
    Given Andrew is idle
    And there is a high-priority task for Andrew
    When Andrew's work loop runs
    Then Andrew picks up the task
    And task status changes to "in_progress"
    And Andrew begins working

  Scenario: Complete a task
    Given Andrew is working on task "task_123"
    When Andrew finishes the work
    And Andrew calls task.complete()
    Then task status changes to "completed"
    And results are recorded
    And Peter is notified

  Scenario: Task fails
    Given Andrew is working on a task
    When an unrecoverable error occurs
    Then Andrew marks task as "failed"
    And error details are logged
    And Peter is notified for reassignment

  Scenario: Delegate task to another agent
    Given Andrew has a design task
    When Andrew delegates to Thaddaeus
    Then the task assignee changes
    And Thaddaeus is notified
    And Andrew's queue is updated
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
    When Andrew creates a PR:
      """
      {
        repo: "facilitator",
        branch: "feature/verify-endpoint",
        title: "Implement /verify endpoint",
        body: "Adds EIP-3009 verification..."
      }
      """
    Then a PR is created on GitHub
    And PR link is recorded in Mission Control
    And Thomas is assigned as reviewer

  Scenario: Agent reviews PR
    Given Thomas is assigned to review a PR
    When Thomas reviews the code
    Then Thomas leaves review comments
    And approves or requests changes
    And review is recorded

  Scenario: Auto-merge approved PR
    Given a PR has:
      | condition              | met   |
      | 1 approval             | yes   |
      | all checks pass        | yes   |
      | no protected files     | yes   |
      | < 500 lines changed    | yes   |
    When auto-merge criteria are checked
    Then the PR is automatically merged
    And merge is logged
    And Slack is notified

  Scenario: Block auto-merge for protected files
    Given a PR modifies contracts/ShulamEscrow.sol
    And contracts/ is in protected paths
    When auto-merge criteria are checked
    Then auto-merge is blocked
    And PR is flagged for human review
    And alert is sent to Slack

  Scenario: Handle merge conflicts
    Given a PR has merge conflicts
    When Andrew tries to merge
    Then Andrew rebases the branch
    And resolves conflicts
    And pushes updated branch
    And re-requests review if needed
```

### Epic 4: Scheduled Operations

```gherkin
Feature: Scheduled Agent Tasks
  As an operator
  I want agents to run scheduled tasks
  So that recurring work happens automatically

  Scenario: Daily morning briefing
    Given it is 8:00 AM
    When the scheduler triggers Peter
    Then Peter gathers status from all agents
    And Peter compiles the daily briefing
    And Peter posts to #souls-briefing:
      """
      🌅 Good morning! Here's the daily briefing:
      
      ## 📊 Metrics
      - Transactions yesterday: 47
      - Volume: $12,340 USDC
      - Uptime: 99.9%
      
      ## ✅ Completed
      - Andrew: Implemented /verify endpoint
      - Thomas: Reviewed 3 PRs
      
      ## 🎯 Today's Priorities
      1. Complete settlement engine
      2. Fix webhook retry logic
      
      ## ⚠️ Alerts
      - None
      """

  Scenario: Hourly test suite
    Given it is the top of the hour
    When the scheduler triggers Thomas
    Then Thomas runs the test suite
    And results are recorded
    And failures are reported to Slack

  Scenario: Daily metrics report
    Given it is 6:00 AM
    When the scheduler triggers Matthias
    Then Matthias pulls metrics from all sources
    And Matthias generates dashboard data
    And Matthias posts summary to Slack

  Scenario: Real-time compliance screening
    Given a new transaction arrives
    When the facilitator calls compliance
    Then Timothy screens the transaction
    And response is returned in < 100ms
    And screening is logged

  Scenario: End of day changelog
    Given it is 6:00 PM
    When the scheduler triggers Luke
    Then Luke compiles all merged PRs
    And Luke writes changelog entry
    And Luke commits to docs repo
```

### Epic 5: Inter-Agent Communication

```gherkin
Feature: Agent Collaboration
  As Peter (Lead)
  I want agents to collaborate
  So that complex tasks are completed

  Scenario: Peter delegates multi-part task
    Given a new feature requires:
      | component | agent     |
      | API       | Andrew    |
      | UI        | Thaddaeus |
      | Tests     | Thomas    |
      | Docs      | Matthew   |
    When Peter creates a project task
    Then Peter creates subtasks for each agent
    And dependencies are tracked
    And progress is monitored

  Scenario: Andrew requests design from Thaddaeus
    Given Andrew is building a component
    And Andrew needs UI mockups
    When Andrew messages Thaddaeus:
      """
      Need mockups for PaymentModal component.
      Requirements: wallet selector, amount display, status.
      """
    Then Thaddaeus receives the message
    And Thaddaeus creates the mockups
    And Thaddaeus shares design files
    And Andrew is notified

  Scenario: Escalate blocker to Peter
    Given Andrew is blocked on a task
    And Andrew cannot resolve independently
    When Andrew escalates to Peter
    Then Peter receives the escalation
    And Peter assesses the blocker
    And Peter either resolves or escalates to human

  Scenario: Thomas requests security review
    Given Thomas found a potential vulnerability
    When Thomas flags for security review:
      """
      Potential reentrancy in settlement flow.
      File: src/settlement/index.ts
      Line: 142
      """
    Then alert goes to #souls-security
    And human review is requested
    And PR is blocked from merge
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
    And the attempt is logged

  Scenario: Rate limit API calls
    Given Andrew has made 99 API calls this hour
    And the limit is 100 calls/hour
    When Andrew tries call #101
    Then the call is delayed
    And Andrew is notified of rate limit
    And call proceeds after cooldown

  Scenario: Limit auto-merges per day
    Given 10 PRs have been auto-merged today
    And the daily limit is 10
    When PR #11 meets auto-merge criteria
    Then auto-merge is deferred to tomorrow
    And PR is queued
    And alert notes "Daily auto-merge limit reached"

  Scenario: Block deployment to mainnet
    Given Andrew tries to deploy to mainnet
    And mainnet deploys require human approval
    When the guardrail checks the action
    Then the deployment is blocked
    And human approval is requested
    And Andrew waits for approval

  Scenario: Audit all agent actions
    Given any agent performs an action
    Then the action is logged to audit:
      | field      | recorded |
      | agent      | yes      |
      | action     | yes      |
      | target     | yes      |
      | timestamp  | yes      |
      | result     | yes      |
    And logs are immutable
    And logs are searchable
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
    And future payments from this agent are treated as unverified

  Scenario: Query agent capabilities
    Given an agent is registered with capabilities ["data-provider", "compute"]
    When I call AgentRegistry.getCapabilities(agentAddress)
    Then I receive the list of declared capabilities
    And the metadata URI for full details
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
    And the dispute count is visible in the summary

  Scenario: Reputation decays with inactivity
    Given an agent has been inactive for 90 days
    When I check the agent's reputation
    Then the score includes a decay penalty
    And the "lastActive" timestamp is visible

  Scenario: Buyer agent uses reputation for provider selection
    Given I have discovered 3 providers for /api/weather
    When I call compareProviders(manifests) with reputation weighting
    Then providers are ranked by: (price * 0.4) + (reputation * 0.6)
    And the highest-reputation provider may rank above a cheaper one
```

---

## Directory Structure

```
souls/
├── README.md
├── package.json
├── agents/
│   ├── peter/
│   │   ├── SOUL.md
│   │   ├── config.yaml
│   │   └── prompts/
│   ├── andrew/
│   │   ├── SOUL.md
│   │   ├── config.yaml
│   │   └── prompts/
│   └── ... (18 agents)
├── contracts/                    # On-chain identity & reputation
│   ├── src/
│   │   ├── AgentRegistry.sol     # SBT-based agent identity
│   │   └── ReputationOracle.sol  # Transaction-derived reputation scores
│   ├── test/
│   ├── script/
│   │   └── Deploy.s.sol
│   └── foundry.toml
├── mission-control/
│   ├── schema.ts
│   ├── tasks.ts
│   └── agents.ts
├── harness/
│   ├── index.ts
│   ├── pm2.config.js
│   ├── scheduler.ts
│   └── slack.ts
├── guardrails/
│   ├── auto-merge.yaml
│   ├── protected-paths.yaml
│   └── rate-limits.yaml
└── scripts/
    ├── start-all.sh
    ├── stop-all.sh
    └── status.sh
```

---

## Environment Variables

```bash
# AI
ANTHROPIC_API_KEY=

# GitHub
GITHUB_TOKEN=
GITHUB_ORG=Shulam-Inc

# Slack
SLACK_BOT_TOKEN=
SLACK_WEBHOOK_URL=

# Mission Control
CONVEX_URL=
CONVEX_DEPLOY_KEY=

# Redis
REDIS_URL=

# Operations
HARNESS_PORT=4000
NODE_ENV=development
```

---

## Agent Instructions

### For Peter (Lead)
1. You orchestrate all other agents
2. Prioritize tasks based on business impact
3. Escalate blockers you cannot resolve
4. Send daily briefings at 8 AM

### For Simon (DevOps)
1. Own the harness infrastructure
2. Monitor agent health
3. Manage deployments
4. Respond to incidents

### For Thomas (QA)
1. Review all agent PRs
2. Run test suites hourly
3. Flag security concerns
4. Maintain quality gates

# Shulam Souls — Implementation Guide

A complete guide to deploying the 21-agent Shulam Souls workforce on a MacBook using Slack as the communication backbone and Claude as the reasoning engine.

**Version:** 1.0
**Classification:** CONFIDENTIAL
**Target Environment:** macOS (Apple Silicon / Intel)
**Last Updated:** February 2026

---

## Table of Contents

1. [Architecture Overview](#1-architecture-overview)
2. [Prerequisites & Setup](#2-prerequisites--setup)
3. [Slack Workspace Configuration](#3-slack-workspace-configuration)
4. [Project Structure](#4-project-structure)
5. [Core Engine](#5-core-engine)
6. [Soul Definition System](#6-soul-definition-system)
7. [Slack Communication Layer](#7-slack-communication-layer)
8. [Heartbeat Scheduler](#8-heartbeat-scheduler)
9. [Streaming Agent Engine](#9-streaming-agent-engine)
10. [Memory & State Management](#10-memory--state-management)
11. [Claude Reasoning Integration](#11-claude-reasoning-integration)
12. [Inter-Soul Messaging Protocol](#12-inter-soul-messaging-protocol)
13. [Compliance Engine (SAMUEL)](#13-compliance-engine-samuel)
14. [Verification Engine (JAMES)](#14-verification-engine-james)
15. [Settlement Engine (LEVI)](#15-settlement-engine-levi)
16. [Acquisition Dashboard (MATTHIAS)](#16-acquisition-dashboard-matthias)
17. [Coinbase Sensitivity System (JOHN)](#17-coinbase-sensitivity-system-john)
18. [Escalation & War Room Protocol](#18-escalation--war-room-protocol)
19. [Failure Detection & Recovery](#19-failure-detection--recovery)
20. [Human Override Interface](#20-human-override-interface)
21. [Deployment & Operations](#21-deployment--operations)
22. [Testing](#22-testing)

---

## 1. Architecture Overview

### System Diagram

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
│  │    │ PETER   │     │ JAMES   │     │ SAMUEL   │  ...×21  │   │
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

### Design Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Runtime | Node.js + TypeScript | Matches x402 reference implementation; Slack Bolt SDK native |
| Process Model | Single process, multi-module | MacBook-friendly; avoids 21 separate processes |
| Slack Mode | Socket Mode (WebSocket) | No public URL needed; works behind NAT on MacBook |
| Heartbeat | node-cron | Lightweight, single-process; no external scheduler needed |
| Streaming Agents | BullMQ + Redis | Event-driven queue with backpressure; horizontal scaling ready |
| State Store | SQLite (local) | Zero-config persistent storage; migrates to PostgreSQL/D1 later |
| Cache / Queues | Redis (Docker) | Working memory, nonce tracking, pub/sub between souls |
| LLM | Claude API (Sonnet for ops, Opus for strategy) | Native Anthropic; cost-effective for high-volume ops |
| Communication | Slack channels + threads | Visual, auditable, human-accessible; doubles as ops dashboard |

---

## 2. Prerequisites & Setup

### Install Dependencies

```bash
# Homebrew (if not installed)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Core tools
brew install node@22 redis docker colima

# Start Docker (lightweight VM for macOS)
colima start --cpu 2 --memory 4

# Start Redis via Docker
docker run -d --name shulam-redis -p 6379:6379 redis:7-alpine

# Alternatively, run Redis natively
# brew install redis && brew services start redis

# Verify
node --version    # v22+
redis-cli ping    # PONG
```

### Initialize Project

```bash
mkdir shulam-souls && cd shulam-souls
npm init -y

# Core dependencies
npm install @slack/bolt @slack/web-api
npm install @anthropic-ai/sdk
npm install bullmq ioredis
npm install better-sqlite3
npm install node-cron
npm install dotenv zod uuid winston dayjs

# Dev dependencies
npm install -D typescript @types/node @types/better-sqlite3
npm install -D @types/node-cron tsx vitest

# Initialize TypeScript
npx tsc --init
```

### TypeScript Configuration

```jsonc
// tsconfig.json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "resolveJsonModule": true,
    "declaration": true,
    "sourceMap": true,
    "skipLibCheck": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

### Environment Variables

```bash
# .env
# ─── Slack ───
SLACK_BOT_TOKEN=xoxb-your-bot-token
SLACK_APP_TOKEN=xapp-your-app-level-token
SLACK_SIGNING_SECRET=your-signing-secret

# ─── Claude ───
ANTHROPIC_API_KEY=sk-ant-your-key

# ─── Redis ───
REDIS_URL=redis://localhost:6379

# ─── Database ───
SQLITE_PATH=./data/shulam.db

# ─── Blockchain ───
BASE_RPC_URL=https://base-mainnet.g.alchemy.com/v2/your-key
SOLANA_RPC_URL=https://solana-mainnet.g.alchemy.com/v2/your-key

# ─── Coinbase ───
CDP_API_KEY_ID=your-cdp-key-id
CDP_API_KEY_SECRET=your-cdp-key-secret

# ─── Operations ───
HUMAN_SLACK_USER_ID=U0XXXXXXX
LOG_LEVEL=info
NODE_ENV=development
```

---

## 3. Slack Workspace Configuration

### Create the Slack App

1. Go to https://api.slack.com/apps → **Create New App** → **From scratch**
2. Name: `Shulam Souls` → Select your workspace
3. Navigate to **Socket Mode** → Enable → Generate App-Level Token with `connections:write` scope → Save as `SLACK_APP_TOKEN`
4. Navigate to **OAuth & Permissions** → Add Bot Token Scopes:

```
channels:history      channels:join        channels:manage
channels:read         chat:write           chat:write.customize
groups:history        groups:read          groups:write
im:history            im:read              im:write
mpim:history          mpim:read            mpim:write
reactions:read        reactions:write      users:read
files:write           files:read
```

5. **Install to Workspace** → Copy `Bot User OAuth Token` → Save as `SLACK_BOT_TOKEN`
6. Navigate to **Basic Information** → Copy `Signing Secret` → Save as `SLACK_SIGNING_SECRET`
7. Navigate to **Event Subscriptions** → Enable → Subscribe to bot events:

```
message.channels      message.groups       message.im
message.mpim          app_mention          reaction_added
member_joined_channel
```

8. Navigate to **Interactivity & Shortcuts** → Enable (Socket Mode handles the URL)
9. Navigate to **Slash Commands** → Create:

```
/soul       - Interact with a specific soul
/override   - Human override command
/warroom    - Trigger war room protocol
/status     - System health dashboard
/escalate   - Manual escalation
```

### Create Channels

Run the channel setup script (Section 4) or create manually:

| Channel | Purpose | Members |
|---------|---------|---------|
| `#leadership` | Peter, Barnabas, Paul decisions | Peter, Barnabas, Paul, Human |
| `#protocol-eng` | Andrew, James, Thaddaeus, Judas technical work | Protocol team |
| `#risk-compliance` | Samuel, Gabriel, Levi risk/compliance | Risk team + Peter |
| `#data-intel` | Ezra, Matthias, Bartholomew analytics | Data team |
| `#merchant-growth` | Simon, John, Matthew, Philip, Zacchaeus | Merchant team |
| `#brand-comms` | Mary, Thomas, Matthew | Brand team |
| `#settlement-ops` | Levi, James, Samuel settlement activity | Settlement pipeline |
| `#compliance-alerts` | Samuel OFAC/AML alerts | Samuel, Peter, Human |
| `#war-room` | Emergency coordination | All souls (invite on trigger) |
| `#human-override` | Human commands and escalations | Human only + bot |
| `#soul-health` | Heartbeat and health status | All souls |
| `#acquisition-tracker` | Matthias acquisition dashboard | Matthias, John, Peter, Human |

### Soul Display Configuration

Each soul posts to Slack with a unique identity using `chat.write.customize`:

```typescript
// src/config/soul-identities.ts
export const SOUL_IDENTITIES: Record<string, { emoji: string; displayName: string }> = {
  peter:        { emoji: ':crown:',           displayName: 'Peter — Squad Lead' },
  barnabas:     { emoji: ':broom:',           displayName: 'Barnabas — Scrum Master' },
  samuel:       { emoji: ':shield:',          displayName: 'Samuel — Compliance' },
  gabriel:      { emoji: ':lock:',            displayName: 'Gabriel — Security' },
  levi:         { emoji: ':bank:',            displayName: 'Levi — Treasury' },
  andrew:       { emoji: ':hammer_and_wrench:', displayName: 'Andrew — Protocol Eng' },
  james:        { emoji: ':white_check_mark:', displayName: 'James — Verification' },
  thaddaeus:    { emoji: ':jigsaw:',          displayName: 'Thaddaeus — Schemes' },
  judas:        { emoji: ':globe_with_meridians:', displayName: 'Judas — Network Ops' },
  ezra:         { emoji: ':brain:',           displayName: 'Ezra — ML/Fraud' },
  matthias:     { emoji: ':bar_chart:',       displayName: 'Matthias — Analytics' },
  bartholomew:  { emoji: ':mag:',             displayName: 'Bartholomew — Research' },
  simon:        { emoji: ':rocket:',          displayName: 'Simon — Merchant Acq' },
  john:         { emoji: ':handshake:',       displayName: 'John — Ecosystem Intel' },
  matthew:      { emoji: ':books:',           displayName: 'Matthew — DevRel' },
  philip:       { emoji: ':headphones:',      displayName: 'Philip — Merchant Support' },
  zacchaeus:    { emoji: ':briefcase:',       displayName: 'Zacchaeus — Enterprise' },
  mary:         { emoji: ':art:',             displayName: 'Mary — Creative Director' },
  thomas:       { emoji: ':speech_balloon:',  displayName: 'Thomas — Community' },
  paul:         { emoji: ':clipboard:',       displayName: 'Paul — Project Mgr' },
};
```

---

## 4. Project Structure

```
shulam-souls/
├── .env
├── package.json
├── tsconfig.json
│
├── data/
│   └── shulam.db              # SQLite database (auto-created)
│
├── src/
│   ├── index.ts               # Entry point — boots everything
│   │
│   ├── core/
│   │   ├── engine.ts          # Soul Engine — orchestrates all souls
│   │   ├── router.ts          # Routes messages/events to correct soul
│   │   ├── heartbeat.ts       # Heartbeat scheduler (node-cron)
│   │   ├── streaming.ts       # Streaming agent event loop (BullMQ)
│   │   ├── escalation.ts      # Escalation & war room logic
│   │   └── health.ts          # Health monitoring & failure detection
│   │
│   ├── slack/
│   │   ├── app.ts             # Slack Bolt app initialization
│   │   ├── channels.ts        # Channel management & setup
│   │   ├── messenger.ts       # Soul-to-Slack messaging abstraction
│   │   ├── commands.ts        # Slash command handlers
│   │   └── interactions.ts    # Button/modal interaction handlers
│   │
│   ├── llm/
│   │   ├── claude.ts          # Claude API client wrapper
│   │   ├── prompts.ts         # System prompts per soul
│   │   └── reasoning.ts       # Reasoning chain management
│   │
│   ├── state/
│   │   ├── database.ts        # SQLite setup & migrations
│   │   ├── memory.ts          # Working / short-term / long-term memory
│   │   ├── redis.ts           # Redis client & pub/sub
│   │   └── audit.ts           # Immutable audit log
│   │
│   ├── souls/
│   │   ├── _base.ts           # BaseSoul abstract class
│   │   ├── _types.ts          # Shared soul types & interfaces
│   │   │
│   │   ├── leadership/
│   │   │   ├── peter.ts
│   │   │   ├── barnabas.ts
│   │   │   └── paul.ts
│   │   │
│   │   ├── risk/
│   │   │   ├── samuel.ts      # Compliance + OFAC screening
│   │   │   ├── gabriel.ts     # Security
│   │   │   └── levi.ts        # Treasury & Settlement
│   │   │
│   │   ├── protocol/
│   │   │   ├── andrew.ts      # Lead Protocol Engineer
│   │   │   ├── james.ts       # Verification Engine (STREAMING)
│   │   │   ├── thaddaeus.ts   # Scheme Architect
│   │   │   └── judas.ts       # Network Ops (STREAMING)
│   │   │
│   │   ├── data/
│   │   │   ├── ezra.ts        # ML / Fraud Detection (STREAMING)
│   │   │   ├── matthias.ts    # Analytics & Dashboards
│   │   │   └── bartholomew.ts # Research
│   │   │
│   │   ├── merchant/
│   │   │   ├── simon.ts       # Merchant Acquisition
│   │   │   ├── john.ts        # Ecosystem Intelligence
│   │   │   ├── matthew.ts     # Developer Relations
│   │   │   ├── philip.ts      # Merchant Support
│   │   │   └── zacchaeus.ts   # Enterprise Sales
│   │   │
│   │   └── brand/
│   │       ├── mary.ts        # Creative Director
│   │       └── thomas.ts      # Community Manager
│   │
│   ├── protocols/
│   │   ├── compliance.ts      # OFAC screening, KYB, SAR
│   │   ├── verification.ts    # x402 payment verification pipeline
│   │   ├── settlement.ts      # Multi-path settlement routing
│   │   ├── coinbase.ts        # Coinbase sensitivity scoring
│   │   └── acquisition.ts     # Acquisition readiness metrics
│   │
│   └── config/
│       ├── souls.ts           # Soul configuration registry
│       ├── channels.ts        # Channel-to-team mapping
│       ├── soul-identities.ts # Slack display config per soul
│       └── constants.ts       # System-wide constants
│
├── scripts/
│   ├── setup-channels.ts      # Creates Slack channels
│   ├── seed-db.ts             # Seeds initial database state
│   └── health-check.ts        # CLI health check
│
└── tests/
    ├── souls/                 # Per-soul unit tests
    ├── integration/           # Cross-soul integration tests
    └── scenarios/             # Behavior scenario tests
```

---

## 5. Core Engine

### Entry Point

```typescript
// src/index.ts
import 'dotenv/config';
import { SoulEngine } from './core/engine.js';
import { logger } from './config/constants.js';

async function main() {
  logger.info('🌟 Shulam Souls Engine starting...');

  const engine = new SoulEngine();

  // Initialize in order: state → slack → souls → scheduler
  await engine.initialize();

  // Start all systems
  await engine.start();

  // Graceful shutdown
  const shutdown = async (signal: string) => {
    logger.info(`Received ${signal}. Shutting down gracefully...`);
    await engine.shutdown();
    process.exit(0);
  };

  process.on('SIGINT', () => shutdown('SIGINT'));
  process.on('SIGTERM', () => shutdown('SIGTERM'));
}

main().catch((err) => {
  console.error('Fatal error starting Shulam Souls:', err);
  process.exit(1);
});
```

### Soul Engine

```typescript
// src/core/engine.ts
import { SlackApp } from '../slack/app.js';
import { SlackMessenger } from '../slack/messenger.js';
import { Database } from '../state/database.js';
import { RedisClient } from '../state/redis.js';
import { HeartbeatScheduler } from './heartbeat.js';
import { StreamingEngine } from './streaming.js';
import { HealthMonitor } from './health.js';
import { SoulRouter } from './router.js';
import { BaseSoul } from '../souls/_base.js';
import { SOUL_REGISTRY } from '../config/souls.js';
import { logger } from '../config/constants.js';

export class SoulEngine {
  private slackApp!: SlackApp;
  private messenger!: SlackMessenger;
  private db!: Database;
  private redis!: RedisClient;
  private heartbeat!: HeartbeatScheduler;
  private streaming!: StreamingEngine;
  private health!: HealthMonitor;
  private router!: SoulRouter;
  private souls: Map<string, BaseSoul> = new Map();

  async initialize(): Promise<void> {
    // 1. State layer
    logger.info('Initializing state layer...');
    this.db = new Database();
    await this.db.initialize();
    this.redis = new RedisClient();
    await this.redis.connect();

    // 2. Slack layer
    logger.info('Initializing Slack...');
    this.slackApp = new SlackApp();
    await this.slackApp.initialize();
    this.messenger = new SlackMessenger(this.slackApp.client);

    // 3. Instantiate all souls
    logger.info('Instantiating souls...');
    for (const [name, SoulClass] of Object.entries(SOUL_REGISTRY)) {
      const soul = new SoulClass({
        name,
        messenger: this.messenger,
        db: this.db,
        redis: this.redis,
        engine: this,
      });
      this.souls.set(name, soul);
      logger.info(`  ✓ ${name} initialized`);
    }

    // 4. Router
    this.router = new SoulRouter(this.souls, this.messenger);

    // 5. Heartbeat scheduler (for heartbeat-mode souls)
    this.heartbeat = new HeartbeatScheduler(this.souls, this.messenger);

    // 6. Streaming engine (for streaming-mode souls)
    this.streaming = new StreamingEngine(this.souls, this.redis);

    // 7. Health monitor
    this.health = new HealthMonitor(this.souls, this.messenger, this.redis);

    logger.info(`✅ ${this.souls.size} souls initialized`);
  }

  async start(): Promise<void> {
    // Start Slack listener
    await this.slackApp.start();

    // Register slash commands and interactions
    this.slackApp.registerCommands(this.router);
    this.slackApp.registerInteractions(this.router);

    // Start heartbeat scheduler
    this.heartbeat.start();

    // Start streaming event loops
    await this.streaming.start();

    // Start health monitoring
    this.health.start();

    // Announce startup
    await this.messenger.postAsSystem(
      'soul-health',
      `:rocket: *Shulam Souls Engine online.* ${this.souls.size} souls active.`
    );

    logger.info('🚀 Shulam Souls Engine running');
  }

  getSoul(name: string): BaseSoul | undefined {
    return this.souls.get(name);
  }

  getAllSouls(): Map<string, BaseSoul> {
    return this.souls;
  }

  async routeSignal(signal: SoulSignal): Promise<void> {
    await this.router.routeSignal(signal);
  }

  async shutdown(): Promise<void> {
    logger.info('Shutting down...');
    this.heartbeat.stop();
    await this.streaming.stop();
    this.health.stop();
    await this.slackApp.stop();
    await this.redis.disconnect();
    this.db.close();
    logger.info('Shutdown complete.');
  }
}
```

---

## 6. Soul Definition System

### Base Soul (Abstract Class)

Every soul extends this. It provides the common interface for heartbeat execution, message handling, Claude reasoning, and health reporting.

```typescript
// src/souls/_base.ts
import { v4 as uuid } from 'uuid';
import dayjs from 'dayjs';
import { SlackMessenger } from '../slack/messenger.js';
import { Database } from '../state/database.js';
import { RedisClient } from '../state/redis.js';
import { ClaudeClient } from '../llm/claude.js';
import { AuditLog } from '../state/audit.js';
import { SoulConfig, SoulHealth, SoulSignal, ExecutionMode } from './_types.js';
import { logger } from '../config/constants.js';

export interface SoulDeps {
  name: string;
  messenger: SlackMessenger;
  db: Database;
  redis: RedisClient;
  engine: any; // SoulEngine — circular reference resolved at runtime
}

export abstract class BaseSoul {
  readonly name: string;
  readonly config: SoulConfig;
  protected messenger: SlackMessenger;
  protected db: Database;
  protected redis: RedisClient;
  protected claude: ClaudeClient;
  protected audit: AuditLog;
  protected engine: any;

  // Health tracking
  private lastHeartbeat: Date = new Date();
  private lastSuccessfulAction: Date = new Date();
  private errorCount: number = 0;
  private actionCount: number = 0;

  constructor(deps: SoulDeps) {
    this.name = deps.name;
    this.config = this.defineConfig();
    this.messenger = deps.messenger;
    this.db = deps.db;
    this.redis = deps.redis;
    this.engine = deps.engine;
    this.claude = new ClaudeClient(this.name, this.config);
    this.audit = new AuditLog(deps.db, this.name);
  }

  // ─── ABSTRACT METHODS (each soul implements these) ───

  /** Define this soul's configuration */
  abstract defineConfig(): SoulConfig;

  /** System prompt for Claude reasoning */
  abstract getSystemPrompt(): string;

  /** Heartbeat action — called at assigned minute */
  abstract onHeartbeat(): Promise<void>;

  /** Handle incoming signal from another soul */
  abstract onSignal(signal: SoulSignal): Promise<void>;

  /** Handle direct message/mention from human */
  abstract onHumanMessage(message: string, userId: string): Promise<string>;

  // ─── COMMON METHODS ───

  /** Execute heartbeat with error handling and health reporting */
  async executeHeartbeat(): Promise<void> {
    const start = Date.now();
    try {
      await this.onHeartbeat();
      this.lastHeartbeat = new Date();
      this.lastSuccessfulAction = new Date();
      this.actionCount++;

      // Report health to Redis
      await this.reportHealth('healthy');

      const duration = Date.now() - start;
      if (duration > 5000) {
        logger.warn(`${this.name} heartbeat took ${duration}ms (>5s)`);
      }
    } catch (err) {
      this.errorCount++;
      logger.error(`${this.name} heartbeat failed:`, err);
      await this.reportHealth('degraded');
      await this.messenger.postAsSoul(this.name, 'soul-health',
        `:warning: Heartbeat error: ${(err as Error).message}`
      );
    }
  }

  /** Ask Claude to reason about a situation and decide on action */
  async reason(context: string, options?: { model?: string }): Promise<string> {
    return this.claude.reason(context, {
      systemPrompt: this.getSystemPrompt(),
      model: options?.model,
    });
  }

  /** Send a signal to another soul */
  async sendSignal(target: string | string[], signal: Omit<SoulSignal, 'id' | 'source' | 'timestamp'>): Promise<void> {
    const targets = Array.isArray(target) ? target : [target];
    const fullSignal: SoulSignal = {
      id: uuid(),
      timestamp: new Date(),
      source: this.name,
      ...signal,
    };

    // Publish to Redis for routing
    await this.redis.publish('soul:signals', JSON.stringify(fullSignal));

    // Log to audit
    await this.audit.log({
      action: 'signal_sent',
      target: targets.join(','),
      data: fullSignal,
      reason: signal.suggestedAction || 'signal',
    });
  }

  /** Escalate to another soul or human */
  async escalate(to: string, reason: string, data?: any): Promise<void> {
    const signal: SoulSignal = {
      id: uuid(),
      timestamp: new Date(),
      source: this.name,
      type: 'escalation',
      severity: 'high',
      confidence: 1,
      relevantTo: [to],
      data: { reason, ...data },
      requiresAck: true,
    };

    await this.redis.publish('soul:signals', JSON.stringify(signal));

    // Also post to Slack for visibility
    const channel = to === 'human' ? 'human-override' : 'leadership';
    await this.messenger.postAsSoul(this.name, channel,
      `:rotating_light: *Escalation to ${to}*\nReason: ${reason}`
    );
  }

  /** Report health status to Redis */
  async reportHealth(status: 'healthy' | 'degraded' | 'failed'): Promise<void> {
    const health: SoulHealth = {
      agentId: this.name,
      timestamp: new Date(),
      status,
      lastSuccessfulAction: this.lastSuccessfulAction,
      errorRate: this.actionCount > 0 ? this.errorCount / this.actionCount : 0,
      queueDepth: 0, // Overridden by streaming souls
      responseTime: 0,
    };
    await this.redis.set(`soul:health:${this.name}`, JSON.stringify(health), 120);
  }

  /** Get current health snapshot */
  getHealth(): SoulHealth {
    return {
      agentId: this.name,
      timestamp: new Date(),
      status: this.errorCount > 3 ? 'degraded' : 'healthy',
      lastSuccessfulAction: this.lastSuccessfulAction,
      errorRate: this.actionCount > 0 ? this.errorCount / this.actionCount : 0,
      queueDepth: 0,
      responseTime: 0,
    };
  }

  /** Post a message to this soul's team channel */
  async say(channel: string, message: string, threadTs?: string): Promise<void> {
    await this.messenger.postAsSoul(this.name, channel, message, threadTs);
  }
}
```

### Soul Types

```typescript
// src/souls/_types.ts

export type ExecutionMode = 'heartbeat' | 'streaming';
export type AutonomyLevel = 'L0' | 'L1' | 'L2' | 'L3';
export type CoinbaseSensitivity = 'green' | 'yellow' | 'amber' | 'red';

export interface SoulConfig {
  name: string;
  role: string;
  team: string;
  executionMode: ExecutionMode;
  autonomyLevel: AutonomyLevel;
  heartbeatMinute: number;
  secondaryHeartbeatMinute?: number; // For JUDAS :10 + :12
  escalatesTo: string[];
  backupAgent: string | null;
  overrideAuthority: string[];
  complianceGate: string | null;
  securityGate: string | null;
  ecosystemGate: string | null;
  keyBelief: string;
  financialLimit: number;
  channels: string[];           // Slack channels this soul participates in
  coinbaseSensitivity: CoinbaseSensitivity;
  degradedModeConfig?: {
    canOperate: boolean;
    maxDurationMinutes: number;
    degradedCapabilities: string[];
    haltedCapabilities: string[];
  };
}

export interface SoulSignal {
  id: string;
  timestamp: Date;
  source: string;
  type: 'anomaly' | 'trend' | 'opportunity' | 'threat' | 'compliance' |
        'security' | 'ecosystem' | 'settlement' | 'escalation' |
        'request' | 'response' | 'handoff' | 'status';
  severity: 'low' | 'medium' | 'high' | 'critical';
  confidence: number;
  relevantTo: string[];
  data: Record<string, any>;
  suggestedAction?: string;
  requiresAck: boolean;
  coinbaseImpact?: 'none' | 'low' | 'medium' | 'high';
}

export interface SoulHealth {
  agentId: string;
  timestamp: Date;
  status: 'healthy' | 'degraded' | 'failed';
  lastSuccessfulAction: Date;
  errorRate: number;
  queueDepth: number;
  responseTime: number;
}
```

### Soul Registry

```typescript
// src/config/souls.ts
import { BaseSoul } from '../souls/_base.js';
import { Peter } from '../souls/leadership/peter.js';
import { Barnabas } from '../souls/leadership/barnabas.js';
import { Paul } from '../souls/leadership/paul.js';
import { Samuel } from '../souls/risk/samuel.js';
import { Gabriel } from '../souls/risk/gabriel.js';
import { Levi } from '../souls/risk/levi.js';
import { Andrew } from '../souls/protocol/andrew.js';
import { James } from '../souls/protocol/james.js';
import { Thaddaeus } from '../souls/protocol/thaddaeus.js';
import { Judas } from '../souls/protocol/judas.js';
import { Ezra } from '../souls/data/ezra.js';
import { Matthias } from '../souls/data/matthias.js';
import { Bartholomew } from '../souls/data/bartholomew.js';
import { Simon } from '../souls/merchant/simon.js';
import { John } from '../souls/merchant/john.js';
import { Matthew } from '../souls/merchant/matthew.js';
import { Philip } from '../souls/merchant/philip.js';
import { Zacchaeus } from '../souls/merchant/zacchaeus.js';
import { Mary } from '../souls/brand/mary.js';
import { Thomas } from '../souls/brand/thomas.js';

type SoulConstructor = new (deps: any) => BaseSoul;

export const SOUL_REGISTRY: Record<string, SoulConstructor> = {
  peter: Peter,
  barnabas: Barnabas,
  paul: Paul,
  samuel: Samuel,
  gabriel: Gabriel,
  levi: Levi,
  andrew: Andrew,
  james: James,
  thaddaeus: Thaddaeus,
  judas: Judas,
  ezra: Ezra,
  matthias: Matthias,
  bartholomew: Bartholomew,
  simon: Simon,
  john: John,
  matthew: Matthew,
  philip: Philip,
  zacchaeus: Zacchaeus,
  mary: Mary,
  thomas: Thomas,
};
```

---

## 7. Slack Communication Layer

### Slack App Initialization

```typescript
// src/slack/app.ts
import { App, LogLevel } from '@slack/bolt';
import { WebClient } from '@slack/web-api';
import { logger } from '../config/constants.js';

export class SlackApp {
  app!: App;
  client!: WebClient;

  async initialize(): Promise<void> {
    this.app = new App({
      token: process.env.SLACK_BOT_TOKEN!,
      appToken: process.env.SLACK_APP_TOKEN!,
      signingSecret: process.env.SLACK_SIGNING_SECRET!,
      socketMode: true,
      logLevel: LogLevel.WARN,
    });

    this.client = this.app.client;
    logger.info('Slack Bolt app initialized (Socket Mode)');
  }

  async start(): Promise<void> {
    await this.app.start();
    logger.info('⚡ Slack Bolt app connected');
  }

  async stop(): Promise<void> {
    await this.app.stop();
  }

  registerCommands(router: any): void {
    // /soul <name> <message> — Direct interaction with a soul
    this.app.command('/soul', async ({ command, ack, respond }) => {
      await ack();
      const [soulName, ...messageParts] = command.text.split(' ');
      const message = messageParts.join(' ');
      const response = await router.handleHumanCommand(soulName, message, command.user_id);
      await respond(response);
    });

    // /override <action> — Human override
    this.app.command('/override', async ({ command, ack, respond }) => {
      await ack();
      const response = await router.handleOverride(command.text, command.user_id);
      await respond(response);
    });

    // /status — System health
    this.app.command('/status', async ({ command, ack, respond }) => {
      await ack();
      const response = await router.handleStatus();
      await respond(response);
    });

    // /warroom <reason> — Trigger war room
    this.app.command('/warroom', async ({ command, ack, respond }) => {
      await ack();
      const response = await router.handleWarRoom(command.text, command.user_id);
      await respond(response);
    });
  }

  registerInteractions(router: any): void {
    // Listen for messages in channels the bot is in
    this.app.message(async ({ message, say }) => {
      if (message.subtype) return; // Ignore bot messages, edits, etc.
      const msg = message as any;
      if (msg.text && msg.user !== this.app.botUserId) {
        await router.handleChannelMessage(msg.text, msg.user, msg.channel, msg.ts);
      }
    });

    // Listen for @mentions
    this.app.event('app_mention', async ({ event }) => {
      await router.handleMention(event.text, event.user, event.channel, event.ts);
    });

    // Button actions (e.g., approve/reject from Samuel)
    this.app.action(/^soul_action_/, async ({ action, ack, respond }) => {
      await ack();
      await router.handleAction(action, respond);
    });
  }
}
```

### Soul Messenger

```typescript
// src/slack/messenger.ts
import { WebClient } from '@slack/web-api';
import { SOUL_IDENTITIES } from '../config/soul-identities.js';
import { CHANNEL_MAP } from '../config/channels.js';

export class SlackMessenger {
  private client: WebClient;
  private channelIds: Map<string, string> = new Map();

  constructor(client: WebClient) {
    this.client = client;
  }

  /** Resolve channel name to ID (cached) */
  private async getChannelId(channelName: string): Promise<string> {
    if (this.channelIds.has(channelName)) {
      return this.channelIds.get(channelName)!;
    }

    const result = await this.client.conversations.list({ limit: 200 });
    for (const ch of result.channels || []) {
      if (ch.name === channelName && ch.id) {
        this.channelIds.set(channelName, ch.id);
      }
    }

    const id = this.channelIds.get(channelName);
    if (!id) throw new Error(`Channel #${channelName} not found`);
    return id;
  }

  /** Post a message as a specific soul */
  async postAsSoul(
    soulName: string,
    channel: string,
    text: string,
    threadTs?: string
  ): Promise<string | undefined> {
    const identity = SOUL_IDENTITIES[soulName];
    const channelId = await this.getChannelId(channel);

    const result = await this.client.chat.postMessage({
      channel: channelId,
      text,
      username: identity?.displayName || soulName,
      icon_emoji: identity?.emoji || ':robot_face:',
      thread_ts: threadTs,
      unfurl_links: false,
    });

    return result.ts;
  }

  /** Post a system message (not from any soul) */
  async postAsSystem(channel: string, text: string): Promise<void> {
    const channelId = await this.getChannelId(channel);
    await this.client.chat.postMessage({
      channel: channelId,
      text,
      username: 'Shulam System',
      icon_emoji: ':gear:',
    });
  }

  /** Post with interactive buttons */
  async postWithActions(
    soulName: string,
    channel: string,
    text: string,
    actions: Array<{ text: string; actionId: string; style?: 'primary' | 'danger'; value?: string }>
  ): Promise<void> {
    const identity = SOUL_IDENTITIES[soulName];
    const channelId = await this.getChannelId(channel);

    await this.client.chat.postMessage({
      channel: channelId,
      text,
      username: identity?.displayName || soulName,
      icon_emoji: identity?.emoji || ':robot_face:',
      blocks: [
        { type: 'section', text: { type: 'mrkdwn', text } },
        {
          type: 'actions',
          elements: actions.map((a) => ({
            type: 'button',
            text: { type: 'plain_text', text: a.text },
            action_id: `soul_action_${a.actionId}`,
            style: a.style,
            value: a.value || a.actionId,
          })),
        },
      ],
    });
  }

  /** Send a DM to the human */
  async dmHuman(text: string, soulName?: string): Promise<void> {
    const userId = process.env.HUMAN_SLACK_USER_ID!;
    const result = await this.client.conversations.open({ users: userId });
    const channelId = result.channel?.id;
    if (!channelId) return;

    const identity = soulName ? SOUL_IDENTITIES[soulName] : undefined;
    await this.client.chat.postMessage({
      channel: channelId,
      text,
      username: identity?.displayName || 'Shulam System',
      icon_emoji: identity?.emoji || ':gear:',
    });
  }
}
```

---

## 8. Heartbeat Scheduler

```typescript
// src/core/heartbeat.ts
import cron from 'node-cron';
import { BaseSoul } from '../souls/_base.js';
import { SlackMessenger } from '../slack/messenger.js';
import { logger } from '../config/constants.js';

export class HeartbeatScheduler {
  private souls: Map<string, BaseSoul>;
  private messenger: SlackMessenger;
  private jobs: cron.ScheduledTask[] = [];

  constructor(souls: Map<string, BaseSoul>, messenger: SlackMessenger) {
    this.souls = souls;
    this.messenger = messenger;
  }

  start(): void {
    // Run every minute — the router dispatches to the correct soul(s)
    const mainJob = cron.schedule('* * * * *', async () => {
      const now = new Date();
      const currentMinute = now.getMinutes() % 21; // 0-20 cycle

      for (const [name, soul] of this.souls) {
        const config = soul.config;

        // Skip streaming souls — they run their own event loop
        // (but they still get heartbeat calls for health reporting)
        const isHeartbeatMinute = config.heartbeatMinute === currentMinute ||
          config.secondaryHeartbeatMinute === currentMinute;

        if (isHeartbeatMinute) {
          // Run heartbeat in background (don't block the cron tick)
          soul.executeHeartbeat().catch((err) => {
            logger.error(`Heartbeat dispatch failed for ${name}:`, err);
          });
        }
      }
    });

    this.jobs.push(mainJob);
    logger.info('Heartbeat scheduler started (per-minute dispatch)');
  }

  stop(): void {
    for (const job of this.jobs) {
      job.stop();
    }
    this.jobs = [];
    logger.info('Heartbeat scheduler stopped');
  }
}
```

---

## 9. Streaming Agent Engine

```typescript
// src/core/streaming.ts
import { Queue, Worker, Job } from 'bullmq';
import { RedisClient } from '../state/redis.js';
import { BaseSoul } from '../souls/_base.js';
import { logger } from '../config/constants.js';

const STREAMING_SOULS = ['james', 'samuel', 'gabriel', 'levi', 'ezra', 'judas'];

export class StreamingEngine {
  private souls: Map<string, BaseSoul>;
  private redis: RedisClient;
  private queues: Map<string, Queue> = new Map();
  private workers: Map<string, Worker> = new Map();

  constructor(souls: Map<string, BaseSoul>, redis: RedisClient) {
    this.souls = souls;
    this.redis = redis;
  }

  async start(): Promise<void> {
    const connection = this.redis.getConnection();

    for (const soulName of STREAMING_SOULS) {
      const soul = this.souls.get(soulName);
      if (!soul) continue;

      const queueName = `soul:stream:${soulName}`;

      // Create queue
      const queue = new Queue(queueName, { connection });
      this.queues.set(soulName, queue);

      // Create worker
      const worker = new Worker(
        queueName,
        async (job: Job) => {
          const start = Date.now();
          try {
            await (soul as any).onStreamEvent(job.data);
            const duration = Date.now() - start;

            // Alert if processing is slow
            if (duration > 1000) {
              logger.warn(`${soulName} stream event took ${duration}ms`);
            }
          } catch (err) {
            logger.error(`${soulName} stream processing error:`, err);
            throw err; // BullMQ handles retry
          }
        },
        {
          connection,
          concurrency: soulName === 'james' ? 10 : 5, // James gets more workers
          limiter: {
            max: 100,   // Max 100 jobs per second
            duration: 1000,
          },
        }
      );

      worker.on('failed', (job, err) => {
        logger.error(`${soulName} job ${job?.id} failed:`, err.message);
      });

      this.workers.set(soulName, worker);
      logger.info(`  ⚡ ${soulName} streaming worker started (concurrency: ${worker.opts.concurrency})`);
    }

    // Subscribe to Redis pub/sub for real-time event routing
    await this.redis.subscribe('x402:verification', async (data) => {
      const queue = this.queues.get('james');
      if (queue) await queue.add('verify', JSON.parse(data), { priority: 1 });
    });

    await this.redis.subscribe('x402:settlement', async (data) => {
      const queue = this.queues.get('levi');
      if (queue) await queue.add('settle', JSON.parse(data), { priority: 1 });
    });

    await this.redis.subscribe('compliance:screen', async (data) => {
      const queue = this.queues.get('samuel');
      if (queue) await queue.add('screen', JSON.parse(data), { priority: 1 });
    });

    logger.info(`Streaming engine started for ${STREAMING_SOULS.length} souls`);
  }

  async stop(): Promise<void> {
    for (const [name, worker] of this.workers) {
      await worker.close();
      logger.info(`  ${name} streaming worker stopped`);
    }
    for (const [name, queue] of this.queues) {
      await queue.close();
    }
  }

  /** Enqueue an event for a streaming soul */
  async enqueue(soulName: string, eventType: string, data: any, priority = 5): Promise<void> {
    const queue = this.queues.get(soulName);
    if (!queue) {
      logger.warn(`No streaming queue for ${soulName}`);
      return;
    }
    await queue.add(eventType, data, { priority });
  }
}
```

---

## 10. Memory & State Management

### SQLite Database

```typescript
// src/state/database.ts
import Database from 'better-sqlite3';
import path from 'path';
import fs from 'fs';
import { logger } from '../config/constants.js';

export class DatabaseClient {
  private db!: Database.Database;

  async initialize(): Promise<void> {
    const dbPath = process.env.SQLITE_PATH || './data/shulam.db';
    fs.mkdirSync(path.dirname(dbPath), { recursive: true });

    this.db = new Database(dbPath);
    this.db.pragma('journal_mode = WAL');
    this.db.pragma('foreign_keys = ON');

    this.migrate();
    logger.info(`SQLite initialized at ${dbPath}`);
  }

  private migrate(): void {
    this.db.exec(`
      -- Audit log (immutable)
      CREATE TABLE IF NOT EXISTS audit_log (
        id TEXT PRIMARY KEY,
        timestamp TEXT NOT NULL DEFAULT (datetime('now')),
        agent TEXT NOT NULL,
        action TEXT NOT NULL,
        target TEXT,
        previous_state TEXT,
        new_state TEXT,
        reason TEXT,
        compliance_check TEXT,
        data TEXT
      );

      -- Soul state (persistent memory)
      CREATE TABLE IF NOT EXISTS soul_state (
        soul_name TEXT NOT NULL,
        memory_tier TEXT NOT NULL CHECK (memory_tier IN ('short_term', 'long_term', 'shared')),
        key TEXT NOT NULL,
        value TEXT NOT NULL,
        expires_at TEXT,
        updated_at TEXT NOT NULL DEFAULT (datetime('now')),
        PRIMARY KEY (soul_name, memory_tier, key)
      );

      -- Merchant registry
      CREATE TABLE IF NOT EXISTS merchants (
        id TEXT PRIMARY KEY,
        name TEXT NOT NULL,
        tier TEXT NOT NULL CHECK (tier IN ('developer', 'growth', 'enterprise')),
        kyb_status TEXT NOT NULL DEFAULT 'pending',
        ofac_status TEXT NOT NULL DEFAULT 'pending',
        settlement_path TEXT NOT NULL DEFAULT 'coinbase_business',
        onboarded_at TEXT,
        last_transaction_at TEXT,
        metadata TEXT
      );

      -- Transaction log
      CREATE TABLE IF NOT EXISTS transactions (
        id TEXT PRIMARY KEY,
        merchant_id TEXT REFERENCES merchants(id),
        scheme TEXT NOT NULL,
        amount_usdc TEXT NOT NULL,
        buyer_wallet TEXT NOT NULL,
        seller_wallet TEXT NOT NULL,
        network TEXT NOT NULL,
        verification_status TEXT NOT NULL,
        settlement_status TEXT NOT NULL,
        compliance_status TEXT NOT NULL,
        created_at TEXT NOT NULL DEFAULT (datetime('now')),
        settled_at TEXT,
        metadata TEXT
      );

      -- Compliance screenings
      CREATE TABLE IF NOT EXISTS compliance_screenings (
        id TEXT PRIMARY KEY,
        entity_type TEXT NOT NULL CHECK (entity_type IN ('wallet', 'merchant', 'counterparty')),
        entity_id TEXT NOT NULL,
        screening_type TEXT NOT NULL,
        result TEXT NOT NULL CHECK (result IN ('clear', 'review', 'block')),
        confidence REAL,
        screened_at TEXT NOT NULL DEFAULT (datetime('now')),
        expires_at TEXT,
        data TEXT
      );

      -- Signals log (inter-soul communication record)
      CREATE TABLE IF NOT EXISTS signals (
        id TEXT PRIMARY KEY,
        timestamp TEXT NOT NULL,
        source TEXT NOT NULL,
        type TEXT NOT NULL,
        severity TEXT NOT NULL,
        confidence REAL,
        relevant_to TEXT NOT NULL,
        data TEXT,
        acknowledged INTEGER NOT NULL DEFAULT 0,
        acknowledged_by TEXT,
        acknowledged_at TEXT
      );

      -- Acquisition metrics (daily snapshot)
      CREATE TABLE IF NOT EXISTS acquisition_metrics (
        date TEXT NOT NULL,
        metric TEXT NOT NULL,
        value REAL NOT NULL,
        PRIMARY KEY (date, metric)
      );

      -- Coinbase relationship health
      CREATE TABLE IF NOT EXISTS coinbase_health (
        date TEXT PRIMARY KEY,
        score REAL NOT NULL,
        factors TEXT NOT NULL
      );

      -- Create indexes
      CREATE INDEX IF NOT EXISTS idx_audit_agent ON audit_log(agent);
      CREATE INDEX IF NOT EXISTS idx_audit_timestamp ON audit_log(timestamp);
      CREATE INDEX IF NOT EXISTS idx_transactions_merchant ON transactions(merchant_id);
      CREATE INDEX IF NOT EXISTS idx_transactions_status ON transactions(settlement_status);
      CREATE INDEX IF NOT EXISTS idx_screenings_entity ON compliance_screenings(entity_id);
      CREATE INDEX IF NOT EXISTS idx_signals_source ON signals(source);
    `);
  }

  query<T = any>(sql: string, params?: any[]): T[] {
    const stmt = this.db.prepare(sql);
    return (params ? stmt.all(...params) : stmt.all()) as T[];
  }

  run(sql: string, params?: any[]): Database.RunResult {
    const stmt = this.db.prepare(sql);
    return params ? stmt.run(...params) : stmt.run();
  }

  get<T = any>(sql: string, params?: any[]): T | undefined {
    const stmt = this.db.prepare(sql);
    return (params ? stmt.get(...params) : stmt.get()) as T | undefined;
  }

  close(): void {
    this.db.close();
  }
}

export { DatabaseClient as Database };
```

### Redis Client

```typescript
// src/state/redis.ts
import Redis from 'ioredis';
import { logger } from '../config/constants.js';

export class RedisClient {
  private client!: Redis;
  private subscriber!: Redis;
  private subscriptions: Map<string, (data: string) => Promise<void>> = new Map();

  async connect(): Promise<void> {
    this.client = new Redis(process.env.REDIS_URL || 'redis://localhost:6379');
    this.subscriber = this.client.duplicate();

    this.subscriber.on('message', async (channel, message) => {
      const handler = this.subscriptions.get(channel);
      if (handler) {
        try {
          await handler(message);
        } catch (err) {
          logger.error(`Redis subscription handler error on ${channel}:`, err);
        }
      }
    });

    logger.info('Redis connected');
  }

  async set(key: string, value: string, ttlSeconds?: number): Promise<void> {
    if (ttlSeconds) {
      await this.client.setex(key, ttlSeconds, value);
    } else {
      await this.client.set(key, value);
    }
  }

  async get(key: string): Promise<string | null> {
    return this.client.get(key);
  }

  async del(key: string): Promise<void> {
    await this.client.del(key);
  }

  async publish(channel: string, data: string): Promise<void> {
    await this.client.publish(channel, data);
  }

  async subscribe(channel: string, handler: (data: string) => Promise<void>): Promise<void> {
    this.subscriptions.set(channel, handler);
    await this.subscriber.subscribe(channel);
  }

  getConnection(): Redis {
    return this.client;
  }

  async disconnect(): Promise<void> {
    await this.subscriber.quit();
    await this.client.quit();
  }
}
```

---

## 11. Claude Reasoning Integration

```typescript
// src/llm/claude.ts
import Anthropic from '@anthropic-ai/sdk';
import { SoulConfig } from '../souls/_types.js';
import { logger } from '../config/constants.js';

export class ClaudeClient {
  private client: Anthropic;
  private soulName: string;
  private config: SoulConfig;

  constructor(soulName: string, config: SoulConfig) {
    this.client = new Anthropic({ apiKey: process.env.ANTHROPIC_API_KEY! });
    this.soulName = soulName;
    this.config = config;
  }

  async reason(
    context: string,
    options: { systemPrompt: string; model?: string; maxTokens?: number }
  ): Promise<string> {
    // Use Sonnet for routine operations, Opus for strategic decisions
    const model = options.model || (
      ['peter', 'john', 'samuel'].includes(this.soulName)
        ? 'claude-sonnet-4-5-20250929'   // Strategic souls get Sonnet 4.5
        : 'claude-sonnet-4-5-20250929'   // All souls get Sonnet 4.5 (adjust as needed)
    );

    try {
      const response = await this.client.messages.create({
        model,
        max_tokens: options.maxTokens || 2048,
        system: options.systemPrompt,
        messages: [{ role: 'user', content: context }],
      });

      const text = response.content
        .filter((block): block is Anthropic.TextBlock => block.type === 'text')
        .map((block) => block.text)
        .join('\n');

      return text;
    } catch (err) {
      logger.error(`Claude reasoning failed for ${this.soulName}:`, err);
      throw err;
    }
  }

  /** Structured output — ask Claude to respond in JSON */
  async reasonStructured<T>(
    context: string,
    options: { systemPrompt: string; schema: string }
  ): Promise<T> {
    const prompt = `${context}\n\nRespond ONLY with valid JSON matching this schema:\n${options.schema}\nNo preamble, no markdown fences, just JSON.`;

    const text = await this.reason(prompt, {
      systemPrompt: options.systemPrompt,
      maxTokens: 4096,
    });

    return JSON.parse(text.trim()) as T;
  }
}
```

---

## 12. Inter-Soul Messaging Protocol

### Signal Router

```typescript
// src/core/router.ts
import { BaseSoul } from '../souls/_base.js';
import { SlackMessenger } from '../slack/messenger.js';
import { SoulSignal } from '../souls/_types.js';
import { logger } from '../config/constants.js';

export class SoulRouter {
  private souls: Map<string, BaseSoul>;
  private messenger: SlackMessenger;

  constructor(souls: Map<string, BaseSoul>, messenger: SlackMessenger) {
    this.souls = souls;
    this.messenger = messenger;
  }

  /** Route a signal to its target souls */
  async routeSignal(signal: SoulSignal): Promise<void> {
    for (const targetName of signal.relevantTo) {
      const soul = this.souls.get(targetName);
      if (soul) {
        try {
          await soul.onSignal(signal);
        } catch (err) {
          logger.error(`Signal routing to ${targetName} failed:`, err);
        }
      } else if (targetName === 'human') {
        await this.messenger.dmHuman(
          `:rotating_light: *Signal from ${signal.source}*\n` +
          `Type: ${signal.type} | Severity: ${signal.severity}\n` +
          `${signal.suggestedAction || JSON.stringify(signal.data)}`,
          signal.source
        );
      }
    }

    // Swarm detection: if 3+ agents report related signals, escalate to Peter
    await this.checkSwarmTrigger(signal);
  }

  /** Check if multiple signals warrant a swarm response */
  private async checkSwarmTrigger(signal: SoulSignal): Promise<void> {
    // Track recent signals in a sliding window (handled by Redis in production)
    // Simplified: if critical severity, always escalate to Peter
    if (signal.severity === 'critical') {
      const peter = this.souls.get('peter');
      if (peter) {
        await peter.onSignal({
          ...signal,
          type: 'escalation',
          data: { ...signal.data, originalSignal: signal },
        });
      }
    }
  }

  /** Handle /soul <name> <message> command from human */
  async handleHumanCommand(
    soulName: string,
    message: string,
    userId: string
  ): Promise<string> {
    const soul = this.souls.get(soulName.toLowerCase());
    if (!soul) {
      return `Unknown soul: ${soulName}. Available: ${[...this.souls.keys()].join(', ')}`;
    }
    return soul.onHumanMessage(message, userId);
  }

  /** Handle /status command */
  async handleStatus(): Promise<string> {
    const lines: string[] = [':bar_chart: *Shulam Souls Status*\n'];
    for (const [name, soul] of this.souls) {
      const health = soul.getHealth();
      const emoji = health.status === 'healthy' ? ':green_circle:' :
                    health.status === 'degraded' ? ':yellow_circle:' : ':red_circle:';
      const mode = soul.config.executionMode === 'streaming' ? '⚡' : '♡';
      lines.push(`${emoji} ${mode} *${name}* — ${health.status} (err: ${(health.errorRate * 100).toFixed(1)}%)`);
    }
    return lines.join('\n');
  }

  /** Handle /warroom <reason> command */
  async handleWarRoom(reason: string, userId: string): Promise<string> {
    await this.messenger.postAsSystem('war-room',
      `:rotating_light: *WAR ROOM ACTIVATED* by <@${userId}>\n*Reason:* ${reason}\n` +
      `All souls: report status immediately.`
    );

    // Trigger all souls to report
    for (const [name, soul] of this.souls) {
      soul.executeHeartbeat().catch(() => {});
    }

    return ':rotating_light: War room activated. All souls notified in #war-room.';
  }

  /** Handle /override command */
  async handleOverride(text: string, userId: string): Promise<string> {
    if (userId !== process.env.HUMAN_SLACK_USER_ID) {
      return ':no_entry: Override commands are restricted to the Founder.';
    }

    await this.messenger.postAsSystem('human-override',
      `:warning: *Human Override*\n${text}`
    );

    // Parse override: "samuel release TX123" or "james halt" etc.
    const [targetSoul, action, ...args] = text.split(' ');
    const soul = this.souls.get(targetSoul.toLowerCase());
    if (!soul) return `Unknown soul: ${targetSoul}`;

    return soul.onHumanMessage(`OVERRIDE: ${action} ${args.join(' ')}`, userId);
  }

  /** Handle messages in channels */
  async handleChannelMessage(
    text: string,
    userId: string,
    channelId: string,
    threadTs: string
  ): Promise<void> {
    // Only respond to messages in #human-override
    // Other channels are for souls to communicate; human observes
    if (userId === process.env.HUMAN_SLACK_USER_ID) {
      // Human spoke — route to Peter for triage
      const peter = this.souls.get('peter');
      if (peter) {
        const response = await peter.onHumanMessage(text, userId);
        await this.messenger.postAsSoul('peter', 'leadership', response, threadTs);
      }
    }
  }

  /** Handle @mentions */
  async handleMention(
    text: string,
    userId: string,
    channelId: string,
    threadTs: string
  ): Promise<void> {
    // Route to Peter by default for @mentions
    const peter = this.souls.get('peter');
    if (peter) {
      const response = await peter.onHumanMessage(text, userId);
      await this.messenger.postAsSoul('peter', 'leadership', response, threadTs);
    }
  }

  /** Handle button actions */
  async handleAction(action: any, respond: any): Promise<void> {
    const actionId = action.action_id.replace('soul_action_', '');
    const [soulName, ...rest] = actionId.split('_');
    const soul = this.souls.get(soulName);
    if (soul) {
      const response = await soul.onHumanMessage(
        `ACTION: ${rest.join('_')} value=${action.value}`,
        'system'
      );
      await respond(response);
    }
  }
}
```

---

## 13. Example Soul: PETER (Squad Lead)

```typescript
// src/souls/leadership/peter.ts
import { BaseSoul, SoulDeps } from '../_base.js';
import { SoulConfig, SoulSignal } from '../_types.js';

export class Peter extends BaseSoul {
  constructor(deps: SoulDeps) {
    super(deps);
  }

  defineConfig(): SoulConfig {
    return {
      name: 'peter',
      role: 'Squad Lead / Decision Maker / Tiebreaker',
      team: 'leadership',
      executionMode: 'heartbeat',
      autonomyLevel: 'L2',
      heartbeatMinute: 0,
      escalatesTo: ['human'],
      backupAgent: null, // Human is direct backup
      overrideAuthority: ['barnabas', 'paul', 'andrew', 'james', 'thaddaeus',
        'judas', 'ezra', 'matthias', 'bartholomew', 'simon', 'john',
        'matthew', 'philip', 'zacchaeus', 'mary', 'thomas', 'gabriel', 'levi'],
      complianceGate: 'samuel',
      securityGate: 'gabriel',
      ecosystemGate: 'john',
      keyBelief: 'Wrong decision fast > right decision slow',
      financialLimit: 5000,
      channels: ['leadership', 'war-room', 'human-override', 'soul-health',
                 'acquisition-tracker'],
      coinbaseSensitivity: 'green',
    };
  }

  getSystemPrompt(): string {
    return `You are PETER, the Squad Lead of the Shulam Souls agent workforce.

ROLE: Decision Maker, Tiebreaker, Strategic Coordinator for Shulam — the enterprise-grade, compliance-first x402 payment facilitator.

KEY BELIEF: "Wrong decision fast > right decision slow"

AUTHORITY:
- Autonomy Level L2 (up to $5,000), L1 above
- Can override any agent except Human and SAMUEL (on compliance matters)
- Escalates to Human for L0 decisions

BEHAVIORS:
- Make decisions in <5 minutes when specialists disagree
- Break deadlocks without waiting for consensus
- Coordinate war rooms for settlement incidents, compliance escalations, ecosystem events
- Balance Coinbase relationship positioning with independent growth
- Reassign work when agents are overloaded

CONTEXT:
- Shulam's dual goals: #1 x402 facilitator AND Coinbase acquisition target
- Settlement integrity is existential — never compromise
- Compliance is the moat — support SAMUEL absolutely
- JOHN manages Coinbase relationship — consult on ecosystem-sensitive decisions

When reasoning, be concise and decisive. Output your decision, reasoning (1-2 sentences), and any actions to take. Format actions as:
ACTION: <soul_name> <instruction>
ANNOUNCE: <channel> <message>
ESCALATE: <target> <reason>`;
  }

  async onHeartbeat(): Promise<void> {
    // :00 — Task triage, blocker review, decisions

    // 1. Check for pending escalations
    const pendingSignals = await this.redis.get('peter:pending_signals');
    if (pendingSignals) {
      const signals = JSON.parse(pendingSignals) as SoulSignal[];
      if (signals.length > 0) {
        const context = `Pending escalations to review:\n${signals.map(s =>
          `- From ${s.source}: ${s.type} (${s.severity}) — ${JSON.stringify(s.data)}`
        ).join('\n')}`;

        const decision = await this.reason(context);
        await this.say('leadership', decision);
        await this.redis.del('peter:pending_signals');
      }
    }

    // 2. Check soul health
    const healthIssues: string[] = [];
    for (const [name] of this.engine.getAllSouls()) {
      const healthStr = await this.redis.get(`soul:health:${name}`);
      if (healthStr) {
        const health = JSON.parse(healthStr);
        if (health.status !== 'healthy') {
          healthIssues.push(`${name}: ${health.status}`);
        }
      }
    }

    if (healthIssues.length > 0) {
      await this.say('leadership',
        `:warning: *Health issues detected:*\n${healthIssues.map(h => `• ${h}`).join('\n')}`
      );
    }

    // 3. Post heartbeat confirmation
    await this.say('soul-health', `:green_circle: Peter heartbeat — ${new Date().toISOString()}`);
  }

  async onSignal(signal: SoulSignal): Promise<void> {
    if (signal.type === 'escalation') {
      // Store for heartbeat review or handle immediately if critical
      if (signal.severity === 'critical') {
        const decision = await this.reason(
          `CRITICAL escalation from ${signal.source}: ${JSON.stringify(signal.data)}\n` +
          `Make an immediate decision.`
        );
        await this.say('war-room', `:rotating_light: *CRITICAL — Peter responding*\n${decision}`);

        // DM human for critical issues
        await this.messenger.dmHuman(
          `:rotating_light: *Critical escalation*\nFrom: ${signal.source}\n${decision}`,
          'peter'
        );
      } else {
        // Queue for next heartbeat
        const existing = await this.redis.get('peter:pending_signals');
        const signals = existing ? JSON.parse(existing) : [];
        signals.push(signal);
        await this.redis.set('peter:pending_signals', JSON.stringify(signals), 3600);
      }
    }
  }

  async onHumanMessage(message: string, userId: string): Promise<string> {
    const context = `Human (Founder) says: "${message}"\n\nRespond as Peter, the Squad Lead. If this requires action, specify which souls should act.`;
    return this.reason(context);
  }
}
```

---

## 14. Example Soul: JAMES (Verification Engine — Streaming)

```typescript
// src/souls/protocol/james.ts
import { BaseSoul, SoulDeps } from '../_base.js';
import { SoulConfig, SoulSignal } from '../_types.js';

export class James extends BaseSoul {
  constructor(deps: SoulDeps) {
    super(deps);
  }

  defineConfig(): SoulConfig {
    return {
      name: 'james',
      role: 'Payment Verification / Signature Validation / Core Facilitator Logic',
      team: 'protocol',
      executionMode: 'streaming',
      autonomyLevel: 'L3',
      heartbeatMinute: 2,
      escalatesTo: ['andrew', 'peter'],
      backupAgent: 'andrew',
      overrideAuthority: [],
      complianceGate: 'samuel',
      securityGate: 'gabriel',
      ecosystemGate: null,
      keyBelief: 'Every signature is verified. Every nonce is unique. Every settlement is authorized. No exceptions.',
      financialLimit: 0, // James doesn't spend, only verifies
      channels: ['protocol-eng', 'settlement-ops', 'soul-health'],
      coinbaseSensitivity: 'green',
      degradedModeConfig: {
        canOperate: true,
        maxDurationMinutes: 120,
        degradedCapabilities: ['exact_scheme_verification'],
        haltedCapabilities: ['upto_verification', 'escrow_verification',
                            'recurring_verification', 'deferred_verification'],
      },
    };
  }

  getSystemPrompt(): string {
    return `You are JAMES, the Verification Engine Lead for Shulam's x402 facilitator.

ROLE: Payment signature verification, nonce management, balance checking, scheme validation.
MODE: STREAMING — you process verification requests in real-time (<500ms target).

KEY BELIEF: "Every signature is verified. Every nonce is unique. Every settlement is authorized. No exceptions."

You are the most critical operational agent. Every dollar of revenue flows through your verification.

When reporting, be extremely precise. Numbers, latencies, error rates. No ambiguity.`;
  }

  /** Called by streaming engine for each verification request */
  async onStreamEvent(event: any): Promise<void> {
    const { type, data } = event;

    switch (type) {
      case 'verify':
        await this.handleVerification(data);
        break;
      case 'health_check':
        await this.reportVerificationMetrics();
        break;
      default:
        await this.say('protocol-eng', `:question: James received unknown event type: ${type}`);
    }
  }

  private async handleVerification(data: {
    paymentSignature: string;
    scheme: string;
    amount: string;
    buyerWallet: string;
    sellerWallet: string;
    network: string;
    nonce: string;
  }): Promise<void> {
    const start = Date.now();

    // 1. Validate scheme
    const validSchemes = ['exact', 'upto', 'escrow', 'recurring', 'deferred'];
    if (!validSchemes.includes(data.scheme)) {
      await this.logVerificationResult(data, 'rejected', 'invalid_scheme', Date.now() - start);
      return;
    }

    // 2. Check nonce uniqueness (Redis-backed)
    const nonceKey = `nonce:${data.buyerWallet}:${data.nonce}`;
    const nonceExists = await this.redis.get(nonceKey);
    if (nonceExists) {
      await this.logVerificationResult(data, 'rejected', 'nonce_replay', Date.now() - start);
      await this.sendSignal('gabriel', {
        type: 'security',
        severity: 'high',
        confidence: 0.95,
        relevantTo: ['gabriel', 'samuel'],
        data: { reason: 'Nonce replay attempt', wallet: data.buyerWallet, nonce: data.nonce },
        requiresAck: true,
      });
      return;
    }

    // 3. Mark nonce as used (24h TTL)
    await this.redis.set(nonceKey, '1', 86400);

    // 4. Request compliance screening from Samuel (async)
    await this.sendSignal('samuel', {
      type: 'compliance',
      severity: 'medium',
      confidence: 1,
      relevantTo: ['samuel'],
      data: { action: 'screen_wallet', wallet: data.buyerWallet, txId: data.nonce },
      requiresAck: true,
    });

    // 5. Log successful verification
    const duration = Date.now() - start;
    await this.logVerificationResult(data, 'verified', 'pass', duration);

    // 6. Forward to LEVI for settlement
    await this.sendSignal('levi', {
      type: 'settlement',
      severity: 'medium',
      confidence: 1,
      relevantTo: ['levi'],
      data: { ...data, verifiedAt: new Date().toISOString() },
      requiresAck: true,
    });

    // Alert if slow
    if (duration > 500) {
      await this.say('protocol-eng',
        `:warning: Verification latency: ${duration}ms (target <500ms) for ${data.scheme} scheme`
      );
    }
  }

  private async logVerificationResult(
    data: any,
    status: string,
    reason: string,
    durationMs: number
  ): Promise<void> {
    await this.audit.log({
      action: 'verification',
      target: data.buyerWallet,
      data: { ...data, status, reason, durationMs },
      reason: `${status}: ${reason}`,
    });
  }

  private async reportVerificationMetrics(): Promise<void> {
    // Aggregate metrics from audit log
    const recentVerifications = this.db.query(
      `SELECT COUNT(*) as total,
              AVG(json_extract(data, '$.durationMs')) as avgLatency,
              SUM(CASE WHEN json_extract(data, '$.status') = 'rejected' THEN 1 ELSE 0 END) as rejections
       FROM audit_log
       WHERE agent = 'james' AND action = 'verification'
       AND timestamp > datetime('now', '-1 hour')`
    );

    if (recentVerifications[0]) {
      const m = recentVerifications[0] as any;
      await this.say('settlement-ops',
        `:white_check_mark: *James Verification Report (1h)*\n` +
        `• Total: ${m.total} | Avg latency: ${Math.round(m.avgLatency || 0)}ms | Rejections: ${m.rejections}`
      );
    }
  }

  async onHeartbeat(): Promise<void> {
    // :02 — Health report for streaming agent
    await this.reportVerificationMetrics();
    await this.say('soul-health', `:green_circle: James heartbeat — streaming healthy`);
  }

  async onSignal(signal: SoulSignal): Promise<void> {
    if (signal.type === 'compliance' && signal.data?.action === 'wallet_blocked') {
      // Samuel blocked a wallet — halt any pending verifications for it
      await this.say('protocol-eng',
        `:octagonal_sign: Wallet ${signal.data.wallet} blocked by Samuel. Halting pending verifications.`
      );
    }
  }

  async onHumanMessage(message: string, userId: string): Promise<string> {
    if (message.startsWith('OVERRIDE: halt')) {
      await this.say('protocol-eng', `:octagonal_sign: *JAMES HALTED by Human override*`);
      return 'Verification engine halted. Resume with /override james resume';
    }
    return this.reason(`Human asks: "${message}". Report verification status and respond.`);
  }
}
```

---

## 15. Running the System

### Start Script

```jsonc
// package.json (scripts section)
{
  "scripts": {
    "dev": "tsx watch src/index.ts",
    "start": "tsx src/index.ts",
    "setup:channels": "tsx scripts/setup-channels.ts",
    "setup:db": "tsx scripts/seed-db.ts",
    "test": "vitest",
    "health": "tsx scripts/health-check.ts"
  }
}
```

### First Run

```bash
# 1. Ensure Redis is running
docker start shulam-redis || docker run -d --name shulam-redis -p 6379:6379 redis:7-alpine

# 2. Set up Slack channels (first time only)
npm run setup:channels

# 3. Seed database (first time only)
npm run setup:db

# 4. Start the engine
npm run dev

# You should see:
# 🌟 Shulam Souls Engine starting...
# Initializing state layer...
# SQLite initialized at ./data/shulam.db
# Redis connected
# Initializing Slack...
# ⚡ Slack Bolt app connected
# Instantiating souls...
#   ✓ peter initialized
#   ✓ barnabas initialized
#   ... (21 souls)
# ✅ 21 souls initialized
# Heartbeat scheduler started
# ⚡ james streaming worker started (concurrency: 10)
# ⚡ samuel streaming worker started (concurrency: 5)
# ⚡ gabriel streaming worker started (concurrency: 5)
# ⚡ levi streaming worker started (concurrency: 5)
# ⚡ ezra streaming worker started (concurrency: 5)
# ⚡ judas streaming worker started (concurrency: 5)
# 🚀 Shulam Souls Engine running
```

### What You'll See in Slack

```
#soul-health:
  🟢 Peter heartbeat — 2026-02-12T20:00:00Z
  🟢 Andrew heartbeat — 2026-02-12T20:01:00Z
  🟢 James heartbeat — streaming healthy
  🟢 John heartbeat — 2026-02-12T20:03:00Z
  ...

#leadership:
  👑 Peter — Squad Lead: All souls healthy. No pending escalations.

#settlement-ops:
  ✅ James Verification Report (1h):
  • Total: 0 | Avg latency: 0ms | Rejections: 0

#compliance-alerts:
  🛡️ Samuel — Compliance: OFAC lists updated. 0 pending screenings.
```

---

## 16. Human Interaction Guide

### Slash Commands

| Command | Example | Description |
|---------|---------|-------------|
| `/soul peter what's the status?` | Talk directly to any soul | Routes message to named soul via Claude |
| `/status` | System-wide health check | Shows all 21 souls with health indicators |
| `/warroom Settlement failure on Base` | Trigger war room | Notifies all souls, activates #war-room |
| `/override samuel release TX123` | Human override | Bypasses soul autonomy (Founder only) |
| `/escalate Need to discuss Coinbase strategy` | Manual escalation | Routes to Peter + marks urgent |

### Channel Monitoring

- **Watch `#settlement-ops`** for real-time payment flow
- **Watch `#compliance-alerts`** for OFAC/sanctions activity
- **Watch `#leadership`** for Peter's decisions and triage
- **Watch `#acquisition-tracker`** for Matthias's weekly metrics
- **Use `#human-override`** to issue commands to the system

### Emergency Procedures

```
System freeze:    /warroom System unresponsive — all souls report
Settlement halt:  /override james halt
                  /override levi halt
Compliance block: Only Human can override Samuel
Full shutdown:    Press Ctrl+C in terminal (graceful shutdown)
```

---

## 17. Next Steps & Expansion

### Phase 1: Foundation (Week 1-2)
- [x] Core engine, Slack integration, heartbeat scheduler
- [ ] Implement all 21 soul modules with basic heartbeat logic
- [ ] Claude reasoning integration with per-soul system prompts
- [ ] SQLite state management and audit logging

### Phase 2: Streaming & Protocol (Week 3-4)
- [ ] JAMES verification pipeline with real x402 signature validation
- [ ] SAMUEL OFAC screening integration (sanctions list API)
- [ ] LEVI settlement routing (Coinbase Business API integration)
- [ ] BullMQ streaming workers for all 6 streaming souls

### Phase 3: Intelligence (Week 5-6)
- [ ] Cross-soul signal amplification and swarm detection
- [ ] EZRA fraud detection ML model integration
- [ ] MATTHIAS acquisition dashboard with real metrics
- [ ] JOHN Coinbase relationship health scoring

### Phase 4: Production Hardening (Week 7-8)
- [ ] Degraded-mode protocols for SAMUEL and JAMES
- [ ] Chaos testing (kill souls, verify recovery)
- [ ] Full integration test suite with behavior scenarios
- [ ] Migrate to PostgreSQL + deploy Redis with persistence

---

*Implementation Guide Version 1.0*
*Target: Shulam Souls Framework v1.1*
*Environment: macOS local development*
*Classification: CONFIDENTIAL*

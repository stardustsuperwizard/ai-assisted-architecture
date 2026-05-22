# Skill: Serverless AI Application Architecture Design

## Overview

This skill captures the conversational architecture design process used to arrive at a
production-grade serverless AI application. It documents the reasoning pattern, decision
framework, and output format so it can be replicated for future projects.

The process is iterative and Socratic — architecture emerges through questions, tradeoffs,
and progressive refinement rather than being prescribed upfront.

-----

## The Core Process

### Phase 1: Establish the Problem Space

Before recommending any technology, understand:

1. **What is the user trying to accomplish?** (functional goal)
1. **Who are the users?** (scale, technical sophistication, household vs enterprise)
1. **What integrations are required?** (third-party APIs, auth requirements)
1. **What are the constraints?** (budget, existing tools, hosting preferences)

Do not jump to architecture until these are clear. The right stack for two household users
is completely different from the right stack for a SaaS product.

**Example from this project:**

> User wanted AI meal planning + Kroger cart integration for personal household use.
> This immediately ruled out over-engineered solutions and pointed toward simplicity.

-----

### Phase 2: Interface First

Before compute or data, establish how users will interact with the system. This decision
cascades into everything else.

**Options to present:**

- Hosted web app (requires frontend + hosting)
- Chat interface (Slack, Discord, Telegram)
- Voice interface (Alexa, custom)
- CLI tool

**Key questions:**

- Does the user want to build a UI or use an existing platform?
- Is this collaborative (multiple users)?
- Is mobile access important?
- What is the user’s familiarity with each platform?

**Lesson from this project:**

> User initially assumed a hosted web app. Suggesting chat interfaces as an alternative
> eliminated the need for frontend development entirely. Telegram emerged as the best fit
> for personal use due to simplicity, bot creation ease, and inline keyboard support.

-----

### Phase 3: Compute Model

Once the interface is established, determine the compute pattern. Present options with
honest tradeoffs rather than prescribing one solution:

|Model                            |Best for                             |Avoid when                     |
|---------------------------------|-------------------------------------|-------------------------------|
|Always-on server (Droplet, EC2)  |Long-running processes, polling      |Cost-sensitive, low traffic    |
|Container (Fargate, App Platform)|Portable workloads, polling supported|Simple use cases               |
|Serverless (Lambda)              |Event-driven, low/variable traffic   |Long-running processes, polling|
|Home hardware (Raspberry Pi)     |Zero cost personal projects          |Reliability required           |

**Critical distinction to explain:**

> Polling requires a continuously running process. Serverless functions shut down after
> each invocation. These are fundamentally incompatible. Webhooks solve this for serverless
> by inverting the communication direction — the external service calls your function
> rather than your function polling the service.

**Lesson from this project:**

> The polling vs webhook distinction was the key insight that unlocked the serverless
> architecture. Once the user understood this, Lambda + webhook became the obvious choice.

-----

### Phase 4: Eliminate Unnecessary Cost Centers

Review each proposed component for unnecessary cost. For personal/small-scale projects,
cost optimization is often architectural simplification:

- API Gateway → Lambda Function URLs (free, simpler)
- RDS → DynamoDB or SQLite (no idle instance cost)
- NAT Gateway → Public subnets or VPC endpoints
- Separate auth service → Secrets Manager

**Pattern:** For each component, ask “is there a simpler AWS-native alternative that
costs less and has fewer moving parts?”

**Lesson from this project:**

> User noticed API Gateway had a cost. Replacing it with Lambda Function URLs eliminated
> a component entirely — not just reduced cost but simplified the architecture.

-----

### Phase 5: AI/Agent Layer Design

When the application involves AI, establish:

1. **What decisions require AI?** (not everything needs an LLM)
1. **What is the routing strategy?** (hardcoded commands vs natural language)
1. **Which model for which task?** (capability vs cost tradeoff)
1. **How does state persist between stateless invocations?**

**Model selection framework:**

|Task type            |Recommended model|Reason                        |
|---------------------|-----------------|------------------------------|
|Intent classification|Haiku            |Simple, deterministic, cheap  |
|Structured extraction|Haiku            |Known output format           |
|Creative planning    |Sonnet           |Quality justifies cost        |
|Complex reasoning    |Sonnet           |Nuanced understanding required|
|Simple confirmations |No LLM           |Deterministic logic only      |

**Agent vs router pattern:**

> Traditional command routing requires predicting every possible user input.
> Passing messages to an LLM agent with defined tools eliminates routing logic entirely —
> the LLM becomes the router. Define tools with clear descriptions; the agent decides
> when to call them based on the message.

**Lesson from this project:**

> The two-model routing pattern emerged from the user asking a cost question. It became
> a portfolio talking point — demonstrating cost optimization thinking to interviewers.
> At small scale it saves pennies; at large scale it saves significantly.

-----

### Phase 6: Data Layer Design

The data layer is one of the most consequential architectural decisions in any application.
Getting it wrong early is expensive to fix later. Do not jump to a specific database until
you have fully understood your data — its shape, its relationships, its volume, and its
lifecycle.

-----

#### Step 1: Inventory Your Data

Before choosing any technology, classify every piece of data the application will touch:

|Classification|Description                |Examples                                   |
|--------------|---------------------------|-------------------------------------------|
|Identity      |Who the users are          |User profiles, credentials                 |
|Preferences   |User-specific configuration|Dietary restrictions, notification settings|
|Domain data   |Core application entities  |Meal plans, orders, products               |
|Transactional |Events and actions         |Cart additions, purchases, audit logs      |
|Conversational|AI/chat history            |Message threads, session context           |
|Cache         |Derived or fetched data    |API responses, computed results            |
|Secrets       |Sensitive credentials      |OAuth tokens, API keys                     |

**Do this inventory before evaluating any database.** The shape and mix of your data
types will often tell you what kind of store you need.

-----

#### Step 2: Define Retention and Lifecycle

For each data type, answer:

- **How long does this data live?** (seconds, days, forever)
- **What triggers deletion or archival?** (time, user action, event)
- **Does it need versioning or history?** (audit trail, rollback)
- **Who owns it?** (user-scoped, household-scoped, global)

**Retention categories:**

|Category      |Lifetime             |Strategy                  |
|--------------|---------------------|--------------------------|
|Ephemeral     |Seconds to hours     |In-memory, short TTL cache|
|Transient     |Days to weeks        |Database TTL attribute    |
|Semi-permanent|Months               |Scheduled archival job    |
|Permanent     |Indefinitely         |Standard storage, backups |
|Secrets       |Permanent but rotated|Dedicated secrets manager |

**Critical rule:** Never mix retention strategies within the same storage location without
explicit separation. Transient data co-mingled with permanent data creates cleanup debt
and security risk.

-----

#### Step 3: Map Access Patterns

This is the most important step and the one most often skipped.

**List every query your application needs to make before evaluating any database.**

For each access pattern, define:

- What is the question? (“Give me all meal plans for this household”)
- What is the input? (household ID)
- What is the output? (list of meal plans, sorted by date)
- How frequently does this run? (every request vs. weekly)
- What is the acceptable latency? (real-time vs. batch)

**Why this matters:** Different databases are optimized for different access patterns.
A database that handles your primary access pattern beautifully may be painful for your
secondary one. Knowing all patterns upfront lets you make an informed choice — or identify
where you need more than one store.

-----

#### Step 4: Choose the Right Database(s)

Match your access patterns and data characteristics to the right tool. It is legitimate
and often correct to use more than one store for different data types.

**Database selection framework:**

|Database type                                |Best for                                                   |Avoid when                            |
|---------------------------------------------|-----------------------------------------------------------|--------------------------------------|
|Relational (PostgreSQL, MySQL)               |Complex relationships, flexible queries, strong consistency|Massive scale, schema-less data       |
|Document (MongoDB, Firestore)                |Flexible schema, nested objects, rapid iteration           |Complex joins, strict consistency     |
|Key-value / Wide-column (DynamoDB, Cassandra)|Known access patterns, massive scale, low latency          |Ad-hoc querying, complex relationships|
|Time-series (InfluxDB, Timestream)           |Metrics, events, telemetry                                 |General application data              |
|Graph (Neptune, Neo4j)                       |Highly connected data, relationship traversal              |Simple or flat data models            |
|Search (Elasticsearch, OpenSearch)           |Full-text search, faceted filtering                        |Primary data store                    |
|Cache (Redis, ElastiCache)                   |Ephemeral data, session state, rate limiting               |Durable storage                       |
|Secrets manager (Secrets Manager, Vault)     |Credentials, tokens, API keys                              |Application data                      |

**Serverless-friendly options by use case:**

|Use case                 |Serverless-compatible options                               |
|-------------------------|------------------------------------------------------------|
|General application data |DynamoDB, Firestore, PlanetScale, Neon (serverless Postgres)|
|Conversational/AI history|DynamoDB, MongoDB Atlas, Firestore                          |
|Cache                    |DynamoDB TTL, ElastiCache Serverless, Momento               |
|Secrets                  |AWS Secrets Manager, HashiCorp Vault                        |
|Search                   |OpenSearch Serverless, Algolia, Typesense                   |

**Scale consideration:** The right database for two users is often wrong for two million.
Be honest about actual scale. Over-engineering for scale that will never arrive adds cost
and complexity without benefit.

-----

#### Step 5: Design Schema Around Access Patterns

Regardless of database type, schema design follows the same principle:

**Access patterns drive schema. Schema never drives access patterns.**

This is the inversion that trips up most developers moving between database paradigms:

- In relational databases: normalize data naturally, then use flexible SQL to query it
- In document databases: embed or reference based on how you read data
- In key-value databases: design keys to match your exact query shapes

The universal rule: **know your reads before you design your writes.**

**Naming conventions matter in every database:**

Consistent, predictable naming reduces cognitive overhead and bugs regardless of platform.
Apply the same instincts you use for file names, environment variables, and code:

- Be explicit about entity type in identifiers
- Use consistent separators
- Include sortable components (dates, sequences) where ordering matters
- Namespace by ownership scope (user, household, global)

See **Appendix A** for DynamoDB-specific implementation of these principles.

-----

#### Step 6: Plan for Data Integrity and Failure

Before finalizing your data layer design, consider:

- **What happens if a write fails halfway through?** (partial updates, transactions)
- **What is the consistency model?** (eventual vs. strong consistency)
- **How do you handle concurrent writes?** (optimistic locking, conditional writes)
- **What is your backup and recovery strategy?**
- **How do you migrate schema changes?** (versioning, backwards compatibility)

These are not afterthoughts — they are design inputs. A database that cannot support
your consistency requirements is the wrong database regardless of other merits.

-----

#### Lesson from This Project

> User felt they were “using DynamoDB wrong even when it worked.” Root cause: designing
> schema before access patterns — SQL thinking applied to a key-value store. The fix was
> not a different database but a different design order. The naming convention insight
> resonated immediately because the user already applied similar instincts elsewhere
> (file naming, environment variables, git branches). **Connect new data modeling concepts
> to patterns the developer already uses confidently.**

-----

### Phase 7: Scheduled Operations

Identify operations that should run on a schedule rather than on-demand:

- **Cleanup/archival** — remove or archive old data
- **Token refresh** — proactive OAuth token renewal
- **Proactive triggers** — initiate workflows without user action
- **Notifications** — remind users of pending actions

**Pattern:** EventBridge cron → dedicated Lambda per job type. Never combine scheduled
jobs with the main handler Lambda. Separation of concerns keeps each function small,
testable, and independently deployable.

**Cron expression reminder (UTC):**

```
cron(minutes hours day-of-month month day-of-week year)
cron(0 13 ? * SUN *)   # Every Sunday 9am UTC
cron(0 4 ? * MON *)    # Every Monday 4am UTC (Sunday midnight ET)
```

-----

### Phase 8: Security Model

For every application, establish security at each layer:

|Layer               |Mechanism                                                        |
|--------------------|-----------------------------------------------------------------|
|Webhook authenticity|Secret token validation on every invocation                      |
|User authorization  |Allowlist of known user identifiers                              |
|Credential storage  |Secrets Manager, never environment variables for sensitive values|
|Data encryption     |Encrypt sensitive fields before DynamoDB storage                 |
|Network             |Function URL auth mode appropriate to trigger type               |

**Key principle:** Defense in depth. Even if one layer fails, the next catches it.

For personal projects: secret token + user ID allowlist is the minimum viable security
posture. For production SaaS: add WAF, IP allowlisting, rate limiting.

-----

### Phase 9: Observability

Production-readiness is demonstrated through observability. For AWS Lambda, AWS PowerTools
is the standard:

```python
from aws_lambda_powertools import Logger, Tracer, Metrics

logger = Logger()
tracer = Tracer()
metrics = Metrics()

@logger.inject_lambda_context
@tracer.capture_lambda_handler
@metrics.log_metrics
def handler(event, context):
    ...
```

This provides:

- Structured JSON logging (searchable in CloudWatch)
- X-Ray distributed tracing (visualize full request flow)
- Custom metrics (track business events like “meal plans generated”)

For portfolio projects: observability signals production-readiness to interviewers more
than almost anything else.

-----

### Phase 10: Infrastructure as Code

Choose IaC tool based on project complexity and user familiarity:

|Tool          |Best for                               |Avoid when                          |
|--------------|---------------------------------------|------------------------------------|
|AWS SAM       |Lambda-centric apps, simpler syntax    |Complex multi-service architectures |
|CDK (Python)  |Python teams, programmatic logic needed|Simple apps (overkill)              |
|Terraform     |Multi-cloud, existing Terraform teams  |AWS-only simple projects            |
|Manual console|Learning/exploration only              |Anything that needs to be reproduced|

For serverless Lambda applications: AWS SAM is the right default.

-----

### Phase 11: Context Transfer Document

At project completion (or handoff to an IDE/Claude Code), generate two documents:

**ARCHITECTURE.md** — permanent project reference:

- Stack summary table
- Architecture diagram (ASCII or Mermaid)
- Lambda function inventory
- DynamoDB table design with key schema
- Security model
- Access patterns
- Portfolio talking points

**PROMPT.md** — AI onboarding document:

- Complete context of all decisions made
- Exact project structure to scaffold
- Code patterns and conventions to follow
- First task instructions
- Constraints and requirements

The PROMPT.md should be written so that a fresh AI session with zero prior context can
immediately begin productive work without re-litigating architectural decisions.

-----

## Anti-Patterns to Avoid

These are mistakes that emerged as questions in this conversation — common traps to
proactively address:

|Anti-pattern                                       |Correct approach                                     |
|---------------------------------------------------|-----------------------------------------------------|
|Design schema before access patterns               |List access patterns first, derive schema second     |
|Choose a database before inventorying data         |Classify data types and lifecycles first             |
|Mix transient and permanent data without separation|Explicit retention strategy per data type            |
|Apply relational thinking to non-relational stores |Understand the paradigm of your chosen database      |
|Use API Gateway when Function URLs suffice         |Default to Function URLs for simple webhook receivers|
|One Lambda for everything                          |Separate Lambda per distinct job type                |
|Store secrets in environment variables             |Dedicated secrets manager for all sensitive values   |
|Polling in a serverless context                    |Webhooks — invert the communication direction        |
|Over-engineer for scale that will never arrive     |Match architecture to actual scale                   |
|Vibe code without understanding                    |Engage with every decision; be able to explain it    |

-----

## The Conversational Pattern

The architecture emerged through this question sequence. Replicate this order for future
projects:

```
1. What are you building and for whom?
2. How will users interact with it? (interface)
3. Where will it run? (compute)
4. Can we eliminate cost centers? (optimization)
5. What AI decisions are needed? (agent design)
6. What data persists and for how long? (data layer)
7. What runs on a schedule? (async jobs)
8. How do we secure it? (security)
9. How do we observe it? (observability)
10. How do we deploy it? (IaC)
11. How do we hand it off? (documentation)
```

Never skip ahead. Interface drives compute. Compute drives data model. Data model drives
security. Each layer depends on the one before it.

-----

## Reusable Outputs

For every project this skill is applied to, produce:

1. **Stack summary table** (technology + reason for each choice)
1. **Architecture diagram** (ASCII minimum, Mermaid preferred)
1. **Function/service inventory** (trigger + purpose for each)
1. **Data inventory table** (entity, classification, retention, owner)
1. **Access pattern table** (query, input, output, frequency)
1. **Security model table** (layer + mechanism)
1. **Schedule table** (if scheduled jobs exist)
1. **ARCHITECTURE.md** (permanent reference)
1. **PROMPT.md** (AI onboarding document)

-----

## Portfolio Considerations

When a project is intended as a portfolio piece, these additional elements increase
interview value:

- **Two-model routing** — demonstrates cost optimization thinking
- **Deliberate data layer design** — demonstrates ability to reason about data beyond just making it work
- **OAuth 2.0 implementation** — demonstrates auth competency
- **Observability** — demonstrates production-readiness thinking
- **IaC** — demonstrates DevOps awareness
- **README with decision rationale** — demonstrates engineering communication

The strongest portfolio projects are ones where the developer can explain not just
*what* was built but *why* each decision was made and what the tradeoffs were.

-----

## Key Insight Summary

> The best architecture for a project is the simplest one that meets the requirements.
> Complexity should be added only when a simpler alternative genuinely cannot solve the
> problem. Every component has a cost — in money, maintenance, and cognitive overhead.
> The goal is to minimize all three while meeting functional requirements.

-----

## Appendix A: DynamoDB Implementation Guide

This appendix applies the platform-agnostic principles from Phase 6 specifically to
DynamoDB. Use this when DynamoDB has been selected as the data store.

-----

### Why DynamoDB Feels Wrong (And How to Fix It)

DynamoDB requires access patterns to be known before schema is designed. This is the
opposite of relational databases where you normalize data and query flexibly later.

```
Relational:  Store data naturally → query however you need later
DynamoDB:    Decide queries first → design schema around those queries
```

If schema is designed first and access patterns added later, you constantly fight the
database. This is the root cause of most DynamoDB frustration.

-----

### Single Table Design

Put all entities in one table, differentiated by key patterns. This is the DynamoDB
best practice — not one table per entity (SQL thinking).

```
❌ Multiple tables (SQL thinking)
   users_table
   meal_plans_table
   conversation_table

✅ Single table (DynamoDB thinking)
   PK                    SK
   USER#mike             PROFILE
   USER#mike             PREFERENCES
   HOUSEHOLD#main        MEAL#2026-05-18
   PRODUCT#chicken       KROGER#0001234
```

-----

### Key Design

Every access pattern needs a key. DynamoDB has two:

- **Partition Key (PK)** — finds the right “bucket” of data
- **Sort Key (SK)** — filters and sorts within that bucket

The correct design order:

```
1. List ALL access patterns
2. Assign PK + SK to each pattern
3. Add GSIs only for patterns that don't fit PK + SK
4. THEN build the table
```

-----

### Naming Convention

```
ENTITY_TYPE#identifier#subidentifier
```

Rules:

- Entity type in ALL_CAPS
- `#` as separator
- ISO dates for natural chronological sort (`MEAL#2026-05-18`)
- Sub-type after entity where needed (`TOKEN#kroger`, `CONVO#2026-05-20`)
- Consistent taxonomy across the entire table

This is the same naming instinct applied to file names, environment variables, and
git branches — just applied to keys.

**Examples:**

```
USER#mike                    → Mike's root record
USER#mike#PREFERENCES        → Mike's preferences
HOUSEHOLD#main#MEAL#2026-05-18  → This week's meal plan
PRODUCT#chicken-breast#KROGER#0001234  → Cached Kroger product
```

-----

### Access Pattern Table (Template)

|Query                   |PK              |SK condition                  |
|------------------------|----------------|------------------------------|
|Get all user data       |`USER#id`       |—                             |
|Get user preferences    |`USER#id`       |`eq(PREFERENCES)`             |
|Get last N meal plans   |`HOUSEHOLD#main`|`begins_with(MEAL#)` + limit N|
|Get today’s conversation|`USER#id`       |`eq(CONVO#date)`              |
|Get cached product      |`PRODUCT#name`  |`begins_with(KROGER#)`        |

-----

### Global Secondary Indexes (GSIs)

Add a GSI only when a required access pattern cannot be answered by PK + SK:

```
Main table:   PK=USER#mike, SK=MEAL#2026-05-18
              → "give me all of Mike's meals" ✅

GSI needed:   → "give me all users who have a plan this week"
              GSI PK: WEEK#2026-05-18
              GSI SK: USER#mike
```

Rule: **one GSI per additional access pattern that doesn’t fit the main key.**

-----

### Data Retention in DynamoDB

|Data type           |Strategy                                |
|--------------------|----------------------------------------|
|Conversation history|DynamoDB TTL attribute (7 days)         |
|Cache data          |DynamoDB TTL attribute (30 days)        |
|Meal plans          |Scheduled Lambda archival (keep 8 weeks)|
|User preferences    |Permanent — no TTL                      |
|OAuth tokens        |Permanent — encrypted at rest           |

Never rely on application code to delete expired data. Use TTL for time-based expiry
and scheduled jobs for business-rule-based archival.
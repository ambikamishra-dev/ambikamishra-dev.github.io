# CaseDesk — Service Cloud Support Hub

A customer-support platform for **Meridian Consumer Goods** (fictional manufacturer), built end-to-end the way a real org grows: record-level security, declarative automation, an Apex trigger framework, a resilient external integration, and an Agentforce agent that deflects cases before they exist.

**Live write-up:** [ambikamishra-dev.github.io](https://ambikamishra-dev.github.io) · Built by [Ambika Mishra](https://www.linkedin.com/in/ambika-mishra23) — Salesforce Certified Platform Developer I & Administrator

## The problem
Meridian's support team drowns in repetitive questions (returns, shipping, warranty) while real order issues wait. CaseDesk answers the repetitive questions with an AI agent grounded on Knowledge, routes what remains into properly-typed Cases, and escalates high-priority work automatically, on a security model that holds up under real user logins.

## Architecture and status

| Layer | What | Status |
|---|---|---|
| C1 · Data model & security | `Order__c` (lookup to Account), Case record types (Product Question / Order Issue), role hierarchy, Private OWD, owner-based sharing rule, permission set | ✅ shipped |
| C2 · Escalation automation | Record-triggered Flow: Priority → High assigns Escalations queue + custom notification | ✅ shipped |
| C3 · Trigger framework | One-trigger-per-object on Case, handler/service classes, recursion control, bypass permission | in progress |
| C4 · SLA batch | Nightly Batch + Schedulable scanning cases open > 48h | planned |
| C5 · LWC case cockpit | Account open-case history on the Case page via `@wire` | planned |
| C6 · External integration | Named Credential → fulfillment API, 4xx/5xx branching, retry Queueable, `Integration_Log__c` | planned |
| C7 · Agentforce agent | Service agent: Knowledge-grounded Q&A subagent ✅, case-creation Flow action + order-status Apex action | in progress |

## Design decisions (the part worth reading)

- **Lookup, not master-detail, for Order → Account.** Orders can exist without an account (guest checkout) and must survive account deletion. Accepted tradeoff: no roll-up summary fields; totals become aggregate queries.
- **Record types over separate objects for the two case flavors.** Same lifecycle, shared queues/escalation/reporting; only fields and picklists differ. Different lifecycle is when you reach for a new object, which is exactly why `Order__c` is one.
- **The sharing rule targets a sibling branch, not the manager.** Private OWD plus the role hierarchy already grants managers visibility downward. A rule aimed at the manager would be dead configuration. Verified with real logins before writing any rule.
- **No hardcoded Ids anywhere.** The escalation queue and the custom notification type are both resolved at runtime by `DeveloperName` via Get Records. Ids differ between environments; API names travel.
- **Agent grounding is deliberate.** Knowledge articles feed an Agentforce Data Library (identifying fields: Title + Article Number; content field: the article body). Instructions constrain the agent to Knowledge-only answers with case creation as the escalation path.
- **Verification by login.** Every security claim is proven with test users (agent sees own cases; manager sees downward for free; nothing leaks sideways without a rule) before the layer is called done.

## Field notes
Real traps hit during the build, documented in the [build diary](https://ambikamishra-dev.github.io/casedesk-build.html): user licenses gating which profiles are even possible, Case record types requiring a Support Process first, record-type deletion order (unhook profile defaults first), and Agentforce Data Library quirks (a field can be identifying or content, not both; category filters hide untagged articles; indexing is not instant).

## Stack
Salesforce Platform · Apex · Flow · LWC · Agentforce (subagents, Data Library, Flow & Apex actions) · SFDX · Git

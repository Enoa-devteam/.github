# Enoa — Vision & Architecture

[English](VISION.md) · [简体中文](VISION.zh-CN.md) · [Organization](https://github.com/Enoa-devteam)

> **Document type:** organization-level vision and reference architecture  
> **Status:** direction and engineering principles, **not** a declaration of a completed integration  
> **Last reviewed:** 2026-11-11

## 1. The idea

**Conversation is the interface. Engineering is the goal.**

A conversational assistant is good at understanding incomplete requests, preserving context, and making interaction approachable. A coding agent is good at operating within a software workspace: inspecting code, editing files, running tests, and producing artifacts. Neither role should have to impersonate the other.

**Enoa aims to connect these two roles into engineering-capable chatbots.** A user should be able to express an intention naturally, without memorizing command syntax, and receive either a useful answer or a traceable attempt to carry out an authorized task.

The central distinction is **talking about an action versus taking an action**. The system must recognize that distinction and enforce it at execution time.

We are building reusable components toward this vision, not claiming a general-purpose autonomous engineer is already available.

## 2. What the system should do

A successful system should:

1. **Understand the conversation.** Retain relevant context and memory without consuming an unbounded prompt.
2. **Choose the right level of work.** Respond in dialogue, use a bounded tool, ask a clarifying question, or recommend a multi-step coding-agent task.
3. **Separate reasoning from authority.** A model or router may propose an action; trusted host policy alone admits it.
4. **Execute with boundaries.** Restrict workspace, permissions, tool exposure, time, resources, and side effects to the granted scope.
5. **Report evidence, not confidence theater.** Distinguish attempted, succeeded, partially succeeded, failed, and uncertain outcomes; include the relevant checks and artifacts.
6. **Recover predictably.** An outage or ambiguous result should not silently repeat an operation or become a claim of success.

We do **not** define success as maximizing the number of agent calls. Ordinary questions should remain ordinary conversations. A short deterministic tool call should not become an expensive coding session.

## 3. Reference architecture

The following is a **target integration model**. Arrows describe responsibilities and intended interaction, not proof that the repositories already communicate over those connections.

~~~text
User / chat platform
        |
        v
Trusted host boundary
(identity, source, context, current grants)
        |
        v
Stella — conversation and orchestration
        |
        +--> Dialogue / memory / answer
        |
        +--> Existing capability routing --> Comes / tool providers
        |                                       |
        |                                       +--> local tools and plugins
        |                                       +--> potential Ligamina MCP tools
        |
        +--> [planned] Alcor decision advice
                     (chat / clarify / delegate / abstain)
                           |
                           v
                 Host admission gate
          (request binding, scope, permissions,
           workspace, execution profile, health)
                           |
                     if admitted
                           v
                  Cometa / coding agent
                    (bounded workspace)
                           |
        +------------------+-------------------+
        |                                      |
        v                                      v
Tool outcomes                          Agent outputs / tests
        |                                      |
        +------------------+-------------------+
                           |
                           v
              Verification and provenance
                           |
                           v
                Conversational feedback
~~~

**Important:** The branch from Stella to Alcor and the admission flow are proposed integration work, not shipped organization-wide behavior. Ligamina's MCP tool surface exists independently; its presence in this drawing does not assert a completed Stella↔Ligamina integration.

### Responsibilities and ownership

| Layer | Primary responsibility | Must not become |
| --- | --- | --- |
| **Stella** — conversational host | Context, dialogue, memory, capability routing, result presentation, and host-side execution boundaries | An unrestricted command interpreter just because a user is chatting |
| **Alcor** — decision adviser | Decide whether the current trusted context merits conversation, clarification, abstention, or *proposed* coding-agent delegation | A source of permissions, approvals, workspace grants, or task execution |
| **Cometa / external coding agent** | Handle admitted multi-step engineering work such as repository inspection, changes, and tests | A self-authorizing executor |
| **Ligamina** — plugin tool surface | Run native Koishi, NoneBot, and AstrBot plugins through their original frameworks; expose commands through an MCP-style interface; produce version-bound compatibility evidence | A guarantee that every third-party plugin is safe or compatible |
| **Trusted host / admission policy** | Bind identity and source request, verify grants and scope, choose permitted execution profiles, maintain idempotency | A model-generated claim such as “authorized=true” |

The host is a **responsibility boundary**, not necessarily a new standalone service. Implementations may distribute its checks across existing components, but they must not delegate authorization to model output.

## 4. Three kinds of requests

The distinction should be visible to users:

| User intent | Preferred behavior | Example |
| --- | --- | --- |
| **Discuss / explain** | Answer conversationally; do not trigger engineering execution | “Why can this test be flaky?” |
| **Inspect / query** | Use an appropriate read-only tool or seek permission for a bounded coding-agent inspection | “Inspect this repository and identify the failing test.” |
| **Modify / operate** | Verify explicit authority and scope before any side effect; otherwise clarify or decline execution | “Apply the fix in this workspace and run the tests.” |

A follow-up such as “go ahead” can refer to a previous **specific** proposal; it is not an unlimited grant. The host must verify the proposal reference, user, revision, target, and still-valid authorization. Quoted text, another user's instruction, and an agent's own suggestion are not execution grants.

The choice of an AI model must not silently change those rules.

## 5. Execution and trust boundaries

The long-term architecture follows these invariants:

- **Trusted origin:** identity, message source, workspace binding, and permissions come from authenticated host state, never from the language model's interpretation.
- **Advisory decisions:** Alcor's recommendations can be rejected by the host. An unknown or abstained decision cannot be promoted into a delegation.
- **Inspect is not modify:** read-only and write-capable execution profiles must be separate and independently verified. Write privileges are not implied by access to a workspace.
- **Bounded tasks:** pass a specific objective, permitted scope, budget/deadline, and a stable request identifier to an executor. For retried submissions, reuse the originating idempotency identity.
- **Untrusted tools:** plugins, retrieved documents, repository files, and agent output are data from a lower-trust boundary. Expose only allowed tools and treat outputs as evidence requiring validation, not instructions to expand authority.
- **Honest outcomes:** timeout, uncertainty, cancellation, partial success, and confirmed success are different states. When delivery or execution is ambiguous, check recorded state rather than replaying side effects blindly.
- **Context discipline:** do not flood Stella's dialogue window with tool definitions, raw logs, or an agent's entire trace. Preserve task references and return bounded, attributed summaries while retaining evidence out of band.
- **Privacy and locality:** local-first is an option, not a blanket claim of zero network traffic. Any online model, external coding agent, platform connector, or networked plugin must have an explicit data-flow boundary.

These are **requirements for an eventual integrated system**. Individual repositories document which guarantees have been implemented and verified.

## 6. How the projects fit together today

This section distinguishes evidence from ambition; it should be updated as integration changes.

| Project | Publicly documented state (2026-11-11 review) | Integration still to establish |
| --- | --- | --- |
| [Stella](https://github.com/Eternal-Wanderer-Vegetable/Stella_project) | Chatbot, memory, capability routing, Comes tool execution, and configurable model deployment are documented in its repository | End-to-end acceptance of the proposed Alcor decision and authorization flow |
| [Alcor](https://github.com/Enoa-devteam/Alcor) | P01 provides a strict decision contract, rules backend, local API/client, and contract tests. Its decisions are **not** permissions and it does **not** submit Cometa/Codex jobs | Calibrated model evaluation, Stella shadow integration, read-only admission, and separately gated write admission |
| [Ligamina](https://github.com/Eternal-Wanderer-Vegetable/Ligamina) | Core/Hosts and MCP command tools have bounded test evidence for selected plugins in Koishi, NoneBot, and AstrBot | Catalog-wide behavioral compatibility, real-platform delivery evidence, and host-specific integration where needed |

Read the project repositories for their **latest** status rather than interpreting this dated snapshot as a release promise. At this review, Stella and Ligamina remain linked to their existing repository owners; Alcor is already under Enoa.

### Primary engineering references

- **Stella:** [Capability Router & Comes](https://github.com/Eternal-Wanderer-Vegetable/Stella_project/blob/main/docs/architecture/capability-system.md) · [Architecture index](https://github.com/Eternal-Wanderer-Vegetable/Stella_project/blob/main/docs/architecture/README.md)
- **Alcor:** [Decision contract v0.1](https://github.com/Enoa-devteam/Alcor/blob/main/docs/contracts/decision-v0.1.md) · [Progress](https://github.com/Enoa-devteam/Alcor/blob/main/progress.md) · [Engineering plan](https://github.com/Enoa-devteam/Alcor/blob/main/docs/plans/2026-10-31-alcor-engineering-plan.md)
- **Ligamina:** [Plugin-as-tool decision (ADR 0002)](https://github.com/Eternal-Wanderer-Vegetable/Ligamina/blob/main/docs/adr/0002-plugins-as-tools.md) · [Bot/agent integration](https://github.com/Eternal-Wanderer-Vegetable/Ligamina/blob/main/docs/guides/bot-integration.md) · [Current verification status](https://github.com/Eternal-Wanderer-Vegetable/Ligamina/blob/main/README.md)

## 7. Delivery and verification strategy

We propose progressing by **evidence-backed gates**, not by promising a fully autonomous experience in one release.

| Gate | What must be demonstrated |
| --- | --- |
| **A. Contracts and local baselines** | Pinned interfaces, deterministic contract tests, negative cases, failure and timeout semantics |
| **B. Decision quality and shadow mode** | Human-adjudicated test sets; false-positive delegation tracking; shadow predictions with **no additional side effects** |
| **C. Admitted read-only workflows** | Real restricted workspace and sandbox, validated identity/scope, end-to-end result attribution, rollback to existing behavior |
| **D. Admitted write workflows** | Explicit grants and approval where needed, limited write profile, reproducible tests, idempotency, recovery, audit and rollback |
| **E. Interoperability and field validation** | Version-pinned plugin behavior coverage and separately verified platform delivery; no universal compatibility claim from a small pilot |

At each gate, record **what was tested**, **what was not tested**, and **what would revoke admission**. A green API test cannot stand in for model-quality evidence, an actual sandbox, or a real platform delivery.

## 8. Non-goals and trade-offs

Enoa is **not** committed to one model provider, one coding-agent implementation, one chatbot framework, or a single operating system. Modular boundaries matter more than forcing every capability into one process.

We do not intend to make every message an execution request, automatically grant filesystem or network privileges, or equate plugin discovery with proven compatibility. Engineering capability must be useful **and** bounded.

Efficiency, local deployment, extensibility, reliability, and safety can pull in different directions. Design decisions should document those trade-offs and keep experiments separate from released guarantees.

## 9. Contributing to this vision

This document defines a common direction, **not a replacement for each project's contracts or ADRs**. Propose architecture changes through issues or pull requests in the affected repository, with concrete scenarios, trust boundaries, alternatives, tests, and migration impact. Cross-project proposals should link the relevant work on both sides.

The best contribution is one that makes the system **more capable without making its behavior less understandable or less verifiable**.

---

*Enoa Development Team — Building the bridge between conversation and engineering.*

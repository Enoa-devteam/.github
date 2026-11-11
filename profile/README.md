# Enoa

**CONVERSATION IS THE INTERFACE. ENGINEERING IS THE GOAL.**

*Open-source chatbots with practical engineering capabilities — connecting conversational AI, developer tools, and coding agents.*

Enoa is an independent open-source development team exploring a specific idea: **a chatbot should be able to do more than answer questions.** It should understand a request in conversation, choose an appropriate capability, and help turn an intention into an actionable engineering workflow.

Our focus is the **integration of chatbots and coding agents**: natural interaction on one side; tools, software development workflows, and verifiable execution on the other. The goal is not to replace either, but to make them work together reliably.

**Enoa 是一个开源开发组织，致力于构建具备实际工程能力的聊天机器人。** 我们希望让自然对话成为任务入口，将聊天机器人的交互与记忆能力，同工具系统和 Coding Agent 的代码理解、修改、测试等工程能力有机结合——不仅讨论问题，也能在明确授权和可验证的流程下协助解决问题。

---

## From Chat to Action

A useful engineering-capable chatbot needs more than a powerful language model. It needs a way to distinguish conversation from requests for action, select the right executor, manage boundaries, and report results back clearly.

Our architectural direction is:

```text
                  Human request
                        |
                        v
              Conversational chatbot
              (dialogue, context, memory)
                        |
                        v
             Intent & capability routing
                        |
             +----------+-----------+
             |                      |
             v                      v
      Tools & plugins          Coding agent
      (bounded actions)     (multi-step code work)
             |                      |
             +----------+-----------+
                        |
                        v
             Results, verification
             & conversational feedback
```

**This is a design direction, not a claim that every integration is already complete.** We are building and validating these parts incrementally.

In practice, that means working toward systems that can:

- **Converse naturally** while maintaining relevant context and memory.
- **Decide when to act** instead of treating every message as either small talk or an unconditional command.
- **Use deterministic tools and plugins** for tasks that do not require an autonomous coding workflow.
- **Delegate appropriate engineering tasks to coding agents**, including code changes and test-oriented workflows, through explicit boundaries.
- **Handle failures visibly** with traceable decisions, controlled permissions, and recoverable execution paths.

## Projects

| Project | Role in the ecosystem |
| --- | --- |
| **[Stella](https://github.com/Eternal-Wanderer-Vegetable/Stella_project)** | The conversational foundation: a memory-aware chatbot with modular capabilities, tool routing, and local/online model options. |
| **[Alcor](https://github.com/Enoa-devteam/Alcor)** | An independent decision engine for recognizing when a request calls for conversation, clarification, or delegation to an external agent. **Integration is under development.** |
| **[Ligamina](https://github.com/Eternal-Wanderer-Vegetable/Ligamina)** | A plugin interoperability and verification layer spanning the NoneBot, Koishi, and AstrBot ecosystems, exposing native plugin capabilities to bots and coding agents. |

Stella and Ligamina currently link to their existing repositories; project ownership and links may evolve as organization migration continues.

These are complementary building blocks, **not a claim of a finished end-to-end product**. Each project documents its own implementation status, limitations, and verification evidence.

## Engineering Principles

- **Conversation-first, action-capable.** Natural interaction is the entry point; useful, bounded execution is the objective.
- **Explicit authority.** A model may recommend an action, but the host system controls whether and how it executes.
- **Reliability over demos.** Prefer tests, observable behavior, failure handling, and reproducible evidence to unsupported capability claims.
- **Efficient by design.** Respect context-window, compute, and deployment constraints; support local-first operation where practical.
- **Open and interoperable.** Build reusable interfaces rather than locking a bot to one model, coding agent, or plugin framework.

## Contribute

We welcome engineering discussions, bug reports, reproducible test cases, documentation improvements, and code contributions.

Start with the repository closest to your interest, review its current status and contribution guidance, and use its Issues or Pull Requests to discuss a concrete change.

---

**Enoa Development Team**  
*Building the bridge between conversation and engineering.*

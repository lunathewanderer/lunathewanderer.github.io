---
title: Agentic AI, in a nutshell
draft: false
date: 2026-11-23T13:04:38.420Z
categories:
  - Learning
lastmod: 2026-11-25T10:17:52.677Z
description: "♫⋆｡♪ ₊˚♬ ﾟ Let's go below zero, and hide from the sun / I'll love you forever where we'll have some fun ♫⋆｡♪ ₊˚♬ ﾟ."
tags:
  - AI/ML
cover: /images/posts/agentic-ai-in-a-nutshell-c48f71/violetslep-1795439293492-5632c1.jpg
banner: /images/posts/agentic-ai-in-a-nutshell-c48f71/violetslep-1795439293492-5632c1.jpg
---

# 1. Introduction

There is no gainsaying the influence of Agentic AI. 

However, to understand them is a different thing. Like, what really differs Agentic AI from the traditional AI, you may wonder?

> Simply put, traditional AI was fundamentally designed to executes specific, preprogrammed, or single-step predictions.
> 
> On the other hand, Agentic AI refers to autonomous AI systems that plan, make decisions, use tools, receive real-emvironment feedback, and execute multi-step reasoning and workflows to pursue open-ended goals with minimal human intervention.

But there is more than meets the eye. And that is the very purpose of this blog. To be specific, this blog will only mainly delve into the two concepts: **Context Engineering** and **Harness Engineering**.

# 2. Context Engineering

## 2.1. `AGENTS.md` & `SKILL.md`

Inside every agents, there is a `Model, or Large Language Model (LLM)`, which kinda shapes how the agent behaves. 

`LLM` does the actual text generation, the reasoning, and by itself, it's our conversational partner. What turns it into an agent is the instruction layer that's wrapped around the model. This brings us to our first terminology: `AGENTS.md`

> `AGENTS.md` is a plain Markdown file placed at the root of a project/code repository to give AI coding agents project-specific context, rules, and instructions. You can think of it as a `README.md` specifically written for AI agents rather than us human readers.

The agent, therefore, execute commands it finds in `AGENTS.md` whenever they are contextually relevant. 

It is also of noted that `AGENTS.md` can be nested, meaning that there can be multiple of them. For example, we can have one at the root, and then multiple other ones for sub-projects with its own set of rules. Importantly, files that are closer to the working directory override the earlier ones, because they apppear later.

Indeed, `AGENTS.md` is centralized, comprehensive, and persistent. **But what about the knowledge that the agent only needs sometimes and is not neccessarily project-specific?** Like, for example, the agent needs to know how to build a PowerPoint deck, but throughout the whole project, it only need to do it only once or twice. Loading all of that context every single time to agent starts would really clog up the context window for no real reason if the task at hand has nothing to do with PowerPoint slides. 

While AI excels at **declarative knowledge** (knowing facts and descriptions about the world), it struggles with **procedural knowledge** (knowing how to execute, adapt, and refine actions in real time). When encounting a task that requires a 67-steps workflow to polish a PowerPoint report for example, it basically has two options: 
- Either somebody needs to prompt it with every single step, all 67 of them, every single time.
- Or the agent would take a guess at it, which can really go astray from the desired result.

This brings us to **[Agent Skills](https://agentskills.io/home)**, the medium via which you add that **Procedural Knowledge** into the agent.

Intrinsically, an **Agent Skill** is a folder (named after the skill), and inside that folder, there is a file called `SKILL.md` acts as an entry point, along with whatever scripts , references, assets, or resources the task needs.

```
skill-name/
├── SKILL.md          # Required: metadata + instructions
├── scripts/          # Optional: executable code
├── references/       # Optional: documentation
├── assets/           # Optional: templates, resources
└── ...               # Any additional files or directories
```

In any `SKILL.md` file, there is two section: the YAML Frontmatter (metadata) and the Markdown Body (instructions).

- **YAML Frontmatter (metadata)**

This section located at the very top of the file enclosed in `---` lines. It is lightweight (around ~100 tokens) and read by the agent at startup for discovery and activation. There is two required fields:
- `name`: Unique identifier using lowercase letters, numbers, and hyphens only (max 64 characters; e.g., "code-review").
- `description`: Concise summary of what the skill does and **when the agent should trigger it** (max 1024 characters).

This part is crucial, since a clear trigger condition would not only save context window (less redundant skill activation) but also determine the effectiveness of the output (an ambiguous condition could result in the agent skipping the skill activation completely, despite it being necessary).

Optional fields of the metadata section may include `version`, `license`, `author`, `repository`, `homepage`, `allowed-tools`, to name but a few.

- **Markdown Body (instruction)**

This is the core operational instructions loaded into context only when the skill is actively triggered, written in plain human languages. 

However, this is where a lot of skills go sideways, because the temptation is to just have the LLM write the skills for you. What you get in so doing is merely a generic mush, since those are **what the model already knew. At the end of the day, the whole point of a skill is your specific way of doing a specific job. Therefore, the content has to come from somewhere the model can't get to on its own.** In order to achieve that, you can apply "Trial-and-error" & "Artifact Synthesis":
- Trial-and-error: Write down what really worked (after you've walked through the task, including the corrections you made along the way). To draw out the best of a workflow, envrionment-specific facts can even defy the most reasonable assumptions.
- Artifact Synthesis: Utilize external references such as old papers, reports, books, comments.

Moreover, in the same spirit of writing an IELTS essay (talking from experience btw), a good instruction body should also:

- Be specific, rather than be generic
- Include relevant examples
- Use numbered steps for procedures
- Set boundaries
- Keep it focused

Nevertheless, the art of writing `SKILL.md` goes beyond those simple tips and tricks. 

Let's say, you have hundreds of **Agent Skills**. Loading all of them into the LLM context window at startup would blow through the token budget before anyone even gets to ask a question, thereby defeating their very meaning in the first place: load a skill only when it deems necessary. 

Therefore, **Agent Skills** use something called **Progressive Disclosure**. Simply put, it discloses additional information only when it is needed, and it is what keeps the context clear. This concept works in three layers.
- The first layer lies in the metadata. At startup, the agent only sees the name and the description of those skills. Even if you have configured many Skills, it only burn a handful of tokens, like reading the Table of Contents.
- The second layer lies in the instruction body. Only when it selects and aptly activates a skill, that's when it actually reads the rest.
- The third layer lies in the resources folders, like `/references`, `/scripts`, or `/assets`. They are also touched upon only when the instruction says so.

So, when the agent finally decide to read a Skill, the instruction body would then be shared with everything else already in the context window. Or in other words, **every single word in the skill body is now competing for the model's attention. Therefore, the less you write, the better.** 

It might seem counterintuitive, since you'd naturally assume a more detailed skill description yields better results. In reality, the AI is already highly capable on its own, and in this context, "writing less" simply means only including context the model wouldn't inherently know from its training data. 

It is recommended that the instruction body of `SKILL.md` file should be under 500 lines, or roughly 5,000 tokens. But what if you really need to write down more than that? Well, remind that Agent Skills is a folder, and you can split the content out into subfolders like `/references`, and the agents will open them only when it actually needs to. This refers to the third layer of **Progressive Disclosure**.

In the same vein, you can make use of deterministic scripts with the folder `/scripts`. Being way too concise may run the risk of rendering the model generate its "guessy" logic on the fly in according with fragile and ambiguous requests. Simply put, it is more reliable to just tell the model to "run this script in this step" rather than having it to make probabilistic decisions. Moreover, this perfectly aligns with that **Progressive Disclosure** spirit right above.

## 2.2. Beyond Agent Skills

With `AGENTS.md` and `SKILL.md` in hand, the agent now knows what to do, **but doing things also means reaching outside the box**. As a matter of course, agents need to reach all kinds of external variables, such as APIs, databases, developer tools, SaaS platforms, so on and so forth, all of which with different variants of interfaces and methodologies for connection. This is where `MCP` comes in handy.

> `Model Context Protocol (MCP)` is an open standard that lets AI applications connect to external data sources, tools, and software systems in a uniform manner. It comes with something called an MCP server, which wraps up an external tool or data source in a standard interface; and now any agent (that can speak MCP) can now communicate with that tool.

`MCP` covers agents talking to tools and data, **but what about agents talking to other agents?** 

> `Agent2Agent (A2A)` is an open protocol that enables independently developed AI agents to communicate and collaborate across different platforms and frameworks. An agent can publish an *Agent Card* that describes its capabilities, how to contact it, and what authentication it requires. A requesting agent can use this information to delegate work. The receiving agent can reply directly or create a task whose progress can be tracked and whose outputs, such as documents or structured data, can be returned as artifacts. This common interface allows agents to coordinate without sharing their underlying models, tools, or internal state.

Another technique to provide knowledge to AI Agents is through ` Retrieval-Augmented Generation (RAG)`, an AI framework that connects large language models (LLMs) to **external, declarative knowledge** sources so they can pull relevant facts before generating an answer. 

When a user submits a query, the system searches an external knowledge source by performing a semantic search. Those matching chunks are returned to the language model as additional context. The model uses this context to generate its response without updating its parameters. 

As a result, `RAG` can improve factual accuracy and give the model access to **declarative knowledge** that was unavailable during training, though its effectiveness depends on the relevance and reliability of the retrieved material.

This brings us to the last point I want to make in this section: **context engineering is analogous to cognitive science**. 

Human us possess three types of memory: 
- Semantic Memory, which are facts *(like I'll definitely be lonely this autumn)*
- Episodic Memory, which are experiences *(like that time when you hold your crush's hand for once)*
- and Procedural Memory, which are skills *(like the muscle memory in how you ride a bike)*

And at the end of the day, agent architectures are none other than mirrors to us human beings:
- Semantic Memory is mapped to the abovementioned knowledge-based RAG
- Episodic Memory is mapped to conversational logs and interaction history, or the context window
- and Procedural Memory is mapped to those `SKILL.md`

Moreover, they can be interconnected as well as complementary. Considering this architecture:
- MCP servers connect agents to the outside world (data, APIs).
- Skills teach agents how to use MCP tools effectively.
- Agent reasoning enables the selection of appropriate skills and tools.

Possibilities abound if you think about it this way.

## 2.3. Can You Trust This Skill?

More often than not, you may stumble upon such Agent Skills on the Internet. They can contain and run executable, as well as the ability to access stuff, all of which makes them powerful. But, that is also what makes trust so important. Skills, however omnipotent they are, can be flawed security-wise, or just straight up containing prompt injection or malware. Those tidbits of "tool poisoning" or "prompt injection" would be delved into in future posts, but for now, the moral of the story is, treat an Agent Skill like any other dependency, which is to say, don't give it the benefit of the doubt right away, but to read what it does and check what it reaches out to.

# 3. Harness Engineering

## 3.1. A brief history of Agentic Engineering

Flashback to the release of ChatGPT, where the context window is merely about 4000 tokens. Such limited amount of context really stymies us from achieving anything substantial, so we started to emerge on how to recycle that bits of memory, do more with less, and quickly expanded to **Context Engineering** via various techniques such as MCP, RAG, Agent Skills, for the purpose of managing the context window more effectively.

**Context Engineering** has been a major update from the primordial **Prompt Engineering**. But with the advent of technology, it is just the underlying for what to come: **Harness Engineering**

## 3.2. The problem with Context Engineering

## 3.3. Subagents

The scope and complexity of modern projects is getting bigger and bigger. And more often than not, just one agent is not enough, especially when the task is
- too big, that would blow out one's context on its own, like reviewing a code base with thousands of files
- too parallel, like checking on 20 different functions where each check is independent, and doing them all at once would be 20 times faster

With a view to augment efficiency, the parent agent spawns multiple child agents to do a specific piece of work, each of which has its own fresh context window. They do their given job and they return a result when it is done, which their parent carries on with its context intact.

## 3.4. Loops

## 3.5. Agent Harness

# References
- [@IBMTechnology](https://www.youtube.com/@IBMTechnology)
- [SoK: Agentic Skills - Beyond Tool Use in LLM Agents](https://arxiv.org/html/2602.20867v1#S1)
- [SecRespond: Benchmarking AI Agents for Real-World Post-Compromise Incident Response](https://arxiv.org/html/2607.26791v1#S3)










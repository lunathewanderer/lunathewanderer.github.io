---
title: Agentic AI, in a nutshell
draft: false
date: 2026-11-23T13:04:38.420Z
categories:
  - Learning
lastmod: 2026-11-24T06:37:51.903Z
description: "♫⋆｡♪ ₊˚♬ ﾟ Let's go below zero, and hide from the sun / I'll love you forever where we'll have some fun ♫⋆｡♪ ₊˚♬ ﾟ."
tags:
  - AI/ML
cover: /images/posts/agentic-ai-in-a-nutshell-c48f71/violetslep-1795439293492-5632c1.jpg
banner: /images/posts/agentic-ai-in-a-nutshell-c48f71/violetslep-1795439293492-5632c1.jpg
---

# I. Introduction

There is no gainsaying the influence of Agentic AI. 

However, to understand them is a different thing. Like, what really differs Agentic AI from the traditional AI, you may wonder?

> Simply put, traditional AI was fundamentally designed to executes specific, preprogrammed, or single-step predictions.
> 
> On the other hand, Agentic AI refers to autonomous AI systems that plan, make decisions, use tools, receive real-emvironment feedback, and execute multi-step reasoning and workflows to pursue open-ended goals with minimal human intervention.

But there is more than meets the eye. And that is the very purpose of this blog.

# II. Anatomy of the Agentic AI

Inside every agents, there is a `Large Language Model (LLM)`, which kinda shapes how the agent behaves. 

`LLM` does the actual text generation, the reasoning, and by itself, it's our conversational partner. What turns it into an agent is the instruction layer that's wrapped around the model. This brings us to our first terminology: `AGENTS.md`

> `AGENTS.md` is a plain Markdown file placed at the root of a project/code repository to give AI coding agents project-specific context, rules, and instructions. You can think of it as a `README.md` specifically written for AI agents rather than us human readers.

The agent, therefore, execute commands it finds in `AGENTS.md` whenever they are contextually relevant. 

It is also of noted that `AGENTS.md` can be nested, meaning that there can be multiple of them. For example, we can have one at the root, and then multiple other ones for sub-projects with its own set of rules. Importantly, files that are closer to the working directory override the earlier ones, because they apppear later.

Indeed, `AGENTS.md` is centralized, comprehensive, and persistent. **But what about the knowledge that the agent only needs sometimes and is not neccessarily project-specific?** Like, for example, the agent needs to know how to build a PowerPoint deck, but throughout the whole project, it only need to do it only once or twice. Loading all of that context every single time to agent starts would really clog up the context window for no real reason if the task at hand has nothing to do with PowerPoint slides. This brings us to term number two: **"Agent Skills"**

Intrinsically, an Agent Skill is a folder (named after the skill), and inside that folder, there is a file called `SKILL.md` acts as an entry point, along with whatever scripts , references, assets, or resources the task needs.

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

However, this is where a lot of skills go sideways, because the temptation is to just have the LLM write the skills for you. What you get in so doing is merely a generic mush, since those are **what the model already knew. At the end of the day, the whole point of a skill is your specific way of doing a specific job. Therefore, the content has to come from somewhere the model can't get to on its own.**

Here are some good practices in writing an instruction


At startup, the agent only sees the name and the description, but when it selects a skill, that's when it actually reads the rest.



# III. Appendix

## MCP & A2A

With `AGENTS.md` and `SKILL.md` in hand, the agent now knows what to do, **but doing things also means reaching outside the box**. As a matter of course, agents need to reach all kinds of external variables, such as APIs, databases, developer tools, SaaS platforms, so on and so forth, all of which with different variants of interfaces and methodologies for connection. This is where `MCP` comes in handy.

> `Model Context Protocol (MCP)` is an open standard that lets AI applications connect to external data sources, tools, and software systems in a uniform manner. It comes with something called an MCP server, which wraps up an external tool or data source in a standard interface; and now any agent (that can speak MCP) can now communicate with that tool.

`MCP` covers agents talking to tools and data, **but what about agents talking to other agents?** Introducing `Agent2Agent (A2A)`, an open standard that lets independent AI agents discover each other, negotiate tasks, and collaborate across different frameworks and vendors. 

For example, we have two agents: one handling procurement and one finance. The procurement agent needs to negotiate a contract, and it needs the finance agent's approval. With A2A, they establish something called "Agent Card" - basically a description of what the agent does and how to talk to it, via with the agents can communicate with each other.




# References
- IBM Technology








---
title: Agentic AI, in a nutshell
draft: false
date: 2026-11-23T13:04:38.420Z
categories:
  - Learning
lastmod: 2026-11-23T15:48:20.313Z
description: "♫⋆｡♪ ₊˚♬ ﾟ. I'll love you forever where we'll have some fun* ♫⋆｡♪ ₊˚♬ ﾟ."
tags:
  - AI/ML
cover: /images/posts/agentic-ai-in-a-nutshell-c48f71/violetslep-1795439293492-5632c1.jpg
banner: /images/posts/agentic-ai-in-a-nutshell-c48f71/violetslep-1795439293492-5632c1.jpg
---

# I. Introduction

There is no gainsaying the influence of Agentic AI. 

However, to understand them is a different thing. Like, what really differs Agentic AI from the traditional AI, you may wonder?

Simply put, traditional AI was fundamentally designed to executes specific, preprogrammed, or single-step predictions.

On the other hand, Agentic AI refers to autonomous AI systems that plan, make decisions, use tools, receive real-emvironment feedback, and execute multi-step reasoning and workflows to pursue open-ended goals with minimal human intervention.

But there is more than meets the eye. And that is the very purpose of this blog.

# II. Anatomy of the Agentic AI

Inside every agents, there is a `Model`, or in other words, `Large Language Model (LLM)`, which kinda shapes how the agent behaves. 

`LLM` does the actual text generation, the reasoning, and by itself, it's our conversational partner. What turns it into an agent is the instruction layer that's wrapped around the model. This brings us to our first terminology: `AGENTS.md`

> `AGENTS.md` is a plain Markdown file placed at the root of a project/code repository to give AI coding agents project-specific context, rules, and instructions. You can think of it as a `README.md` specifically written for AI agents rather than us human readers.

The agent, therefore, execute commands it finds in `AGENTS.md` whenever they are contextually relevant. 

It is also of noted that `AGENTS.md` can be nested, meaning that there can be multiple of them. For example, we can have one at the root, and then multiple other ones for sub-projects with its own set of rules. Importantly, files that are closer to the working directory override the earlier ones, because they apppear later.

Indeed, `AGENTS.md` is centralized, comprehensive, and persistent. **But what about the knowledge that the agent only needs sometimes and is not neccessarily project-specific?** Like, for example, the agent needs to know how to build a PowerPoint deck, but throughout the whole project, it only need to do it only once or twice. Loading all of that context every single time to agent starts would really clog up the context window for no real reason if the task at hand has nothing to do with PowerPoint slides. This brings us to term number two: `SKILL.md`



# Reference

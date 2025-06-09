---
title: "Three Months with AI: Building, Coding, and Contributing"
date: 2025-06-09T10:00:00+01:00
tags: ["ai", "openstack", "development", "experimentation"]
draft: false
---

# Three Months with AI: From Experiment to Daily Workflow

Over the past three months, I've been intentionally exploring how AI can fit into my software development workflow. Here's what I learned.

## Building with AI: The Prototype Experiment

My initial foray into AI tooling was building a prototype to understand the fundamentals. I spent about 6 hours creating [ca-bhfuil](https://github.com/SeanMooney/ca-bhfuil), a simple experiment in combining multiple AI components:

- **LangChain and LangGraph** as the agent framework
- **Ollama** for local LLM hosting  
- **sqlite-vec** as a basic vector database for RAG experimentation

The result was a very simple prototype using hardcoded data to demonstrate
storing commit messages in a vector database, prompting an LLM to use RAG
to retrieve relevant data, and generating responses. You can see an [example response in the commit message](https://github.com/SeanMooney/ca-bhfuil/commit/c1df8b24a50098e6a27946d103d962349f40c2a2).

This is far from being a useful tool, but it effectively demonstrated the
basics of building an AI agent. The project is on hold for now, but I may
return to the concept in the future.

## AI in Daily Development

### The IDE Evolution

For several years, I've had AI available in my IDE through both VS Code
and Emacs. Last quarter, I was using **Continue** with local models hosted on
**Ollama**. In fact, that was how I mainly used AI for the last 12 months due to
changes in my company's AI policy. This quarter, I briefly experimented with
the new **Granite Code** extension, but it was a regression from Continue for
my workflow since I run Ollama remotely — a use case they don't focus on.

### Exploring AI Coding Agents

In Q2, I took time to experiment with other AI coding agents, specifically
**RooCode**, **Aider**, and **Claude Code**.

#### RooCode: The Power User's Dream

**RooCode** is a powerful agentic open-source agent that extends VS Code. One
of the AI YouTubers I follow released custom modes for Roo called
[**Micromanager mode**](https://github.com/adamwlarson/RooCodeMicroManager),
which prompted me to explore using local models for routine AI code
changes while reserving larger models for orchestration and complex tasks.

The idea was to have a local-first approach to AI tooling. Unfortunately,
the hardware I currently have for running LLMs doesn't quite have enough
memory to work effectively with RooCode. Models advanced enough to handle
large amounts of context and support tool calling, like the latest release
of [**Devstral**](https://mistral.ai/news/devstral), are tuned to run on
systems with 32GB of RAM. My M4 Mac Mini has 16GB, and quantized models
that do fit proved unreliable or too slow to be effective.

However, I did use RooCode with hosted models via **OpenRouter** and **Google Cloud**
to develop a [Nova specification](https://review.opendev.org/c/openstack/nova-specs/+/951222), and I
wrote a [detailed blog post](https://www.seanmooney.info/blog/ai-to-spec/)
about that process, focusing on how I used AI to work around my dyslexia
and produce higher-quality initial versions of specifications.

#### Aider and Gemini Deep Research: The Eventlet Challenge

**TL;DR**: Used AI to research and debug OpenStack's Python 3.13 compatibility issues, exploring everything from high-level analysis to experimental code solutions. While the technical solutions weren't production-ready, AI proved valuable for rapid prototyping and problem exploration.

OpenStack has a significant problem: it uses a concurrency framework called Eventlet (a Python library that enables cooperative multitasking) which is incompatible with Python 3.13, preventing OpenStack from running on the latest Python version. This became an excellent test case for AI-assisted debugging and research.

##### Understanding the Problem

I used **Gemini's** deep research capabilities to explore the background and potential paths forward. After spending an hour or two working with AI to better understand how changes in Python's threading model due to **PEP 703** might cause the errors we're observing, I produced a comprehensive analysis document.

Unfortunately, the research made a strong argument for why it's more important to move off Eventlet quickly rather than trying to extend it to newer Python versions. The conclusion was clear: this isn't a problem we should solve by patching Eventlet.

##### Exploring Technical Solutions

Despite the research conclusions, I wanted to explore potential workarounds. After 3 hours of trying to use AI to develop a simple [reproducer](https://github.com/eventlet/eventlet/issues/1032), I used **Gemini** to brainstorm several different possible fixes in Nova, producing another technical analysis document.

The brainstorming session explored several approaches:

**Arena Allocation Strategy**: Originally I wanted to explore if we could take an arena allocation strategy (a memory management technique where objects are allocated in large blocks) to try and keep the eventlet stack frame alive, but since the underlying bug is with liveness tracking of the frame and the objects in them, that likely won't work and it's not easy to do in Python anyway.

**Thread Local Storage**: My next idea was to use thread local storage (data that's unique to each thread) or a module level WeakKeyDict to store strong references to the HTTP clients, keyed off the eventlet greenthread (Eventlet's lightweight thread equivalent). While clever, it's also too clever and not something we should sprinkle over the code base.

**Context Manager Approach**: In the end we arrived at possibly using a context manager to extend the lifecycle, so I iterated with **Ollama** and some local models to create a hold context manager and then refined it with more powerful models.

##### The Hold Context Manager Experiment

The most interesting experiment was developing a context manager to extend object lifecycles:

<details>
<summary>Click to see the Hold context manager implementation</summary>

```python
import threading
from typing import Any, Optional, TypeVar, Dict, Union, cast

T = TypeVar('T')

class Hold:
    """
    Context manager that holds an object in thread-local storage to prevent it from being collected.
    
    The context manager creates a namespace in thread-local storage keyed by the context manager's
    identity (using its memory address) and stores the object there. This ensures the object remains
    accessible even if the garbage collector attempts to deallocate it during context switches in
    eventlet green threads.
    """
    
    def __init__(self, obj: T) -> None:
        self.obj = obj
        self._key = (id(self), hash(obj))  # Precompute key once

    def __enter__(self) -> T:
        threading_locals = threading.local()
        setattr(threading_locals, self._key, self.obj)
        return self.obj

    def __exit__(self, exc_type, exc_value, traceback) -> None:
        threading_locals = threading.local()
        try:
            delattr(threading_locals, self._key)
        except Exception:
            pass
        return None
```
</details>

I have not proved to myself that this is a good approach either, so for now I'm capturing this experiment in this blog post for future reference.

##### AI vs Human Solutions

The most fascinating part of this exercise was comparing AI-generated solutions with human expertise. For one of the reported issues, we had a concrete reproducer, so I used Aider with OpenRouter to generate a [possible solution](https://github.com/eventlet/eventlet/pull/1044).

What was particularly interesting: both **DeepSeek** and a human developer ([Hervé's approach](https://github.com/eventlet/eventlet/pull/1031)) independently identified the same problematic function. When I asked AI to generate a solution without mentioning Hervé's work, it focused on the same code area.

The AI solution probably isn't correct, but this convergence suggests AI can effectively identify problem areas, even when the proposed fixes need significant human refinement.


## Creating an AI Style Guide for OpenStack

**TL;DR**: Developed a comprehensive style guide for using AI with OpenStack development, iterating through multiple AI tools to create both human-readable guidelines and tool-optimized versions.

Moving from debugging to documentation, my next major AI experiment focused on creating reusable guidelines for AI assistance in open source development.

As I used AI more with open source and OpenStack in particular, I wanted
to create an AI style guide to encode important information for AI to
follow. One thing that became apparent quickly is that paid models are
still much more powerful than local models, even though local models are
rapidly catching up.

Regardless of whether it's hosted or local, open or closed, any usage of
AI with open source has critical requirements that must be followed.

I created the [OpenStack AI Style Guide](https://github.com/SeanMooney/openstack-ai-style-guide) through an iterative process:

1. **Initial Draft**: Used **Aider** with **DeepSeek-R1** to summarize style guidance from the [OpenStack hacking repo](https://github.com/openstack/hacking/blob/master/HACKING.rst) and its [style checks](https://github.com/openstack/hacking/tree/master/hacking/checks).

2. **Enhancement**: Asked **Aider** to add non-conflicting enhancements from Google's latest Python style guide and **PEP 8**.

3. **Policy Integration**: Uploaded to **Claude.ai** and iterated with **Claude Sonnet 4** to incorporate guidance from:
   - [OpenInfra AI Policy](https://openinfra.org/legal/ai-policy)
   - [DCO replacement resolution](https://raw.githubusercontent.com/openstack/governance/refs/heads/master/resolutions/20250520-replace-the-cla-with-dco-for-all-contributions.rst)

4. **OpenStack Conventions**: Refined the style to better reflect how [OpenStack commit messages](https://wiki.openstack.org/wiki/GitCommitMessages) are written.

5. **Tool Optimization**: Used **Claude** to distill the guide into a tool-focused version that minimizes context length and token usage alongside the comprehensive guide.

After about 20 iterations with **Claude** and several earlier versions, I used **Claude Code** to create a repository that would be easy for AI tools to consume. The end result is the [**OpenStack AI Style Guide**](https://github.com/SeanMooney/openstack-ai-style-guide).

## Beyond Code: AI-Enhanced Development Environment

**TL;DR**: Used AI to modernize and maintain my Emacs configuration, turning a procrastinated maintenance task into a streamlined, accessibility-focused development environment.

While most of my AI experiments focused on coding and documentation, I also explored how AI could improve my daily development environment. This led to a significant overhaul of my Emacs configuration—an exercise that perfectly demonstrated AI's potential for maintenance tasks I'd been avoiding.

### The Emacs Modernization Project

For years I had put off creating a proper Emacs config, preferring minimal customization. But when VS Code started getting on my nerves, I decided to give building out a sophisticated Emacs environment a proper shot. Rather than using Doom or Spacemacs, I stripped everything back to basics and started a [literate config from scratch](https://github.com/SeanMooney/emacs/blob/master/lit.org).

The transformation was dramatic: AI helped me consolidate scattered settings into proper use-package blocks, fix critical startup errors, and embrace true literate programming principles. The result was nearly 300 lines of reduction while dramatically improving maintainability.

**Key AI-Assisted Improvements:**
- Proper use-package conventions for both third-party packages and built-in settings
- Accessibility features including dyslexia-friendly font presets with OpenDyslexic
- Distraction-free writing environments via olivetti
- Comprehensive spell-checking workflows with writegood-mode and codespell integration
- Direct Claude Code integration for AI-assisted configuration management

This experiment highlighted an important insight: **AI excels at maintenance tasks we tend to procrastinate on**. The maintenance overhead that had kept me from building a custom environment became manageable with agentic coding tools like Claude Code and Aider.

### The Accessibility Connection

The Emacs modernization tied directly back to my earlier AI writing experiments. By integrating AI assistance directly into my editing environment, I created a seamless workflow for the kind of writing enhancement that proved so valuable for technical documentation.


## Lessons Learned: Three Months of AI Integration

After experimenting with AI across prototyping, debugging, documentation, and environment configuration, several clear patterns emerged about where AI genuinely helps versus where it falls short.

### AI Excels At:
- **Structured writing and editing**: Transforming rough technical concepts into polished documentation
- **Research synthesis**: Combining information from multiple sources into coherent analysis
- **Iterative refinement**: Improving drafts through focused feedback loops
- **Accessibility**: Helping work around learning disabilities like dyslexia
- **Maintenance tasks**: Handling tedious configuration and cleanup work that humans procrastinate on
- **Pattern recognition**: Identifying similar code structures or problem areas across different contexts

### AI Struggles With:
- **Domain expertise**: Deep institutional knowledge about OpenStack's architecture and conventions
- **Community context**: Understanding the social and political aspects of open-source development
- **Hardware limitations**: Local models require significant resources for complex tasks
- **Tool integration**: Many AI coding tools aren't optimized for remote or self-hosted setups
- **Production readiness**: Generated solutions often need significant human refinement for real-world use

### The Sweet Spot

The most effective approach isn't having AI replace human expertise, but using it as a collaborative partner.
AI handles the mechanical aspects of writing and research while human expertise guides architectural decisions,
technical trade-offs, and community dynamics.

It's worth noting that AI won't replace your writing entirely. Your initial draft and ideas remain essential. AI excels at enhancing the skeleton you create, making content more readable and coherent.

## Looking Forward

This three-month experiment convinced me that AI can be a powerful tool for OpenStack development when used thoughtfully. Rather than replacing human expertise, AI augments it by helping experienced contributors work more efficiently and potentially helping newer contributors learn faster.

Using AI to learn is a double-edged sword. When I first used AI (Copilot) to help me work in Go, it made me feel productive in the new language environment much faster than traditional learning methods.

However, that ease of onboarding comes at a cost. Without struggling through challenges, you won't embed knowledge as deeply. I don't recommend new programmers rely heavily on AI until they've built critical thinking skills and can reason about the effects of their changes. AI will confidently lead you down the wrong path, and as tools become more powerful, analyzing their suggestions becomes more important, not less.

The real success isn't in any individual tool or technique, but in discovering how AI can lower barriers to high-quality technical contribution. My writing workflow isn't perfect yet, but it's significantly better than before and continues to evolve.

The future I'm excited about isn't one where AI writes code for us. Instead, I envision AI helping us express our original intent more clearly and providing tools that enable us to write better code ourselves.

---

*If you're interested in experimenting with AI in your OpenStack work, check out the [OpenStack AI Style Guide](https://github.com/SeanMooney/openstack-ai-style-guide) and feel free to reach out on IRC (sean-k-mooney) or through the usual OpenStack channels.*

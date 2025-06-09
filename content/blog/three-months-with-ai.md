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
and Emacs. Last quarter, I was using Continue with local models hosted on
Ollama. In fact that was how i mainly used ai for the last 12 month due to
changes my company  ai policy. This quarter, I briefly experimented with
the new Granite Code extension, but it was a regression from Continue for
my workflow since I run Ollama remotely — a use case they don't focus on.

### Exploring AI Coding Agents

In Q2, I took time to experiment with other AI coding agents, specifically
RooCode, Aider, and Claude Code.

#### RooCode: The Power User's Dream

RooCode is a powerful agentic open-source agent that extends VS Code. One
of the AI YouTubers I follow released custom modes for Roo called
[Micromanager mode](https://github.com/adamwlarson/RooCodeMicroManager),
which prompted me to explore using local models for routine AI code
changes while reserving larger models for orchestration and complex tasks.

The idea was to have a local-first approach to AI tooling. Unfortunately,
the hardware I currently have for running LLMs doesn't quite have enough
memory to work effectively with Roo. Models advanced enough to handle
large amounts of context and support tool calling, like the latest release
of [Devstral](https://mistral.ai/news/devstral), are tuned to run on
systems with 32GB of RAM. My M4 Mac Mini has 16GB, and quantized models
that do fit proved unreliable or too slow to be effective.

However, I did use Roo with hosted models via OpenRouter and Google Cloud
to develop a [Nova specs](https://review.opendev.org/c/openstack/nova-specs/+/951222), and I
wrote a [detailed blog post](https://www.seanmooney.info/blog/ai-to-spec/)
about that process, focusing on how I used AI to work around my dyslexia
and produce higher-quality initial versions of specifications.

#### Aider and Gemini Deep Research

OpenStack has a significant problem: it uses a concurrency framework
called Eventlet, which is incompatible with Python 3.13, preventing
OpenStack from running on 3.13. I used Gemini's deep research capabilities
to explore the background and potential paths forward.

I spent an hour or two working with AI to better understand how changes in
Python's threading model due to PEP 703 might cause the errors we're
observing, producing a report: Debugging Eventlet GC Python 3.13

Unfortunately, first documents make a strong argument for why it's more
important to move off Eventlet quickly rather than trying to extend it to
newer Python versions.
as it is actully hosted in my work accout unlike the rest of my example i
wont link ti here.

after 3 hour of trying to use ai to develop a simple repoducer https://github.com/eventlet/eventlet/issues/1032
i used gemini to also brainstorm several diffent posisbel fixes in nova
produceing yet another report - Nova HTTP Client Lifetime Options

orgially i wanted to exproe if we could perhaps take an arena allcoation stragiy ot try and keep the eventlet stack frame alive
bug since the underlying bug is with livenesss trackingk fo the frame and hte object in them that likely wont work and
it not easy to do in ptyhon anyway.

my next idea was to thread local storage or a module level  WeakKeyDict to store strong refence to the http clinets
key off the eventlet greenthread.

whiel that is a clever solution it also a bit too clever and not somethign we shoudl sprinkel over the code base.
in the end we arrive at possibel using a context manager to extend the lifecyle os i tierated with ollam and some local modes
to create a hold context manager and then refined it to more powerful models.

```
import threading
from typing import Any, Optional, TypeVar, Dict, Union, cast

T = TypeVar('T')
"""
Type variable for generic types.
"""

class Hold:
    """
    Context manager that holds an object in thread-local storage to prevent it from being collected.
    
    The context manager creates a namespace in thread-local storage keyed by the context manager's
    identity (using its memory address) and stores the object there. This ensures the object remains
    accessible even if the garbage collector attempts to deallocate it during context switches in
    eventlet green threads.
    
    Attributes:
        obj (T): The object being held during the context
    
    Methods:
        __enter: Stores the object in thread-local storage and returns it
        __exit: Removes the object from thread-local storage and handles cleanup
    """
    
    def __init__(self, obj: T) -> None:
        """
        Initialize the Hold context manager with the object to hold.
        
        Args:
            obj: The object that will be kept alive during the context's lifetime
        """
        self.obj = obj
        self._key = (id(self), hash(obj))  # Precompute key once

    def __enter__(self) -> T:
        """
        Enter the runtime context.
        
        Creates a unique namespace in thread-local storage using the context manager's identity.
        This namespace allows other context managers to maintain separate storage while still
        being able to prevent object collection.
        
        Returns:
            T: The stored object
        """
        threading_locals = threading.local()
        setattr(threading_locals, self._key, self.obj)
        return self.obj

    def __exit__(self,
                 exc_type: Optional[type] = None,
                 exc_value: Optional[BaseException] = None,
                 traceback: Optional[Union[None, 'traceback.TracebackType']] = None) -> None:
        """
        Exit the runtime context.
        
        Removes the object from the thread-local storage namespace to prevent memory leaks.
        If an exception occurred within the context, it will be handled by the context consumer
        and not by this manager.
        
        Args:
            exc_type: The exception type, if any
            exc_value: The exception instance, if any
            traceback: The exception traceback, if any
            
        Returns:
            None: No exception handling is done by this manager
        """
        threading_locals = threading.local()
        
        # Handle cleanup - remove the stored object from thread-local storage
        try:
            delattr(threading_locals, self._key)
        except Exception:
            pass
        
        # Return None (meaning no exception is handled by this context manager)
        return None

``

i have nto proved to my self that htis is a good approch either so for now i will capture this in this blog post for later me.

finally for 
For one of the two reported issues, we had a concrete reproducer, so I
used Aider (a terminal-based AI coding agent) with OpenRouter to generate
a [possible solution](https://github.com/eventlet/eventlet/pull/1044).
i think a better solution may be to actully refactor oslo.service in
eventlet mode to not use fork but it was interestign to me at least
that deepseek and a human both narrowed in on the same function. https://github.com/eventlet/eventlet/pull/1031
i tried generating a solution with ai (without telling it about hervé work)
to see if it coudl find a solution that would pass eventlets unit
tests. since herve mentioned that "this patch broke more 40 unit tests
(local run)" but didnt say which.

so without biaising the ai usign the same repoducer i wanted to see if it
coudl come up with a solution and it did. is it correct? proably not but
it is too the same part of the code and at least somewhat related to
hervé's approch.


## Creating an AI Style Guide for OpenStack

As I used AI more with open source and OpenStack in particular, I wanted
to create an AI style guide to encode important information for AI to
follow. One thing that became apparent quickly is that paid models are
still much more powerful than local models, even though local models are
rapidly catching up.

Regardless of whether it's hosted or local, open or closed, any usage of
AI with open source has critical requirements that must be followed.

I created the [OpenStack AI Style Guide](https://github.com/SeanMooney/openstack-ai-style-guide) through an iterative process:

1. **Initial Draft**: Used Aider with DeepSeek-R1 to summarize style guidance from the [OpenStack hacking repo](https://github.com/openstack/hacking/blob/master/HACKING.rst) and its [style checks](https://github.com/openstack/hacking/tree/master/hacking/checks).

2. **Enhancement**: Asked Aider to add non-conflicting enhancements from Google's latest Python style guide and PEP 8.

3. **Policy Integration**: Uploaded to Claude.ai and iterated with Claude Sonnet 4 to incorporate guidance from:
   - [OpenInfra AI Policy](https://openinfra.org/legal/ai-policy)
   - [DCO replacement resolution](https://raw.githubusercontent.com/openstack/governance/refs/heads/master/resolutions/20250520-replace-the-cla-with-dco-for-all-contributions.rst)

4. **OpenStack Conventions**: Refined the style to better reflect how [OpenStack commit messages](https://wiki.openstack.org/wiki/GitCommitMessages) are written.

5. **Tool Optimization**: Used Claude to distill the guide into a tool-focused version that minimizes context length and token usage alongside the comprehensive guide.

After about 20 iterations with Claude and several earlier versions, I used Claude Code to create a repository that would be easy for AI tools to consume. The end result is the [OpenStack AI Style Guide](https://github.com/SeanMooney/openstack-ai-style-guide).

## enhancing my emacs config

  Over the past few days, my Emacs configuration has undergone significant evolution, while i started with a curated literate config it has transitioning from a
  basic setup functioanl setup to a sophisticated config with enhanced ai integration.
  
  The most dramatic change was a complete
  structural refactor that consolidated scattered settings into proper use-package blocks, fixed critical
  startup errors, and embraced true literate programming principles—reducing the codebase by nearly 300
  lines while dramatically improving maintainability.

while there wasnt anything wrong persay in how i organised the most of the
section before there is a use_package convention where use_package can
be used to configure inbuilt setting not jsut third party packages.
before i used that inconsistently but ai was able to help me refine
my config an fix some bugs along the way.

  The configuration has also gained a strong focus on accessibility and professional writing. New additions
   include dyslexia-friendly font presets with OpenDyslexic, distraction-free writing environments via
  olivetti, and comprehensive spell-checking workflows with writegood-mode and codespell integration.
  Perhaps most notably, I've integrated Claude Code directly into the workflow, enabling AI-assisted
  configuration management and code review. The result is a modern, accessible, and AI-enhanced Emacs
  environment that prioritizes both developer productivity and inclusive design.
  
This is perhaps partly an exersie in spite drivern developemnt rather then
something i expect others to use. for years i have put off ever creating
my own emacs config as i tend to not heivlly customise the tools i use.
but a few months ago vscode just got on my nerves so i unearthed my old
custom emacs config and decied to actully give building it out proably a shot. instead of usinge doom or spacemacs i stript everythign back to basics
and started a literate config form scratch. the result.
https://github.com/SeanMooney/emacs/blob/master/lit.org

part of the reaosn i avoided this for so long is the maintance overhead
of buildign a custom envioenment like this. i think with agentic
coding tools like claude_code or adier i can do that with less of a time
sync then beofre so i will likely continue building this out ot make
may day to day life better.


## What I Learned

### AI Excels At:
- **Structured writing and editing**: Transforming rough technical concepts into polished documentation
- **Research synthesis**: Combining information from multiple sources into coherent analysis
- **Iterative refinement**: Improving drafts through focused feedback loops
- **Accessibility**: Helping work around learning disabilities like dyslexia

### AI Struggles With:
- **Domain expertise**: Deep institutional knowledge about OpenStack's architecture and conventions
- **Community context**: Understanding the social and political aspects of open-source development
- **Hardware limitations**: Local models require significant resources for complex tasks
- **Tool integration**: Many AI coding tools aren't optimized for self-hosted setups.

### The Sweet Spot

The most effective approach isn't having AI replace human expertise, but using it as a collaborative partner.
AI handles the mechanical aspects of writing and research while human expertise guides architectural decisions,
technical trade-offs, and community dynamics.

As an aside AI will not replace all of your writing, draft prose is still an important initially input.
it can however enhance the skeleton you lay down to make the content more readable an coherent.

## Looking Forward

This three-month experiment convinced me that AI can be a powerful tool for OpenStack development when used thoughtfully.
It's not a replacement for human expertise,
but it can augment it—helping experienced contributors work more efficiently and potentially helping newer contributors learn faster.

using Ai to learn is a double edged sword.
As i observed when first usign Ai (copilot) to help me work in golang more quickly,
ai can make you feel productive in a new coding environment like a new language.

that ease of onboradign comes at a cost, if you dont stuggele and over come challanges as you "learn"
then you wont embded the knowledge as deeply. As such i still do not recommend using ai hevillry to
new programmer. if you have not buidl the skills of critical thinkink and reasoning about the effects
of your chagne ai will happily lead you down the worng path and as tools get more powerful its more
important not less to analyse there actions.

The real success isn't in the individual tools or techniques, but in
discovering how AI can lower barriers to high-quality technical
contribution. I dont yet have have a perfect writign wrokflow, but
what i have built is now at least better then before an i will
likely evolve it more going forward.

The future I'm excited about isn't one where AI writes code for us, but one
where it helps you express your orgininal intent more clearly and provide tools for you to write better code.

---

*If you're interested in experimenting with AI in your OpenStack work, check out the [OpenStack AI Style Guide](https://github.com/SeanMooney/openstack-ai-style-guide) and feel free to reach out on IRC (sean-k-mooney) or through the usual OpenStack channels.*

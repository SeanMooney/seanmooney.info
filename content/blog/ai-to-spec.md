---
title: "Auto-Correcting Sean-Speak: Beyond Spell-Check to Real AI Assistance"
date: 2025-06-08T10:00:00+01:00
tags: ["openstack", "nova", "LLMs", "specifications", "development"]
draft: false
---

# Using AI to Write OpenStack Nova Specifications: A Real-World Experiment.

## Background and Motivation

After attending the 2025.2 PTG and seeing the extensive backlog of specs that
need to be written, reviewed, and implemented, I decided to run an experiment:
can AI meaningfully help with the OpenStack specification process? Not just the
writing, but the actual design thinking and technical architecture work that
goes into a good Nova spec.

The motivation was simple - we have more good ideas than we have time to
properly document them. Between my day job, core review responsibilities, and
the general time constraints we all face, turning a concept into a
well-structured, technically sound specification can take days or weeks. What
if AI could help accelerate that process?

### The Personal Challenge

As many of ye may know, i am severely _severely_ dyslexic.
While the English dialect i speak is technically Hiberno-English, or Irish
English, calling the raw output of my writing that is a stretch. Core reviewers
for years have been kind enough to help translate the `sean-speak` that leaves
my fingertips back into English.

Over the years i have developed several strategies to deal with this, but
there have been times where creating a document with all the technical rigor
required in something like a spec or blog post may take me 8-10 hours just for
it to take me an additional 3-4 hours just to proof read and spell check it
with multiple tools like Grammarly, my browser spell checker and editors
spellcheck just for there to still be issues when a human reads it.

So when on a good day it takes me only half as long to correct a document as
to write the base content and on a bad day it can literally take 2-3 times as
long to do so, there is potentially a lot to be gained by using AI to aid in
my writing.

### The Broader Impact

This isn't just about me though - there are likely other contributors in the
OpenStack community and beyond who face similar challenges. Learning
disabilities, non-native English speakers, contributors who are brilliant
engineers but struggle with written communication - AI assistance could be a
game changer for inclusive participation in open source communities. If this
experiment proves successful, it might open doors for more diverse voices to
contribute to technical discussions that have traditionally required strong
written English skills.

### The Timing

Well why now is a good question but with a pretty simple answer:
* Like many companies, Red Hat would like us all to explore AI and what we can
  use it for day to day.
* Until recently AI tools were auto-complete on steroids and kind of terrible
  at this, but in the last 3-6 months with release of LLMs like DeepSeek-R1 and
  Google Gemini Flash 2.5, coupled with tools like RooCode, the landscape has
  changed a lot.
* I was asked to review a technical solution proposal drafted by one of our
  very talented Solution Architects Greg Procunier. What better time than when
  you are provided a problem statement, use cases and even a potential approach
  to apply my knowledge of nova to how I would address the problem? After all,
  the best way to learn how to use a tool is to try and use it to address a
  real world problem.

### The Challenge: CPU Performance Tiers

As is disappointingly common with nova the concept that Greg outlined in
[openstack-cgroup-tiering](https://github.com/gprocunier/openstack-cgroup-tiering)
is something that we have discussed tangentially many times over the years. In
fact in his technical plan Greg explains how a combination of old and very old
openstack features can be combined to achieve the usecase without any code
change.

So why a new spec? simple it should be possible for mere mortals to achieve
the use case without over a decade of contributing to nova or cloud solution
architecture experience to define cpu performance level in OpenStack!

**Technical Background**: The existing approach requires deep knowledge of Nova's scheduler filters, Placement service resource classes, libvirt domain XML generation, cgroups configuration, and the interactions between all these components. Greg's proposal aimed to make CPU performance tiers as simple as selecting a flavor.

## The Experiment Setup

**TL;DR**: Used RooCode with Google Gemini Flash 2.5 and DeepSeek-R1 to iteratively develop a comprehensive Nova specification, transforming AI's initial rough attempt into a production-ready design through systematic refinement.

There is a standard in writing often used in U.S. military communication
known as BLUF (Bottom line up front). Well I'm Irish so Better late than
never, the intent of this blog is not to discuss The details of how to modify
nova to support "CPU Performance Service Levels using CPU Shares and Placement
Resource Classes". Instead the goal is to discuss the experience and process
of using ai to write the spec. By necessity some of the details of the spec
will also be addressed here but as a framework for the conversation rather
than as the focus.

### The Failed First Attempts

This is actually my 3rd attempt as initially i tried to use Gemini pro and
the web chat interface to do this followed by using google docs gemini
integration and both failed spectacularly.

The web chat interface couldn't maintain context across the multiple reference
documents I needed - it would forget about the nova-specs template when I was
asking about Greg's technical plan, or lose track of OpenStack Placement
concepts when discussing scheduler changes. Google Docs integration was even
worse - it kept trying to "help" by reformatting my RST syntax and couldn't
handle the complexity of cross-referencing multiple technical documents.

### Finding the Right Tool: RooCode

So for this experiment i wanted to provide a more powerful agentic coding
environment where i could group reference content in folders locally for LLMs
to use.

To do this i selected RooCode, an agentic AI coding environment that allows
you to work with AI models in a more interactive way than simple chat
interfaces. Unlike basic AI assistants, RooCode can maintain context across
multiple files, execute code, and work with local repositories. It provides
different "modes" - essentially different system prompts and capabilities - for
different types of work.

The week before i had been playing with Adam Larson's [MicroManager](https://github.com/adamwlarson/RooCodeMicroManager) mode.
That's a topic for a different blog but most of my interaction were done in the
Senior Engineer mode with some usage of roo's default Architect mode. The key
advantage was being able to provide the AI with access to the entire nova-specs
repository, placement documentation, and Greg's reference material
simultaneously.

For this blog you can effectively assume that the delta between RooCodes
default system prompt and my setup was

```
You are my expert programmer named Roo Sr. You are an expert programmer, that
is free to implement functionality across multiple files. You take general
guidelines about what needs to be done, and solve the toughest problems. You
will look at the context around the problem to see the bigger picture of the
problem you are working on, even if this means reading multiple files to
identify the breadth of the problem before coding.
```

I also created a custom prompt enhancer prompt to improve my initial requests:

```
You are Promptly, an AI assistant specializing in refining prompts for LLMs.
If any aspect of the task is unclear, ask clarifying questions before
proceeding.

# Output Format
-  reply with only the enhanced prompt, no conversation, explanations, lead-in,
   bullet points, placeholders, or surrounding quotes
- Keep content concise and easy to read

# Context/Inputs
- You are an LLM assisting in crafting prompts for advanced AI systems
- Enhance clarity and precision of user-submitted prompts
- Balance brevity with accuracy (avoid excessive token usage while maintaining
  precision)

# Constraints/Guidelines
- Correct obvious typos/spelling errors
- Use plain language without technical jargon
- Do not provide solutions to the original prompt
- Do not include instructions for handling ambiguous input or edge cases
-  It's very important that you focus on the question the user has. When using
   tools, always pass required parameters
- Think about your responce carefully, use the sequentialthinking tool to help
  reason about the prompt you are enhancing.

# Priorities
1. **Accuracy**
2. **Efficiency**
3. **Clarity**

Generate an enhanced version of this prompt:
```

Is this a good prompt enhancer prompt? Probably not, but it worked well enough with my semi-local LLMs that I decided to use it for this experiment.

### Model Selection: Local vs. Hosted

This is as good a time as any to talk about models.
Originally I wanted to try and do this using models that I ran locally
on a 16GB M4 Mac Mini with Ollama. I tried a number of different models
to do this but RooCode's system prompt uses too much context to work
with models that fit on this hardware. So while I did use Gemma 3:12B-IT-QAT
as the local model for the prompt enhancer, I selected
Google Gemini Flash 2.5 and DeepSeek-R1:free.

Why deepseek? DeepSeek-R1 is specifically designed for reasoning tasks and
includes a "thinking" process where you can see the model's internal reasoning
before it responds. For complex technical architecture decisions like designing
a Nova scheduler weigher, having a model that can work through the problem
step-by-step seemed valuable. The reasoning traces also help you understand why
the AI made certain design choices.

Why gemini flash? Beyond being very fast and Google's $300 free credits, Gemini
Flash 2.5 is particularly good at following complex instructions and working
with structured documents like RST files. It's also more reliable at tool
calling and file manipulation than many other models, which was crucial for
actually editing the specification document. Originally i wanted to have the
Architecture mode use the long thinking deepseek and programmer modes use gemini
flash to apply the changes to the spec file.

### Data Organization

What about data management? Well, it may be old-fashioned but just use Git.
In this case I created an empty Git repo and added my reference material
as Git submodules.

### The Test Case

So what was I actually trying to build? The core idea was to let users pick CPU
performance tiers - think Gold, Silver, Bronze - where each tier gets guaranteed
proportional CPU time through Linux cgroups and the CFS scheduler. All
integrated with OpenStack Placement so the scheduler actually knows what's
available.

This seemed like a perfect test case for AI assistance because:
- It's complex enough to be interesting but not so complex it would take months
- It touches multiple OpenStack services (Nova, Placement, potentially Watcher)
- You need to understand both OpenStack's scheduler architecture and Linux
  kernel features
- There are clear use cases but the implementation needs careful design work

Basically, it's the kind of thing that sounds simple when you describe it to
someone but turns into a 700-line specification when you actually think through
all the details.

### Pre-AI Design Process: The Human Foundation

Before diving into AI assistance, I want to emphasize that this wasn't a case of asking AI to design something from scratch. The human expertise and preparation phase was crucial.

Normally I gather use case and requirement before sitting down to design
something like this and ruminate on it for a bit before writing anything down.

This time was only slightly different as Greg had done much of that
in writing up their solution plan. As I had a personal engagement taking
my mother to the doctor for a checkup i took the, unusual for me, step
of printing out a dead tree edition of Greg's [document](https://github.com/gprocunier/openstack-cgroup-tiering/blob/main/README.md) and marked it up with a Sharpe as i waited.

The reason i bring this up is to make it clear that before i started
trying to use ai to design a solution, i read the relevant reference material
in multiple media(online and physically), i reflected on the content
assessing how it i would use my nova knowledge to refactor the solution
to be more applicable upstream, and just left the ideas percolate
in my subconscious for about a week to 10 days before involving AI.

That mean that when i did start writing i had a direction and a sense
for how i wanted to use the reference material to express my intent.

## The AI-Assisted Process

**TL;DR**: Evolved from AI's basic initial attempt to a comprehensive specification through seven iterations, using targeted prompts to address specific technical gaps and architectural requirements.

### Getting Started: Specification Structure and the Path to v0

One of the most time-consuming parts of writing a Nova spec is getting the
structure right and ensuring you've covered all the necessary sections.

**Context for Non-OpenStack Readers**: Nova is OpenStack's compute service that manages virtual machines across thousands of physical servers. Nova specs are comprehensive technical design documents (similar to RFCs or ADRs in other projects) that must be written and approved before any significant new feature can be implemented. They typically run 300-800 lines and require deep understanding of distributed systems, virtualization, and OpenStack's complex service interactions.

The 3 most important sections of a Nova spec are as follows

- **Problem Description**: This is the what, expressing what you want to build

- **Use Cases**: This is the why, the motivation for why this problem is
  important

- **Proposed Changes**: This is the How, Breaking down the implementation into
  logical components

**Technical Context**: Nova specs must also address complex concerns like database migrations, API versioning, upgrade compatibility across thousands of servers, integration with other OpenStack services (Placement, Neutron, Cinder), hypervisor-specific implementations, and detailed testing strategies.

Getting this right without deep expertise is not something that anyone
does correctly the first time. This is the foundation of the spec,
so that's where I started.

My initial prompt where I first used AI in this process was:

```
i would like to create a new nova-spec to describe how to support cpu teiring. some refernce matilal is aviable in @/reference/  and a local copy of the nova-specs git repo is cloned at @/nova-specs . the spec template for the current release is @/nova-specs/specs/2025.2-template.rst : i would like to  analsys the refence mataila and using the existing spec as as example create a new spec to describe how to implemente "cpu performance service level using cpu shares"
```

But why write a prompt by hand when AI can improve it?

```
Develop a Nova specification document detailing the implementation of CPU performance service levels using CPU shares. Utilize the existing Nova specification template located at `/nova-specs/specs/2025.2-template.rst` as a guide. Analyze the reference material available in `/reference/` and leverage the template structure and style. The specification should comprehensively describe the design, implementation details, operational considerations, and testing procedures for a system that allows users to request and guarantee CPU performance tiers based on CPU share allocation.  Specifically address how CPU shares will be managed, how performance guarantees are enforced, how users interact with the system (API, UI), and how the system integrates with existing Nova components. Include diagrams and examples to illustrate key concepts and workflows.  The target audience is Nova developers and operators.
```
This shows what RooCode's built-in enhancer can do, but I also wrote my own.
Can it do better?

```
Create a Nova specification document detailing CPU performance service levels using CPU shares, for Nova developers and operators. Use `/nova-specs/specs/2025.2-template.rst` as the template and `/reference/` for style and structure. The specification should comprehensively cover design, implementation, operational considerations, and testing for a system enabling users to request and guarantee CPU performance tiers via CPU share allocation.

**Focus areas:**

*   **CPU Share Management:** Detail allocation, tracking, and adjustment mechanisms. Address conflicts and scaling.
*   **Performance Guarantee Enforcement:** Describe how requested performance levels are consistently met, considering hypervisor limitations, oversubscription, and workload characteristics. Include mitigation strategies for performance degradation.
*   **User Interaction:** Define RESTful API and UI interfaces for requesting, monitoring, and managing CPU share allocations. Provide example API requests and responses.
*   **Nova Integration:** Explain integration with existing Nova components (scheduler, compute, API, etc.). Identify impacts and required modifications.
*   **Oversubscription Handling:** Detail the strategy for managing oversubscription, including share prioritization and performance impact under high load.
*   **Testing Procedures:** Outline unit, integration, and performance tests to validate functionality and performance.
*   **Diagrams and Examples:** Include diagrams illustrating key concepts (e.g., share allocation workflow, performance guarantee enforcement) and practical examples demonstrating API usage and configuration.

Adhere to the template's structure and style, providing a detailed and actionable guide. Consider future extensibility and integration with other performance management features.
```
That's much better.

I should note that there are some aspects of this prompt that will steer the AI in slightly the wrong direction, like "monitoring" and "UI interfaces", but I wanted to have something concrete to start iterating on.

### V0: AI's First Attempt - A Reasonable Start

The AI's initial specification was a reasonable starting point that captured the basic concept and use cases. You can view the complete v0 specification here:

**[📄 View AI's Initial Specification (v0)](/static/specs/cpu-performance-v0.rst)**

Key characteristics of the v0 draft:
- Basic problem statement and use cases
- Proposed direct API parameters for user control  
- Simple cgroups enforcement approach
- Database schema changes (new `cpu_shares` column)
- High-level implementation concepts with placeholders

Was this a good spec? Well no, it didn't properly follow the spec format, but more importantly it missed a lot of the nuance that makes a Nova spec actually implementable.

It did capture the problem statement and use-cases reasonably well, but it made
some fundamental mistakes. For example, it suggested adding new API parameters
that users could set directly - which sounds reasonable until you realize that
Nova doesn't work that way. Users interact with Nova through flavors and images,
not by setting arbitrary parameters on server creation. It also proposed a new
database column without considering upgrade impacts, and didn't understand how
Nova's scheduler actually works with Placement.

A good Nova spec needs to show understanding of existing patterns, explain how
the new feature integrates with current workflows, and demonstrate that the
author understands the operational complexity of deploying changes across
thousands of compute nodes. The v0 draft read like someone who understood the
general concept but had never actually worked with Nova's codebase.

But was any of that a show stopper? No. It was actually a pretty good starting point to build from - the core technical concept was sound, and having something concrete to iterate on is better than staring at a blank page.

### The Transformation Process: From v0 to v7

Before diving into the final result, let me explain the refinement process that transformed AI's basic first attempt into a comprehensive, production-ready specification.

#### Setting Up the Collaboration

First an overview of how i used Senior engineer mode.

One thing about AI is they are good at role-play, so first I set up a new chat with the following prompt, clearing all previous context.

```
As Nova project maintainer, i want to revise this design proposal to align with Nova's goals and architecture.
Provide insights on its technical accuracy, clarity, and completeness, to ensure the proposed change, fulfils the use cases
and addresses the problem descriptions. I will iteratively provide modifications; collaborate to apply them.
I will provide additional documentation describing Nova and the proposal as context.
The design document is in reStructuredText format and we wil produce a new revision that will be stored in a new file.
initally do not take any action until i ask you do, just acknowledge this prompt and wait for my request.
```

This at least I hoped would put the AI in the right "mindset" to help me correct the issues with the v0 draft.

Before I go over the main prompt I used to revise the spec, I'll also note that I did manually edit the spec in between prompts to address issues that were quicker to fix by hand and also to write some additional prose.

To make revisions to the spec, I first created a prompt which I iterated on once or twice with the prompt enhancer. I saved the major prompts to a file for later reference, but I also wrote smaller prompts to make minor tweaks between each revision. For example, when I manually added content I ran:
```
Review the document at @/nova-specs/specs/cpu-performance-service-level.rst for technical accuracy, spelling, grammar, and consistency between the problem statement, use cases, and proposed solution.
```
The final output is as follows, but I don't expect you to read it in detail. I've included it to show how the spec evolved from AI's first attempt to what I considered acceptable for a first draft to submit for review.

### V7: The Final Implementation-Ready Specification

After seven iterations of refinement, the specification evolved into a comprehensive, production-ready design. You can view the complete final specification here:

**[📄 View Final Specification (v7)](/static/specs/cpu-performance-v7.rst)**

Key improvements in the v7 specification:
- **Comprehensive technical definitions** and algorithmic descriptions
- **Placement service integration** with standardized `VCPU_SHARES` resource class
- **Detailed ResourceProviderWeigher** with complete pseudocode implementation
- **Flavor-driven configuration** (no API changes required)
- **Production deployment considerations** and upgrade compatibility
- **Mathematical formulations** with practical examples

### Summary of v0 -> v7 Evolution

**TL;DR**: Transformed from a basic API-centric approach with database changes to a sophisticated Placement-integrated solution using flavors, standardized resource classes, and a custom scheduler weigher.

For those who don't want to read the full spec, here is an AI-generated summary of what changed.

The specification evolved significantly through seven iterations,
transforming from a basic concept to a comprehensive technical design.

#### Title and Scope Changes

**v0:** "CPU Performance Service Levels using CPU Shares"
**v7:** "CPU Performance Service Levels using CPU Shares and Placement Resource Classes"

The scope expanded to include full OpenStack Placement service integration.

#### Major Technical Changes

**Resource Management Approach:**
- **v0**: Direct API parameters (`cpu_shares`) with basic cgroups enforcement
- **v7**: Flavor-driven approach using `quota:cpu_shares_multiplier` with `VCPU_SHARES` resource class and Placement integration

**Scheduler Integration:**
- **v0**: Vague mention of "a new scheduler filter or weigher might be required"
- **v7**: Detailed `ResourceProviderWeigher` specification with complete algorithm, pseudocode, and "most boring host" selection strategy

**User Interface:**
- **v0**: New API parameters for direct user control
- **v7**: No API changes - flavor-based configuration with administrative controls

#### New Sections Added

The final version includes several sections absent from v0:
- **Definitions**: Comprehensive technical glossary
- **ResourceProviderWeigher**: Complete algorithm specification with pseudocode
- **Configuration Examples**: Practical deployment guidance
- **Detailed Diagrams**: ASCII art showing resource contention scenarios
- **Comprehensive Alternatives**: Analysis of rejected approaches with reasoning
- **Implementation Details**: Specific work items and dependencies

#### Technical Depth Improvements

**v0 Specification:**
- Basic problem statement and use cases
- High-level implementation concepts
- Placeholders for key sections
- Simple API-centric approach

**v7 Specification:**
- Detailed technical definitions and algorithms
- Complete configuration management strategy
- Comprehensive upgrade compatibility mechanisms
- Production-ready implementation plan
- Mathematical formulations and example calculations

#### Architectural Sophistication

The final version demonstrates significantly more architectural sophistication:
- Multi-layered enforcement (Placement + CFS scheduler)
- Integration with existing OpenStack patterns (flavor validation framework)
- Configuration-driven feature activation for upgrade compatibility
- Resource provider tree processing algorithms
- Sophisticated weighing mechanisms for intelligent host selection

The evolution represents a transformation from conceptual proposal to implementation-ready specification
with comprehensive technical detail and production deployment considerations.

Sean: ^ While not wrong, AIs can be a bit of a suck-up.

#### The Iterative Refinement Process

Right, so back to the actual work - what prompts did I use to get from the AI's first rough attempt to something that didn't make me cringe when I read it?

I started with something relatively simple, because if there's one thing I've learned about working with AI, it's that you don't dump your entire design vision on it at once.

```
the first change i woudl like to make is summerise the current proposal as a
short paragaph in the Alternatives
seation with a title of "cpu service level via custom resources classes"
after that we will work togehter to rework the spec to use a single
standard resouces class called "VCPU_SHARES"
```
In other words, I started by summarizing the initial spec as an alternative and then made the first big change: using a single resource class instead of one per tier.

This was one of the core simplifications needed to make this feature easy to use.

Next, I got a little more ambitious and the next improvement I identified was to add automatic reporting instead of using provider.yaml files.
```
Summarize the currnt proposal to report available inventories in the
Alternatives section using the `provider.yaml` file, titled "Reporting
VCPU_SHARE via provider.yaml."
Then update the proposal to state that the libvirt driver will automatically
report a `VCPU_SHARES` inventory on the root resource provider for each
hypervisor.
The allowed range for CPU shares in the cgroups v2 API is [1, 10000]. A new
`vcpu_share_multiplier` configuration option will be added, with a minimum value
of 1 and a maximum value of 10000, the default will be 1000.
Reporting of `VCPU_SHARES` will be controlled by a `report_vcpu_shares` boolean
option in the libvirt section of the `nova.conf`.
 When enabled, the libvirt driver will report a quantity of
 `vcpu_share_multiplier * count(cpu_share_set)` as `VCPU_SHARES`.
 ```

So at this point, the spec described the problem and use cases, introduced a new standard resource class, and described updating the libvirt driver to report that automatically.

#### Adding Scheduler Intelligence

Next step: scheduling.

```
Document the design and functionality of the `ResourceProviderWeigher` in
@/nova-specs/specs/cpu-performance-service-level.rst. The
`ResourceProviderWeigher` calculates a weight by comparing instance resource and
trait requests to placement provider summaries.

Enhance the `HostState` object in @/nova/nova/scheduler/host_manager.py with the
provider summary dictionary. This dictionary is obtained from the placement API
(see @/placement/api-ref/source/allocation_candidates.inc and
@/placement/api-ref/source/samples/allocation_candidates/ for examples) when
Nova requests allocation candidates.

The weighting algorithm should adhere to these guidelines:

1. **Input:** The algorithm takes the `provider_summaries` data structure, which
represents a forest of trees encoded as a map. Node relationships are defined by
`parent_provider_uuid` (with the root having `null`) and `root_provider_uuid`.
Resource providers are modeled as a mapping of UUIDs to resource class
inventories (capacity and usage).

2. **Resource Aggregation:** Transform the tree into a map of resource class to
free capacity (capacity - used), aggregating capacity and usage for identical
resource classes across multiple providers within the tree.

3. **Trait Aggregation:** Create a flattened set of all traits reported by
resource providers in a given tree, indexed by the root provider UUID.

4. **Weight Calculation:** Based on requested resources and traits, produce a
weight between 0 and 1.

5. **Weighting Formula:** For each available resource, calculate the arithmetic
mean of `(1 - (requested amount of resource class) / (free capacity of resource
class))` across all resource classes in the provider tree. Add to this the count
of requested traits divided by available traits. Normalize the result by
dividing by 2.

Reference existing Nova weighers (e.g., @/nova/nova/scheduler/weights/) for
implementation guidance, especially the `MetricsWeigher` and
`ImagePropertiesWeigher` for configuration options.

The `ResourceProviderWeigher` should support the following configurable options:

*   `ResourceProviderWeigher` multiplier
*   List of resource classes to use for weighing (default: all)
*   List of traits to include when weighing (default: all)
```

This prompt represented a significant step up in complexity. The main elements of note are that i provided it the paths to documentation for the placement allocation_candidates api and the api samples, named the relevant Classes in the nova scheduler `HostState`, and provided the host_manager.py as context.

With that context i described how i wanted the `ResourceProviderWeigher` to work.

This got pretty close but needed several refinements to actually express my intent. This and the prior prompts were the largest changes to the spec, and it was here that I took the time to make most of my manual changes.

#### Final Polish and Technical Context

Form this point my prompts were more polishing rather then large changes.

```
Review @/nova-specs/specs/cpu-performance-service-level.rst for consistency in
style, information presentation, and section content, using
@/nova-specs/specs/train/implemented/cpu-resources.rst as a detailed example.
```
The CPU resources spec is one of our most complex specs and as a result one of the better written.

```
can you add a short introduction to
@/nova-specs/specs/cpu-performance-service-level.rst describing how the linux
CFS scheduler works in the context fo cpu_shares and how vms VCPU kvm threads
are schdule to host cores and how time is allcoated to each process.
```

## Results and Evaluation

**TL;DR**: AI didn't make the process faster, but significantly improved quality by helping structure complex technical concepts and eliminating spelling/grammar issues before human review.

### The Punch Line
Did Ai make faster or easier to write the spec... No not with this setup
but it did result in a higher quality spec with most if not all of the spelling issues address before humans other then my self had to read them.

The time spent setting up and debugging this setup especially the time spent
trying to use local models with the limited ram availability negated any speed up that could otherwise have been achieved.

If or when I do this again, I think I will change the agentic agent choice to either use Aider or Claude Code. RooCode could also be very effective if I choose a more powerful LLM that better supports RooCode's complex system prompt and tool calling conventions.

### The Final Result

**TL;DR**: Produced a 700-line, production-ready Nova specification covering problem definition, technical architecture, implementation plan, testing strategy, and alternatives analysis.

After about a week of iterative work, spread out as an hour or two in the
evening we produced a comprehensive specification that includes:

- **Complete problem definition** with concrete use cases
- **Detailed technical architecture** using Placement resource classes
- **Implementation plan** broken down by component (scheduler, libvirt driver, API)
- **Configuration options** with sensible defaults
- **Testing strategy** including functional and integration tests
- **Documentation requirements** for operators and users
- **Alternative approaches** and why they were rejected

The specification ended up being about 700 lines and a little over
3500 words and covers all the sections you'd expect in a Nova spec.

## What Worked Well

### AI Strengths in Technical Documentation

#### Design Iteration and Refinement

Iteration works well when working with LLMs and AI agents.
With enough context and prompting you can get it to do some impressive refactoring but it still need your guidance. If you provide it with
high quality examples it can provide high quality output.

#### Technical Writing and Clarity

The AI was great at helping structure complex technical concepts in a way that would be accessible to reviewers who might not be deeply familiar with cgroups or the placement service internals.

An example of this is the ascii diagram of the vm vcpu to host cpus mappings.

In the past I have drawn them using https://asciiflow.com/#/
which is great but I think Gemini did a great job.
Initially I used DeepSeek but Gemini I think captured it better.

## What Was Challenging

### AI Limitations in Complex Domains

#### Domain Expertise Limitations

Here's the thing - while the AI knows a lot about OpenStack in general,
it doesn't have the kind of deep institutional knowledge you get from years
of actually working on Nova. I found myself constantly having to correct
assumptions about how features actually work, or provide context about why
we made certain design decisions years ago.

For example, the AI initially suggested approaches that would have
completely broken CPU pinning, or didn't account for the fact that the
libvirt driver has some... let's call them "quirks" in how it handles
certain configurations.

#### Community and Process Knowledge

The AI doesn't understand the social aspects of the OpenStack development
process - what kinds of changes are likely to be controversial, which core
reviewers have strong opinions about certain areas, or how to phrase things
to avoid bikeshedding in reviews.

It also doesn't know about informal agreements or "soft" rules that aren't
documented anywhere but are understood by the community. You know, the kind
of things you learn from being in IRC discussions at 2am when someone's
trying to debug a weird corner case.

#### Understanding Upgrade Complexity

Upgrade impact is very important to nova.
we avoid designs that require db changes or api changes when they are not
strictly required. That does not mean we do not add new apis or db tables
but we treat such changes as a high bar.

In Nova specifically, database schema changes require all the nova
controller services (API, scheduler, conductor) that have access to the
database to be upgraded simultaneously. This makes all the compute agents
and the workloads that reside on them unmanageable for the duration of the
controller upgrades. API changes have backward compatibility implications
that can affect users for years, and must be supported across multiple
OpenStack releases.

The original approach the AI suggested had a new db column to store the
cpu_shares and allowed end users to request them directly. While neither of
these are technically required to achieve the goal, the AI lacked the
context to know that those are expensive things to do in something like
nova, whereas db changes and api changes may be relatively cheap in a
typical web application.

### Technical Tool Limitations

**TL;DR**: RestructuredText's use of dashes conflicted with git diff syntax, causing AI tool confusion and edit failures when making cross-section changes.

#### Restructured Text Format Confusion

Here's something that caught me completely off guard, though it's obvious in hindsight.

RestructuredText uses `-------` everywhere - literally under every heading. You know what else uses `-------`? Git diffs.

So when I was working within a single section, everything was fine. But as
soon as I tried to make changes across larger parts of the spec, the AI
would get confused by all the dashes and start treating section dividers as
diff markers. Tool call failures everywhere, and poor Roo couldn't figure
out what I actually wanted it to change.

This is partly down to the RST format but more a limitation of both Roo
and the models I selected: `google-gemini-flash-2.5` and `deepseek/deepseek-r1:free`.
Gemini is better at applying diffs and tool calls than deepseek but
neither are really trained for this. If I used a model that was specifically trained for this
it would have gone more smoothly.

Aider and Claude Code, or other tools like Zed, Windsurf, or Cursor may do better.

## Lessons Learned

### The Collaborative Model Works Best

#### AI as a Collaborative Partner

The most effective approach wasn't having the AI write the spec for me, but rather using it as a collaborative partner.

The AI was excellent at:
- Helping structure and organize complex information
- generating summaries and diagrams
- comparing refernce information to determine how to improve the existing
  proposal to incorporate that context.

#### Human Expertise Remains Essential

My years of experience with Nova and OpenStack were essential for:
- Knowing what's actually implementable vs. what sounds good on paper
- Understanding the community and review process
- Catching subtle interactions with existing features
- Making pragmatic trade-offs between ideal solutions and mergeable code

### Process Insights

#### Iteration and Refinement

The best results came from multiple rounds of iteration. The first draft
was pretty rough, but each iteration I ask for specific problems to be
addressed and incrementally made the specification significantly better.

## Would I Do This Again?

### Evaluating Success Against Goals

This is an interesting question.

As I alluded to before, if my goal was to get things done faster then I don't think that was a success. Fortunately that was not my goal - my goal was to provide a higher quality initial version of a spec for upstream review.

I wanted to try and mitigate some of the mental load of written communications and mask the impacts of my dyslexia. That goal was definitely achieved.

Was it easy? No, this still took a lot of effort to drive. Would I do it again with AI? Yes, on balance I think I would.

## Next Steps and Future Applications

### Community Review

The specification is now ready for community [review](https://review.opendev.org/c/openstack/nova-specs/+/951222).
I am curious to see how the community responds to an AI-assisted
specification. Will reviewers be able to tell? Will the quality be
noticeably different?

### Expanding the Approach

I'm also planning to use this approach for other specifications I've been
putting off due to time constraints. If the community review goes well, I
might write up some guidelines for others who want to experiment with
AI-assisted spec writing once i gain more experience with it.

For now I have created https://github.com/SeanMooney/openstack-ai-style-guide
to help with future AI usage in OpenStack.


## Implications for the OpenStack Community

**TL;DR**: AI assistance could democratize technical contribution, accelerate complex projects like Eventlet removal, and help preserve institutional knowledge while raising questions about review processes.

This experiment raises some interesting questions for our community:

### Accelerating Innovation

If AI can help experienced contributors produce higher quality specifications faster, can it help use implement them.

Eventlet removal is a very complex undertaking.
open api schema definition are another community goal with a lot
of detailed work. Could AI help achieve those goals faster?

### Democratizing Contribution

Could AI assistance help newer contributors write better specifications by helping them understand OpenStack architecture patterns and community conventions?

What about those with learning disabilities like dyslexia?
visual disabilities via enhanced TTS and voice interaction?

### Review Process Evolution

How should reviewers approach AI-assisted specifications?
What new review criteria might we need to consider?

### Knowledge Preservation

One unexpected benefit was that working with the AI to explain OpenStack
concepts helped me articulate institutional knowledge that might not be
well documented elsewhere. Could AI assistance be a tool for better
knowledge transfer?

## Final Thoughts

This experiment convinced me that AI can be a powerful tool for OpenStack
development when used thoughtfully. It's not a replacement for human
expertise, but it can be guided by it - helping experienced contributors
work more efficiently and potentially helping newer contributors learn
faster.

What started as a personal accessibility challenge became something bigger:
a proof of concept that AI-assisted technical writing can produce genuinely
useful results. The specification that emerged from this process isn't just
"good enough" - it's comprehensive, technically sound, and ready for
serious community review.

But perhaps more importantly, this experiment points toward a future where
the barriers to contributing high-quality technical documentation are
significantly lower. If AI can help someone with severe dyslexia produce a
700-line Nova specification, imagine what it could do for brilliant
engineers who struggle with English as a second language, or newcomers who
understand the technology but don't yet know the community conventions.

The success here wasn't in replacing human judgment - it was in augmenting
human expertise. Every architectural decision, every technical trade-off,
every understanding of community dynamics came from years of OpenStack
experience. The AI simply helped translate that knowledge into clear,
well-structured prose.

I'm excited to see how this specification is received by the community and
whether others will experiment with similar approaches. More importantly,
I'm curious to see if this opens doors for contributions from voices we
might not have heard otherwise.

The real test isn't whether the AI wrote a good spec - it's whether this
approach can help more people contribute to the technical discussions that
shape OpenStack's future.

---
*The specification is now under [community review](https://review.opendev.org/c/openstack/nova-specs/+/951222). I'm interested in feedback on both the specification itself and the AI-assisted development process. Feel free to reach out on IRC (sean-k-mooney) or through the usual OpenStack channels.*

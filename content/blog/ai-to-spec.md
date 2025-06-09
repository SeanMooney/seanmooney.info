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

### Personal note
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

This isn't just about me though - there are likely other contributors in the
OpenStack community and beyond who face similar challenges. Learning
disabilities, non-native English speakers, contributors who are brilliant
engineers but struggle with written communication - AI assistance could be a
game changer for inclusive participation in open source communities. If this
experiment proves successful, it might open doors for more diverse voices to
contribute to technical discussions that have traditionally required strong
written English skills.

### Why now
Well why now is a good question but with a pretty simple answer:
* Like many companies, RedHat would like us all to explore ai and what we can
  use it for day to day.
* until recently ai tools were auto complete on steroids and kind of terrible
  at this but in the last 3-6 months with release of LLMs like deepseek-R1 andc
  google-gemini-flash-2.5, coupled with tools like RooCode the landscape has
  changed a lot.
* I was asked to review a technical solution proposal drafted by one of our
  very talented Solution Architects Greg Procunier. What better time than when
  you are provided a problem statement, use cases and even a potential approach
  to apply my knowledge of nova to how I would address the problem? After all,
  the best way to learn how to use a tool is to try and use it to address a
  real world problem.

### The what
As is disappointingly common with nova the concept that Greg outlined in
[openstack-cgroup-tiering](https://github.com/gprocunier/openstack-cgroup-tiering)
is something that we have discussed tangentially many times over the years. In
fact in his technical plan Greg explains how a combination of old and very old
openstack features can be combined to achieve the usecase without any code
change.

So why a new spec? simple it should be possible for mere mortals to achieve
the use case without over a decade of contributing to nova or cloud solution
architecture experience to define cpu performance level in OpenStack!

## The Experiment Setup

There is a standard in writing often used in U.S. military communication
known as BLUF (Bottom line up front). Well I'm Irish so Better late than
never, the intent of this blog is not to discuss The details of how to modify
nova to support "CPU Performance Service Levels using CPU Shares and Placement
Resource Classes". Instead the goal is to discuss the experience and process
of using ai to write the spec. By necessity some of the details of the spec
will also be addressed here but as a framework for the conversation rather
than as the focus.

### Tool Selection and Configuration

This is actually my 3rd attempt as initially i tried to use Gemini pro and
the web chat interface to do this followed by using google docs gemini
integration and both failed spectacularly.

The web chat interface couldn't maintain context across the multiple reference
documents I needed - it would forget about the nova-specs template when I was
asking about Greg's technical plan, or lose track of OpenStack Placement
concepts when discussing scheduler changes. Google Docs integration was even
worse - it kept trying to "help" by reformatting my RST syntax and couldn't
handle the complexity of cross-referencing multiple technical documents.

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

i also created a custom prompt enhancer prompt

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

Is this a good prompt enhancer prompt? probably not but it one that work well
enough with my semi local llms that i decide to use it for this experiment.

This is as good a time as any to talk about models.
originally i wanted to try and do this using models that i ran locally
on a 16G m4 mac mini with ollama. i tried a number of different models
to do this but RooCode system prompt uses too much context to work
with models that fit on this hardware. So while i did use gemma3:12b-it-qat
as the local model for the prompt enhancer, i selected
google-gemini-flash-2.5 and deepseek/deepseek-r1:free.

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

What about data management, well it may be old fashioned but just use git.
in this case i created an empty git repo and added my reference material
as git sub modules.

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

### Pre-AI Design Process

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

### Specification Structure and the path to v0

One of the most time-consuming parts of writing a Nova spec is getting the
structure right and ensuring you've covered all the necessary sections.

For those not familiar with OpenStack, Nova is the compute service that manages
virtual machines, and Nova specs are technical design documents that must be
written and approved before any significant new feature can be implemented.
They're similar to RFCs or design docs in other projects.

The 3 most important sections of a Nova spec are as follows

- **Problem Description**: This is the what, expressing what you want to build

- **Use Cases**: This is the why, the motivation for why this problem is
  important

- **Proposed Changes**: This is the How, Breaking down the implementation into
  logical components

getting this right without deep expertise is not something that anyone
does correctly first time. This is the foundation of the spec
so thats where i started.

my initial prompt and where i first used ai in this process was.

```
i would like to create a new nova-spec to describe how to support cpu teiring. some refernce matilal is aviable in @/reference/  and a local copy of the nova-specs git repo is cloned at @/nova-specs . the spec template for the current release is @/nova-specs/specs/2025.2-template.rst : i would like to  analsys the refence mataila and using the existing spec as as example create a new spec to describe how to implemente "cpu performance service level using cpu shares"
```

but why am i writing a prompt by hand when ai can do it better.

```
Develop a Nova specification document detailing the implementation of CPU performance service levels using CPU shares. Utilize the existing Nova specification template located at `/nova-specs/specs/2025.2-template.rst` as a guide. Analyze the reference material available in `/reference/` and leverage the template structure and style. The specification should comprehensively describe the design, implementation details, operational considerations, and testing procedures for a system that allows users to request and guarantee CPU performance tiers based on CPU share allocation.  Specifically address how CPU shares will be managed, how performance guarantees are enforced, how users interact with the system (API, UI), and how the system integrates with existing Nova components. Include diagrams and examples to illustrate key concepts and workflows.  The target audience is Nova developers and operators.
```
This shows what RooCode's inbuilt enhancer can do but I also wrote my own
can it do better?

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
Thats much better.

i will note that there are some aspects of this prompt that will steer the ai in
slightly the wrong direction like "monitoring" and "UI interfnaces"
but i wanted to have a some concreate to start iterating on.

### V0 AIs first attempt.

```rst
===============================================================================
CPU Performance Service Levels using CPU Shares
===============================================================================

Problem Description
-------------------

Users currently lack a standardized and reliable method within Nova to request and guarantee specific levels of CPU performance for their instances. While existing mechanisms like CPU pinning or NUMA topology awareness provide some control, they are often complex to manage and do not offer a simple, quantifiable way to express desired performance tiers across a shared infrastructure. This leads to unpredictable performance, especially in oversubscribed environments, making it difficult for users to run performance-sensitive workloads with confidence.

The goal is to introduce a mechanism that allows users to request CPU performance service levels based on CPU shares, providing a more predictable and manageable approach to CPU resource allocation and performance guarantees.

Use Cases
~~~~~~~~~

*   A user wants to launch a database server instance and ensure it receives a guaranteed minimum level of CPU performance, even under high load from other instances on the same host.
*   A user is running a batch processing job that requires a consistent amount of CPU time to complete within a specific timeframe. They need to request a performance tier that guarantees this.
*   A cloud administrator wants to offer different tiers of compute service (e.g., "Bronze", "Silver", "Gold") based on guaranteed CPU performance, allowing users to choose the level that best suits their workload and budget.
*   A user needs to temporarily boost the CPU performance of an existing instance to handle a peak load, and then revert to the standard performance level.

Proposed Change
---------------

Introduce a new mechanism in Nova that leverages CPU shares (using control groups - cgroups) to define and enforce CPU performance service levels. Users will be able to specify a desired CPU share value when launching or resizing an instance, which will translate to a relative proportion of available CPU resources on the host.

This approach provides a flexible and granular way to manage CPU performance, allowing for oversubscription while still providing a mechanism for performance guarantees based on the allocated shares.

CPU Share Management
~~~~~~~~~~~~~~~~~~~~

*   **Allocation:** CPU shares will be allocated to instances based on a new flavor extra spec or image metadata. The value will represent a relative weight, with higher values indicating a greater proportion of CPU resources. A default share value will be applied if none is specified.
*   **Tracking:** Nova will track the allocated CPU shares for each instance and the total shares allocated on a compute host. This information will be used by the scheduler and compute drivers.
*   **Adjustment:** Users will be able to adjust the CPU share allocation for a running instance via a resize operation.
*   **Conflicts and Scaling:** The system will need to handle potential conflicts when the total requested shares exceed the available CPU resources on a host. This will involve prioritization based on share values and potentially admission control. Scaling will be handled by the underlying cgroup mechanism.

Performance Guarantee Enforcement
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

*   **Mechanism:** CPU shares, managed via cgroups, provide a mechanism for proportional sharing of CPU time. While not a strict reservation, higher share values translate to a greater opportunity to consume CPU cycles when the system is contended.
*   **Hypervisor Limitations:** The implementation will need to consider how different hypervisors (e.g., KVM, VMware) expose and manage CPU shares or similar concepts. Abstraction layers may be required.
*   **Oversubscription:** In oversubscribed environments, the performance guarantee is relative to the total shares allocated. Instances with higher shares will receive a larger portion of the available CPU time.
*   **Workload Characteristics:** The actual performance experienced by an instance will still depend on its workload characteristics. The system guarantees a proportion of CPU time, not a specific throughput or latency.
*   **Mitigation Strategies:** Monitoring CPU utilization and performance metrics will be crucial. If performance degradation is detected for instances with guaranteed shares, mitigation strategies could include workload migration, reducing oversubscription, or alerting administrators.

User Interaction
~~~~~~~~~~~~~~~~

*   **RESTful API:**
    *   Extend the server creation and resize APIs to accept a `cpu_shares` parameter (or similar) in the request body.
    *   Add a new API endpoint to view the current CPU share allocation for an instance.
    *   Potentially add an API endpoint to view the total allocated CPU shares on a compute host (for administrators).

    Example API Request (Create Server):

    .. code-block:: json

        {
            "server": {
                "name": "my-performance-instance",
                "imageRef": "...",
                "flavorRef": "...",
                "networks": [...],
                "cpu_shares": 2048
            }
        }

    Example API Response (Show Server Details):

    .. code-block:: json

        {
            "server": {
                ...,
                "cpu_shares": 2048,
                ...
            }
        }

*   **UI Interfaces:**
    *   Update the Horizon dashboard to include an input field for CPU shares during instance launch and resize.
    *   Display the allocated CPU shares on the instance details page.

Nova Integration
~~~~~~~~~~~~~~~~

*   **Scheduler:** The scheduler will need to be aware of the requested `cpu_shares` and the available CPU resources and allocated shares on compute hosts to make informed placement decisions. A new scheduler filter or weigh
    er might be required.
*   **Compute:** The compute driver will be responsible for configuring the hypervisor (e.g., libvirt) to set the appropriate CPU shares for the instance's cgroup.
*   **API:** The API layer will handle the user requests for specifying and viewing CPU shares, performing validation, and interacting with the scheduler and compute components.
*   **Database:** The Nova database schema will need to be updated to store the `cpu_shares` value for each instance.
*   **Impacts and Required Modifications:** This feature will require modifications to the Nova API, scheduler, compute drivers, and database schema. Careful consideration of backward compatibility and rolling upgrades will be necessary.

Oversubscription Handling
~~~~~~~~~~~~~~~~~~~~~~~~~

*   **Strategy:** The primary strategy for oversubscription handling will rely on the proportional sharing provided by CPU shares. Instances with higher share values will receive a proportionally larger amount of CPU time when the system is oversubscribed.
*   **Share Prioritization:** The cgroup mechanism inherently prioritizes instances based on their share values.
*   **Performance Impact under High Load:** Under extreme oversubscription, even instances with high share values may experience performance degradation. Monitoring and alerting will be crucial to identify such situations. Administrators may need to adjust oversubscription ratios or migrate workloads.

Testing Procedures
------------------

*   **Unit Tests:** Unit tests will be required for new or modified code in the API, scheduler, compute drivers, and database layer.
*   **Integration Tests:** Integration tests will verify the interaction between different Nova components (API, scheduler, compute) and the hypervisor when creating, resizing, and deleting instances with specified CPU shares.
*   **Performance Tests:** Performance tests will be essential to validate that the allocated CPU shares translate to the expected relative performance under various load conditions and oversubscription levels. These tests should measure metrics like CPU utilization, throughput, and latency for instances with different share values. Tempest tests should be developed to cover the user-facing API interactions and basic functionality.

Diagrams and Examples
----------------------

*   **Share Allocation Workflow Diagram:** An ASCII diagram illustrating the flow of a request to create an instance with specified CPU shares, showing how the request is handled by the API, scheduler, and compute components, and how the CPU shares are configured on the compute host via cgroups.
*   **Performance Guarantee Enforcement Diagram:** An ASCII diagram illustrating a scenario with multiple instances on a single host with different CPU share allocations, showing how CPU time is distributed among them under contention.
*   **API Usage Examples:** Include detailed examples of the RESTful API requests and responses for creating, showing details of, and resizing instances with CPU shares.
*   **Configuration Examples:** Provide examples of how to configure flavors or image metadata to include default or allowed CPU share values.

Alternatives
------------

(Placeholder for analysis of alternative approaches, e.g., strict CPU reservation, percentage-based allocation, etc.)

Impacts
-------

*   **Data Model Impact:** Add a `cpu_shares` column to the instances table.
*   **REST API Impact:** Modify the create server and resize server API requests and responses. Add new API endpoints for viewing CPU share information. This requires the `APIImpact` flag.
*   **Security Impact:** (Placeholder for security considerations, e.g., potential for denial-of-service if not properly validated).
*   **Notifications Impact:** Versioned notifications may be required for changes to instance CPU share allocations.
*   **Performance Impact:** The overhead of cgroup management should be minimal. The primary performance impact will be on instance performance based on their allocated shares and the overall host load.

Upgrade Considerations
----------------------

Careful consideration is needed for rolling upgrades to ensure compatibility with existing instances and APIs. The new `cpu_shares` field should be nullable in the database initially, and default values applied for instances created before the feature is fully deployed.

Implementation Plan
-------------------

*   Phase 1: Database schema changes and API modifications to accept and store `cpu_shares`.
*   Phase 2: Scheduler integration to consider `cpu_shares` during instance placement.
*   Phase 3: Compute driver implementation to configure cgroups with the specified CPU shares.
*   Phase 4: Add API endpoints for viewing CPU share information.
*   Phase 5: Implement Horizon UI changes.
*   Phase 6: Develop comprehensive unit, integration, and performance tests.
*   Phase 7: Update documentation.

Testing Requirements
--------------------

See "Testing Procedures" section above. Tempest tests are required for API coverage.

Documentation Impact
--------------------

Update the Nova API reference, the Nova administrator guide (for configuration), and the Nova user guide (for requesting CPU shares).
```

is this a good spec?
Well no, it didn't properly follow the spec format, but more importantly it
missed a lot of the nuance that makes a Nova spec actually implementable.

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

But is any of that a show stopper? No. It was actually a pretty good starting
point to build from - the core technical concept was sound, and having something
concrete to iterate on is better than staring at a blank page.

### preview of V7

First an overview of how i used Senior engineer mode.

one thing about ai is they are good a role play so first i setup a new chat
with the following prompt clearing all previous context.

```
As Nova project maintainer, i want to revise this design proposal to align with Nova's goals and architecture.
Provide insights on its technical accuracy, clarity, and completeness, to ensure the proposed change, fulfils the use cases
and addresses the problem descriptions. I will iteratively provide modifications; collaborate to apply them.
I will provide additional documentation describing Nova and the proposal as context.
The design document is in reStructuredText format and we wil produce a new revision that will be stored in a new file.
initally do not take any action until i ask you do, just acknowledge this prompt and wait for my request.
```

This at least i hope would put the ai in the right "mindset" to help me correct
the issue with v0 draft

Before I go over the main prompt i used to revise the spec ill also note
that i did also manually edit the spec in between prompts to
address issues that were quicker to fix by hand and also to write some
additional pros.

To make revisions to the spec i first created a prompt which i iterated on once
or twice with the prompt enhancer. i saved the major prompts to a file for later
but i also wrote some min prompt to tweak the change between each revisions
minorly. for example when i manually added content i ran.
```
Review the document at @/nova-specs/specs/cpu-performance-service-level.rst for technical accuracy, spelling, grammar, and consistency between the problem statement, use cases, and proposed solution.
```
the final out put is as follow but i do not expect you to read it in detail.
i have included to show how the spec evolved form AIs first attempt to what
i considered accpable for a first draft to submit for review.

v7
```RST
==============================================================================
CPU Performance Service Levels using CPU Shares and Placement Resource Classes
==============================================================================

This specification proposes a mechanism within Nova to allow users to
request and receive predictable CPU performance service levels for their
instances. By leveraging CPU shares managed via control groups (cgroups)
and integrating with OpenStack Placement using standardized resource
classes, we aim to provide a standardized and quantifiable way for users
to express desired performance tiers and for cloud administrators to
enforce performance guarantees, particularly in oversubscribed
environments. This addresses the current lack of a reliable method
for users to ensure consistent CPU performance for their workloads.

Problem description
===================

Users currently lack a standardized and reliable method within Nova to
request and guarantee specific levels of CPU performance for their
instances. While existing mechanisms like CPU pinning or NUMA topology
awareness provide some control, they are often complex to manage and do not
offer a simple,
quantifiable way to express desired performance tiers across a shared
infrastructure, leading to unpredictable performance, especially in
oversubscribed environments, making it difficult for users to run
performance-sensitive workloads with confidence.

The goal is to introduce a mechanism that allows users to request CPU
performance service levels based on CPU shares, providing a more
predictable and manageable approach to CPU resource allocation and
performance guarantees.

Definitions
-----------

CPU shares
    A mechanism within the Linux kernel's control groups (cgroups) to provide
    a proportional share of CPU time to processes or groups of processes
    when the CPU is contended. Higher share values result in a greater
    proportion of CPU time.

cgroups (Control Groups)
    A Linux kernel feature that limits, accounts for, and isolates the
    resource usage (CPU, memory, disk I/O, network, etc.) of a collection
    of processes.

OpenStack Placement
    An OpenStack service that tracks resource provider inventories and
    allocations, enabling the scheduler to make informed decisions about
    where instances can be hosted.

Resource Class
    A named category of resources managed by the Placement service (e.g.,
    VCPU, MEMORY_MB, DISK_GB). Custom resource classes can be defined for
    specific needs.

Flavor Extra Spec
    Key-value pairs associated with a Nova flavor that provide additional
    configuration options or constraints for instances created from that
    flavor.

VCPU_SHARES
    A proposed standard resource class in OpenStack Placement representing
    a unit of CPU performance based on CPU shares.

ResourceProviderWeigher
    A Nova scheduler component that calculates a weight for each potential
    host based on its resource availability and other factors, influencing
    host selection during scheduling.

Over subscription
    The practice of allocating more virtual resources (like VCPUs) than the
    physical resources available on a host, relying on the assumption that
    not all allocated resources will be used simultaneously.

CFS Scheduler
    The Linux Completely Fair Scheduler (CFS)
    In virtualized environments using KVM, each virtual CPU (vCPU) operates
    as a host thread scheduled by CFS across physical cores.
    The kernel scheduler distributes these vCPU threads dynamically,
    allocating time slices based on their assigned shares. This ensures:
    * Proportional CPU access during contention periods
    * Fair distribution across all runnable threads
    * Predictable performance through share-based prioritization
      under contention

Use Cases
---------

* A user wants to launch a database server instance and ensure it receives
  a guaranteed minimum level of CPU performance, even under high load
  from other instances on the same host.
* A user is running a batch processing job that requires a consistent amount
  of CPU time to complete within a specific timeframe. They need to request
  a performance tier that guarantees this.
* A cloud administrator wants to offer different tiers of compute service
  (e.g., "Bronze", "Silver", "Gold") based on guaranteed CPU performance,
  allowing users to choose the level that best suits their workload and
  budget.
* As a cloud administrator, I want to achieve higher utilization of my
  infrastructure by allowing lower performance tier instances to use more
  resources when there is low contention but reduce them to a lower
  level of performance when higher priority workloads become active.
  For example, running lower priority batch processing jobs when higher
  priority customer workloads are idle.

Proposed change
===============

Introduce a new mechanism in Nova that leverages CPU shares (using control
groups - cgroups) to define and enforce CPU performance service levels.
Administrators will be able to specify a desired CPU share multiplier
value when defining flavors that users can select when launching or
resizing an instance. This translates to a relative proportion of
available CPU resources on the host.

To achieve this, a new ``quota:cpu_shares_multiplier`` flavor extra spec
and ``VCPU_SHARES`` resource class will be introduced. The
``cpu_shares_multiplier`` will be used to calculate the number of
``VCPU_SHARES`` allocations to request based on the number of VCPUs
requested by the flavor.

This approach provides a flexible and granular way to manage CPU
performance, allowing for oversubscription while still providing a
mechanism for performance guarantees based on the allocated shares.

The validation of the new ``quota:cpu_shares_multiplier`` flavor extra spec
will leverage the existing flavor extra spec validation framework
introduced in the `Flavor Extra Spec Validator`_ specification. This
framework allows for the definition and validation of flavor extra specs,
ensuring correct usage and providing better documentation. A validator
for ``quota:cpu_shares_multiplier`` will be added to the
``nova.api.validation.extra_specs.quota`` module, alongside other
quota-related extra spec validators.

.. _Flavor Extra Spec Validator: https://specs.openstack.org/openstack/nova-specs/specs/ussuri/implemented/flavor-extra-spec-validators.html

The primary mechanism for enforcing per-host, per-tier capacity will be through
the use of OpenStack Placement resource classes. This will be combined with
a new ``ResourceProviderWeigher``. Enforcement at the hypervisor level
will be done by the Linux `CFS scheduler`_.

.. _`CFS scheduler`: https://docs.kernel.org/scheduler/sched-design-CFS.html

CPU Share Management
--------------------

* Allocation: CPU shares will be allocated to instances based on a new
  ``quota:cpu_shares_multiplier`` flavor extra spec. The value will represent
  a relative weight, with higher values indicating a greater proportion of
  CPU resources.
* Example:
    Given the allowed range for CPU shares (1-10,000) and assuming a VM
    can use up to 100 vCPUs, the following ``cpu_shares_multiplier``
    values may be appropriate:
    Gold: 100, Silver: 50, Bronze: 25.

.. Note::

  ``quota:cpu_shares_multiplier`` and ``quota:cpu_shares`` are two ways of
  specifying the same constraint. CPU shares as defined via libvirt applies
  to the domain as a whole rather than per vCPU. As a result, a flavor with
  10 VCPUs and ``quota:cpu_shares=100`` would get equal CPU time as a flavor
  with
  1 VCPU and ``quota:cpu_shares=100`` during CPU contention. By expressing
  shares
  as a multiplier (``quota:cpu_shares_multiplier=100``), the intuitive
  expectation that a 10-core VM should receive 10 times the CPU resources
  is maintained under contention. Administrators can choose between using
  ``quota:cpu_shares`` or ``quota:cpu_shares_multiplier`` based on their
  preference. Translation to Placement allocations will be opt-in via
  configuration for upgrade compatibility.

Performance Guarantee Enforcement via Placement
-----------------------------------------------

A standardized ``VCPU_SHARES`` resource class with weighted allocations
will be used to enforce performance tiers through Placement:

* **Automatic Inventory Reporting:** The libvirt driver will automatically:

  - Report ``VCPU_SHARES`` inventory based on host capabilities.
  - Calculate capacity as ``vcpu_share_multiplier * len(cpu_shared_set)``
  - Respect cgroups v2 API limits of [1, 10000] shares.
  - If and only if enabled via a new ``report_vcpu_shares`` config option.

  Configuration options:

  .. code-block:: ini

      [libvirt]
      report_vcpu_shares = true
      # Range: 1-10000
      # min: 1
      # max: 10000
      # default: 100
      vcpu_share_multiplier = 100

  * Default multiplier provides granularity for 1% increments and up to 100
    vCPUs per VM.
  * The multiplier scales the number of VCPUs requested by the flavor into
    VCPU_SHARES units. The total VCPU_SHARES capacity on a host is calculated
    based on the number of physical cores available for sharing
    (``cpu_shared_set``)
    and the ``vcpu_share_multiplier`` (e.g., 64 cores * 100 multiplier = 6,400
    shares capacity).

* Placement Enforcement: The scheduler ensures the sum of allocated shares
  never exceeds the host's advertised capacity. Shares act as consumption
  coefficients rather than absolute limits, preserving the proportional
  guarantee model.
* Overall VCPU Coordination: The standard VCPU inventory and its
  ``cpu_allocation_ratio`` will still be used as the default CPU capacity
  mechanism. The ``cpu_allocation_ratio`` will also be applied to the
  ``VCPU_SHARES`` inventory to ensure oversubscription works as expected.

ResourceProviderWeigher
-----------------------

The ``ResourceProviderWeigher`` calculates host weights by analyzing
provider trees from Placement's allocation candidates data. This weigher
enables capacity-aware scheduling decisions based on both resource
availability and trait compatibility. The weigher behavior can be
summarized as follows: "select the most boring host" where boring is
defined as having the least number of traits or resource classes.

Input Data Structure
^^^^^^^^^^^^^^^^^^^^^

The weigher processes ``provider_summaries`` data which represents a
forest of resource provider trees. Each tree is encoded as a map with:

- ``resources``: Map of resource class to (capacity, used) tuples
- ``traits``: Set of trait names
- ``parent_provider_uuid``: UUID of parent provider (null for root)
- ``root_provider_uuid``: UUID of tree root provider

Algorithm Implementation
^^^^^^^^^^^^^^^^^^^^^^^^^

1. Tree Flattening
   * Group providers by root UUID
   * Aggregate resource capacities and traits across each tree

2. Resource Calculation:
  * For each available resource class in the provider tree:
  * Sum available capacity across all providers in tree
  * Calculate utilization ratio: ``1 - (requested amount of resource
    class) / (free capacity of resource class)``
  * Calculate the arithmetic mean of these ratios across all available
    resource classes.

 3. Trait Calculation:
    - Collect all traits from tree providers
   * Calculate ratio: ``(count of requested traits present on provider) /
(count of all available traits on provider)``

4. Weight Composition:
   - Combine resource and trait scores: ``(resource_score + trait_score) / 2``
   - Apply multiplier from scheduler configuration

Pseudocode Implementation
^^^^^^^^^^^^^^^^^^^^^^^^^

Here is a pseudocode example illustrating the `ResourceProviderWeigher` logic:

.. code-block:: python

    def _weigh_object(self, host_state, request_spec):
        # host_state contains provider_summaries
        provider_summaries = host_state.provider_summaries
        requested_resources = request_spec.get('resources', {})
        requested_traits = request_spec.get('traits', set())

        resource_ratios = []
        trait_ratios = []

        # 1. Tree Flattening and Aggregation (Simplified)
        # In reality, this involves iterating through the forest and
        # aggregating resources and traits for each root provider tree.
        # For pseudocode, assume we have aggregated_resources and
        # aggregated_traits for the current host's provider tree.
        aggregated_resources = {} # {resource_class: {'capacity': X, 'used': Y}}
        aggregated_traits = set() # {trait_name1, trait_name2, ...}

        # 2. Resource Calculation
        available_resource_classes = aggregated_resources.keys()
        if not available_resource_classes:
            # Handle case with no available resources
            resource_score = 0.0
        else:
            for res_class in available_resource_classes:
                capacity = aggregated_resources[res_class]['capacity']
                used = aggregated_resources[res_class]['used']
                free_capacity = capacity - used

                # Avoid division by zero if free capacity is 0
                if free_capacity > 0:
                    # Use 0 for requested amount if not in the request
                    requested_amount = requested_resources.get(res_class, 0)
                    utilization_ratio = 1.0 - (requested_amount / free_capacity)
                    resource_ratios.append(utilization_ratio)
                else:
                    # If no free capacity, this resource class contributes 0 to the score
                    resource_ratios.append(0.0)

                    # Calculate the arithmetic mean of ratios across all
                    # available resource classes
                    resource_score = sum(resource_ratios) / len(resource_ratios)
                    \
                        if resource_ratios else 0.0

        # 3. Trait Calculation
        if not aggregated_traits:
             # Handle case with no available traits.
             trait_score = 0.0
        else:
            matched_traits_count =
            len(requested_traits.intersection(aggregated_traits))
            available_traits_count = len(aggregated_traits)
            # Calculate ratio: (count of requested traits present) /
            # (count of all available traits)
            trait_score = (
                (matched_traits_count / available_traits_count)
                if available_traits_count > 0 else 0.0)

        # 4. Weight Composition
        # The goal is to select the most boring host (least traits/resources).
        # A lower resource_score (closer to 0) means higher utilization
        # ratios (closer to 1), which happens when requested amount is close
        # to free capacity. This is not what we want. We want a higher score
        # for less utilized resources. The current formula 1 - (requested /
        # available) gives a higher score for less utilized. This seems
        # correct for the resource part.

        # For traits, a higher trait_score means the requested traits and
        # available traits are closely aligned. As a host will never have fewer
        # traits than requested, we want to select a host where the trait_score
        # is as close to 1 as possible because this means there are few extra
        # traits that were not requested. We want a higher score as (more
        # boring = less unrequested traits). The current formula (matched
        # traits) / (available traits) gives a higher score for more matched
        # traits.

        # Combine resource and adjusted trait scores
        # A higher combined score indicates a more "boring" host (less
        # utilized resources, fewer requested traits present).
        combined_score = (resource_score + trait_score) / 2.0

        # Apply multiplier from scheduler configuration.
        weight = combined_score * self.weight_multiplier(host_state)

        return weight


Configuration
^^^^^^^^^^^^^^
Configure via ``nova.conf``:

.. code-block:: ini

    [filter_scheduler]
    # Weight multiplier (default: 0.0)
    resource_provider_weight_multiplier = 1.0

    # Resource classes to consider (default: all)
    resource_provider_resources = VCPU_SHARES,MEMORY_MB

    # Traits to require (default: all)
    resource_provider_traits = HW_CPU_X86_AVX2,HW_CPU_X86_SSE

Example calculation for a VM requesting 2 VCPUs and SSD trait::

  HostA resources: VCPU avail=4
  HostA traits: SSD, HW_CPU_X86_AVX2
  Resource score: 1 - (2/4) = 0.5
  Trait score: (1/2) = 0.5
  Final weight: (0.5 + 0.5)/2 = 0.5

Diagrams and Examples
---------------------

* Virtual Machine Resource Contention Diagram:

  .. code-block:: bash

    +-----------------+   +-----------------+   +-----------------+
    |   Gold Instance |   |  Silver Inst 1  |   |  Silver Inst 2  |
    |   (1 vCPU)      |   |  (1 vCPU)       |   |  (1 vCPU)       |
    |   Shares: 100   |   |  Shares: 50     |   |  Shares: 50     |
    +-----------------+   +-----------------+   +-----------------+
            |                     |                     |
            v                     v                     v
    +-----------------+   +-----------------+   +-----------------+
    |  Bronze Inst 2  |   |  Bronze Inst 3  |   |  Bronze Inst 1  |
    |  (1 vCPU)       |   |  (1 vCPU)       |   |  (1 vCPU)       |
    |  Shares: 25     |   |  Shares: 25     |   |  Shares: 25     |
    +-----------------+   +-----------------+   +-----------------+
            |                     |                     |
            +----------+----------+----------+----------+
                      |                     |
                      v                     v
            +---------------------------------+
            |     Aggregate Host CPU Pool     |
            | (Managed by CFS based on shares)|
            +---------------------------------+
                      ^     ^     ^     ^
                      |     |     |     |
            +---------------------------------+
            |    Host CPU 1   |    Host CPU 2 |
            |    Host CPU 3   |    Host CPU 4 |
            +---------------------------------+

  Total Host Capacity: 400 shared units (4 CPUs * 100 units/CPU)
  Total Allocated Shares: 100 (Gold) + 100 (Silver) + 75 (Bronze) = 275 units
  Remaining Capacity: 400 - 275 = 125 units

  .. note::
     Under contention, CPU time is allocated proportionally to shares across all
     cores.
     At any given time 2 vms can reside on the same core, but the Linux
     scheduler will
     allocate cpu time proportionally to the shares.

  .. note::
     Gold (100 sh): ~36.4% of total CPU time
     Silver (50 sh each): ~18.2% of total CPU time each
     Bronze (25 sh each): ~9.1% of total CPU time each


* Configuration Examples: Provide examples of:

  - Flavor extra specs using share multipliers:

    .. code-block:: console

        # Create flavors with different CPU share multipliers
        $ openstack flavor create --vcpus 1 --ram 1024 --disk 10 bronze
        $ openstack flavor set bronze --property
        quota:cpu_shares_multiplier=25.0

        $ openstack flavor create --vcpus 1 --ram 2048 --disk 20 silver
        $ openstack flavor set silver --property
        quota:cpu_shares_multiplier=50.0

        $ openstack flavor create --vcpus 1 --ram 4096 --disk 40 gold
        $ openstack flavor set gold --property quota:cpu_shares_multiplier=100.0

  - Runtime configuration via nova.conf:

    .. code-block:: ini

        [libvirt]
        report_vcpu_shares = true
        vcpu_share_multiplier = 100  # Default value, scales physical cores
                                     # to shares

        [compute]
        cpu_shared_set = 1-112  # Map to physical cores

Alternatives
------------

* **Reporting VCPU_SHARE via provider.yaml:**
  Initial approach required manual configuration of provider.yaml files to
  report VCPU_SHARES inventory:

  .. code-block:: yaml

      resource_provider:
        name: compute-node01
        inventories:
          VCPU_SHARES:
            total: 100000
            allocation_ratio: 1.0

  **Limitations:**
  - Static configuration unable to adapt to host changes.
  - No direct correlation with cgroups capacity.
  - Manual maintenance overhead.

* **CPU Service Level via Custom Resource Classes (Original Proposal):**
  The initial approach proposed creating multiple custom resource classes
  (e.g., ``CUSTOM_VCPU_GOLD``, ``CUSTOM_VCPU_SILVER``) to represent
  performance tiers. This required:

  - Defining separate resource classes for each service tier
  - Complex flavor extra specs combining vCPU and custom resources
  - Per-host inventory management for each tier.

  **Reason for revision:** Simplified maintenance through standardized
  resource class while maintaining Placement-based guarantees through
  share weighting.

* **Watcher-Based Enforcement (Reactive):** OpenStack Watcher could be
  used to monitor per-host, per-tier CPU usage and reactively migrate
  instances to rebalance the load and enforce policies. This would involve
  developing a custom Watcher strategy that queries Nova/Placement for
  allocations and potentially Telemetry for actual CPU utilization.
* **Pros:** Can provide ongoing compliance and potentially optimize
  placement based on real-time performance metrics. Can act as a safety
  net if initial placement is not strictly enforced.
* **Cons:** Reactive approach means potential for temporary SLO breaches
  until Watcher acts. Relies on accurate telemetry data. Adds complexity
  with an additional service to deploy and manage.
* **Reason for not choosing as primary:** While Watcher can complement
    the Nova scheduler approach by providing ongoing monitoring and
    optimization, it is not ideal for strict deterministic enforcement
    at the time of placement. The Placement-based approach prevents
    violations from occurring in the first place.

* **Static Host Aggregates:** Using host aggregates to group hosts by
  intended service tier capacity and relying on aggregate-specific
  scheduler filters.
* **Pros:** Relatively simple to implement for basic tiering.
* **Cons:** Less flexible than Placement custom resources for dynamic
inventory and fine-grained control. Does not easily support mixing
tiers on the same host with guaranteed per-tier capacity limits.
Requires manual management of host aggregate membership.

* **Percentage-Based Allocation:** Allowing users to request a percentage
  of host CPU resources instead of raw shares.
* **Pros:** Potentially more intuitive for users.
* **Cons:** Translating a percentage request to a consistent share value
  across hosts with varying total CPU capacity can be complex. May not
  align directly with the underlying cgroup share mechanism which is
  relative.


Security impact
---------------

None

Notifications impact
--------------------

None

Other end user impact
---------------------

* The Horizon dashboard has limited support for displaying inventories
  of resource classes to administrators. While it could be extended to
  display ``VCPU_SHARES``, that is not currently planned and is out of
  scope. Administrators will be able to query this information via the
  existing OpenStack client, so no end user impact is expected.

Performance Impact
------------------

* The overhead of cgroup management by the compute driver should be
  minimal, as it leverages existing kernel features and libvirt
functionality. As a result, it effectively amounts to reporting
inventories of ``VCPU_SHARES``, weighing hosts based on the available
resource classes in the provider summaries and generating the
appropriate libvirt xml to have it enforce cpu shares using the Linux
scheduler.

* The primary performance impact will be on instance performance based on
  their allocated shares and the overall host load. Instances with higher
  shares are expected to receive a proportionally larger amount of CPU
  time under contention.
* The scheduler's interaction with Placement to check resource inventory
  should have minimal performance impact, as Placement is designed for
  scalable resource tracking. However, the overhead of the new
  ``ResourceProviderWeigher`` will impact scheduler performance. To
  mitigate this impact, it will be disabled by default via a default
  multiplier of ``0.0``.

Other deployer impact
---------------------

None

Developer impact
----------------

None

Data model impact
-----------------
None

while the Hoststate object in the scheudler will need to be extened
to provide access to the Provider Summeries. That is an internal
change and does not affect the overall datamodel.

REST API impact
---------------

None

Upgrade impact
--------------

* Careful consideration is needed for rolling upgrades to ensure
  compatibility with existing instances.
* Following the pattern established by the pci in Placement feature, a
  new config option to report CPU shares in Placement will be added to
  the libvirt section and a corresponding config option to request CPU
  shares will be added to the filter scheduler section.

 .. code-block:: ini

   [libvirt]
   report_vcpu_shares = true|false  # Default: false
   vcpu_share_multiplier = 100  # Default value

   [filter_scheduler]
   query_placement_for_vcpu_shares = True | False (Default)

 As with pci in Placement when share reporting is enabled it should not be
 disabled again. When activated, on startup, the nova compute agent will need
 to reshape existing allocations for instances in Placement to request CPU
 shares if they use either the ``quota:cpu_shares`` or
 ``quota:cpu_shares_multiplier`` flavor extra specs.


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  sean-k-mooney

Other contributors:
  deepseek-r1
  gemini-flash-2.5

Feature Liaison
---------------

Feature liaison:
  sean-k-mooney

Work Items
----------

1. Implement ``VCPU_SHARES`` resource class support in Placement.
2. Add libvirt driver integration for automatic inventory reporting.
3. Develop ``ResourceProviderWeigher`` scheduler component.
4. Create new flavor extra spec validation logic.
5. Implement configuration options for upgrade compatibility.
6. Add documentation and release notes.

Dependencies
============

* A new resource class needs to be added to os-resource classes.

Testing
=======

Comprehensive testing is required to ensure the feature functions correctly and
provides the expected performance guarantees.

* **Unit Tests:**
  Unit tests will be provided to cover the basic operations of the weigher,
  xml generation and other relevant components.
* **Functional Tests:** Verify the interaction between different Nova
  components (API, scheduler, compute) and the hypervisor/Placement.
  New functional tests will be required to validate the reshape logic.
  They can also be used to assert that shares are reported correctly
 to Placement and that the translation mechanism from flavor extra spec
 to allocations works as expected.

* **Tempest Tests:**

  In general, as this primarily builds on standard Placement logic, there
  is not much additional Tempest logic required. With that said, it
  should be possible to add a Tempest test that creates a flavor that
  uses the new flavor extra specs and assert that the instance allocations
  contain the relevant VCPU_SHARES. Alternatively, we can tweak the
  flavors in one of our existing jobs to use this feature for all tests.

Documentation Impact
====================

* Update the Nova API reference to document the changes to the server
  creation, resize, and show details APIs, including the new ``cpu_shares``
  parameter and its behavior.
* Update the Nova administrator guide to explain how to configure compute
  nodes to expose custom tier-specific VCPU resources to Placement via
  provider config files, how to define flavors with appropriate extra
  specs, and how to configure related policy rules.
* Update the Nova user guide to explain how users can request CPU
  performance service levels by selecting flavors with specific CPU
  shares or by specifying the ``cpu_shares`` parameter directly (if
  allowed by policy).

References
==========

* `OpenStack cgroup tiering README <https://github.com/gprocunier/openstack-cgroup-tiering/blob/b205e34c0c62fa6cd451a44659c648595f0d19ae/README.md>`_

History
=======

.. list-table:: Revisions
   :header-rows: 1

  * - Release Name
    - Description
  * - 2025.2 Flamingo
    - Introduced CPU performance service levels using CPU shares and
      Placement standardized resource classes.
```

### Summary of v0 -> v7 Evolution

For those that dont want to read the spec here is an ai generated
summary of what changed.

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

Sean: ^ while not wrong AIs can be a bit of a suck up.

### The Evolution from v0 to v7

Right, so back to the actual work - what prompts did I use to get from the AI's first rough attempt to something that didn't make me cringe when I read it?

I started with something relatively simple, because if there's one thing I've learned about working with AI, it's that you don't dump your entire design vision on it at once.

```
the first change i woudl like to make is summerise the current proposal as a
short paragaph in the Alternatives
seation with a title of "cpu service level via custom resources classes"
after that we will work togehter to rework the spec to use a single
standard resouces class called "VCPU_SHARES"
```
In other words, I started by summarising the initial spec as an alternative
and then made the first big change: using a single resource class instead of 1 per tier.

This is one of the core simplification needed to make this easy to use.

next i got a little more ambiguous and the next imporvement i identifed was  to add automatic reporting instead of using provider.yaml
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

so at this point the spec describes the problem and use case and
introduces a new standard resource class and describes updating the libvirt
driver to report that automatically.

next step schduling.

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

This is getting even more ambitious.
so there is a lot to take in with this prompt
the main element of note are that i provided it the paths to documentation
for the placement allcoation_canddiates api and the api samples

I also named the relevant Classes in the nova scheduler `HostState`
and provided the host_manager.py as context.

With that context i described how i wanted the `ResourceProviderWeigher`
to work.

This got pretty close but needed several refinements to actually express my
intended. this and the prior prompts were the largest changes to the spec and it was here that i took the time to make most of my manual changes.

Form this point my prompts were more polishing rather then large changes.

```
Review @/nova-specs/specs/cpu-performance-service-level.rst for consistency in
style, information presentation, and section content, using
@/nova-specs/specs/train/implemented/cpu-resources.rst as a detailed example.
```
the cpu-resouces spec is one of our most complex specs and as a result one
of the better written.

```
can you add a short introduction to
@/nova-specs/specs/cpu-performance-service-level.rst describing how the linux
CFS scheduler works in the context fo cpu_shares and how vms VCPU kvm threads
are schdule to host cores and how time is allcoated to each process.
```

## Results and Evaluation

### The Punch Line
Did Ai make faster or easier to write the spec... No not with this setup
but it did result in a higher quality spec with most if not all of the spelling issues address before humans other then my self had to read them.

The time spent setting up and debugging this setup especially the time spent
trying to use local models with the limited ram availability negated any speed up that could otherwise have been achieved.

If or when i do this again i think i will alter the agentic agent
choice to either use aider or claud_code. roo could also be very effective
if i choose a more powerful LLM that better supports roo complex system prompt and tool calling conventions.

### The Final Result

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

### Design Iteration and Refinement

Iteration works well when working with LLMs and AI agents.
With enough context and prompting you can get it to do some impressive refactoring but it still need your guidance. If you provide it with
high quality examples it can provide high quality output.

### Technical Writing and Clarity

The AI was great at helping structure complex technical concepts in a way that would be accessible to reviewers who might not be deeply familiar with cgroups or the placement service internals.

An example of this is the ascii diagram of the vm vcpu to host cpus mappings.

In the past i have drawn them using https://asciiflow.com/#/
which is great but i think gemini did a great job.
inially i used deepseek but gemini i think catured it better.

## What Was Challenging

### Domain Expertise Limitations

Here's the thing - while the AI knows a lot about OpenStack in general,
it doesn't have the kind of deep institutional knowledge you get from years
of actually working on Nova. I found myself constantly having to correct
assumptions about how features actually work, or provide context about why
we made certain design decisions years ago.

For example, the AI initially suggested approaches that would have
completely broken CPU pinning, or didn't account for the fact that the
libvirt driver has some... let's call them "quirks" in how it handles
certain configurations.

### Community and Process Knowledge

The AI doesn't understand the social aspects of the OpenStack development
process - what kinds of changes are likely to be controversial, which core
reviewers have strong opinions about certain areas, or how to phrase things
to avoid bikeshedding in reviews.

It also doesn't know about informal agreements or "soft" rules that aren't
documented anywhere but are understood by the community. You know, the kind
of things you learn from being in IRC discussions at 2am when someone's
trying to debug a weird corner case.

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

### Restructured text

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

aider and claud_code or other tools like  zed, windsurf or cursor
may do better.

## Lessons Learned

### AI as a Collaborative Partner

The most effective approach wasn't having the AI write the spec for me, but rather using it as a collaborative partner.

The AI was excellent at:
- Helping structure and organize complex information
- generating summaries and diagrams
- comparing refernce information to determine how to improve the existing
  proposal to incorporate that context.

### Domain Expertise Still Critical

My years of experience with Nova and OpenStack were essential for:
- Knowing what's actually implementable vs. what sounds good on paper
- Understanding the community and review process
- Catching subtle interactions with existing features
- Making pragmatic trade-offs between ideal solutions and mergeable code

### Iteration and Refinement

The best results came from multiple rounds of iteration. The first draft
was pretty rough, but each iteration I ask for specific problems to be
addressed and incrementally made the specification significantly better.

## Would I Do This Again?

This is an interesting question.

As i alluded to before if my goal was to get things down faster then
i don't think that was a success. fortunately that was not my goal
my goal was to provide a higher quality initial version of a spec for
upstream review.

i wanted to try and mitigate some of the mental load of written
communications and mask the impacts of my dyslexia. That goal was
definitely achieved.

Was it easy?  no this still took a lot of effort to drive.
would i do it again with ai. yes on balance i think i would.

## Next Steps

The specification is now ready for community [review](https://review.opendev.org/c/openstack/nova-specs/+/951222).
I am curious to see how the community responds to an AI-assisted
specification. Will reviewers be able to tell? Will the quality be
noticeably different?

I'm also planning to use this approach for other specifications I've been
putting off due to time constraints. If the community review goes well, I
might write up some guidelines for others who want to experiment with
AI-assisted spec writing once i gain more experience with it.

For now I have created https://github.com/SeanMooney/openstack-ai-style-guide
to help with future AI usage in OpenStack.


## Implications for the OpenStack Community

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

---
layout: post
title: DDIA Notes 1, Foundations of Data Systems
categories: book
tags: [systems]
description: >
toc:
  sidebar: right
media_subpath: /assets/img/cs61b/
thumbnail: /assets/img/ddia/book-cover.png
---

## 1. Reliable, Scalable, and Maintainable Applications

Many applications today are data-intensive, as opposed to compute-intensive. A data-intensive application is typically built from standard building blocks (**data systems**): databases, caches, search indexes, stream processing, and batch processing etc.

When building an application, we need to figure out which tools and which approaches are the most appropriate for the task at hand. 

### Thinking About Data Systems
We typically think of <i>databases, queues, caches</i>, etc. as being very different categories of tools. But we should lump them all together under an umbrella term like **data systems** for two main reasons:

- The boundaries between the categories are becoming blurred.
  - **Redis** datastores that are also used as message queues
  - **Apache Kafka** message queues with database-like durability guarantees
- Increasingly many applications now have demanding or wide-ranging requirements. 
  - the work is broken down into tasks that can be performed efficiently on a single tool 
  - those different tools are stitched together using application code

<div class="row justify-content-sm-center">
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid path="/assets/img/ddia/1/1-1.png" class="img-fluid rounded z-depth-1" zoomable=true caption="One possible architecture for a data system that combines several components."%}
  </div>
</div>

When you combine several tools in order to provide a service, the service’s interface
or application programming interface (API) usually hides those implementation
details from clients. Now you have essentially created a new, special-purpose data
system from smaller, general-purpose components.

Three concerns that are important in most software systems: Reliability, Scalability and Maintainability. 

### Reliability
Reliability means making systems work correctly, even when faults occur. 

- Hardware Faults
  - It happens <i>all the time</i> when you have a lot of machines.
    - e.g., hard disks crash, RAM becomes faulty, the power grid has a blackout, someone unplugs the wrong network cable
  - **Hardware redundancy** was sufficient for most applications, since it makes total failure of a single machine fairly rare.
    - When one component dies, the redundant component can take its place while the broken component is replaced. 
  - There is a move toward systems that can tolerate the loss of entire machines, with **software fault-tolerance techniques**. Reasons:
    - more applications have begun using larger numbers of machines, which proportionally increases the rate of hardware faults
    - some cloud platforms are designed to prioritize flexibility and elasticityi over single-machine reliability
- Software Faults
  - Hardware faults are usually random and independent from each other, but software fault is a kind of **systematic error** within the system.
    - harder to anticipate and tend to cause many more system failures
  - The bugs often lie dormant for a long time until they are triggered by **an unusual set of circumstances**.
    - the software is making some kind of assumption about its environment, which is usually true but eventually stops being true for some reason
  - No quick solution; small things can help
    - carefully thinking about assumptions and interactions
    - thorough testing
    - process isolation
    - measuring, monitoring, and analyzing system behavior in production

- Human Errors
  - Humans design, build software systems, keep the systems running. Even when they have the best intentions, **humans are known to be unreliable**.
    - configuration errors by operators were the leading cause of outages
  - The best systems combine several approaches to make our systems reliable, in spite of unreliable humans:
    - **Design** systems in a way that minimizes opportunities for error
      - e.g., well-designed abstractions, APIs, and admin interfaces
    - **Decouple** the places where people make the most mistakes from the places where they can cause failures.
      - sandbox
    - **Test** thoroughly at all levels, from unit tests to whole-system integration tests and manual tests.
    - Allow quick and easy **recovery** from human errors, to minimize the impact in the case of a failure.
      - fast to roll back, roll out new code gradually
    - Set up detailed and clear **monitoring**.
      - performance metrics, error rates

Reliability is important. Unreliable applications may cause lost productivity, legal risks, lost revenue and damage to reputation.

### Scalability

Scalability means having strategies for keeping performance good, even when load 
increases.

- Describing Load
  - Load can be described with a few numbers which we call **load parameters**. The best choice of parameters depends on the architecture of your system.
    - e.g., requests per second to a web server, the ratio of reads to writes in a database, the number of simultaneously active users in a chat room, the hit rate on a cache
  - Example: Two ways of implementing <i>Post tweet</i> and <i>Home timeline</i> on Twitter
    - Approach 1: using a global collection of tweets
    - Approach 2: maintaining a cache for each user’s home timeline (like a mailbox)
    - **the distribution of followers per user** (maybe weighted by how often those users tweet) is a key load parameter for discussing scalability

<div class="row justify-content-sm-end">
  <div class="col-sm-10 mt-3 mt-md-0">
    {% include figure.liquid path="/assets/img/ddia/1/1-2.png" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
</div>

<div class="row justify-content-sm-end">
  <div class="col-sm-10 mt-3 mt-md-0">
    {% include figure.liquid path="/assets/img/ddia/1/1-3.png" class="img-fluid rounded z-depth-1" zoomable=true %}
  </div>
</div>

- Describing Performance
  - batch processing system, **throughput**
    - the number of records we can process per second, or 
    - the total time it takes to run a job on a dataset of a certain size
  - online systems, **response time**
    - the time between a client sending a request and receiving a response
  - **percentiles** are often used in contracts that define the expected performance and availability of a service
    - service level objectives (SLOs)
    - service level agreements (SLAs)
- Approaches for Coping with Load
  - Rethink your architecture on load increase.
  - **scaling up** (vertical scaling, moving to a more powerful machine)
  - **scaling out** (horizontal scaling, distributing the load across multiple smaller machines)
    - Taking stateful data systems from a single node to a distributed setup can introduce a lot of additional complexity.
    - Common wisdom in the past was to keep your database on a single node (scale up) until scaling cost or high-availability requirements forced you to make it distributed.
    - It is conceivable that **distributed data systems will become the default** in the future, as the tools and abstractions for distributed systems get better.
  - **Elastic** systems can automatically add computing resources when they detect a load increase.
    - useful if load is highly unpredictable
    - manually scaled systems are simpler and may have fewer operational surprises


### Maintainability

Maintainability has many facets, but in essence it’s about making life better for the engineering and operations teams who need to work with the system. It is well known that the majority of the cost of software is not in its initial development, but in its **ongoing maintenance**. 

- Operability: easy to keep the system running smoothly
  - Providing visibility into the runtime behavior and internals of the system, with good **monitoring**
  - Providing good **documentation** and an easy-to-understand operational model ("If I do X, Y will happen")
- Simplicity: easy for new engineers to understand the system, by removing as much complexity as possible from the system.
  - Possible symptoms of complexity
    - explosion of the state space, tight coupling of modules, tangled dependencies, inconsistent naming and terminology, hacks aimed at solving performance problems, special-casing to work around issues elsewhere, etc.
  - **accidental complexity**: it is not inherent in the problem that the software solves (as seen by the users) but arises only from the implementation.
    - One of the best tools we have for removing accidental complexity is **abstraction**.
      - hide a great deal of implementation detail behind a clean, simple-to-understand facade
- Evolvability: easy for engineers to make changes to the system in the future, adapting it for unanticipated use cases as requirements change
  - closely linked to its simplicity and its abstractions

## 2. Data Models and Query Languages
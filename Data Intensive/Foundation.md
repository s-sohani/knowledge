Many applications need to:
- databases
- cache
- search indexes
- stream processing
- batch processing
There are many database systems with different characteristics, because different applications have different requirements. There are various approaches to caching, several ways of building search indexes, and so on. When building an application, we still need to figure out which tools and which approaches are the most appropriate for the task at hand.

In recent years, numerous new tools for data storage and processing have emerged, optimized for various specific use cases and defying traditional categories. As a result, tasks are distributed among specialized tools, which are then integrated via application code.
![[Pasted image 20240527063735.png]]

Now you have essentially created a new, special-purpose data system from smaller, general-purpose components.

If you are designing a data system or service, a lot of tricky questions arise. How do
you ensure that the data remains correct and complete, even when things go wrong
internally? How do you provide consistently good performance to clients, even when
parts of your system are degraded? How do you scale to handle an increase in load?
What does a good API for the service look like?

## Reliability
• The application performs the function that the user expected.
• It can tolerate the user making mistakes or using the software in unexpected
ways.
• Its performance is good enough for the required use case, under the expected
load and data volume.
• The system prevents any unauthorized access and abuse.

The things that can go wrong are called faults, and systems that anticipate faults and
can cope with them are called fault-tolerant or resilient.

### Types of faults
#### Hardware Faults
When we think of causes of system failure, hardware faults quickly come to mind.
Hard disks crash, RAM becomes faulty, the power grid has a blackout, someone
unplugs the wrong network cable.

To prevent this type of failure we can use **Raid** Hard Disks or use **Multi-machine redundancy**.

#### Software Error
- A software bug that causes every instance of an application server to crash when given a particular bad input. For example, consider the leap second on June 30, 2012, that caused many applications to hang simultaneously due to a bug in the Linux kernel
- A runaway process that uses up some shared resource—CPU time, memory, disk space, or network bandwidth.
- A service that the system depends on that slows down, becomes unresponsive, or starts returning corrupted responses.
- Cascading failures, where a small fault in one component triggers a fault in another component, which in turn triggers further faults.

In those circumstances, it is revealed that the software is making some kind of assumption about its environment—and while that assumption is usually true, it eventually stops being true for some reason.
For example, in a message queue, that the number of incoming messages equals the number of outgoing messages

#### Human Errors
How do we make our systems reliable, in spite of unreliable humans?
- Design systems in a way that minimizes opportunities for error for example develop **API** to do things.
- Decouple the places where people make the most mistakes from the places where they can cause failures like **sandbox**.
- Test thoroughly at all levels, from unit tests to whole-system integration tests and manual tests.
- Allow quick and easy recovery from human errors.
- Set up detailed and clear monitoring, such as performance metrics and error rates.

## Scalability
For example the number of simultaneously active users.
- When you increase a load parameter and keep the system resources (CPU, memory, network bandwidth, etc.) unchanged, how is the performance of your system affected?
- When you increase a load parameter, how much do you need to increase the resources if you want to keep performance unchanged?

Even if you only make the same request over and over again, you’ll get a slightly different response time on every try. In practice, in a system handling a variety of requests, the response time can vary a lot. We therefore need to think of response time not as a single number, but as a distribution of values that you can measure.
![[Pasted image 20240527073055.png]]

Random additional latency could be introduced by a context switch to a background process, the loss of a network packet and TCP retransmission, a garbage collection pause, a page fault forcing a read from disk, mechanical vibrations in the server rack.


### Approaches for Coping with Load
- vertical scaling
- horizontal scaling

A system that is designed to handle 100,000 requests per second, each 1 kB in size, looks very different from a system that is designed for 3 requests per minute, each 2 GB in size—even though the two systems have the same data throughput.


## Maintainability
It is well known that the majority of the cost of software is not in its initial development, but in its ongoing maintenance
fixing bugs, keeping its systems operational, investigating failures, adapting it to new platforms, modifying it for new use cases.

legacy system: System that out of date and 
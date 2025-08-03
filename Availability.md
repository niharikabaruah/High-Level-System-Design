# Availability

Availability is the measure of a system's uptime and its ability to be accessible and usable when needed. It's often expressed as a percentage of time that a system is operational.

High availability is a critical requirement for many systems. For example, an e-commerce website needs to be available 24/7, because any downtime can result in lost revenue and damage to the brand's reputation.

### How is Availability Measured?

Availability is measured in "nines". Here's a table that shows what different levels of availability mean in terms of downtime per year:

| Availability % | "Nines" | Downtime per year |
| :------------- | :------ | :---------------- |
| 90%            | One nine  | 36.53 days        |
| 99%            | Two nines | 3.65 days         |
| 99.9%          | Three nines | 8.77 hours        |
| 99.99%         | Four nines | 52.6 minutes      |
| 99.999%        | Five nines | 5.26 minutes      |
| 99.9999%       | Six nines | 31.56 seconds     |

Achieving a higher level of availability requires more complex and expensive systems. "Five nines" is often considered the gold standard for high availability.

### How to Improve Availability:

1.  **Eliminate Single Points of Failure (SPOFs):** A single point of failure is a component of a system that, if it fails, will cause the entire system to fail. To improve availability, you need to identify and eliminate SPOFs.

    *   **Redundancy:** This is the most common way to eliminate SPOFs. It involves duplicating critical components of the system. For example, if you have one web server, you can add a second one. If the first one fails, the second one can take over.

2.  **Load Balancing:** A load balancer distributes incoming traffic across multiple servers. This not only improves performance but also improves availability. If one server goes down, the load balancer can detect the failure and redirect traffic to the remaining healthy servers.

3.  **Failover:** This is the process of automatically switching to a redundant or standby system in the event of a failure.

    *   **Active-Passive Failover:** In this setup, you have a primary server (active) and a secondary server (passive). The passive server is on standby and only takes over if the active server fails.
    *   **Active-Active Failover:** In this setup, you have multiple active servers, and the load is distributed among them. If one server fails, its traffic is redirected to the other active servers.

4.  **Monitoring and Alerting:** You need to have a robust monitoring system in place to detect failures as soon as they happen. When a failure is detected, the system should automatically trigger an alert so that the operations team can investigate and resolve the issue.

5.  **Geographic Distribution:** For mission-critical systems, you might want to distribute your servers across multiple geographic locations. This way, if there's a power outage or a natural disaster in one location, your system can still be available from another location.

### Availability vs. Reliability

Availability and reliability are related but distinct concepts:

*   **Availability:** Is the system up and running?
*   **Reliability:** How long can the system run without failure?

A system can be highly available but not very reliable. For example, a system that crashes every 5 minutes but reboots in 1 second could be considered highly available, but it's not reliable.

In system design, we strive for both high availability and high reliability.

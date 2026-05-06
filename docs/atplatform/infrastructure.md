---
description: How we scale and provide resilience.
icon: gear-complex-code
---

# Infrastructure

The atPlatform is designed to be distributed and allows people to run their own infrastructure for atDirectory and atServer services on their own networks. Here, we show how at a high level how Atsign runs the Internet atPlatform Infrastructure.

Atsign services are monitored by an independent third party for uptime and can be seen here:

{% embed url="https://status.atsign.com/" %}
UpTimeRobot Reports
{% endembed %}

## atDirectory

Atsign runs the Internet atDirectory, which has to be resilient and dependable. To provide that level of service, we use Google's Cloud Platform, Kubernetes, containers, and a distributed in-memory database.&#x20;

<figure><picture><source srcset="../.gitbook/assets/Auto-Sized - Dark.png" media="(prefers-color-scheme: dark)"><img src="../.gitbook/assets/Auto-Sized - Light.png" alt="Architecture Diagram of the atDirectory infrastructure"></picture><figcaption><p>Highly available design</p></figcaption></figure>

The atDirectory runs in a GCP Virtual Private Cloud. This VPC also houses an auto-scaling Kubernetes cluster which is spread across multiple datacenters and availability zones.

The atDirectory service is found on the well-known DNS address `root.atsign.org` on port `64.` This is load balanced across the atDirectory containers. These containers, through an internal load balancer, to read-only in-memory databases containing the Atsign to Fully Qualified Domain Name (FQDN) and port number mappings for all Atsigns.

The read-only databases are kept up to date with a single read-write database. This database is updated by the registrar [website](https://my.atsign.com/), which is run in another Kubernetes cluster.

This design has proved to be reliable and allows upgrades in place without downtime. It automatically scales as load increases by spinning up more containers, and, if required, by adding new machines to the cluster itself. GCP's platform and Kubernetes have demonstrated resilience during data center or hardware issues, and have self-healed the infrastructure.

## atServers

Each Atsign has its own dedicated personal data store, called an "atServer," running as a Docker container within a Docker Swarm. We run a number of Docker Swarms and can move atServers from one swarm to another. However, for high availability, we rely on the Docker Swarm's manager nodes to orchestrate and ensure each atServer is up and running even if hardware fails within a swarm.

Why Docker Swarm and not Kubernetes? Kubernetes is an excellent choice for groups of containers that provide a service like the atDirectory or websites. But, Kubernetes does not scale down well for thousands or millions of tiny independent containers like atServers.  Docker Swarm also provides highly resilient networking and is very lightweight.

<figure><picture><source srcset="../.gitbook/assets/Docker Swarm - Dark.png" media="(prefers-color-scheme: dark)"><img src="../.gitbook/assets/Docker Swarm - Light.png" alt=""></picture><figcaption><p>resilient atServer Cluster</p></figcaption></figure>

The FQDN and port number for a given Atsign from the atDirectory is connected to the Docker Swarm. Each Docker Swarm node will route the port number to the right container on the swarm via its internal VXLAN. The Manager Nodes are responsible for ensuring each container is running and available across the whole swarm.

For data requiring persistent storage beyond the Docker Swarm, encrypted atServer data gets transferred to a highly resilient NetApp Cloud Volume managed by GCP. This cloud volume functions as a network file system accessible to the atServers. &#x20;

All infrastructure components are distributed across multiple data centers and availability zones, and have proven to be highly reliable with very little downtime of individual atServers during failures or upgrades.

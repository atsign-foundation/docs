---
description: The atServer is one of the most important parts of the Atsign Platform.
icon: server
---

# atServer

## What is an atServer?

An atServer is both a personal data service for storing encrypted data owned by an Atsign, and a rendezvous point for information exchange. An atServer is responsible for the delivery of encrypted information to other atServers, from which the owners of those Atsigns can then retrieve the data.

An Atsign's "personal" data on its atServer is stored as encrypted ciphertext, decryptable only with that atSign's edge cryptographic [atKeys](../atsign/atkeys.md), unless the data is explicitly made public.

## atServer Functionality

* Cryptographic authentication of client devices.
* Cryptographic authentication of other atServers.
* Persistence of encrypted data on behalf of the controlling Atsign.
* Caching of data shared by others with the controlling Atsign.
* Notification of data change events to clients (edge devices) and other atServers to facilitate delivery of information shared with them.
* Synchronization of data with multiple clients (edge devices).
* TLS wire encryption from clients to atServers using SSL certificates.
* Mutually authenticated TLS 1.2/1.3 wire encryption between atServers using SSL certificates.

## How to connect to atServers

1. Use a tool such as [OpenSSL](https://www.openssl.org/) to establish a TLS connection to `root.atsign.org:64`.

```shellscript
openssl s_client -connect root.atsign.org:64
```

2. Enter an Atsign (without the `@` symbol)

Example:

```shellscript
@rv_am
45ddba00-ee9a-5cb5-a219-44f40fab8991.swarm0001.atsign.zone:6121
```

3. That is the host and port of the atServer belonging to the Atsign "@rv\_am". You may cancel the original connection and establish a new one with the new host and port.

```shellscript
openssl s_client -connect 45ddba00-ee9a-5cb5-a219-44f40fab8991.swarm0001.atsign.zone:6121
```

3. Once connected, you can execute Atsign Protocol commands and interact with the Atsign's atServer.

## How we run atServers

Each Atsign has its own dedicated personal data store, called an "atServer," running as a Docker container within a Docker Swarm. We run a number of Docker Swarms and can move atServers from one swarm to another. However, for high availability, we rely on the Docker Swarm's manager nodes to orchestrate and ensure each atServer is up and running even if hardware fails within a swarm.

Why Docker Swarm and not Kubernetes? Kubernetes is an excellent choice for groups of containers that provide a service like the atDirectory or websites. But, Kubernetes does not scale down well for thousands or millions of tiny independent containers like atServers.  Docker Swarm also provides highly resilient networking and is very lightweight.

<figure><img src="../../.gitbook/assets/image (29).png" alt=""><figcaption><p>Resilient atServer architecture</p></figcaption></figure>

The FQDN and port number for a given Atsign from the [atDirectory](../atdirectory.md) is connected to the Docker Swarm. Each Docker Swarm node will route the port number to the right container on the swarm via its internal VXLAN. The Manager Nodes are responsible for ensuring each container is running and available across the whole swarm.

For data requiring persistent storage beyond the Docker Swarm, encrypted atServer data gets transferred to a highly resilient NetApp Cloud Volume managed by GCP. This cloud volume functions as a network file system accessible to the atServers. &#x20;

All infrastructure components are distributed across multiple data centers and availability zones, and have proven to be highly reliable with very little downtime of individual atServers during failures or upgrades.

## GitHub

The [atServer](https://github.com/atsign-foundation/at_server) code is fully open-source.

{% embed url="https://github.com/atsign-foundation/at_server" %}

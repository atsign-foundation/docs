---
description: >-
  Learn about what the atDirectory is and how it works with other components on
  the platform.
icon: book-bookmark
---

# atDirectory

## Summary

In order for an Atsign to communicate with another one on the Internet, we need to locate the atServer that can send and receive information securely on its behalf.

The location of an atServer is found using the atDirectory service (`root.atsign.org:64`). This directory returns the DNS address and port number of the atServer for any Atsign that it has a record for. The atDirectory service contains no information about the owner of the Atsign.

Atsign runs the Internet atDirectory, which has to be resilient and dependable. To provide that level of service, we use Google's Cloud Platform, Kubernetes, containers, and a distributed in-memory database.&#x20;

## How to connect to the atDirectory

1. Use a tool such as [OpenSSL](https://www.openssl.org/) to establish a TLS connection to root.atsign.org:64.

```shellscript
openssl s_client -connect root.atsign.org:64
```

2. Enter an Atsign (without the `@` symbol)

```shellscript
@rv_am
45ddba00-ee9a-5cb5-a219-44f40fab8991.swarm0001.atsign.zone:6121
```

3. Tada! That is the host and port of the atServer belonging to the Atsign "@rv\_am".
4. Subsequently, you can establish an additional openssl session with that new host and port using the same method in step 1.

## How we run atDirectories

The atDirectory runs in a GCP Virtual Private Cloud. This VPC also houses an auto-scaling Kubernetes cluster which is spread across multiple datacenters and availability zones.

The atDirectory service is found on the well-known DNS address `root.atsign.org` on port `64.` This is load balanced across the atDirectory containers. These containers, through an internal load balancer, to read-only in-memory databases containing the Atsign to Fully Qualified Domain Name (FQDN) and port number mappings for all Atsigns.

The read-only databases are kept up to date with a single read-write database. This database is updated by the registrar [website](https://my.atsign.com/), which is run in another Kubernetes cluster.

This design has proved to be reliable and allows upgrades in place without downtime. It automatically scales as load increases by spinning up more containers, and, if required, by adding new machines to the cluster itself. GCP's platform and Kubernetes have demonstrated resilience during data center or hardware issues, and have self-healed the infrastructure.

<figure><img src="../.gitbook/assets/image (28).png" alt="Highly available design"><figcaption><p>Highly available design</p></figcaption></figure>


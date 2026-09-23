---
description: >-
  The protocol that defines client-to-atServer and atServer-to-atServer
  communication.
icon: train-subway-tunnel
---

# Atsign Protocol

## What is the Atsign Protocol?

The Atsign Protocol communicates via layer 7, the application layer of the OSI model, over TCP/IP. The protocol defines how actors communicate with one another: client-to-atDirectory, client-to-atServer, and atServer-to-atServer.

## Atsign Protocol Specification

{% hint style="info" %}
The Atsign Protocol communicates via layer 7, the application layer of the OSI model, over TCP/IP.
{% endhint %}

The Atsign Protocol is an application protocol that enables data sharing between Atsigns. You can learn more about the Atsign Protocol by reading the [specification on GitHub](https://github.com/atsign-foundation/at_protocol/blob/trunk/specification/atsign_protocol_specification.md). The Atsign Protocol uses TCP/IP and TLS but does not specify how data itself is encrypted, that is the job of the atSDK and atClient libraries.

<p align="center"><a href="https://github.com/atsign-foundation/at_protocol/blob/trunk/specification/atsign_protocol_specification.md" class="button primary" data-icon="train-subway-tunnel">Go to Atsign Protocol Specification on GitHub</a></p>

## How do I use the Atsign Protocol?

At a rudimentary level, the Atsign protocol is just a language (or set of rules) that both client sand atServers must follow in order to communicate effectively with one another. The below example shows you how to speak the Atsign Protocol in a client-to-atServer scenario.

1. Use a tool like OpenSSL to establish a TLS connection with our production atDirectory.

```shellscript
openssl s_client -connect root.atsign.org:64
```

2. Enter your Atsign into the prompt and find out your atServer's host and port.

Example:

```shellscript
@rv_am
45ddba00-ee9a-5cb5-a219-44f40fab8991.swarm0001.atsign.zone:6121
```

3. Make a second connection to that new host:port (this is the atServer's host and port)

```shellscript
openssl s_client -connect 45ddba00-ee9a-5cb5-a219-44f40fab8991.swarm0001.atsign.zone:6121
```

4. You can now execute Atsign Protocol verbs.&#x20;
5. `scan` is one Atsign Protocol verb that will display public records for that Atsign.

Example:

```shellscript
@scan
data:["publickey@rv_am","signing_publickey@rv_am"]
```

6. `lookup:` is another Atsign Protocol verb that will fetch the raw data for that record.

Example:

```shellscript
@lookup:publickey@rv_am
data:MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAobv3bVJ7mMkQRd4JPygpdFTwlIZpcuiyRefE4xt9uKPmNHM+38vzKNjGXWJ6U3nO7XkOYnEgzWao2cmID3unUruaj6LdPNlGizkCnl8RmXJNle6Wruxyb9oBSMxDvTVPBJHLTCe8FgIYPhV1Y0COWoglZTjOBL5WY8/h5rwLlDP5jPOLJl0SfY8Hesb9Ki5Ixc8wLlM/ilL6Ay05B2nfFNlyDu8YuwjogvsWnbZJeAnySQRqfCUwG62pE9OcVcyPeO3eVMop72h7c6T30JdaqtCrVH+QD5ngbfXVy0usFoNtXmVQewakyy5StDrJ3Ok88RJzs1rao8OYAxRxzZJVaQIDAQAB
```


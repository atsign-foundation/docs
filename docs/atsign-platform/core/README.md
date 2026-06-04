---
description: An overview of Atsign's core pillars of technology
icon: layer-group
---

# Atsign Platform Overview

## TL;DR

Atsign Platform allows people, entities and things to communicate privately and securely without having to know about the intricacies of the underlying IP network. The Atsign Protocol is the application protocol used to communicate, and Atsigns are the addresses on the protocol. All cryptographic keys are cut at the edge by the Atsign owner, meaning only the receiving and sending Atsigns see data in the clear.

Atsign Platform can be used to send data synchronously or asynchronously, and can be used as a data plane, or a control plane, or both, simultaneously at Internet scale.

<figure><picture><source srcset="../../.gitbook/assets/Atsign Platform dark.svg" media="(prefers-color-scheme: dark)"><img src="../../.gitbook/assets/Atsign Platform light.svg" alt="Diagram of Atsign&#x27;s Core Technology"></picture><figcaption></figcaption></figure>

<details>

<summary>Relationships</summary>

Every **atServer** is associated with _one_ **Atsign**, and each atServer stores _many_ **atRecords.**

When provided an **Atsign**, the **atDirectory** returns a _DNS address_ and _port number_ for its **atServer.**

The **Atsign Protocol** is the _application layer protocol_ used to communicate with an **atServer.**

</details>

## atServer

An atServer is both a personal data service for storing encrypted data owned by an Atsign, and a rendezvous point for information exchange. An atServer is responsible for the delivery of encrypted information to other atServers, from which the owners of those Atsigns can then retrieve the data.

{% hint style="info" %}
Unless explicitly made public, atServers only store encrypted data and do not have access to the cryptographic keys, nor the ability to decrypt the stored information.
{% endhint %}

<details>

<summary>atServer Functionality</summary>

* Cryptographic authentication of client devices.
* Cryptographic authentication of other atServers.
* Persistence of encrypted data on behalf of the controlling Atsign.
* Caching of data shared by others with the controlling Atsign.
* Notification of data change events to clients (edge devices) and other atServers to facilitate delivery of information shared with them.
* Synchronization of data with multiple clients (edge devices).
* TLS wire encryption from clients to atServers using SSL certificates.
* Mutually authenticated TLS 1.2/1.3 wire encryption between atServers using SSL certificates.

</details>

## atDirectory

In order for an Atsign to communicate with another one on the internet, we need to locate the atServer that can send and receive information securely on its behalf.

The location of an atServer is found using the atDirectory service (`root.atsign.org:64`). This directory returns the DNS address and port number of the atServer for any Atsign that it has a record for. The atDirectory service contains no information about the owner of the Atsign.

## Atsign Protocol

{% hint style="info" %}
The Atsign Protocol communicates via layer 7, the application layer of the OSI model, over TCP/IP.
{% endhint %}

The Atsign Protocol is an application protocol that enables data sharing between Atsigns. You can learn more about the Atsign Protocol by reading the [specification](https://github.com/atsign-foundation/at_protocol/blob/trunk/specification/at_protocol_specification.md). The Atsign Protocol uses TCP/IP and TLS but does not specify how data itself is encrypted, that is the job of the atSDK and atClient libraries.

## atSDK

atSDKs provide developers with Atsign Platform specific building tools in a number of languages and for a number of operating systems and hardware. The atSDK allows developers to rapidly develop applications that use Atsign Platform.

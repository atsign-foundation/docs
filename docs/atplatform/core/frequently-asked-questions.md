---
icon: question
---

# Frequently Asked Questions

## Frequently Asked Questions

This page answers common questions about the AtPlatform: how Atsigns, AtServers, and the AtDirectory work together to provide end-to-end encrypted, identity-first communication.

### Atsign

#### What is an Atsign?

An Atsign (e.g. `@alice`) is a resolvable address, similar in concept to an email address. Anyone or anything can have an Atsign — people, organizations, AI agents, and devices (IoT sensors, gateways, routers, etc.).

#### What is an Atsign used for?

An Atsign is used to securely exchange information with other Atsigns without risk of surveillance, impersonation, or data theft. The exchange can be a pure data exchange, or it can trigger an action on the receiving system.

#### What characters can be used to form an Atsign?

Atsigns can be made of any combination of UTF-7 characters up to 55 characters in length, provided they are not already registered. This provides a name space of roughly 10^224 Atsigns. Some combinations may be unavailable or restricted (e.g. trademarked names, already registered Atsigns).

### AtServers

#### What is an AtServer?

An AtServer is a private datastore for encrypted data owned by a single Atsign, and a rendezvous point for information exchange. It is responsible for delivering encrypted information to other AtServers, where the recipient can then retrieve it. **An AtServer only stores encrypted information and never has access to the cryptographic keys.**

#### What functions does an AtServer perform?

An AtServer provides:

* Cryptographic authentication of client devices
* Cryptographic authentication of other AtServers
* Persistence of encrypted data on behalf of the controlling Atsign
* Caching of data shared by others with the controlling Atsign
* Notification of data change events to clients and other AtServers
* Monitoring of notifications from other AtServers
* Synchronization of data across multiple clients (edge devices)
* TLS wire encryption from clients to AtServers
* Mutually authenticated TLS 1.2/1.3 wire encryption between AtServers

#### What does an AtServer consist of?

An AtServer is a small dockerized service that can be deployed almost anywhere. It requires persistent storage and publicly addressable network connectivity. Each Atsign gets its own isolated AtServer instance.

This distributed, single-tenant architecture means there is no centralized service mixing data from many Atsigns together — unlike traditional multi-tenant web services that present an attractive target for hackers.

#### Where are AtServers hosted?

There are two options:

* **Atsign-hosted AtServers** — Run by Atsign in the cloud with automated deployment and maintenance. Atsign cannot see private data because it is encrypted with keys Atsign never has access to.
* **Self-hosted AtServers** — Run by the Atsign owner on platforms ranging from a Raspberry Pi to GCP, AWS, or on-premises infrastructure.

#### How does authentication work with an AtServer?

The very first authentication uses a shared secret (a "cram key") (CRAM stands for Challenge Response Authentication Mechanism). After that, an RSA-2048 or ECC key pair is generated and used for all subsequent authentication via PKAM (Public Key Authentication Mechanism).

To create additional keys that grant access to the same AtServer with namespace restrictions, you can use APKAM (Application Public Key Authentication Mechanism). APKAM allows individual applications to authenticate with their own keypair, scoped to only the namespaces they need.

#### How do AtServers protect the data they hold?

Each AtServer holds data for a single owner, which limits the appeal of an attack. If an attacker did gain access to an AtServer's datastore, all private data is encrypted with cryptographic keys that the AtServer does not hold. It is therefore _provably true_ that the data cannot be decrypted from the AtServer alone — unlike systems that rely on policy and procedure to safeguard centralized data.

#### What is the size of an AtServer? How much can it store?

An AtServer can store as much data as its underlying storage system allows. For Atsign-hosted AtServers, the default configuration allocates 50MB of memory to the docker container, which is sufficient for most Atsigns representing people or things. Busier Atsigns (e.g. for companies) can be allocated more CPU and RAM. Contact us at support@atsign.com

### AtDirectory

#### What does "resolvable address" mean for an Atsign?

For an Atsign to communicate with another, it needs to locate the AtServer that handles its traffic. This lookup is done through the AtDirectory service (`root.atsign.org`), which returns the DNS address and port number of the AtServer for any Atsign it has a record for. The AtDirectory contains no information about the owner of the Atsign.

### Edge Devices

#### What is an edge device and what is edge-to-edge encryption?

An edge device is any device located at the edge of a network that produces or consumes data — smartphones, personal computers, servers, IoT devices, gateways, etc.

Edge-to-edge encryption is an extension of end-to-end encryption: data is encrypted at all times during transmission and is only decrypted on edge devices. Atsigns store their cryptographic keys exclusively on the edge device. The AtDirectory is never used to route personal data — it is only used for DNS lookups. AtServers transmit, receive, and store encrypted data without any access to the keys needed to decrypt it. Only data that has been intentionally made public is stored in the clear.

#### How does the AtPlatform eliminate network attack surfaces from devices?

All connections from an AtPlatform-enabled device are _outbound_ to its AtServer. Inbound communications must be encrypted and sent to the device's AtServer; the device picks them up the next time it connects (or immediately, if already connected).

This means edge devices have no need for open listening ports, no need for a known IP address, and no need for direct network reachability. The result: no network attack surface.

#### Does the AtPlatform require static IPs?

No. Because all connections are outbound from the edge device to its AtServer, device IP addresses can change freely. AtServers themselves do have static DNS addresses.

#### How are devices addressable when no ports are open?

The AtPlatform provides bidirectional communication originating outbound from the device. The device is reachable as long as it has internet access — no port forwarding required, even behind firewalls or NAT.

To send a message to a device, you send it to that device's AtServer. The AtServer authenticates you as a permitted sender and notifies the device. If the device is connected, it receives the message immediately; if not, it receives it as soon as it next connects.

#### Does the AtPlatform require passwords?

No. Authentication is mutual, cryptographic, and does not depend on shared secrets like passwords between endpoints. This eliminates the entire class of credential-compromise attacks against centralized credential stores.

#### What is the latency?

In a typical case where `@atsign_1` sends an end-to-end-encrypted message to `@atsign_2`, once the socket connections are established, end-to-end latency excluding speed-of-light is typically between 4 and 15 milliseconds — including encryption, decryption, and the AtServer's store-and-forward work.

#### What hardware and operating systems are supported?

**Processors**

* x64
* ARM (ARMv7, ARMv8, M1/M2 Apple Silicon)
* RISC-V (Beta)
* Older or more exotic CPUs are supported via the C atSDK

**Operating Systems**

* Linux
* macOS
* Windows
* iOS
* Android

**Other**

* Microcontrollers via the C atSDK

#### Will this work on very low-power hardware?

Yes, provided there is a microcontroller with enough RAM to handle crypto operations. Extensive testing has been done on Raspberry Pi Pico (192kB RAM) and various ESP32-based platforms. 128kB MCUs are likely workable; below that, an additional processor is generally needed.

### Encryption

#### Why does the AtPlatform use encryption?

To make it _provably true_ that data is accessible only to the intended parties. Two properties make this work:

1. Asymmetric cryptographic keys are generated and kept on the edge device where the data is created.
2. Symmetric encryption keys used to share data are themselves encrypted with those asymmetric keys, then exchanged via the atProtocol.

This means only the creator and the intended recipient ever have the keys to decrypt the data. No infrastructure operator — including Atsign — can access private data.

#### What encryption algorithms are used?

The AtPlatform uses both symmetric and asymmetric encryption. Currently:

* **Symmetric**: AES-256 (used to encrypt the actual data, including data streams)
* **Asymmetric**: RSA-2048 and ECC (used for authentication, data signing, and exchanging symmetric keys)

The protocol is designed to be extensible, so additional algorithms (e.g. post-quantum) can be added as needed.

#### How is data transmitted between two Atsigns?

Suppose Alice (`@alice`) wants to share data with Bob (`@bob`):

1. Alice's device generates a new AES key and encrypts the data with it.
2. Alice's device stores the encrypted data on her AtServer.
3. Alice's device encrypts the AES key using Bob's RSA public key, and stores that encrypted key on her AtServer too.
4. Alice's AtServer notifies Bob's AtServer that data is available.
5. Bob's device retrieves the encrypted AES key, decrypts it with his RSA private key, then uses the AES key to decrypt the data.

#### What types of cryptographic keys does the AtPlatform manage?

**Authentication**

* `pkamPublicKey` / `pkamPrivateKey` (asymmetric) — public key held on the AtServer, private key held only on the edge. The technology supports holding the private key in a secure element and delegating sign/encrypt/decrypt operations to it.

**Asymmetric encryption**

* `encryptionPublicKey` / `encryptionPrivateKey` — public key held on the AtServer. When one Atsign needs to share a symmetric key with another, it retrieves the recipient's encryption public key, encrypts the symmetric key with it, and sends it. This keypair is also used for data signing and signature verification.

**Symmetric encryption**

* Symmetric keys are used to encrypt the actual data being exchanged. They are securely exchanged using the asymmetric encryption keypairs above.
* Atsigns also generate symmetric keys to encrypt data they want to store privately for themselves.

#### How are these keys protected?

Asymmetric keypairs are generated on the edge device, and private keys never leave it. There is no centralized key management. Ideally, keys are generated and held in a secure enclave (TPM, SIM, etc.), with private keys isolated from other forms of access.

#### What happens if I lose my keys?

Atsign cannot retrieve lost keys — they were generated on your device and Atsign never had access to them. However, you still own your Atsign and can reset it by contacting `support@atsign.com`. You can then generate a new keyset and continue using the same Atsign.

#### How are stolen or compromised devices handled?

There are several options depending on the use case:

1. The Atsign can be reset on the `my.atsign.com` portal, invalidating any compromised keys.
2. Keys can be cut in hardware (SIM or TPM) so the private key is never exposed directly.
3. Keys can be exported and imported across devices. (This is being phased out in favor of a model where each Atsign supports multiple keys across multiple devices, with a master key that can revoke access for any individual device — protected by a second factor such as biometrics.)

#### Are there any other security features?

In addition to end-to-end encryption of private data, TLS validates the authenticity of AtServers and encrypts wire traffic between clients and their AtServer. Communication between AtServers is mutually authenticated and TLS-encrypted.

### Data Model

#### What is the data exchange and persistence model?

The atProtocol's default persistence model is a simple key-value store. Record IDs have a syntactical structure that is also used in the protocol exchanges between clients and AtServers.

The record ID structure is:

```
[cached:]<visibility scope>:<entity ID>.<namespace>@<owner Atsign>
```

Record IDs are always stored in lowercase.

**Visibility scope** — defines who can see and access the data:

* **Public** (e.g. `public:location.some_app@alice`) — available to anyone, surfaces in unauthenticated AtServer scans, and is not encrypted.
* **Private** (e.g. `@bob:phone.some_app@alice`) — shared privately with a specific Atsign. Only shows up in scans for the owner and recipient after authentication, and can only be decrypted by the owner and recipient.

**Entity ID and namespace** (e.g. `@bob:work.email.an_app@alice`):

* Entity IDs can be any alphanumeric string. Because record IDs are stored in lowercase, snake\_case is recommended (`home_phone_number`, not `homePhoneNumber`).
* The namespace (`an_app` above) controls which applications can read, write, and synchronize a given record.

**Owner Atsign** — indicates which Atsign created the record.

**Cached prefix** (e.g. `cached:@bob:phone.buzz@alice`) — indicates the record was created by another Atsign, encrypted and shared with you, with metadata permitting it to be cached on your datastore. This enables offline support and improves performance.

#### Can I share different subsets of a device's data with different parties?

Yes. You can configure a device so that different datasets are sent to different recipients, each uniquely encrypted for the intended recipient.

### Integration

#### Can I integrate the AtPlatform with existing cloud and IoT platforms?

Yes. The AtPlatform is powered by the open atProtocol and provides SDKs for integration with most stacks. There are open-source libraries, widgets, and connectors available on [Atsign's GitHub organization](https://github.com/atsign-foundation). The default cloud is GCP, but the platform is compatible with any cloud or on-premises infrastructure.

#### How do I assign an Atsign to a device?

There are several options:

* Assigned manually via a physical connection or OTA update.
* Pre-assigned by the manufacturer of the device or SIM.

For IoT devices, a best practice is to define separate Atsigns for administration and data ownership. This separation makes it easy to keep device administration distinct from data ownership and privacy.

#### Can I change the Atsign on a device?

It depends. If the Atsign is embedded in an eSIM/iSIM, it generally cannot be changed. Otherwise it is possible — but in practice, it is more common to change the _designated owner_ of a device (e.g. when the device is sold) rather than the Atsign itself.

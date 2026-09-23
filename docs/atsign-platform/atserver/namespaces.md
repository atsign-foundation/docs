---
description: >-
  Learn about namespaces in the platform and what aspects of the platform
  concern them most
icon: signature
---

# Namespaces

## What are namespaces?

Namespaces help keep applications and different message types separate. They are used in many places like [atrecords.md](atrecords.md "mention"), [events.md](events.md "mention") and [enrollments.md](../atsign/enrollments.md "mention"). Find out where they're used in [#namespace-footprint](namespaces.md#namespace-footprint "mention").

## Namespace examples

There are two types of namespaces: [#application-namespaces](namespaces.md#application-namespaces "mention") and [#hierarchical-namespaces](namespaces.md#hierarchical-namespaces "mention").

### Application namespaces

Useful for keeping all of your messages under one large namespace. These namespaces are Atsigns that you should own (like `@wavi`) to validate the ownership of your namespace.

Examples are:

* `wavi`
* `noports`
* `sshrvd`
* `sshnp`
* `my_app`

### Hierarchical namespaces

Hierarchical namespaces are generally used for sorting AtRecords and events, and are more specific than application namespaces.

Examples are:

* `contacts.my_app`
* `group_2.groups.policy`  and `group_1.groups.policy`

They ascend in specificity from left to right. In these examples, you may hold all your contact AtRecords under the `contacts.my_app` namespace. Similarly, you may have all the group policy rules related to the group name "group\_2" under the `group_2.groups.policy`.&#x20;

## Reserved namespaces

Namespaces starting with `__` belong to the SDK and atServer operations. Never write to them directly. Find out where they're used in [#namespace-footprint](namespaces.md#namespace-footprint "mention").

<table data-search="false"><thead><tr><th>Namespace</th><th>Purpose</th></tr></thead><tbody><tr><td><code>__manage</code></td><td>Enrollment management. <code>rw</code> here means "can approve enrollments"</td></tr><tr><td><code>__pkams</code></td><td>PKAM key material</td></tr><tr><td><code>__global</code></td><td>Cross-namespace scope</td></tr><tr><td><code>__rpcs</code></td><td><code>AtRpc</code> request/response traffic</td></tr><tr><td><code>__shared_keys</code></td><td>Symmetric keys shared with other Atsigns</td></tr><tr><td><code>__public_keys</code></td><td>Additional encryption public keys</td></tr><tr><td><code>__self_keys</code></td><td>Additional self encryption keys</td></tr><tr><td><code>__rr</code></td><td><code>AtCollection</code> read receipts</td></tr><tr><td><code>*</code></td><td>In <a data-mention href="../atsign/enrollments.md">enrollments.md</a>, this means "all namespaces"</td></tr></tbody></table>

## Namespace footprint

Namespaces have impacts in many places such as:

* [#reserved-namespaces](atrecords.md#reserved-namespaces "mention") — you cannot assign certain namespaces to your AtKeys
* [#reserved-namespaces](events.md#reserved-namespaces "mention") — you cannot assign certain namespaces to your notifications
* [#namespace-scoped-permissions](../atsign/enrollments.md#namespace-scoped-permissions "mention") — certain namespaces are important to keep note of when enrolling

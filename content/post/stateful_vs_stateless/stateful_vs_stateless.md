---
title: "Firewalls: stateful vs. stateless"
description: "The key differences explained in simple terms"
date: 2023-07-24T19:56:52-07:00
draft: false
tags: ["good to know"]
categories: ["networking"]
---

Whether it's an on-prem firewall or in the cloud, it's important to understand the fundamental distinctions in firewall types. In this post, I will summerize the differences between stateful and stateless filtering in simple and basic terms.

## Stateful filtering

Stateful filtering, is a firewall technique that tracks the state of network connections and makes decisions (such as allow/deny) based on the context of those connections. When a packet passes through a stateful firewall, the firewall keeps track of the state of the connection by creating an entry in a state table to monitor the connection's status.

The state table contains information such as:

- Source's IP address and port
- Destination's IP address and port
- Connection status (e.g. established, new, related, or invalid).

By maintaining this state information, the firewall can **allow inbound traffic** that is part of an outbound connection initiated from within the network. This "allow rule" does not have to be explicitly configured in the firewall to permit traffic from the destination back to the source. That's whay they're called "context-aware", because this type is designed to understand the context of the conntection, where it's coming from, where it's going to, and who generated the connection.

Imagine your application sending a request, and when the destination sends the response back, the firewall just goes, 'Oh, I've got this! It's coming from a destination that a client from my network initiated the connection with. No need to have the destination IP in the allow-list; we're all good to go!'.

The firewall will continue blocking traffic that does not **correspond to an existing connection** or violates the state table rules. Security Groups on major cloud providers typically operate in a stateful manner.

### Advantages of Stateful Filtering

- Simplicity in implementation: No need to add an allow rule for the destination address if the connection originated from a client within the network.
- Improved security: Only allows traffic from established connections within our network.

## Stateless Filtering

Stateless filtering, is a firewall technique that examines individual packets without considering the context or state of the connection to which they belong. Each packet is analyzed independently based on predefined filtering rules. If a client within the network sends a request to a remote destination, the destination address must be on the firewall's allow list, otherwise the client won't receive a response.

Think of it like the customs kiosks at an airport in some country. They check arrivals and departures for travelers separately. The passengers' origin does not generally impact their work rules. All they know is whether a passenger is arriving or departing and whether they are permitted to enter or leave the country based on predefined rules.

In stateless filtering, the firewall evaluates each packet's headers, such as source and destination IP, ports, and protocol type. If a packet matches one of the preconfigured rules, it is allowed or denied based on that rule's criteria.

Stateless firewalls do not maintain a state table to track connection states, meaning they do not know established connections. As a result, they cannot dynamically allow response traffic for outgoing connections or handle traffic related to established connections.

In AWS, Network Access List (NACLs) operate in a stateless manner.

### Advantages of Stateless Filtering

- Efficiency. Each packet is evaluated in isolation without the overhead of maintaining a state table.
- Useful for basic traffic filtering and access control based on packet headers.

## Scope of implementation

The fundamental concepts of stateful and stateless firewalls are consistent across firewall appliance manufacturers as well as cloud providers. However, the implementation of stateless and stateful firewall functionality (i.e. how they do it) can vary.

For example, in AWS, stateful firewall functionality is implemented using Security Groups, which operate at the instance level. And stateless firewall functionality can be achieved through Network ACLs (Access Control Lists - NACLs), which operate at the subnet level.

## Comparison

- Stateful filtering is more sophisticated and provides better security by considering the state of connections. It can dynamically allow responses to outgoing traffic and handle related traffic.
- Simplicity in usage: they're good when simpler filtering rules based on static rules are sufficient.

## Finally

In practice, firewalls often utilize both stateful and stateless filtering at various levels, such as subnet, or gateway, to provide complete security. Stateful filtering is commonly used for connection tracking, while stateless filtering allows for quick packet evaluation.

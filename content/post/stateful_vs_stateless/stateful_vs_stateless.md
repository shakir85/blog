---
title: "Firewalls: stateful vs. stateless"
description: "Explained in simple terms to help you make informed decisions for your security strategy"
date: 2023-07-24T19:56:52-07:00
draft: false
tags: ["good to know"]
categories: ["networking"]
---

Within the realm of firewalls, two major concepts are prominent: **stateless** and **stateful** firewalls. These two distinct approaches serve different purposes, and the decision of whether to implement stateful or stateless firewall roles depends on your specific needs and objectives.

Whether in your data center or in the cloud, it's crucial to grasp the fundamental distinctions in firewall types. In this post, I will summerize the differences between stateful and stateless filtering in simple and basic terms.

## Stateful filtering

Stateful filtering, is a firewall technique that tracks the state of network connections and makes decisions based on the context of those connections. When a packet passes through a stateful firewall, the firewall keeps track of the state of the connection to which the packet belongs. It creates an entry in a state table to monitor the connection's status.

The state table contains information such as:

- Source IP address an port
- Destination IP address and port
- Source port
- Connection status (e.g. established, new, related, or invalid).

By maintaining this state information, the firewall can **allow inbound traffic** that is part of an established connection initiated from within the network without having to explecitly ask the firewall to allow traffic from the destination back to source.

Imagine your application sending a request, and when the destination sends the response back, the firewall just goes, 'Oh, I've got this! It's coming from a destination that a client from our network initiated the connection with. No need to add the destination address to the allow-list; we're all good to go!

The firewall will continue blocking traffic that does not correspond to an existing connection or violates the state table rules. As an example, Security Groups on major cloud providers typically operate in a stateful manner.

### Advantages of Stateful Filtering

- Provides improved security by allowing only traffic that belongs to established connections from the network.
- Can dynamically handle and allow responses to outgoing traffic initiated from the internal network.

## Stateless Filtering

Stateless filtering, is a firewall technique that examines individual packets without considering the context or state of the connection to which they belong. Each packet is analyzed independently based on predefined filtering rules.

Think of it as the customs kiosks at an airport in some country. They check the arrivals/departures travelers separately. They do not care where the passenger is from (generally!). All they know is that a passenger is arriving/departing and whether or not they are permitted to enter/leave the country based on pre-defined rules.

In stateless filtering, the firewall evaluates each packet's headers, such as source and destination IP addresses, source and destination ports, and protocol type. If a packet matches one of the preconfigured rules, it is allowed or denied based on that rule's criteria.

Stateless firewalls do not maintain a state table to track connection states, meaning they do not know established connections. As a result, they cannot dynamically allow response traffic for outgoing connections or handle traffic related to established connections.

In AWS, Network Access List (NACLs) operate in a stateless manner.

### Advantages of Stateless Filtering

- Simplicity and efficiency, as each packet is evaluated in isolation without the overhead of maintaining a state table.
- Useful for basic traffic filtering and access control based on packet headers.

## Comparison

- Stateful filtering is more sophisticated and provides better security by considering the state of connections. It can dynamically allow responses to outgoing traffic and handle related traffic.
- Stateless filtering is simpler and faster but cannot track connection states, making it less effective in handling complex scenarios.

## Finally
In practice, modern firewalls often use stateful and stateless filtering techniques to provide comprehensive security and performance. They use stateful filtering for connection tracking and stateless filtering for quick packet evaluation.

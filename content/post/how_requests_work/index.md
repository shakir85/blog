---
title: "How requests go from your app to the internet?"
description: "A simplified explanation of the journey from 'Request' to 'Content'"
date: 2024-01-29T08:34:22-07:00
categories: ["networking"]
tags: ["grasping essentials"]
draft: true
---

Ever wondered what happens when you type a website and hit enter or send a request through your app? It's a classic job interview question that can sometimes leave the interviewee feeling a bit uneasy.

In this post, I will explain what occurs when we request something from the internet in a very simplified way. I'll focus on the networking side, keeping it simple and straightforward. I'll intentionally avoid the DNS shenanigans, as it involves multiple components.

## Step 1: Source port selection

When an application (a process) initiates a request, the operating system will select a source port for the outgoing communication. This source port is usually selected from the ephemeral port range (49152–65535) and is used to uniquely identify the communication session. For example, the source port could be 45956, as indicated in the above diagram. So, the request will contain the server's address, where the process is running on, and the port number which will play critical role in the next steps.

## Step 2: Destination port on outbound request

The destination port in the outbound request is the port that the server is listening on. In our example, it's port 443 since it's a HTTPS request. The combination of the destination IP address (172.4.66.3) and destination port (443) identifies the specific service on the server that the client is trying to reach.

## Step 3: Router Network Address Translation (NAT)

When the traffic reaches the gateway (typically a router or firewall), NAT comes into play. The gateway replaces the source IP and port with its own public IP and a new port number. This is done to allow multiple devices within the internal network to share the same public IP address. This is what NAT does in essence. So, in our example, the source IP becomes the gateway's public IP (11.22.33.44), and a new source port (e.g., 1189) is assigned.

## Step 4: Response handeling

Now the request has been sent from our network. Then when the server responds, it sends the response to the public IP and port of our gateway (11.22.33.44:1189). The gateway, based on its NAT table, knows which internal device (identified by the original source IP and port) should receive this response. The gateway then forwards the response to the appropriate internal IP and port.

## Step 5:Port Forwarding

This mechanism of associating an incoming response with the correct internal client based on the port number is often referred to as port forwarding. The gateway keeps track of the translations it performed and uses this information to forward responses to the correct internal device.

Finally the operating system, based on the port number coming in the response, will be able to hand the data load back to the service.

## Caviate

Keep in mind that specific network configurations, security, and routing roles might introduce variations on how internal traffic is routed.

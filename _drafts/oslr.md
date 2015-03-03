---
title: Optimized Link State Routing (OLSR) Protocol
layout: post
date: 2015-02-12
author: James Keener
---

OLSR, as defined in [RFC 7181](tools.ietf.org/html/rfc7181) is an example
of a [Link-State routing protocol](http://en.wikipedia.org/wiki/Link-state_routing_protocol).
designed for [Mobile Ad Hoc Networks (MANETs)](http://en.wikipedia.org/wiki/Mobile_ad_hoc_network)
and as such has to deal with a network topology that is expected to change
often and the devices attached to the network may likly have constrained
processing capabilites and power usage.

Due to the changing topology, MANETs should be self-forming and
self-healing. Self-forming means that no central configuration should be
required. Self-healing means that if a node leaves the network, packets
that were being routed by it should be directed elsewhere without error.

*Steps when first joining*

*Steps to chosing MPR*

*Finding out about new and lost peers*


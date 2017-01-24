---
layout: post
title:  "Configuring a basic IPv6 network using IPv6 neighbor discovery (autoconfig)"
date:   2017-01-24 11:30:12 +0100
categories: ccie ipv6 nd
---

# Basic network topology

![Basic IPv6 Topology]({{ site.url }}/assets/Basic-IPv6-Lab-With-ND-Topology.jpg)

# Autoconfig router configuration

By default, Cisco router advertisse all the prefixes configured on the interface. Configuring the autoconfig server is just as simple as configuring an IPv6 address.

{% highlight apache %}
R1#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R1(config)#ipv6 unicast-routing
R1(config)#
R1(config)#interface Ethernet0/0
R1(config-if)# ipv6 address 2001:1234:5678::1/64
R1(config-if)#!

{% endhighlight %}

# Autoconfig client configuration 

On the autoconfig clients, the IPv6 address is configured dynamically.

{% highlight apache %}
R3(config)#ipv6 unicast-routing
R3(config)#
R3(config)#interface Ethernet0/0
R3(config-if)# ipv6 address autoconfig
R3(config-if)#!

{% endhighlight %}

# Verification

## on R1

{% highlight apache %}
R1#sh ipv6  int eth0/0
Ethernet0/0 is up, line protocol is up
  IPv6 is enabled, link-local address is FE80::A8BB:CCFF:FE00:100
  No Virtual link-local address(es):
  Global unicast address(es):
    2001:1234:5678::1, subnet is 2001:1234:5678::/64
{% endhighlight %}


## on R3

{% highlight apache %}
R3#sh ipv6  int eth0/0
Ethernet0/0 is up, line protocol is up
  IPv6 is enabled, link-local address is FE80::A8BB:CCFF:FE00:300
  No Virtual link-local address(es):
  Stateless address autoconfig enabled
  Global unicast address(es):
    2001:1234:5678:0:A8BB:CCFF:FE00:300, subnet is 2001:1234:5678::/64 [EUI/CAL/PRE]
      valid lifetime 2591871 preferred lifetime 604671
{% endhighlight %}


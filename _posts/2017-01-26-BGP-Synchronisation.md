---
layout: post
title:  "BGP Synchronisation and internal redistribution"
date:   2017-01-26 8:30:12 +0100
categories: ccie bgp
---

# Summary

**BGP synchronisation** forces the bgp process to accept a prefix received from a ibgp peer only if an igp already installed a route for the same prefix in the routing table.



# Basic Topology


![Basic BGP Topology](/assets/BGP-Synchronisation-1.png)


# Issue

Let see what happens in the routing table of R4 regarding the route to reach the Loopback of R1 (1.1.1.1)

## Without redistribution of BGP inside IGP


### With synchronisation off(default)

Without synchronisation we can see the prefix 1.1.1.1 is accepted by the bgp process of R4 and market as **best**

{% highlight text%}
R4#sh ip  bgp 1.1.1.1
BGP routing table entry for 1.1.1.1/32, version 2
Paths: (1 available, best #1, table default)
  Advertised to update-groups:
     8
  Refresh Epoch 1
  65001
    2.2.2.2 (metric 21) from 2.2.2.2 (2.2.2.2)
      Origin IGP, metric 0, localpref 100, valid, internal, best
      rx pathid: 0, tx pathid: 0x0

{% endhighlight %}

And we can see the prefix is also installed in the routing table of R4 and forwarded to R5

{% highlight apache %}

R4#sh ip  route 1.1.1.1
Routing entry for 1.1.1.1/32
  Known via "bgp 65234", distance 200, metric 0
  Tag 65001, type internal
  Last update from 2.2.2.2 00:00:12 ago
  Routing Descriptor Blocks:
  * 2.2.2.2, from 2.2.2.2, 00:00:12 ago
      Route metric is 0, traffic share count is 1
      AS Hops 1
      Route tag 65001
      MPLS label: none

R5#sh ip route 1.1.1.1
Routing entry for 1.1.1.1/32
  Known via "bgp 65005", distance 20, metric 0
  Tag 65234, type external
  Last update from 192.168.45.4 00:12:19 ago
  Routing Descriptor Blocks:
  * 192.168.45.4, from 192.168.45.4, 00:12:19 ago
      Route metric is 0, traffic share count is 1
      AS Hops 2
      Route tag 65234
      MPLS label: none

{% endhighlight %}


But we have a big reachability issue. R3, in the core of the OSPF area doesn\'t know how to route this packets, blackholing all the packet.



{% highlight text %}
R3#sh ip route 1.1.1.1
% Network not in table
{% endhighlight %}


### With synchronisation on

{% highlight text %}
R4#conf t
Enter configuration commands, one per line.  End with CNTL/Z.
R4(config)#router bgp 65234
R4(config-router)# synchronization
R4(config-router)# ex
R4# clear ip bgp *
{% endhighlight %}


When we activate  bgp synchronisation, the prefix 1.1.1.1 is refused by the bgp process of R4, marked as **not synchronized**, not injected to the routing table and not forwarded to R5.


{% highlight text%}
R4#sh ip  bgp 1.1.1.1
BGP routing table entry for 1.1.1.1/32, version 0
Paths: (1 available, no best path)
Flag: 0x820
  Not advertised to any peer
  Refresh Epoch 1
  65001
    2.2.2.2 (metric 21) from 2.2.2.2 (2.2.2.2)
      Origin IGP, metric 0, localpref 100, valid, internal, **not synchronized**
      rx pathid: 0, tx pathid: 0
R4#sh ip  route 1.1.1.1
% Network not in table
R5#sh ip  route 1.1.1.1
% Network not in table
{% endhighlight %}



## Reinjecting the BGP routes inside the IGP.


If we reinject the route to 1.1.1.1 inside de OSPF area 0, R4 learns the route to reach it.

{% highlight apache %}

R4#sh ip route 1.1.1.1
Routing entry for 1.1.1.1/32
  Known via "ospf 1", distance 110, metric 1
  Tag 65001, type extern 2, forward metric 30
  Last update from 192.168.34.3 on Ethernet0/1, 00:10:18 ago
  Routing Descriptor Blocks:
  * 192.168.34.3, from 2.2.2.2, 00:10:18 ago, via Ethernet0/1
      Route metric is 1, traffic share count is 1
      Route tag 65001

{% endhighlight %}

And we can see that the prefix is now accepted by the BGP process and marked as **synchronized, best**

{% highlight apache %}

R4#sh ip bgp 1.1.1.1
BGP routing table entry for 1.1.1.1/32, version 2
Paths: (1 available, best #1, table default, RIB-failure(17))
  Advertised to update-groups:
     10
  Refresh Epoch 1
  65001
    2.2.2.2 (metric 21) from 2.2.2.2 (2.2.2.2)
      Origin IGP, metric 0, localpref 100, valid, internal, synchronized, best
      rx pathid: 0, tx pathid: 0x0


{% endhighlight %}

And announced to R5

{% highlight text %}

R5#sh ip route 1.1.1.1
Routing entry for 1.1.1.1/32
  Known via "bgp 65005", distance 20, metric 0
  Tag 65234, type external
  Last update from 192.168.45.4 00:20:41 ago
  Routing Descriptor Blocks:
  * 192.168.45.4, from 192.168.45.4, 00:20:41 ago
      Route metric is 0, traffic share count is 1
      AS Hops 2
      Route tag 65234
      MPLS label: none

{% endhighlight %}

## Conclusion

bgp synchronisation is a method to limit the injection of incorrectly routed traffic to external peer.

In real network, its not used because synchronizing the ebgp and igp routes is not applicable in most scenario.






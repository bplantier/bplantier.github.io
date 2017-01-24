---
layout: post
title:  "IPv6 Last Hope Security"
date:   2017-01-20 11:30:12 +0100
categories: ccie ipv6 security
---

# IPv6 First Hop Security

IPv6 addressing is continously growing and replacing legacy IPv4 in campus and datacenter.
This new method of addressing devices also introduces new challenges regarding network security.

Several mechanism are available to address those challenges:

# IPv6 Prefix Glean / Destination Glean

Inspect ND and DHCP messages.
Populates binding table

# IPv6 Snooping

{% highlight ruby %}
Router(config)# ipv6 snooping policy policy1
Router(config-ipv6-snooping)#?
IPv6 snooping policy configuration mode:
  data-glean         binding recovery by data traffic source address gleaning
  default            Set a command to its defaults
  destination-glean  binding recovery by data traffic destination address
                     gleaning
  exit               Exit from snooping policy configuration mode
  limit              Specifies a limit
  no                 Negate a command or set its defaults
  prefix-glean       Glean prefixes in RA and DHCP-PD traffic
  protocol           Sets the protocol to glean (default all)
  security-level     setup security level
  tracking           Override default tracking behavior
  trusted-port       setup trusted port
Router(config-ipv6-snooping)#security-level ?
  glean    Glean addresses
  guard    inspect and drop un-authorized messages (default)
  inspect  glean and Validate message
Router(config-ipv6-snooping)# exit
Router(config)# interface Gigabitethernet 0/0/1
Router(config-if)# ipv6 snooping attach-policy policy1
. 
.
.
Device# show ipv6 snooping policies interface gigabitethernet 0/0/1
Target               Type  Policy               Feature        Target range
Gi0/0/1              PORT  my_policy            Destination Gu vlan all
Gi0/0/1              PORT  policy1              Snooping       vlan all
{% endhighlight %}



# IPv6 Source Guard / Prefix Guard

{% highlight apache %}
Router(config)#ipv6 source-guard policy source01
Router(config-sisf-sourceguard)#?
IPv6 sourceguard policy configuration mode:
  default   Set a command to its defaults
  deny      Block data traffic
  exit      Exit from sourceguard policy configuration mode
  no        Negate a command or set its defaults
  permit    Allow data traffic
  trusted   Bridge all data traffic
  validate  Validate the src of the received data traffic
{% endhighlight %}


# IPv6 Destination Guard

{% highlight apache %}
Router> enable
Router# configure terminal
Router(config)# ipv6 destination-guard policy destination01
Router(config)# interface GigabitEthernet 1
Router(config-if)# ipv6 destination-guard attach-policy destination01

Router# show ipv6 destination-guard policy destination01
Destination guard policy Destination: 
  enforcement always
        Target: Gi1  
{% endhighlight %}


# IPv6 RA Guard 

Not avalaible on routers.

{% highlight apache %}

Switch(config)#ipv6 nd raguard policy rapolicy
Switch(config-nd-raguard)#?
IPv6 RA guard policy configuration mode:
  default              Set a command to its defaults
  device-role          Sets the role of the device attached to the port
  exit                 Exit from RA guard policy configuration mode
  hop-limit            Enable verification of the advertised Hop count limit
  managed-config-flag  Enable verification of the advertised M flag
  match                Match a particular prefix-list or access-list
  no                   Negate a command or set its defaults
  other-config-flag    Enable verification of the advertised O flag
  router-preference    Enable verification of the advertised Router Preference
                       flag
  trusted-port         Setup trusted port
{% endhighlight %}


# DHCPv6 Guard

{% highlight apache %}
Switch(config)#ipv6 dhcp guard policy  dhcpguard01
Switch(config-dhcp-guard)#?
IPv6 dhcp guard policy configuration mode:
  default       Set a command to its defaults
  device-role   Sets the role of the device attached to the port
  exit          Exit from DHCP Guard policy configuration mode
  no            Negate a command or set its defaults
  trusted-port  This is a trusted port (no policing)

Switch(config-dhcp-guard)#device-role ?
  client  Attached device is a client (default)
  server  Attached device is a dhcp server

Switch(config-dhcp-guard)#device-role server
Switch(config-dhcp-guard)#?
IPv6 dhcp guard policy configuration mode:
  default       Set a command to its defaults
  device-role   Sets the role of the device attached to the port
  exit          Exit from DHCP Guard policy configuration mode
  match         dhcp filtering
  no            Negate a command or set its defaults
  preference    dhcp server advertised preference filter
  trusted-port  This is a trusted port (no policing)

Switch(config-dhcp-guard)#
{% endhighlight %}



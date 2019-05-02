---
layout: post
title:  "Setting up the Network: Part 2 (using Snort)"
date:   2019-04-01
categories: ['Final Year Project']
---
Category: **Final Year Project**

For Intrusion Prevention:

![My helpful screenshot](/assets/Part2.jpeg)

The topology for this is as following: <br/>
1. The Cisco switch is configured with a VLAN <br/>
2. Victim computers are connected to ports on the switch through the internet. All of them are on the same VLAN.  <br/>
3. A computer running Snort IDS/IPS has two ethernet interfaces, one is connected to a port on the switch, while the other is connected to the router. <br/>
4. Mobile broadband connected to the router provides internet access.<br/>
5. Attacker(s) access the victim network by wirelessly connecting.<br/>

Configurations on the victim computers, switch and router remain the same as in [Part 1]({{ site.baseurl }}{% link _posts/2019-03-12-Setting up the Network Part 1.markdown %}).

The main difference in the network topology from [Part 1]({{ site.baseurl }}{% link _posts/2019-03-12-Setting up the Network Part 1.markdown %}) is that the router is no longer connected directly to the switch. There is now a computer running Snort IDS/IPS with two ethernet interfaces, one connected to a port on the switch, and the other to a port on the router in the middle. So all traffic entering and leaving the victim network through the router now passes through this computer. An attacker's traffic now coming in wirelessly through the router, to reach any computer in the victim network will have to pass through Snort.

Note: Basic knowledge of what Snort is, is assumed hereby.

Configuration of the computer (my laptop) running Snort IDS/IPS:<br/>
<pre>
# First bridged interface
auto eth0
iface eth0 inet manual
    up ifconfig $IFACE 0.0.0.0 up
    up ip link set $IFACE promisc on
    post-up ethtool -K $IFACE gro off
    post-up ethtool -K $IFACE lro off
    down ip link set $IFACE promisc off
    down ifconfig $IFACE down

# Second Bridged Interface
auto eth1
iface eth1 inet manual
    up ifconfig $IFACE 0.0.0.0 up
    up ip link set $IFACE promisc on
    post-up ethtool -K $IFACE gro off
    post-up ethtool -K $IFACE lro off
    down ip link set $IFACE promisc off
    down ifconfig $IFACE down
</pre>

I changed the interfaces file on my laptop to contain the above. This set both the ethernet interfaces to unmanaged.

With Snort v3 installed, I activated it in inline mode and used the Data Acquisition library's (DAQ) Afpacket module to make a transparent bridge between the two interfaces. By using the Afpacket module Snort itself bridges the interfaces used, no prior bonding/bridging is required. <br/>

DAQ is a library that separates fuctions such as packet capture from Snort. This helps in not changing Snort configurations directly if any of the modules of DAQ are used. The library's main purpose is to allow inline functionality, so that Snort can be used as an IPS instead of just being a passive IDS sensor, and to encourage flexibility so customized modules dependent on these functions could easily be created.

DAQ library has the following modules if Snort3 is installed as is without any extra configurations or modules: <br/>
Run the following command to see the list of modules you currently have installed and the modes that they can be enabled in. <br/>
<pre>
snort --daq-list
</pre> 

![My helpful screenshot](/assets/daqList.png)

**Pcap**: The default DAQ, used for sniffer and IDS modes. If snort is run w/o any DAQ arguments, it will operate as it always did using this module.<br/>
**Ipfw**: Inline on OpenBSD and FreeBSD.<br/>
**Dump**: Allows testing of  various inline mode features like injection and normalization.<br/>
**Afpacket**: Functions similar to the pcap DAQ but with better performance. Enables Snort to be inline on Linux using two bridged interfaces.<br/>

Since afpacket does not depend on IP routing like ipfw does, kernel forwarding dos not need to be enabled here. The Snort sensor is positioned on the network between two network segments with an interface connected to each and then transparently forwards traffic between its two interfaces so both network segments appear to be part of one broadcast domain.

The following changes were made in the /etc/snort/snort.lua config file to check if afpacket worked as it should: <br/>
1. Changed **'any'** in **HOME_NET = 'any'** to the network address of the victim network.<br/>
2. In section 6. configure detection, added **include = '/path/to/rules.txt'** and **mode = inline** within the **ips = { }** braces.<br/>
3. Created a file called rules.txt and added an all encompassing rule that blocked any incoming traffic from any IP and any port to any IP and to any port, with this line **drop ip any any -> any any (msg:"dropped")** <br/>

I then ran Snort from the terminal with:<br/>
**sudo snort --daq afpacket -Q -c /home/rumasi/SNORTC/etc/ snort/snort.lua -i eth0:eth1 -L dump**<br/>
-Q for starting Snort in inline mode.<br/>
-c followed by path of config file to explicitly direct which file to use. <br/>
-i for interfaces that need to be briged.<br/>
-L for logging mode, here "-L dump" means the traffic is dumped on the terminal.<br/>

With Snort running I connected wirelessly to the router with my mobile and requested the site hosted on the webserver and voila! I couldn't access it anymore. And after terminating Snort, in the statistics, a number of packets blocked were clearly shown in the "blocked" line.


 




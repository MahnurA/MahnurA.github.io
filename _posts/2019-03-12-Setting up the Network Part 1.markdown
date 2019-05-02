---
layout: post
title:  "Setting up the Network: Part 1"
date:   2019-03-12
categories: ['Final Year Project']

---
Category: **Final Year Project**

For creating the Dataset: 

Network Topology

![My helpful screenshot](/assets/Part1.jpeg)

This setup was to create the dataset later used to train the model. As no detection or prevention was required at this stage, Snort was not used here. 

The victim network, as the name suggests, comprise of computers that are targeted by the attacker(s). All of the computers within this network are on the same subnet, and each is connected through the ethernet to a port on the Cisco Catalyst 2960-X series switch. Each port on the switch is in the same VLAN. 
I've set up the wired ethernet on each computer statically to a specific IP on the same subnet, the netmask is 255.255.255.0 and the gateway and DNS is the IP of the router. Automatic routing is disabled. 

I also edited the network interface file by sudo nano /etc/network/interfaces.d/* command. So the file now looks like the following: 
<pre>
# the loopback network interface
auto lo
iface lo inet loopback

# the primary network interface
auto eno1 
iface eno1 inet static 
address 10.1.100.2
netmask 255.255.255.0
network 10.1.100.0
broadcast 10.1.100.255
gateway 10.1.100.4 
</pre>

After similarly setting up the rest of the computers in the victim network, the computers can now talk to each other through the switch. A router configured to with the IP, 10.1.100.4, is connected to the switch through a port in the same VLAN as the victims. A mobile broadband dongle is connected to this router to provide access to the internet.  

The Cisco switch is set up with a  Switched Port Analyzer (SPAN) port, mirroring the traffic from the ports where the victim computers are connected to this port. The following was used to create this functioning:
<https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst2960x/software/15-0_2_EX/network_management/configuration_guide/b_nm_15ex_2960-x_cg/b_nm_15ex_2960-x_cg_chapter_0111.html>

Special caution has been followed to ensure that this SPAN port is not overloaded which would make it prone to dropping packets. So at a maximum two ports are mirrored to this one at a time. 

A separate computer is used to capture the traffic on this SPAN port through Wireshark listening in on the respective ethernet interface in promiscous mode. 

All computers run Ubuntu 18.04. One of them is a webserver running Damn Vulnerable Web Application (DVWA) that I set up after installing and configuring the following required applications: <br/>
Apache v2.4.29<br/>
MySQL v5.7.25<br/>
PhpMyAdmin v7.2.15<br/>

Another computer is the SSH server running the service through OpenSSH. 

A computer running Kali Linux is used as an attacker, which connects to the router wirelessly, and can then target and access the services running on the victim computers. The attacker's traffic is routed through the router, through the respective port on the switch, where it also gets mirrored to the SPAN port, and then finally reaches the port it's directed for on the victim computer. 

The attacker computer(s) also have their IPs statically set. This was done only to make it easier to tag the pcap file.



      




















---
layout: post
title:  "Realtime Feature Extraction and its problems"
date:   2019-05-31
categories: ['Final Year Project']
---
Category: **Final Year Project**

Over the years as machine learning has been applied to network security in the hopes to move away from signature based Network Intrusion Detection Systems, a large part of the research to achieve this has been dedicated to finding the most accurate combination of algorithm(s), which is a fair goal to pursue, however, what is and what remains of equal importance is having the ability to detect attacks in realtime or near realtime.

Since the project I undertook hoped to tackle in detection, and possibly later, prevention, near realtime was of ultimate importance. Next, of almost equal importance was that packets once detected either way as Benign or Not, needed to be labelled as such *packet-wise.*  

Typically, when thousands and millions of packets are analyzed, whether for Machine Learning or for general network traffic analysis by network administrators, softwares used for that analysis convert movement of singular packets as seen in PCAP files, into flows.

### What are network flows? ### 

A **Flow** is an end to end connection that shows how the data is flowing through the network.

Five characteristics are associated with each flow: **Source IP**, **Destination IP**, **Source Port**, **Destination Port**, and **Protocol**. Packets having the same value for each of these five characteristics belong to the same flow. A flow starts when a packet having that particular 5 tuple values is first seen in the traffic. Every subsequent packet having those same 5 tuple values gets categorized in that flow, or has the same flow ID. This is the universally accepted way of deriving flows from raw network data.

However, what remains in muddy waters is the exact way a flow is terminated. All tools use timeout values to determine when to end a flow. The timeout value is a countdown that is usually started from the last packet seen of that particular flow. If another packet is not observed within the next 5 milliseconds or however long the timeout value is defined at, the flow is terminated. If a packet of this flow is then seen after the timeout window, that packet will not be categorized in the same flow, but will be in a new flow record. Uptil here it's all straightforward, the discord lies in what should the timeout value be set at?

Different tools have different timeout values. Since the longer the timeout value the more memory greedy the tool. So this value is usually set low. Some tools, like [CICFLowMeter](http://netflowmeter.ca/netflowmeter.html) do allow you to fiddle around with that number. So, if the tool is slow, on your head be it.

Timeout values also affect the accuracy of a flow record. Flows with TCP as their protocol and dealing with application layer software will go on for far longer than a 3-way handshake and an instant disconnection classic to checking if a port is open or closed. The former might extend for several seconds while the latter would most usually finish up in a couple of milliseconds. Thus, if the timeouts are set too low, all packets that should be in the same flow might not be categorized as such, decreasing the accuracy of flows. 

This is how unidirectional flows work. To lessen the amount of data being observed to an even lesser amount, most tools combine packets to obtain *bi-directional* flows.

**Bi-directional flows** are similar to uni-directional flows. They have the same five characteristics, namely, Source IP, Destination IP, Source Port, Destination Port, and Protocol. However, here, Source and Destination, both IPs and Ports, when swapped *also* belong to the same flow. So a flow and its response flow combined, make up a bi-directional flow. 

A PCAP file, processed and converted to a CSV containing bi-directional flows can be used to extract unique features. Since a bi-directional flow is the entire conversation that one machine holds with another, how that conversation is carried out can tell us a lot about what type of conversation it might be. 

### Flows and Features ###



........................................................................................................................................
This post is under construction and will be updated soon!
........................................................................................................................................


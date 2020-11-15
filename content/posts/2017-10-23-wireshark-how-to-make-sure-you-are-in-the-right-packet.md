---
title: 'Wireshark: How to make sure, you are in the right packet!!'
author: sitereal
type: post
date: 2017-10-23T17:32:50+00:00
url: /2017/10/23/wireshark-how-to-make-sure-you-are-in-the-right-packet/
featured_image: /wp-content/uploads/2017/10/AAEAAQAAAAAAAA2yAAAAJDBiMDFhYmJmLTVjY2YtNDllMS1hOTdlLTlkZTkyMWRkNWU4Mg.png
categories:
  - Networking

---
&nbsp;

Some weeks ago, we had a strange issue: We have an hl7 messaging service, and one of the clients that send us some messages, was telling there´s some delay with our queues. We had to put a sniffer in the client, client firewall, server and 4 firewalls that are in our environment.

The client tells us, that some packets are getting stuck for 3 seconds, and we saw that this packet, was retransmitting. Here we could see the first SYN and the tcp retransmission 3 seconds later.

<img class="alignnone wp-image-12 size-full" src="http://sitereliabilityengineer.io/wp-content/uploads/2017/10/1.png" alt="" width="1660" height="101" srcset="http://sitereliabilityengineer.net/wp-content/uploads/2017/10/1.png 1660w, http://sitereliabilityengineer.net/wp-content/uploads/2017/10/1-800x49.png 800w, http://sitereliabilityengineer.net/wp-content/uploads/2017/10/1-1440x88.png 1440w, http://sitereliabilityengineer.net/wp-content/uploads/2017/10/1-1600x97.png 1600w" sizes="(max-width: 1660px) 100vw, 1660px" /> 

Sometimes, you don´t have the time synchronized in all the systems, so you will need to make sure which packet you are displaying. Here I will show you the Packet bytes pane of the first SYN and below, the packet bytes from the TCP retransmission:

<img class="alignnone size-full wp-image-13" src="http://sitereliabilityengineer.io/wp-content/uploads/2017/10/2.png" alt="" width="552" height="308" /> 

First SYN &#8211; Packet bytes pane. The identification value is 0x4253

<img class="alignnone size-full wp-image-14" src="http://sitereliabilityengineer.io/wp-content/uploads/2017/10/3.png" alt="" width="598" height="239" /> 

TCP Retransmission &#8211; Packet bytes pane. The identification´s value: 0x4521

Now I will show you the server side *.pcap, where we only see one of the packets. The first SYN wasn´t reaching the server.

<img class="alignnone size-full wp-image-15" src="http://sitereliabilityengineer.io/wp-content/uploads/2017/10/4.png" alt="" width="1578" height="661" srcset="http://sitereliabilityengineer.net/wp-content/uploads/2017/10/4.png 1578w, http://sitereliabilityengineer.net/wp-content/uploads/2017/10/4-800x335.png 800w, http://sitereliabilityengineer.net/wp-content/uploads/2017/10/4-1440x603.png 1440w" sizes="(max-width: 1578px) 100vw, 1578px" /> 

Now we could see on the server side, the IP identification of the packet, that corresponds, with the TCP Retransmission. Value: 0x4521

The problem was in the client´s firewall, but the propose of this message, was trying to identify a TCP retransmission packet, because identifying the packet with the time isn´t a good idea.

Maybe this is too obvious for most of you, but I think it could help for some people.
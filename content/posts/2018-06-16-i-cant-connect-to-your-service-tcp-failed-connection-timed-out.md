---
title: 'Networking Problem: I can’t connect to your service (tcp) failed: Connection timed out'
author: sitereal
type: post
date: 2018-06-16T01:56:32+00:00
url: /2018/06/16/i-cant-connect-to-your-service-tcp-failed-connection-timed-out/
featured_image: /wp-content/uploads/2018/06/iptables.png
categories:
  - Linux
  - Networking
  - Site reliability

---
#### **Imagine that a friend is trying to connect to one of your services and he mention that when he tries to connect, finally displays a  &#8220;(tcp) failed: Connection timed out&#8221;**

The first thing, I go and check if I could connect to the service, then I&#8217;ll check if the service is working properly, if it&#8217;s right, I will go and check the firewall&#8230;..

Wow, I have all open in iptables, everybody could connect to that service, but I need to deal with my friend and tell him something! Because he told me, that he doesn&#8217;t have any rule that could block the connections.

#### **First of all, I&#8217;m going to try to simulate this problem.**

I open the port listening in X ip.

nc -l 127.0.0.2 3000

&nbsp;

**Then I start sniffing:**

tcpdump -vvv -s0 -i lo -w lo.pcap

&nbsp;

**With netcat I also try to connect to the service:**

nc -v -z 127.0.0.2 3000  
nc: connect to 127.0.0.2 port 3000 (tcp) failed: Connection timed out

&nbsp;

**And now I open the .pcap with wireshark.**

127.0.0.1 is my FRIENDS IP and 127.0.0.2 is the service in port 3000.

Here we could see, how my FRIEND/CLIENT send me a SYN, but when I answer with the SYN,ACK the client send me a retransmission of the SYN, and here it&#8217;s where the loop starts, because I also have to send him again a SYN,ACK.

The first thing that I think: the origin is blocking the incoming SYN,ACK

<img class="alignnone size-full wp-image-179" src="http://sitereliabilityengineer.io/wp-content/uploads/2018/06/wireshark.png" alt="" srcset="http://sitereliabilityengineer.net/wp-content/uploads/2018/06/wireshark.png 1608w, http://sitereliabilityengineer.net/wp-content/uploads/2018/06/wireshark-300x48.png 300w, http://sitereliabilityengineer.net/wp-content/uploads/2018/06/wireshark-1440x229.png 1440w, http://sitereliabilityengineer.net/wp-content/uploads/2018/06/wireshark-800x127.png 800w, http://sitereliabilityengineer.net/wp-content/uploads/2018/06/wireshark-1600x255.png 1600w" sizes="(max-width: 1608px) 100vw, 1608px" /> 

&nbsp;

**So I ask my friend for the RULES, and here they are:**

<img class="alignnone size-full wp-image-180" src="http://sitereliabilityengineer.io/wp-content/uploads/2018/06/iptables2.png" alt="" srcset="http://sitereliabilityengineer.net/wp-content/uploads/2018/06/iptables2.png 1035w, http://sitereliabilityengineer.net/wp-content/uploads/2018/06/iptables2-300x30.png 300w, http://sitereliabilityengineer.net/wp-content/uploads/2018/06/iptables2-800x80.png 800w" sizes="(max-width: 1035px) 100vw, 1035px" /> 

&nbsp;

<img class="alignnone size-full wp-image-181" src="http://sitereliabilityengineer.io/wp-content/uploads/2018/06/thatsAllFolks-e1529114136188.jpeg" alt="" />
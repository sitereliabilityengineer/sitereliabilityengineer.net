---
title: Your site is blocked from a region and you don´t know why? Maybe SNI related?
author: sitereal
type: post
date: 2019-02-09T01:15:58+00:00
url: /2019/02/09/your-site-is-blocked-from-a-region-and-you-dont-know-why-maybe-sni-related/
featured_image: /wp-content/uploads/2019/02/blocked_site.png
pipdig_meta_geographic_location:
  - Dublin
categories:
  - Networking
  - Site reliability

---
The other day at work, we received issues from customers. They told us that they couldn´t access to their web Instance. In all the cases, the origin was the same country.

The first thing that I ask, is to give me a screenshot of the error, but it wasn´t a great help. 

We also asked for a telnet instance 443<figure class="wp-block-image">

<img src="http://sitereliabilityengineer.net/wp-content/uploads/2019/02/1.png" alt="" class="wp-image-192" srcset="http://sitereliabilityengineer.net/wp-content/uploads/2019/02/1.png 814w, http://sitereliabilityengineer.net/wp-content/uploads/2019/02/1-300x56.png 300w, http://sitereliabilityengineer.net/wp-content/uploads/2019/02/1-800x150.png 800w" sizes="(max-width: 814px) 100vw, 814px" /> </figure> 

My surprise was that, telnet worked well, but then&#8230;.<figure class="wp-block-image">

<img src="http://sitereliabilityengineer.net/wp-content/uploads/2019/02/image.png" alt="" class="wp-image-194" srcset="http://sitereliabilityengineer.net/wp-content/uploads/2019/02/image.png 978w, http://sitereliabilityengineer.net/wp-content/uploads/2019/02/image-300x70.png 300w, http://sitereliabilityengineer.net/wp-content/uploads/2019/02/image-800x186.png 800w" sizes="(max-width: 978px) 100vw, 978px" /> </figure> 

With curl doesn´t work well. Maybe It´s pointing to something related to SSL.

We decided to sniff on the client side, loadbalancer, etc&#8230;<figure class="wp-block-image">

<img src="http://sitereliabilityengineer.net/wp-content/uploads/2019/02/2sni.png" alt="" class="wp-image-193" srcset="http://sitereliabilityengineer.net/wp-content/uploads/2019/02/2sni.png 1129w, http://sitereliabilityengineer.net/wp-content/uploads/2019/02/2sni-300x18.png 300w, http://sitereliabilityengineer.net/wp-content/uploads/2019/02/2sni-800x47.png 800w" sizes="(max-width: 1129px) 100vw, 1129px" /> </figure> 

We could see RSTs just after the client Client Hello. It indicates that it could be a problem with the handshake, maybe ciphers, etc etc..

Here I don´t have the loadbalancer .pcap files, but as far as I remember, in the loadbalancers we also received RSTs. So what´s sending the RSTs?

Let´s examine the Client Hello inside the .pcap file.<figure class="wp-block-image">

<img src="http://sitereliabilityengineer.net/wp-content/uploads/2019/02/image-1.png" alt="" class="wp-image-195" srcset="http://sitereliabilityengineer.net/wp-content/uploads/2019/02/image-1.png 678w, http://sitereliabilityengineer.net/wp-content/uploads/2019/02/image-1-300x219.png 300w" sizes="(max-width: 678px) 100vw, 678px" /> </figure> 

Let´s go down until Server Name Indication extension. After server Name length. It should be the name of the severname (I deleted in this case)

What´s the Server Name Indication Extension: Server Name Indication, often abbreviated SNI, is an extension to TLS that allows multiple hostnames to be served over HTTPS from the same IP address.

Let´s try to change the headers with curl or openssl.<figure class="wp-block-image">

<img src="http://sitereliabilityengineer.net/wp-content/uploads/2019/02/image-2.png" alt="" class="wp-image-196" srcset="http://sitereliabilityengineer.net/wp-content/uploads/2019/02/image-2.png 1155w, http://sitereliabilityengineer.net/wp-content/uploads/2019/02/image-2-300x193.png 300w, http://sitereliabilityengineer.net/wp-content/uploads/2019/02/image-2-800x515.png 800w" sizes="(max-width: 1155px) 100vw, 1155px" /> </figure> 

Here you could see how it worked. Why? Let´s go and open the new .pcap file.<figure class="wp-block-image">

<img src="http://sitereliabilityengineer.net/wp-content/uploads/2019/02/image-3.png" alt="" class="wp-image-197" srcset="http://sitereliabilityengineer.net/wp-content/uploads/2019/02/image-3.png 682w, http://sitereliabilityengineer.net/wp-content/uploads/2019/02/image-3-300x201.png 300w" sizes="(max-width: 682px) 100vw, 682px" /> </figure> 

In this new .pcap we can´t see the the SNI extension and we can´t also see any of the RST<figure class="wp-block-image">

<img src="http://sitereliabilityengineer.net/wp-content/uploads/2019/02/image-4.png" alt="" class="wp-image-198" srcset="http://sitereliabilityengineer.net/wp-content/uploads/2019/02/image-4.png 977w, http://sitereliabilityengineer.net/wp-content/uploads/2019/02/image-4-300x20.png 300w, http://sitereliabilityengineer.net/wp-content/uploads/2019/02/image-4-800x53.png 800w" sizes="(max-width: 977px) 100vw, 977px" /> </figure> 

Ok, if we have more than one site in that ip address it would be a mess, because we would access to the first loaded certificate, but in this case we only wanted to see that the problem was with SNI filtering.  


Here you have a nice website to check a website from different agents. In this case agents in China and other regions, to see the differences.

<https://www.websitepulse.com/tools/china-firewall-test>

SNI filtering is used very often by Internet providers to block access to torrent sites or similar. In this case it might be related to the great firewall (China).
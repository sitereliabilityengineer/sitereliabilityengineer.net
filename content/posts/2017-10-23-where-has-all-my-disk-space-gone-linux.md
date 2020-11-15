---
title: Where has all my disk space gone? (linux)
author: sitereal
type: post
date: 2017-10-23T17:46:50+00:00
url: /2017/10/23/where-has-all-my-disk-space-gone-linux/
featured_image: /wp-content/uploads/2017/10/1a.jpg
categories:
  - Linux

---
Sometimes I receive some nagios alerts, displaying a high usage of a filesystem. The first thing I do, a df -h and after that I du -csh /directory (that I suspect could be guilty).

My surprise came after the du, the du tell me /directory is innocent!!! Let me show you an example.

<img class="alignnone size-full wp-image-21" src="http://sitereliabilityengineer.io/wp-content/uploads/2017/10/2a.png" alt="" width="462" height="75" /> 

With df we see 17Gb used

<img class="alignnone size-full wp-image-22" src="http://sitereliabilityengineer.io/wp-content/uploads/2017/10/3a.png" alt="" width="295" height="47" /> 

The du displays 7.9Gb used! Something strange happens

I´m a bit abset-minded so I start to du -csh /directory1, then /usr/loca/directory200, then /directory3000 until I remember!! Maybe the file is deleted, but not truncated first, and the file descriptor stills open??? !! Ohh, lets see&#8230;.

So I execute: &#8216;lsof -X | grep &#8220;(deleted)\|COMMAND&#8221;|more&#8217; and I see lot of stuff&#8230;..

<img class="alignnone size-full wp-image-23" src="http://sitereliabilityengineer.io/wp-content/uploads/2017/10/4a.jpg" alt="" width="1603" height="189" srcset="http://sitereliabilityengineer.net/wp-content/uploads/2017/10/4a.jpg 1603w, http://sitereliabilityengineer.net/wp-content/uploads/2017/10/4a-800x94.jpg 800w, http://sitereliabilityengineer.net/wp-content/uploads/2017/10/4a-1440x170.jpg 1440w, http://sitereliabilityengineer.net/wp-content/uploads/2017/10/4a-1600x189.jpg 1600w" sizes="(max-width: 1603px) 100vw, 1603px" /> 

Now I see there are lots of deleted files using space. They are &#8220;deleted&#8221; but the fd stills open. For example file .out00029 is using 54Mb. The lsof displays the usage in bytes, but we could do &#8220;56952801 / (1024.0 * 1024.0) = 54Mb&#8221; to get Mb

Now I go to /proc/29937/fd (29937 is the PID of the process that haves the fd open) and I do ls:

<img class="alignnone size-full wp-image-24" src="http://sitereliabilityengineer.io/wp-content/uploads/2017/10/5a.jpg" alt="" width="1378" height="71" srcset="http://sitereliabilityengineer.net/wp-content/uploads/2017/10/5a.jpg 1378w, http://sitereliabilityengineer.net/wp-content/uploads/2017/10/5a-800x41.jpg 800w" sizes="(max-width: 1378px) 100vw, 1378px" /> 

This file haves two file descriptors, still used by the PID 29937 (it´s a weblogic server). In the lsof you could see fd 1w and 2w and in the ls you could see 2 and 1, just after the time. The w means that the file descriptor is marked as writeable.

One trick to free that space is to truncate the fd directly. I do it this way, but there are more ways to do it:

\# cd /proc/29937/fd and # :> 1 (to truncate file descriptor 1) or # :> 2 (truncate fd 2)

I´ve created a .py for displaying deleted files, that are still in use (fd open) and ordering this fd by size and displaying the location of the fd, so you could truncate it. Take care!! You have to know what you are doing, because you could delete something important.

Now I´m going to put a screenshot of the output of my script. My apologies with my bad code, I´m not a rock star programming, but I try to do it in &#8216;my&#8217; best way.

<img class="alignnone size-full wp-image-25" src="http://sitereliabilityengineer.io/wp-content/uploads/2017/10/6a.jpg" alt="" width="1442" height="180" srcset="http://sitereliabilityengineer.net/wp-content/uploads/2017/10/6a.jpg 1442w, http://sitereliabilityengineer.net/wp-content/uploads/2017/10/6a-800x100.jpg 800w, http://sitereliabilityengineer.net/wp-content/uploads/2017/10/6a-1440x180.jpg 1440w" sizes="(max-width: 1442px) 100vw, 1442px" /> 

And here the location of the .py in my github:

https://github.com/SiteReliabilityEngineering/sre/blob/master/deleted\_files\_fd_open

I hope, it could help!!!
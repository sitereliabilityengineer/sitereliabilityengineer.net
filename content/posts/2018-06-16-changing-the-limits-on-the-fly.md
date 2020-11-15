---
title: Changing the limits on the fly
author: sitereal
type: post
date: 2018-06-16T00:37:38+00:00
url: /2018/06/16/changing-the-limits-on-the-fly/
featured_image: /wp-content/uploads/2018/06/limits.png
pipdig_meta_geographic_location:
  - London
categories:
  - Linux
  - Site reliability

---
### **I never did this before, but now that I know that it works, I will do it more often.**

To show how it works, I changed the number of processes (nproc) hard limit of the user koteo in the limits.conf to 10

With the prlimit command I display the limits of the first PID of user &#8220;koteo&#8221;, that matches with the limits of the file limits.conf

<img class="alignnone size-full wp-image-170" src="http://sitereliabilityengineer.io/wp-content/uploads/2018/06/1.png" alt="" srcset="http://sitereliabilityengineer.net/wp-content/uploads/2018/06/1.png 802w, http://sitereliabilityengineer.net/wp-content/uploads/2018/06/1-300x177.png 300w, http://sitereliabilityengineer.net/wp-content/uploads/2018/06/1-800x473.png 800w" sizes="(max-width: 802px) 100vw, 802px" /> 

Now I execute bash until I get the error &#8220;Cannot fork&#8221;.

<img class="alignnone size-full wp-image-171" src="http://sitereliabilityengineer.io/wp-content/uploads/2018/06/2.png" alt="" srcset="http://sitereliabilityengineer.net/wp-content/uploads/2018/06/2.png 793w, http://sitereliabilityengineer.net/wp-content/uploads/2018/06/2-300x59.png 300w" sizes="(max-width: 793px) 100vw, 793px" /> 

I execute prlimit with the parameter &#8211;nproc=1024:1024 (soft:hard) and the parameter &#8211;pid $pid (we get $pid from pgrep) . We just changed the soft and hard limit to 1024, as you could see at the bottom of the next screenshot.

<img class="alignnone size-full wp-image-175" src="http://sitereliabilityengineer.io/wp-content/uploads/2018/06/3.png" alt="" srcset="http://sitereliabilityengineer.net/wp-content/uploads/2018/06/3.png 944w, http://sitereliabilityengineer.net/wp-content/uploads/2018/06/3-300x44.png 300w, http://sitereliabilityengineer.net/wp-content/uploads/2018/06/3-800x117.png 800w" sizes="(max-width: 944px) 100vw, 944px" /> 

Now I can execute again the bash command, after the error &#8220;Cannot fork&#8221;.

<img class="alignnone size-full wp-image-173" src="http://sitereliabilityengineer.io/wp-content/uploads/2018/06/4.png" alt="" srcset="http://sitereliabilityengineer.net/wp-content/uploads/2018/06/4.png 798w, http://sitereliabilityengineer.net/wp-content/uploads/2018/06/4-300x68.png 300w" sizes="(max-width: 798px) 100vw, 798px" /> 

Here I show you, that we actually have more than 10 processes for koteo user.

<img class="alignnone size-full wp-image-174" src="http://sitereliabilityengineer.io/wp-content/uploads/2018/06/5.png" alt="" srcset="http://sitereliabilityengineer.net/wp-content/uploads/2018/06/5.png 771w, http://sitereliabilityengineer.net/wp-content/uploads/2018/06/5-300x110.png 300w" sizes="(max-width: 771px) 100vw, 771px" /> 

&nbsp;

If I&#8217;m not wrong, this started working with kernels 2.6.32+

I know that in few versions of the kernel works &#8220;echo -n &#8220;Max processes=SOFT\_L:HARD\_L&#8221; > /proc/$PID/limits&#8221; but not in the one that I have. It displays the error:

\`write(2, &#8220;: Invalid argument&#8221;, 18: Invalid argument) = 18\`

&nbsp;

&nbsp;
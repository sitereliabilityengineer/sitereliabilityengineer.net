---
title: Some tips for finding performance issues in Linux
author: sitereal
type: post
date: 2017-10-23T17:52:10+00:00
url: /2017/10/23/some-tips-for-finding-performance-issues-in-linux/
featured_image: /wp-content/uploads/2017/10/1b.jpg
categories:
  - Linux

---
Sometimes we have some trouble with processes that demands lot of IO. There´s a great tool for that, iotop.

I executed fio, to generate some read stress:

mkdir -p /tmp/data ; fio &#8211;runtime=300 &#8211;time_based &#8211;name=random-read &#8211;rw=randread &#8211;size=128m &#8211;directory=/tmp/data

<img class="alignnone size-full wp-image-29" src="http://sitereliabilityengineer.io/wp-content/uploads/2017/10/2b.jpg" alt="" width="1345" height="93" srcset="http://sitereliabilityengineer.net/wp-content/uploads/2017/10/2b.jpg 1345w, http://sitereliabilityengineer.net/wp-content/uploads/2017/10/2b-800x55.jpg 800w" sizes="(max-width: 1345px) 100vw, 1345px" /> 

As you could see, fio is on the top of the iotop&#8217;s view, and the io is at 99%, displaying the DISK READ K/s.

If you don´t have iotop, you could do it with a little script. It´s not going to give you all that information that iotop shows, but is really good.

Basicly, this script shows the process in &#8220;D&#8221; State. Processes that are waiting for I/O are commonly in an &#8220;uninterruptible sleep&#8221; state or &#8220;D&#8221;; given this information we can simply find the processes that are constantly in a wait state.

cd /proc ; for pid in [0-9]* ; do awk &#8216;$2 == &#8220;D&#8221; {print &#8220;The process with PID: &#8216;${pid}&#8217; is in &#8216;D&#8217; State&#8221;;}&#8217; $pid/status ; done

<img class="alignnone size-full wp-image-30" src="http://sitereliabilityengineer.io/wp-content/uploads/2017/10/3b.jpg" alt="" width="1145" height="62" srcset="http://sitereliabilityengineer.net/wp-content/uploads/2017/10/3b.jpg 1145w, http://sitereliabilityengineer.net/wp-content/uploads/2017/10/3b-800x43.jpg 800w" sizes="(max-width: 1145px) 100vw, 1145px" /> 

You could see some detailed information with iostat.

iostat -xdy 2 5 (x= Display extended statistics, d= Display the device utilization and y= Omit first report with statistics since system boot)

We see that, %util is very high. It´s very useful to see the r/s w/s (In this case the problem are the reads)

<img class="alignnone size-full wp-image-31" src="http://sitereliabilityengineer.io/wp-content/uploads/2017/10/4b.jpg" alt="" width="1011" height="496" srcset="http://sitereliabilityengineer.net/wp-content/uploads/2017/10/4b.jpg 1011w, http://sitereliabilityengineer.net/wp-content/uploads/2017/10/4b-800x392.jpg 800w" sizes="(max-width: 1011px) 100vw, 1011px" /> 

### Sometimes we could reach the limit of &#8220;open files&#8221;.

Error: &#8220;Too many open files (24)&#8221;

You could see the total of files opened in the system with a simple shell script or lsof:

\# for pid in /proc/[0-9]* ; do echo $(ls $pid/fd | wc -l) ; done | sort -n | awk &#8216;{ SUM += $1} END { print SUM }&#8217;

\# lsof -Xn -a -d ^mem -d ^cwd -d ^rtd -d ^txt -d ^DEL | wc -l

<img class="alignnone size-full wp-image-32" src="http://sitereliabilityengineer.io/wp-content/uploads/2017/10/5b.jpg" alt="" width="1055" height="57" srcset="http://sitereliabilityengineer.net/wp-content/uploads/2017/10/5b.jpg 1055w, http://sitereliabilityengineer.net/wp-content/uploads/2017/10/5b-800x43.jpg 800w" sizes="(max-width: 1055px) 100vw, 1055px" /> 

If you need, to display the info, only for one user, you will need to pass the argument -u $USER to lsof

If you&#8217;ve overpassed the user limit, you will need to change this limit with:

&nbsp;

<pre class="brush: php; title: ; notranslate" title="">Changing the limit for the user: (Edit your .profile and add it or change it
in limits.conf

* ulimit -Hn $NEW_LIMIT ($HOME/.profile)

* Or maybe you will need to do this change globally for all the users, 
editing /etc/sysctl.conf and modifying the value of fs.file-max = $NEW_GLOBAL_LIMIT
</pre>
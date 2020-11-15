---
title: Run commands remotely via ssh, using groups of hosts lists
author: sitereal
type: post
date: 2017-11-15T22:21:03+00:00
url: /2017/11/15/run-commands-remotely-via-ssh-using-groups-of-hosts-lists/
featured_image: /wp-content/uploads/2017/11/sshhero1-e1510784806425.png
categories:
  - Bash

---
### 

### Some time ago I made two scripts:

  1. Script named as: multiple\_ssh\_exec.sh \* When executed, it displays a menu. \* In the first lines it gives the host groups names. \* You will need to input as first option a host group name. \* On the second option you will need to input a Username. * Third option: input a command to execute on remote host.
  2. Script named: multiple\_ssh\_exec\_w\_args.sh \* This scripts require arguments from the cli. \* First arg: -f the file with the host groups \* Second arg:-s to show the host group names (requires -f ) \* Third arg: -n to define the host group name (requires -f) \* Fourth arg:-u username to use with ssh (requires -f -n) \* Fifth arg: -c command to use on remote host (requires -f -n -u)

You could take a look here:

<https://github.com/SiteReliabilityEngineering/sre/tree/master/multiple_ssh>

One of the script&#8217;s !

<pre class="brush: python; title: ; notranslate" title="">#!/bin/sh
# Koldo Oteo &lt;koldo.oteo1@gmail.com&gt;
#

### FUNCTIONS

#
usage()
{
cat &lt;&lt;EOF
Usage:   $0 [OPTION]
Example: $0 -f hosts_file.txt -s (it displays the host groups)
	 $0 -f hosts_file.txt -n host_group -u USER -c command (runs command on the host group)
        Options:
                -h,     Print help.
                -f,     File with host groups.
		-s,	Show host groups names.
                -n,     Host group name.
                -u,     User you use to login via ssh.
                -c,     Command to run in remote server.
EOF
exit 0
}

#
show_hgroups()
{
echo -e "Available host groups:\n"
echo -e "##########"
grep -e '\[[a-zA-Z]' $f | tr -d '[/]'
echo -e "##########"
echo -e "\n"
}

#
exec_comm()
{
for i in $(sed -n -e '/\['$n'\]/,/\[\/'$n'\]/ p' "$f" | sed -e '1d;$d')
        do ssh  "$u"@$i "$c"
done
}

###

while getopts ":hf:sn:u:c:" opts; do
    case "${opts}" in
        h)
	    usage 
            ;;
        f)
            f=${OPTARG}
            ;;
        s)
	    s=1
            ;;
        n)
            n=${OPTARG}
            ;;
        u)
            u=${OPTARG}
            ;;
        c)
            c=${OPTARG}
            ;;
        *)
            usage
            ;;
    esac
done

#
[[ -z "$1" ]] && usage

if

[ -n "$f" ] && [ -n "$c" ]  && [ -n "$u" ] && [ -n "$c" ]
then
	exec_comm
fi

if [ -n "$f" ] && [ -n "$s" ]
then
	show_hgroups
fi
</pre>
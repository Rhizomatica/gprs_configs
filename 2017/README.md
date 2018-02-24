(E)GPRS

Taking advantage of the existence of this repo, I'll dump here some of the config files in use now that we are 
in 2017...18

:-)

### Making use of the limited bandwidth available with GPRS ###


----------


Technical Documents on GPRS written around the time of first deployments often refer to "high speed" data. 

Of course in the context of today's inter-networks the terms "high speed" and "GPRS" have no place in the same sentence. Most modern Cell Phones would saturate the available capacity faster than you can say "PDP Context Activated".

Most of this traffic will be to the app stores and other cloud services associated with the devices.

Following all that calling home, the rest of the traffic that will saturate your limited capacity will be related to the so called "social networking" apps that come by default installed on most devices.

A good first step to some control over this is to limit DNS resolution. [1] This is enough to make most mobiles back off.

`dnsmasq` offers a quick one stop solution for this. 

Configure the GGSN to assign your local dnsmasq server as DNS, then add an appropriate firewall rule to catch all that sneaky DNS traffic that will try to use 8.8.8.8 and the like, so assuming your mobile side network is 10.20.0.0/16 and your dnsmasq server is 172.16.0.1:

	iptables -t nat -A PREROUTING -s 10.20.0.0/16 ! -d 172.16.0.1/32 -p udp -m udp --dport 53 -j DNAT --to-destination 172.16.0.1:53
	iptables -t nat -A PREROUTING -s 10.20.0.0/16 ! -d 172.16.0.1/32 -p tcp -m tcp --dport 53 -j DNAT --to-destination 172.16.0.1:53

Drop a file into `/etc/dnsmasq.d` with content:

	no-resolv

and then something like:

	server=/whispersystems.org/192.168.1.1

where 192.168.1.1 is your upstream DNS.

Now your dnsmasq will only be able to resolv for the domains you specify with server= lines. Everything else will return NXDOMAIN. Note that as some distros setup dnsmasq as a local resolver when you install it, this is your local DNS as well, you'll prevent your own local DNS resolver from working :-(

Just fix up `/etc/resolv.conf` for now, and deal with stopping your distro redirecting your DNS to localhost later.


----------


There will be apps that have hard coded IPs and/or other ways of getting out, so you could also just limit traffic by destination port, see the iptables-save file in this repo for examples.

But better than port based limiting, is limiting by AS number. This offers an easy way to save a lot of bandwidth by stopping all that traffic that will be caused by social networking sites, but still allow access to the real internet. 

This script snippet will get the AS for a domain and create some iptables rules to deal with traffic to that AS:

Save it and do something like:

`./script.sh some_book_or_other.com`


	#!/bin/sh
	IP=`nslookup $1 | grep -E '[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}' | tail -n1 | cut -d\  -f2`
	AS=`wget -q -O - http://ipinfo.io/$IP/org | cut -f1 -d \  | sed -e 's/AS//'`
	
	NETWORKS=`wget -O - http://stat.ripe.net/data/announced-prefixes/data.yaml?resource=$AS|grep prefix\:|grep -v \:\:|awk '{print $3}'`

	iptables -t mangle -N as_check
	iptables -t mangle -N as_do

	for i in $NETWORKS; do echo "iptables -t mangle -A as_check -d $i -j as_do" ; done

You might want to modify the 1st line depending on your nslookup output, or just do it better.


Then do what you will with the traffic in the as_do chain, for example:

	iptables -t mangle -A as_do -j MARK --set-mark 1

then you can send back tcp-reset based on the mark, again see the iptables-save example.

Well done! Now we can use "slow speed" internet again! Yay!





[1]
 *Seriously people, fsck "net neutrality" There never was any such thing. Ask anybody who ever tried to host services at home. If there was net neutrality we wouldn't need "the cloud".*



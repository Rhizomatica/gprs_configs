
Config files for running the GGSN and GGSN on the SysmoBTS

On the BTS Side, you would need to activate ipv4 forwarding.
It seems you also need to bring up a network route to the ggsn tunnel interface

So you need something like:

    /sbin/sysctl -w net.ipv4.ip_forward=1

in rc.local or otherwise

AND

    /sbin/route add -net 10.20.0.0/16 dev tun0

after the ggsn has started


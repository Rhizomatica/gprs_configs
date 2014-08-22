gprs_configs
============

# Test configuration for GPRS. Includes config files for OpenBSC and SysmoBTS.

Most of the configuration have been taken from:

    openbsc.osmocom.org/trac/wiki/OpenBSC_GPRS

However, these instructions are specific to the nanoBTS, so we had to do some things differently :)

All the packages can be installed directly by adding this repository to /etc/apt/sources.list.d/rhizomatica.list :

    deb http://dev.rhizomatica.org/ubuntu/ precise main

(assuming an installation of Ubuntu 12.04 amd64).

Then:

    apt-get install osmocom-nitb osmocom-sgsn openggsn

Should install everything you need (on the BSC side).


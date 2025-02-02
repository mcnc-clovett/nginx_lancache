# Nginx LanCache

On premises caching or micro-CDN solution for Microsoft, Google and Adobe products.

## *This version makes changes to the config to work on Debian-based distros*

*The nginx.conf file included is to be used in Linux distributions such as Debian and Ubuntu.*

## Overview

The Nginx Lancache is an on premise caching solution initially designed for schools, but its application can be used anywhere.

Using DNS interception for well-known and high-volume domain names at the local level, requested files can be cached on premises.

Running for 12 months on a network of 13,500 users distributed over 33 different independent links, total bandwidth savings netted 160.55 terabytes of data - an average of 13.38 terabytes per month. The caches had an average efficiency ratio of 3.58:1.

## Intercepted Zones

Microsoft Zones:

       download.windowsupdate.com
       
       *.download.windowsupdate.com
       
       tlu.dl.delivery.mp.microsoft.com
       
       *.tlu.dl.delivery.mp.microsoft.com
       
       officecdn.microsoft.com
       
       officecdn.microsoft.com.edgesuite.net

Google Chrome/ChromeOS Zones:

       dl.google.com
       
       *.gvt1.com

Adobe Zones:

       ardownload.adobe.com
       
       ccmdl.adobe.com
       
       agsupdate.adobe.com
	   

* All zone's need a base '@' A record and a wildcard * record pointing to your on premises cache.

## Implementation Method 1 (Automated Install)

This method is for sites with a pre-existing onsite DNS server.

1. Create a linux VM (Debian was used for this variation) with 2GB ram and 100GB disk space, and install Nginx.
2. Place the `setup-cache.sh` file on the new server and run as root.
3. On your local DNS server, install the zones listed in the "Intercepted Zones" section above with both base and wildcard A records pointing to the IP address you gave your linux VM.

## Implementation Method 2 (Local DNS Option)

This method is for sites with a pre-existing onsite DNS server.

1. Create a linux VM (Debian was used for this variation) with 2GB ram and 100GB disk space, and install Nginx.
2. Implement the nginx.conf configuration and change the two resolver locations to an upstream DNS server that you WILL NOT use
   for the local DNS interception - eg. 8.8.8.8 / 8.8.4.4.
3. On your local DNS server, install the zones listed in the "Intercepted Zones" section above with both base and wildcard A records pointing to the IP address you gave your linux VM.

## Implementation Method 3 (Local DHCP Option)

This method is for sites with no onsite DNS server.

1. Create a linux VM (CentOS was used for this variation) with 2GB ram and 100GB disk space, and install Nginx.
2. Implement the nginx.conf configuration and change the two "forwarders" locations to an upstream DNS server that you WILL NOT use
   for the local DNS interception - eg. 8.8.8.8 / 8.8.4.4.
3. Install "bind" and implement the /etc/named.conf and /etc/named/intercept.zone files. Change the DNS forwarders in named.conf to
   your own forwarders - eg. 8.8.8.8 / 8.8.4.4. Edit the /etc/named/intercept.zone file to point to the IP of your linux vm.
4. On your local DHCP server, change the DNS server option to point to your linux VM.

## Design

The caches are designed for direct connectivity (no proxy) or transparent proxy.

Technically you can point any host to the onsite cache, however the more selective the better.

It only caches HTTP content - SSL is passed through as an SNI proxy only, however the config file has options to rate limit this traffic.

The cache ignores X-Accel-Expires, Expires, Cache-Control, Set-Cookie, and Vary headers. It also ignores query strings as part of
the cache key.

The cache also downloads large files at one 16MB slice at a time. Cache locking prevents the client from pulling the file at the same
time as a cache filling operation, thus reducing bandwidth utilisation. In theory, this should mean that only one instance of any file
is ever downloaded.

Some configuration within the nginx.conf file restricts caching based on a HEAD request. Some updates (for whatever reason) do a HEAD request, then fail to download the actual file. This sometimes causes nginx to download the file into the cache when it is not needed.

---
title: OpenVPN and Iptables
description: I recently setup an OpenVPN server, I mostly followed the fantastic Digital Ocean (DO) guide, however I ended up using iptables instead of ufw. Since setting up my iptables configuration correctly was probably the one thing that gave me the most trouble I thought I'd share.
date: 2016-07-16
draft: false
type: post
---

I recently setup an [OpenVPN](https://openvpn.net/) server, I mostly followed the [fantastic Digital Ocean (DO) guide](https://www.digitalocean.com/community/tutorials/how-to-set-up-an-openvpn-server-on-ubuntu-14-04), however I ended up using [iptables](https://en.wikipedia.org/wiki/Iptables) instead of [ufw](https://help.ubuntu.com/community/UFW). Since setting up my iptables configuration correctly was probably the one thing that gave me the most trouble I thought I'd share.

<img class="post-img" src="/images/iptables-family-guy.gif" alt="iptables be like" title="iptables be like"/>

I setup OpenVPN using a TUN adapter for my OpenVPN server because 1: that's what the DO guide does and 2: I don't plan to do any bridging. However, if you are using a TAP adapter you can read more about it on the [OpenVPN website](https://community.openvpn.net/openvpn/wiki/BridgingAndRouting).

The iptables rules file I ended up using looked something like the following. 

{{< highlight kernel-config >}}

*filter

# Allow all loopback (lo0) traffic and reject traffic
# to localhost that does not originate from lo0.
-A INPUT -i lo -j ACCEPT
-A INPUT ! -i lo -s 127.0.0.0/8 -j REJECT

# Allow ping.
-A INPUT -p icmp -m conntrack --ctstate NEW --icmp-type 8 -j ACCEPT

# Allow SSH connections.
-A INPUT -p tcp --dport 22 -m conntrack --ctstate NEW -j ACCEPT

# Allow OpenVPN connections.
-A INPUT -p udp --dport 1194 -m conntrack --ctstate NEW -j ACCEPT

# Allow inbound traffic from established connections.
# This includes ICMP error returns.
-A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

# Reject all other inbound.
-A INPUT -j REJECT

# Allow traffic initiated from VPN to access LAN.
-A FORWARD -i tun0 -o eth0 -s 10.8.0.0/24 -d 10.0.0.4/24 -m conntrack --ctstate NEW -j ACCEPT

# Allow established traffic to pass back and forth.
-A FORWARD -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT

# Drop invalid packets.
-A INPUT -m conntrack --ctstate INVALID -j DROP

COMMIT

*nat

# Masquerade all traffic from VPN clients.
-A POSTROUTING -o eth0 -s 10.8.0.0/24 -j MASQUERADE

COMMIT

{{< / highlight >}}

There's a few basic non-OpenVPN related items in there - e.g. allow SSH, ping, but overall it's a pretty barebones iptables configuration since I plan to just run OpenVPN on this server. Also note that this only for IPv4 traffic and NOT IPv6.

Finally, to load the rules so that your server is using them it's a matter of running a simple one line command.

{{< highlight bash >}}

sudo iptables-restore openvpniptable.rules

{{< / highlight >}}

And that's it! Assuming you followed the rest of the DO guide you should now have a shiny new OpenVPN server using iptables up and running. Happy VPNing.
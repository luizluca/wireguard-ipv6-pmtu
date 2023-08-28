# wireguard-ipv6-pmtu

Wireguard interface uses by default an MTU of 1420. That considers as the
worst case scenario two peers connecting using IPv6 over an MTU=1500 link

```
1500 -40(IPv6 header) -8(UDP header) -32(wg header) = 1420
```

However, whenever the MTU between peers is smaller (like when PPPoE is in
use), the 1420 is not enough, fragmenting the encrypted packets. This would
lead either to a less performant connection (ip fragment overhead) to a
broken connection (if fragmented packets are dropped by ISP).

When peers are connected using IPv4, the default 1420 is enough (considering
a standard 20-byte IPv4 header) as it really requires 1440.

```
1500 -20(IPv4 header) -8(UDP header) -32(wg header) = 1440
```

The extra 20 bytes are enough to fit most tunneling protocol, like PPPoE+L2TP

This script look for cached PMTU routes and updates all wireguard routes to that
peer to match the usable MTU. It will also adapt if the outgoing interface uses a
smaller MTU, even when there is no PMTU.

Once the route MTU is reduced, wireguard traffic will not trigger "packet too big"
answer. If nothing else sends a large packet, the PMTU route will expire and this
script will remove the MTU reduction from all allowed ip routes. Once a new large
packet is sent, a new "packet too big" would be generated and this script would
again re-add the MTU reduction. There is not a known way to refresh a cached PMTU
route to avoid this cycle.

This script must run periodically or, ideally, triggered when a PMTU change occurs.
However, "ip monitor" does not emit an event for cached routes.

***PMTU DOES NOT WORK*** with busybox ip as it does not show mtu field. Please, use iproute2.
(for OpenWrt, ip-tiny is enough). However, it would still work if the smaller MTU
in the path is a local interface as it also considers local interface MTU.

It was only tested with busybox ash.

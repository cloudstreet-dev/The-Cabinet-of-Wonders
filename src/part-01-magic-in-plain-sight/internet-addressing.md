# The Internet's actual addressing system

The Internet does not know how to reach your computer. It does not have a route to your home network. It does not have a route to your phone, your laptop, or the server you are reading this on. What it has is a set of rough generalizations — large blocks of IP addresses owned by various organizations, and a continuous shouting match between routers about who can reach which blocks. From that shouting match, every packet finds its way to the right machine, somewhere on a planet of billions of devices.

The shouting match is BGP. It is held together by handshake agreements between human network operators, and it has no central authority. It has been the routing fabric of the Internet since 1989 and it carries every packet you have ever sent.

## The structure of the addressing space

The Internet is divided into about 75,000 Autonomous Systems (ASes) — Comcast, Cloudflare, your university, a regional ISP in Mongolia, Google. Each AS owns one or more IP prefixes. A prefix is a range of IP addresses written `1.2.3.0/24`, meaning "the addresses with the same first 24 bits as 1.2.3.0," which is 256 addresses.

Inside an AS, the operator routes packets however they like. Between ASes, neighbors announce to each other "I can reach this prefix, here is the path." That is BGP.

A BGP route announcement contains, at minimum:

```
prefix:    1.2.3.0/24
AS path:   65001 65002 65003
next hop:  198.51.100.7
```

Read right to left: AS 65003 owns the prefix. AS 65002 announced it to AS 65001 ("I can reach 1.2.3.0/24 via 65003"). AS 65001 added itself to the path and announced it to its neighbors. Each AS along the way adds itself before passing the announcement on. The AS path is both a record of how the announcement reached you and a loop-prevention mechanism — if you see your own AS in the path, drop the announcement.

When a packet arrives at an AS for `1.2.3.5`, the router looks up the longest matching prefix in its routing table and forwards toward the announced next hop. "Longest matching prefix" means more specific announcements win: if both `1.2.3.0/24` and `1.2.0.0/16` match, the /24 takes precedence.

## What "the right path" even means

There is no objective shortest path. Each AS chooses among the announcements it has received using its own policy. The standard tie-breaks are:

1. Highest local preference (administrative weight set by the operator).
2. Shortest AS path.
3. Lowest origin type (IGP < EGP < incomplete).
4. Lowest MED (multi-exit discriminator, a hint from the neighbor).
5. eBGP over iBGP.
6. Lowest IGP cost to the next-hop.
7. Tiebreaker by router ID.

The first criterion is an editorial choice. ASes prefer paths through their own customers (who pay them) over peers (free), and peers over upstream providers (who they pay). This is the underlying economics — the Gao-Rexford rules — and it is what makes BGP routes resemble valid commercial paths.

So the path your packet takes from a coffee shop in Berlin to a server in Tokyo is not the path of minimum hops. It is the path each AS along the way found most commercially attractive among the announcements it had at that moment, mediated by economic relationships none of them describe to anyone else, and the next packet might take a different one.

## The wild parts

**Route announcements are claims, not proofs.** Until very recently, anyone could announce any prefix, and their neighbors would either propagate it or not. In 2008, Pakistan Telecom briefly announced YouTube's prefix to block it inside Pakistan; the announcement leaked to a peer, then to the broader Internet, and YouTube was unreachable for two hours. In 2018, an Amazon DNS prefix was hijacked and used to redirect cryptocurrency users. The fix, RPKI (Resource Public Key Infrastructure), has been creeping toward universal adoption for over a decade, with around 50% of prefixes now signed.

**Routes flap.** A single fiber cut in the Red Sea can change which AS path 200 unrelated networks select for a destination. Most BGP traffic is route updates, not application traffic. Routers receive on the order of one update per second steady-state, with bursts during incidents.

**The default-free zone is one giant routing table.** Tier-1 ISPs (the small set of ASes that have no transit provider, only peers) carry "the full table" — currently around 970,000 IPv4 prefixes and 220,000 IPv6 prefixes. Each router with a full table needs RAM and TCAM space proportional to the table size. As the table grows, hardware refreshes become unavoidable; the famous "512K problem" of August 2014 was the moment the IPv4 table exceeded the default TCAM allocation in many Cisco routers, causing widespread outages until operators reconfigured.

**Aggregation matters.** If an AS owns `10.0.0.0/16`, it can announce that one prefix instead of 256 separate /24s, drastically reducing the global table. Operators do this where their internal routing allows. The pressure to aggregate fights the pressure to be specific (because longest prefix wins, more specific announcements steal traffic). The whole Internet routing table is the equilibrium of those forces.

## The picture

```
       AS 1               AS 2 (transit)              AS 3
        |                       |                       |
   +---------+            +-----+-----+            +---------+
   |  origin  |---peer----| transit   |---customer-|  origin  |
   |  AS for  |           | provider  |            |  AS for  |
   | 1.2/16  |            | (paid by  |            |  9.8/16  |
   +---------+            |  AS 1, 3) |            +---------+
        |                 +-----+-----+                  |
        | announces             |                        |
        | 1.2/16                |                        |
        |  with path [AS1]      |                        |
        |                       v                        |
        |                announces 1.2/16                |
        |                with path [AS2 AS1]             |
        |                       |                        |
        +-----------------------+------------------------+
                       packets to 9.8.x.x
                       follow the longest match
                       through the cheapest AS path
                       each AS has chosen
```

## The handshake

A BGP session between two routers is a TCP connection on port 179 with a continuously open exchange. After an initial OPEN message and capabilities negotiation, the two routers send each other every prefix they have, then increments forever. There is no resync mechanism in standard BGP — if the session resets, the routers redo the full table dump.

The trust model is: I run my router. You run yours. We agreed by email last week that you would announce me prefixes A and B, and I would announce you prefix C, and we will pay each other (or not) according to a handshake-and-paperwork contract. The protocol does not enforce any of this. The protocol assumes you will honor the agreement, and if you misbehave, your peers will eventually depeer you and the Internet will route around the dispute.

## The wonder

There is no map of the Internet. There is no central registry of paths. There is no algorithm computing best routes globally. There is a routing system held together by economic agreements, written in human contracts and email threads, where every router in the world is in conversation with a few neighbors, each of whom is in conversation with a few neighbors, and the union of those conversations happens to converge on something that almost always lets your packet find the right place.

It is not that the system is robust to failures. It is that the system is *constituted* by failures and successes — by ASes coming and going, by fiber cuts and route hijacks and policy changes — and the protocol's only job is to make the current state of the world propagate fast enough that the average packet finds its way before things change again. That works at all is a small daily miracle.

## Where to go deeper

- Geoff Huston, *bgpreport.potaroo.net* — long-running statistics on the global routing system, written by one of the few people who reads BGP for a living.
- Gao and Rexford, *Stable Internet Routing Without Global Coordination*, IEEE/ACM TON 2001. Why economic policies do not break BGP convergence.

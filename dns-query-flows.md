# DNS Query Flows and Failure Modes

This is a response to a technical interview question asking me to describe DNS request and response packet flows for three query types, then explain how each behaves after a domain registration lapses.

---

## Setup

- **Domain:** fakedomain123.com (fictional)
- **A record:** www.fakedomain123.com → 1.2.3.4
- **Authoritative nameservers:** ns1.foodns.com, ns2.foodns.com (fictional)
- **TTL:** 3600 seconds (1 hour)

---

## Part 1: Domain Active

### A. `dig +trace www.fakedomain123.com`

The `+trace` flag tells your local machine to bypass its recursive resolver and walk the DNS hierarchy itself, starting from the root. Normally a DNS lookup delegates that work to your ISP's resolver; `+trace` makes the process visible.

**Step 1: Root servers.**
Your machine consults its root hints file and sends a query to a root server (e.g., `a.root-servers.net`) asking for the `.com` TLD servers.

**Step 2: TLD servers.**
Your machine queries a `.com` TLD server (e.g., `a.gtld-servers.net`) for `fakedomain123.com`. The TLD server responds with a referral -- it does not know the A record, but it knows which nameservers are authoritative for the domain: `ns1.foodns.com` and `ns2.foodns.com`.

**Step 3: Authoritative nameservers.**
Your machine queries `ns1.foodns.com` directly. Because this server owns the zone, it answers immediately from its zone file and returns the A record for `www.fakedomain123.com`.

```
$ dig +trace www.fakedomain123.com

; <<>> DiG 9.18.1 <<>> +trace www.fakedomain123.com
;; global options: +cmd

.                       518400  IN  NS  a.root-servers.net.
.                       518400  IN  NS  b.root-servers.net.
;; Received 239 bytes from 192.168.1.1#53(192.168.1.1) in 12 ms

com.                    172800  IN  NS  a.gtld-servers.net.
com.                    172800  IN  NS  b.gtld-servers.net.
;; Received 840 bytes from 198.41.0.4#53(a.root-servers.net) in 28 ms

fakedomain123.com.      172800  IN  NS  ns1.foodns.com.
fakedomain123.com.      172800  IN  NS  ns2.foodns.com.
;; Received 172 bytes from 192.5.6.30#53(a.gtld-servers.net) in 34 ms

www.fakedomain123.com.  3600    IN  A   1.2.3.4
fakedomain123.com.      3600    IN  NS  ns1.foodns.com.
fakedomain123.com.      3600    IN  NS  ns2.foodns.com.
;; Received 88 bytes from 198.51.100.1#53(ns1.foodns.com) in 18 ms
```

---

### B. `dig @ns1.foodns.com www.fakedomain123.com A`

This query targets the authoritative nameserver directly, bypassing the hierarchy entirely. Because `ns1.foodns.com` owns the zone, it does not need to ask anyone else -- it checks its zone file and answers immediately.

```
$ dig @ns1.foodns.com www.fakedomain123.com A

; <<>> DiG 9.18.1 <<>> @ns1.foodns.com www.fakedomain123.com A
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 44712
;; flags: qr aa rd; QUERY: 1, ANSWER: 1, AUTHORITY: 2, ADDITIONAL: 1

;; QUESTION SECTION:
;www.fakedomain123.com.         IN  A

;; ANSWER SECTION:
www.fakedomain123.com.  3600    IN  A   1.2.3.4

;; AUTHORITY SECTION:
fakedomain123.com.      3600    IN  NS  ns1.foodns.com.
fakedomain123.com.      3600    IN  NS  ns2.foodns.com.

;; Query time: 22 msec
;; SERVER: 198.51.100.1#53(ns1.foodns.com)
;; MSG SIZE  rcvd: 103
```

Note the `aa` flag (Authoritative Answer) in the header. This flag is set only when the responding server owns the zone. You will not see it on responses from recursive resolvers or cache.

---

### C. `dig @8.8.8.8 www.fakedomain123.com A`

Google's public resolver at 8.8.8.8 accepts recursive queries. If the record is already in its cache, it answers immediately. If not, it performs the same hierarchy walk described in Part A on your behalf -- root servers, TLD servers, authoritative nameservers -- caches the result for the duration of the TTL, and returns the answer to you.

```
$ dig @8.8.8.8 www.fakedomain123.com A

; <<>> DiG 9.18.1 <<>> @8.8.8.8 www.fakedomain123.com A
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 19283
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; QUESTION SECTION:
;www.fakedomain123.com.         IN  A

;; ANSWER SECTION:
www.fakedomain123.com.  3541    IN  A   1.2.3.4

;; Query time: 31 msec
;; SERVER: 8.8.8.8#53(8.8.8.8)
;; MSG SIZE  rcvd: 66
```

Note the TTL of 3541 rather than 3600. Google fetched and cached this record 59 seconds ago; the TTL is counting down. Also note the absence of the `aa` flag -- 8.8.8.8 is a recursive resolver, not an authoritative server.

---

## Part 2: After the Registrar Removes the Delegation

When you stop paying your registrar, they eventually remove your NS records from the `.com` TLD zone. (In practice this often isn't immediate: registrars typically have grace periods, and some point a lapsed domain at parking nameservers instead. This walkthrough assumes the delegation has been removed.) Your authoritative nameservers still exist and may still hold your zone data, but the official pointer to them is gone. The hierarchy no longer knows how to reach them.

---

### A. `dig +trace www.fakedomain123.com` (post-lapse)

The trace succeeds through the root step. When your machine queries the `.com` TLD servers for `fakedomain123.com`, the TLD responds with `NXDOMAIN`. There is no referral -- the domain no longer exists in the `.com` zone. The `.com` SOA record appears in the authority section, which is how a server proves a negative answer and tells resolvers how long to cache it. The trace stops here.

```
$ dig +trace www.fakedomain123.com

; <<>> DiG 9.18.1 <<>> +trace www.fakedomain123.com
;; global options: +cmd

.                       518400  IN  NS  a.root-servers.net.
.                       518400  IN  NS  b.root-servers.net.
;; Received 239 bytes from 192.168.1.1#53(192.168.1.1) in 12 ms

com.                    172800  IN  NS  a.gtld-servers.net.
com.                    172800  IN  NS  b.gtld-servers.net.
;; Received 840 bytes from 198.41.0.4#53(a.root-servers.net) in 28 ms

;; status: NXDOMAIN
com.    900     IN  SOA a.gtld-servers.net. nstld.verisign-grs.com. (
                        2024080101 1800 900 604800 900 )
;; Received 115 bytes from 192.5.6.30#53(a.gtld-servers.net) in 31 ms

;; (Trace stops here -- no referral to ns1.foodns.com returned)
```

The distinction matters when troubleshooting. `NXDOMAIN` means the name does not exist in the zone at all. `NOERROR` with an empty answer section (sometimes called NODATA) means the name exists but has no records of the type you asked for. A removed delegation produces `NXDOMAIN`.

---

### B. `dig @ns1.foodns.com www.fakedomain123.com A` (post-lapse)

This query bypasses the hierarchy entirely and speaks directly to the nameserver. Whether it succeeds depends entirely on the nameserver provider.

There is no DNS protocol rule that dictates when a zone must be deleted after registration lapses. Some providers suspend zones automatically within minutes. Others leave the configuration in place for days or longer. The behavior is provider-specific and not predictable from the outside.

**While the zone is still active on ns1.foodns.com:** The response is identical to Part 1B. The nameserver does not know or care that the TLD delegation has been removed.

**After the provider suspends or deletes the zone:**

```
$ dig @ns1.foodns.com www.fakedomain123.com A

; <<>> DiG 9.18.1 <<>> @ns1.foodns.com www.fakedomain123.com A
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: SERVFAIL, id: 38291
;; flags: qr; QUERY: 1, ANSWER: 0, AUTHORITY: 0, ADDITIONAL: 0

;; QUESTION SECTION:
;www.fakedomain123.com.         IN  A

;; Query time: 18 msec
;; SERVER: 198.51.100.1#53(ns1.foodns.com)
;; MSG SIZE  rcvd: 32
```

Some providers will return `REFUSED` instead of `SERVFAIL`, depending on their implementation.

---

### C. `dig @8.8.8.8 www.fakedomain123.com A` (post-lapse)

This breaks in phases, governed by two different TTLs: the A record's (3600 seconds) and the NS delegation's from the `.com` zone (172800 seconds, or 2 days).

**Phase 1 -- Within the A record's TTL (up to 1 hour after lapse):**
The resolver's cache still holds the A record. Queries return it normally. The lapse is invisible to clients using 8.8.8.8 during this window.

**Phase 2 -- A record expired, NS delegation still cached (up to 2 days):**
To refresh the A record, the resolver does not have to start from the root. If it still holds the cached NS records for `fakedomain123.com`, it can go straight to `ns1.foodns.com`. As long as the nameserver provider is still serving the zone, the answer comes back normally. Resolver behavior varies here: some re-validate the delegation with the parent more aggressively than others.

**Phase 3 -- Delegation no longer cached:**
The resolver walks the hierarchy, reaches the `.com` TLD servers, and gets the same `NXDOMAIN` described in Part 2A. It returns `NXDOMAIN` to the client and caches that negative answer for the duration set in the SOA record (900 seconds here). The same thing happens sooner if the provider has already removed the zone and `ns1.foodns.com` stops answering.

```
$ dig @8.8.8.8 www.fakedomain123.com A

; <<>> DiG 9.18.1 <<>> @8.8.8.8 www.fakedomain123.com A
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 57104
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; QUESTION SECTION:
;www.fakedomain123.com.         IN  A

;; AUTHORITY SECTION:
com.    900     IN  SOA a.gtld-servers.net. nstld.verisign-grs.com. (
                        2024080101 1800 900 604800 900 )

;; Query time: 44 msec
;; SERVER: 8.8.8.8#53(8.8.8.8)
;; MSG SIZE  rcvd: 115
```

`NXDOMAIN` means "this domain does not exist." From Google's perspective at this point, that is correct -- the TLD has no record of the delegation.

---

## Summary

| Query | Domain Active | Delegation Removed | Later |
|---|---|---|---|
| `dig +trace` | Resolves via root → TLD → auth | Fails at TLD; NXDOMAIN | Same |
| `dig @ns1.foodns.com` | Authoritative answer (aa flag) | Authoritative answer (aa flag) while the provider still serves the zone | SERVFAIL or REFUSED once the provider removes the zone |
| `dig @8.8.8.8` | Recursive answer from cache or hierarchy | Cached A record (up to 1 hour), then answers via cached NS delegation (up to 2 days) | NXDOMAIN |

The most operationally important detail: a lapsed registration does not cause immediate universal failure. Clients using a recursive resolver like 8.8.8.8 may keep resolving the domain for as long as the resolver holds a cached copy of the delegation, up to the 2-day NS TTL, as long as the nameserver still serves the zone. Direct queries to the authoritative nameserver work until the provider removes the zone. That's why a lapsed domain can look "half broken," working for some users and failing for others. Troubleshooting DNS failures with `+trace` is the fastest way to determine whether the problem is at the TLD delegation layer or somewhere else in the chain.

---

[← Back to Home](https://github.com/mjburak/aboutme/blob/main/README.md)

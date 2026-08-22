# DigitalOcean Valkey: The Complete Guide to Managed Caching — How to Set Up a Cluster, Which Plan Fits Your Workload, and Is It Worth Switching From Redis? (With Full Pricing Breakdown and Migration Walkthrough)

If you've been running Redis in production for any length of time, you probably remember the day in March 2024 when the license changed. Suddenly the open-source in-memory store you'd built your caching layer on wasn't quite so open-source anymore, and a quiet scramble began across the industry to figure out what comes next. The short answer turned out to be **Valkey** — a community-governed fork of Redis 7.2.4, stewarded by the Linux Foundation, and backed by names like AWS, Google, and Oracle. And if you're already on DigitalOcean, or thinking about moving there, the longer answer is **DigitalOcean Valkey**, the managed caching service that quietly replaced the old Managed Redis product.

This piece walks through what DigitalOcean Valkey actually is, where it fits, how the plans break down, and the things you'll want to know before you spin up your first cluster. No sales pitch — just the details you'd want if a friend who'd already done it was explaining it over coffee.

## What Valkey Actually Is (And Why It Exists)

Valkey is, in plain terms, a fork of Redis at version 7.2.4. When Redis moved away from its permissive BSD license to a more restrictive dual license model, a chunk of the open-source community — including maintainers, cloud providers, and large end users — forked the last fully open-source release and kept building on it under the original BSD terms. The result is a project that's API-compatible with Redis, wire-compatible with Redis clients, and intended as a drop-in replacement for most workloads.

The practical implication is straightforward: if your application talks to Redis today using any standard client library — `ioredis`, `redis-py`, `jedis`, `StackExchange.Redis`, `valkey-glide`, whatever — it can talk to Valkey tomorrow without code changes. The commands are the same, the data structures are the same, the response semantics are the same. You're moving infrastructure, not rewriting application logic.

DigitalOcean's pitch for its managed Valkey offering leans on this compatibility heavily, and for good reason. The product is positioned as the natural successor to Managed Caching (their previous managed Redis service), and existing clusters can be converted over with relatively little friction. You can read more on the [Valkey product page](https://bit.ly/DigitaLocean) if you want the official framing.

## Why "Managed" Matters Here

Running Valkey (or Redis) yourself isn't hard. Running it *well* in production — with backups, automatic failover, monitoring, security patches, version upgrades, connection management, and capacity planning — is a different and much more tedious activity. A managed service offloads all of that.

What DigitalOcean's managed Valkey gives you, specifically:

- **One-click cluster provisioning** through the control panel or API. You pick a region, a plan, and a few configuration options, and a cluster comes up in minutes.
- **Automatic failover** to a standby node if the primary fails. With two standby nodes, the cluster survives even a double-node failure.
- **Encryption in transit and at rest** by default. Clusters live in your account's private VPC network, with public access gated by a trusted-sources allowlist.
- **Automated maintenance windows** you can schedule, so version updates happen on your terms rather than at a random 3 AM.
- **Per-slot metrics** for visibility into what each shard is actually doing — memory, hit/miss rates, connected clients, keyspace stats.
- **Autoscaling for storage** so clusters can absorb growing data demand without manual resizing.
- **Standby nodes** you can add or remove at any time, in the same region as the primary, to dial up or down your resilience posture.

There are also a few things the managed service explicitly does *not* give you, which we'll cover in the limits section — knowing the gaps is as important as knowing the features.

## Common Use Cases: Where Valkey Earns Its Keep

Valkey is an in-memory key-value store, which means it shines anywhere you need low-latency access to data that either changes often or gets read constantly. The classic patterns:

- **Application caching** — caching database query results, API responses, rendered templates, computed values. This is the bread-and-butter use case and probably 80% of what people actually run Valkey for.
- **Session storage** — keeping user session data in a fast, shared store so any application server can serve any user. Particularly useful behind a load balancer.
- **Rate limiting and throttling** — counting requests per user or per IP in a sliding window, using atomic operations like `INCR` and `EXPIRE`. DigitalOcean even published a [tutorial on building an API rate limiter with managed Valkey](https://bit.ly/DigitaLocean), which is a good starting project if you want to get a feel for the service.
- **Real-time leaderboards** — using sorted sets (`ZADD`, `ZRANGE`, `ZRANK`) to maintain rankings for games, contests, or "top N" dashboards. Gaming platforms lean on this heavily.
- **Message queues and job dispatch** — simple producer/consumer patterns using lists (`LPUSH`/`BRPOP`) or streams. Not a full message broker like Kafka, but lightweight and often enough.
- **Real-time analytics** — counters, bitmaps, hyperloglogs for approximate uniques, time-series-style aggregations.

What Valkey is *not* great at: durable, transactional primary storage where you need ACID guarantees, complex joins, or large datasets that don't fit in memory. For those, pair it with PostgreSQL or MySQL — which DigitalOcean also offers as managed services — and let Valkey handle the hot path.

## The Plan Lineup: What You Get For Your Money

DigitalOcean prices Valkey clusters by node size, billed hourly with a monthly cap. There are seven standard plans, ranging from a $15/month starter to a $960/month beast. Here's the full table — every plan currently listed on the [official pricing page](https://bit.ly/DigitaLocean), nothing omitted:

| Plan (RAM / vCPU) | Disk | Hourly | Monthly (cap) | Best For | Get Started |
| --- | --- | --- | --- | --- | --- |
| 1 GiB / 1 vCPU | 10 GiB | $0.02232 | $15.00 | Dev, testing, tiny caches | [Start 1 GiB Plan](https://bit.ly/DigitaLocean) |
| 2 GiB / 1 vCPU | 30 GiB | $0.04464 | $30.00 | Small production caches, HA entry point | [Start 2 GiB Plan](https://bit.ly/DigitaLocean) |
| 4 GiB / 2 vCPUs | 60 GiB | $0.08929 | $60.00 | Growing apps, mid-size session stores | [Start 4 GiB Plan](https://bit.ly/DigitaLocean) |
| 8 GiB / 4 vCPUs | 140 GiB | $0.17857 | $120.00 | Production workloads, heavier traffic | [Start 8 GiB Plan](https://bit.ly/DigitaLocean) |
| 16 GiB / 6 vCPUs | 290 GiB | $0.35714 | $240.00 | Larger caches, multi-app shared stores | [Start 16 GiB Plan](https://bit.ly/DigitaLocean) |
| 32 GiB / 8 vCPUs | 600 GiB | $0.71429 | $480.00 | High-traffic production, real-time workloads | [Start 32 GiB Plan](https://bit.ly/DigitaLocean) |
| 64 GiB / 16 vCPUs | 1.19 TiB | $1.42857 | $960.00 | Largest workloads, heavy analytics | [Start 64 GiB Plan](https://bit.ly/DigitaLocean) |

A few things worth knowing about how these plans work in practice:

**Single node vs. high availability.** The prices above are per node. A *single node* cluster at the 1 GiB tier costs $15/month — that's your entry point, but it's not highly available (though it does have automatic failover within the node). A *high availability* cluster means a primary plus at least one matching standby node, so the 2 GiB HA entry point is $30/month primary + $30/month standby = $60/month total. You can add or remove standbys at any time, but you can't add a standby to the smallest 1 GiB plan — that one is solo only.

**Traffic is free.** Bandwidth to and from managed databases does not count against your transfer allowance. That's a quietly meaningful detail if you're running a write-heavy cache that's constantly churning data.

**Storage is bundled.** Unlike some other DigitalOcean managed database engines where storage is priced separately per GiB, Valkey plans come with a fixed disk size tied to the plan. You can't independently scale storage — you scale the whole node.

**Premium AMD and Premium Intel variants.** As of April 2026, managed databases support additional Premium AMD and Premium Intel plan options with higher network throughput than the regular plans shown above. If your workload is network-bound rather than memory-bound, those are worth a look during cluster creation.

You can explore the full plan matrix and sign up via the [managed databases pricing page](https://bit.ly/DigitaLocean).

## How to Actually Set Up a Cluster

The mechanics are simple enough that you can do it on a coffee break:

1. **Sign in** to your DigitalOcean account (or create one — new accounts often come with free credit to get started).
2. **Navigate to Databases** in the left sidebar, then **Create**.
3. **Choose Valkey** as the engine.
4. **Pick a region** — note that standby nodes must be in the same region as the primary.
5. **Choose a plan** from the table above. For first experiments, the 1 GiB single-node plan is enough; for anything resembling production, start at 2 GiB with a standby.
6. **Configure optional standby nodes** — zero, one, or two standbys for HA.
7. **Set trusted sources** — the IP ranges allowed to connect. By default the cluster is locked down; you'll add your Droplets' VPC or your office IP here.
8. **Click Create.** Within a few minutes you'll have a connection string, port, and credentials.

Connecting from your application uses a standard Redis-compatible connection string with TLS required. With `valkey-cli`, that's the `--tls` flag. From application code, point your existing Redis client at the provided host and port with TLS enabled and you're done — no client library changes, no API differences.

## Migrating From Redis (or From Another Provider)

If you're already running Redis somewhere — self-hosted, another cloud, or DigitalOcean's old Managed Redis — moving to Valkey is mostly a data migration exercise, not a code change.

**From DigitalOcean's old Managed Caching (Redis).** DigitalOcean provides a documented conversion path from existing Caching clusters to Valkey. Because the wire protocol and command set are compatible, this is the smoothest migration case — same control panel, same connection patterns, just the engine swaps underneath.

**From self-hosted Redis or another cloud.** Use `valkey-cli`'s `MIGRATE` command, or dump-and-restore with `valkey-cli --rdb` followed by restore, or a replication-based cutover where you point a Valkey replica at your existing Redis master, let it sync, then promote. The Valkey project maintains a [migration guide](https://valkey.io/topics/migration/) covering standalone and cluster topologies.

**What's not supported.** A few specific migration paths are explicitly off the table on DigitalOcean Valkey: you can't migrate from AWS ElastiCache using the online migration feature, and you can't continuously migrate one DigitalOcean managed cluster to another DigitalOcean managed cluster (account-to-account). For those cases, fall back to dump-and-restore.

## The Honest Part: Limits and Gotchas

No managed service is complete, and pretending otherwise doesn't help anyone. Here's what DigitalOcean Valkey *doesn't* do, straight from the official limits documentation:

- **No backups, no point-in-time recovery, no restoring from backups.** This is the big one and worth saying twice. Valkey clusters on DigitalOcean do not support backups. If you delete data, it's gone. Design accordingly — treat Valkey as a cache or ephemeral store, not as your system of record. (For persistent storage, use PostgreSQL or MySQL, which do support backups.)
- **No read-only nodes.** Unlike PostgreSQL or MySQL managed clusters, you can't add read replicas to scale read throughput.
- **No native connection pooling** at the server level. You'll want client-side pooling in your application.
- **No query statistics, no current/long-running query view.** Less introspection than some other engines.
- **No third-party ACL clients.** Access control lists are managed through the DigitalOcean control panel and API, not via external tools.
- **No IPv6 trusted sources.** Trusted source rules are IPv4 only for now.
- **You can't resize a cluster while it's being upgraded.** Pick a maintenance window and a resize window that don't overlap.
- **No cluster-to-cluster online migration within DigitalOcean.** Use dump-and-restore.

There's also a connection rate limit worth knowing: each CPU in your cluster can handle up to **200 new connections per second**. Burst beyond that and connections get rejected until the next second. The fix is standard — use a connection pool in your client so you're not opening new connections constantly. On the plus side, the absolute connection ceiling is generous: up to 10,000 simultaneous connections, or 4 per megabyte of memory (whichever is larger). A 4 GiB node can hold 16,384 concurrent connections; a 1 GiB node tops out at 10,000.

A handful of commands are restricted for performance and security reasons — `CONFIG`, `DEBUG`, `BGSAVE`, `SHUTDOWN`, `MIGRATE`, `MONITOR`, `REPLICAOF`, `ACL`, and friends. You won't be able to reconfigure the server at runtime from a client, which is a sensible default for a managed service. Lua scripting (`EVAL`, `EVALSHA`, `FCALL`, etc.) is fully available.

## How It Compares: DigitalOcean Valkey vs. The Alternatives

It's worth being clear about where DigitalOcean's offering sits in the landscape, because "managed Valkey" is becoming a crowded category:

- **AWS ElastiCache for Valkey** — the incumbent for AWS shops. Deeply integrated with the AWS ecosystem, IAM, VPC, and so on. More expensive at the low end, complex to configure, and a heavier operational surface. The right choice if you're already all-in on AWS and want everything in one billing relationship.
- **Google Cloud Memorystore for Valkey** — similar story for GCP-native shops. Tight integration, premium pricing, fewer knobs.
- **Upstash** — serverless, per-request pricing model. Excellent for spiky or low-volume workloads where you don't want to pay for idle capacity. Less suited to steady high-throughput workloads where the per-request math gets expensive fast.
- **Self-hosted Valkey on a VM** — cheapest in raw compute, but you own everything: backups, failover, monitoring, upgrades. Often a false economy for small teams.
- **DigitalOcean Valkey** — sits in the middle. Predictable flat monthly pricing starting at $15, fully managed, simple control panel, no per-request surprise bills, no ecosystem lock-in. The trade-off is the absence of backups (which the hyperscalers do offer) and a smaller feature set than the cloud-native incumbents.

For a startup, indie developer, or small-to-mid-size team that wants managed caching without the AWS-bill-anxiety or the operational drag of self-hosting, DigitalOcean's positioning is genuinely attractive. For an enterprise already running on AWS or GCP with deep integration needs, the hyperscaler offerings probably still win on integration if not on price.

## Pricing in Practice: A Few Scenarios

Abstract pricing tables are fine, but concrete scenarios help. A few common ones:

**The hobby project.** You're running a small web app with a few hundred users and want to cache database query results. The 1 GiB / 1 vCPU plan at **$15/month** is plenty. No standby, no HA — if it goes down for a minute, your app falls back to the database and nobody notices.

**The growing SaaS.** A few thousand users, session storage in Valkey, some computed-result caching. Start at the 2 GiB plan with one standby for HA — **$60/month** total ($30 primary + $30 standby). Add a second standby if you want to survive a double failure.

**The production workhorse.** Heavy traffic, real-time leaderboards, rate limiting for a public API. The 8 GiB / 4 vCPU plan with two standbys — **$360/month** ($120 primary + $120 × 2 standbys). Comfortable headroom for tens of thousands of concurrent connections.

**The analytics-heavy platform.** Large in-memory working set, lots of sorted set operations, hyperloglog cardinality estimates. The 32 GiB plan with two standbys — **$1,440/month** ($480 + $480 × 2). At this scale you'll want to verify you actually need all that RAM in memory vs. tiering some data to disk-backed storage.

A reminder: traffic to and from managed databases is free, so the only variable is the cluster cost itself. No bandwidth surprises.

## A Note on the Redis vs. Valkey Decision

If you're starting fresh and wondering whether to pick Redis or Valkey at all — the answer mostly depends on your licensing comfort and your trajectory. Valkey is BSD-licensed and Linux-Foundation-governed, which means it'll stay open-source indefinitely and won't be subject to a future license change like the one that prompted the fork. Redis has since walked back somewhat and re-open-sourced portions, but the trust gap exists.

For most greenfield workloads today, Valkey is the pragmatic default — same performance, same protocol, same ecosystem, more durable licensing posture. If you have existing Redis infrastructure and tooling that you don't want to disturb, staying on Redis is fine too. The two will likely coexist for years, and most clients work with both interchangeably.

The thing I'd actually decide first is *managed vs. self-hosted*. That decision has way more impact on your day-to-day than Redis vs. Valkey does. If you go managed, DigitalOcean's offering is a solid, fairly-priced option in a crowded field. If you go self-hosted, Valkey is a low-risk swap from Redis.

## Getting Started Without Overthinking It

If you've read this far and want to actually try it, the path is short. The [DigitalOcean sign-up page](https://bit.ly/DigitaLocean) takes a few minutes, new accounts typically come with free credit to play with, and you can have a 1 GiB Valkey cluster running inside of five minutes from there. Point an existing Redis client at it, run a few `SET` and `GET` commands, and see for yourself that nothing in your code needs to change.

If you're coming from DigitalOcean's old Managed Redis, the conversion is documented and low-friction. If you're coming from elsewhere, a dump-and-restore migration gets you moved over an evening. Either way, the goal is the same: stop spending your evenings on cache server maintenance and get back to the part of your job that actually builds something. [👉 Spin up your first Valkey cluster here](https://bit.ly/DigitaLocean).

## Quick FAQ

**Is Valkey really a drop-in replacement for Redis?**
For the vast majority of workloads, yes. The command set, wire protocol, data structures, and client libraries are compatible. There are edge cases with newer Redis-specific features or modules, but standard caching, session storage, and queueing patterns work identically.

**Does DigitalOcean Valkey support backups?**
No. This is the most important limitation to understand going in. Valkey clusters on DigitalOcean do not support backups or point-in-time recovery. Treat it as a cache, not as your system of record.

**Can I add high availability later?**
Yes. You can add or remove standby nodes at any time, as long as you're on a plan that supports standbys (the 1 GiB plan does not). Standbys must be in the same region as the primary.

**What's the connection limit?**
Up to 10,000 simultaneous connections, or 4 per megabyte of memory — whichever is larger. New connection rate is capped at 200 per second per CPU; use client-side connection pooling to handle bursts.

**Can I use my existing Redis client library?**
Yes. Any Redis-compatible client works with Valkey with no code changes — just enable TLS in the connection config, since DigitalOcean Valkey requires encrypted connections.

**How is Valkey billed?**
Hourly, with a monthly cap equal to the plan's monthly price. Traffic to and from managed databases is free and doesn't count against your bandwidth allowance.

**Can I migrate from AWS ElastiCache?**
Not via the online migration feature. Use a dump-and-restore approach instead.

If you're weighing a move, the [👉 managed Valkey product page](https://bit.ly/DigitaLocean) has the current details, and the cluster creation flow walks you through the rest. The hardest part is usually deciding which plan to start with — and if you're not sure, the honest answer is to start one size smaller than you think you need, watch the metrics for a week, and resize up. The whole point of a managed service is that resizing is a click, not a project.

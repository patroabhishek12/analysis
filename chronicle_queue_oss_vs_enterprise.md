# Chronicle Queue: Open Source vs. Chronicle Queue Enterprise

| Dimension | Chronicle Queue (Open Source) | Chronicle Queue Enterprise | Explanation |
|---|---|---|---|
| **License / cost** | Apache 2.0, free, source on GitHub (OpenHFT/Chronicle-Queue) | Commercial license; pricing not public — requires contacting Chronicle Software sales | Apache 2.0 lets you use, modify, and even embed the OSS version commercially for free. Enterprise is a paid product layered on top, sold directly rather than via public price list. |
| **Core single-host functionality** | Full: memory-mapped, off-heap, append-only queue; automatic indexing per message for fast random access; appenders (writers) and tailers (readers) as lightweight Java objects with no broker or TCP connection needed; automatic file rolling (e.g., daily .cq4 files) | Same core engine, plus tuning/controls layered on top | The fundamental "microsecond messaging that stores everything to disk" capability is entirely open source — you don't need Enterprise to get low-latency, durable, single-machine queuing. |
| **Cross-host replication** | Not supported — no built-in way to move data between JVMs on different machines | Core Enterprise feature: TCP/UDP replication between hosts, including active-active replication with fast failover and consistency guarantees during outages | This is the single biggest differentiator. If your architecture needs the queue's data available on more than one machine, Enterprise (or a wrapper you build yourself) is required — OSS has no supported answer. |
| **Encryption** | Not included | Encrypts stored queue files and in-flight messages, aimed at regulatory needs (e.g., GDPR) | Regulated industries (finance, healthcare) needing encryption-at-rest/in-transit for the queue itself need Enterprise. |
| **Tail-latency control** | Good average latency, but higher and less predictable outliers/tail latency under load | Includes specific mechanisms to tighten outliers — e.g., ring buffer support for high throughput on slower filesystems, and reported 99th–99.99th percentile latencies under ~40 microseconds even with replication | If your use case cares about worst-case (p99/p99.9) latency rather than just average latency — common in trading — Enterprise's tuning features matter more than in general-purpose messaging. |
| **Rollover / scheduling controls** | Basic roll-cycle configuration (e.g., roll daily) | Adds timezone-aware rollover scheduling, useful for aligning file boundaries with market/business calendars across regions | Relevant to global trading operations where "the trading day" doesn't align with UTC midnight. |
| **Compression** | Not included | Supports Chronicle Wire Enterprise real-time compression (delta/object-based), reportedly reducing message size 10x or more with minimal added latency | Matters when replicating large volumes over constrained network links. |
| **Cross-language interoperability** | Primarily JVM languages (Java/Kotlin/Scala) | Adds supported interop for sharing queue data with Rust, Python, and C++ | OSS is realistically a Java-ecosystem tool; Enterprise extends reach into polyglot trading/analytics stacks. |
| **Support model** | Community support only (GitHub issues, forums) — no SLA | Commercially supported by Chronicle Software's engineers, with the implication of SLA-backed support (specific terms are only available from sales) | For production financial systems, having a vendor accountable for support/bugs is often a compliance or risk-management requirement, not just a convenience. |
| **Typical adopters** | Individual developers, smaller systems, or teams that only need single-host durability and are comfortable building their own replication/HA layer | Banks, exchanges, and trading firms needing multi-host HA, regulatory-grade encryption, and vendor support | The split roughly mirrors "get the core low-latency engine free" vs. "pay for the operational/compliance features an institution needs to run it in production at scale." |

## Resiliency & Recovery: OSS vs. Enterprise

| Dimension | Chronicle Queue (Open Source) | Chronicle Queue Enterprise | Explanation |
|---|---|---|---|
| **Resiliency (fault domain)** | Durable against a process/JVM crash only — data is memory-mapped straight to disk, so a restart doesn't lose it. No resilience against a host failure: there's no second copy anywhere, so a dead disk or machine takes the queue down with it | Adds host-level resiliency via active-active replication — a live, continuously-synced copy on a second (or more) machine, so a single host failure doesn't take the system down | OSS protects you from your process crashing; Enterprise protects you from your machine dying. That's the core resiliency gap. |
| **Recovery Point Objective (RPO — data you could lose)** | No built-in RPO story; recovery depends entirely on your own backup/snapshot strategy, since there's no replica to fall back to | Replication runs at microsecond-level lag (commonly ~10–50µs even under replication) with support for acknowledged replication, where a write isn't considered safe until the replica confirms it — pushing RPO close to zero | With Enterprise, at most the sliver of unacknowledged in-flight data is at risk if a host fails. With OSS, you could lose everything since your last backup. |
| **Recovery Time Objective (RTO — time to get back up)** | Manual recovery — rebuild/restart on the same or replacement host and reload from disk; RTO depends on your ops process (minutes or more) | Built for fast/automatic failover to the replica, so the system keeps serving with little to no manual intervention — RTO is close to instant | Enterprise turns a host failure into a near-transparent failover event; OSS turns it into an ops incident. |

## Bottom line

The free, open-source Chronicle Queue gives you the actual core value proposition — a broker-less, memory-mapped, microsecond-latency, durable log — with no cost and no vendor lock-in. Chronicle Queue Enterprise doesn't change that core engine; it adds the things a regulated, multi-datacenter, production trading system typically needs on top of it: cross-host replication with failover, encryption, tail-latency tuning, compression, cross-language support, and vendor-backed support. For a single-host, single-language use case, OSS is often sufficient; once you need HA across machines or compliance-grade encryption, Enterprise becomes close to mandatory since OSS has no supported alternative path to get there.

## Sources

- [What is Chronicle Queue Enterprise? — Chronicle Software](https://chronicle.software/tech-hub/technical-information/chronicle-queue/what-is-chronicle-queue)
- [Chronicle Queue Enterprise — Chronicle Software](https://chronicle.software/products/chronicle-queue)
- [Chronicle Queue Enterprise — Chronicle Software](https://chronicle.software/queue-enterprise/)
- [Chronicle Queue Enterprise (Old) — Chronicle Software](https://chronicle.software/queue-enterprise-old/)
- [Acknowledged replication of Queue across the network — Chronicle Software](https://chronicle.software/acknowledged-replication-of-queue-across-the-network/)
- [Reducing Tail Latencies with Chronicle Queue Enterprise — Chronicle Software](https://chronicle.software/reducing-tail-latencies-with-chronicle-queue-enterprise/)
- [GitHub - OpenHFT/Chronicle-Queue](https://github.com/OpenHFT/Chronicle-Queue)
- [Chronicle-Queue How it works — GitHub docs](https://github.com/OpenHFT/Chronicle-Queue/blob/ea/docs/How_it_works.adoc)
- [Chronicle-Queue LICENSE — GitHub](https://github.com/OpenHFT/Chronicle-Queue/blob/ea/LICENSE)
- [Introduction to Chronicle Queue — Baeldung](https://www.baeldung.com/java-chronicle-queue)

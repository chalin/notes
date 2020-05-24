Fauna Blog | Spanner vs. Calvin: distributed consistency at scale

## Spanner vs. Calvin: distributed consistency at scale

 Daniel J. Abadi  April 06, 2017

![9d396180-1aa8-11e7-9bac-249811510f34.png](../_resources/1e03fe6fdd8e5e2e4eedd1655dc3fdfd.png)

*[Daniel J. Abadi](http://cs-www.cs.yale.edu/homes/dna/) is an Associate Professor at Yale University. He does research primarily in database system architecture and implementation. He received a Ph.D. from MIT and a M.Phil from Cambridge.*

## Introduction

In 2012, two research papers were published that described the design of geographically replicated, consistent, ACID compliant, transactional database systems. Both papers criticized the proliferation of NoSQL database systems that compromise replication consistency and transactional support, and argue that it is possible to build extremely scalable, geographically-replicated systems without giving up consistency and transactional support.

>

> It is possible to build extremely scalable, geographically-replicated systems without giving up consistency and transactional support.

The first of these papers was the [Calvin paper](http://cs-www.cs.yale.edu/homes/dna/papers/calvin-sigmod12.pdf), published in SIGMOD 2012. A few months later, Google published their [Spanner paper](https://static.googleusercontent.com/media/research.google.com/en//archive/spanner-osdi2012.pdf) in OSDI 2012. Both of these papers have been cited many hundreds of times and have influenced the design of several modern “NewSQL” systems, including [FaunaDB](https://fauna.com/).

Recently, Google released a beta version of their Spanner implementation, available to customers who use Google Cloud Platform. This development has excited many users seeking to build on Google’s cloud, since they now have a reliably scalable and consistent transactional database system to use as a foundation.

However, the availability of Spanner outside of Google has also brought it more scrutiny: what are its technical advantages, and what are its costs? Even though it has been five years since the Calvin paper was published, it is only now that the database community is asking me to directly compare and contrast the technical designs of Calvin and Spanner.

>

> The availability of Spanner outside of Google has also brought it more scrutiny: what are its technical advantages, and what are its costs?

The goal of this post is to do exactly that—compare the architectural design decisions made in these two systems, and specifically focus on the advantages and disadvantages of these decisions against each other as they relate to performance and scalability.

This post is focused on the protocols described in the original papers from 2012. Although the publicly available versions of these systems likely have deviated from the original papers, the core architectural distinctions remain the same.

## The CAP theorem in context

Before we get started, allow me to suggest the following: Ignore the CAP theorem in the context of this discussion. Just forget about it. It’s not relevant for the type of modern architectural deployments discussed in this post where network partitions are rare.

Both Spanner and Calvin replicate data across independent regions for high availability. And both Spanner and Calvin are technically CP systems from CAP: they guarantee 100% consistency (serializability, linearizability, etc.) across the entire system. Yes, when there is a network partition, both systems make slight compromises on availability, but partitions are rare enough in practice that developers on top of both systems can assume a fully-available system to many 9s of availability.

>

> Both Spanner and Calvin are technically CP systems from CAP, but partitions are rare enough in practice that developers can assume a fully-available system to many 9s of availability.

If you didn’t believe me in 2010 when I explained the [shortfalls of using CAP](http://dbmsmusings.blogspot.co.il/2010/04/problems-with-cap-and-yahoos-little.html) to understand the practical consistency and availability properties of modern systems, maybe you will believe the author of the CAP theorem himself, Eric Brewer, who [recommends against](https://static.googleusercontent.com/media/research.google.com/en//pubs/archive/45855.pdf) analyzing Spanner through the lens of the CAP theorem.

## Ordering transactions in time

Let us start our comparison of Calvin and Spanner with the most obvious difference between them: Spanner’s use of “TrueTime” vs. Calvin’s use of “preprocessing” (or “sequencing” in the language of the original paper) for transaction ordering. In fact, most of the other differences between Spanner and Calvin stem from this fundamental choice.

A serializable system provides a notion of transactional ordering. Even though many transactions may be executed in parallel across many CPUs and many servers in a large distributed system, the final state (and all observable intermediate states) must be as if each transaction was processed one-by-one. If no transactions touch the same data, it is trivial to process them in parallel and maintain this guarantee.

>

> A serializable system provides a notion of transactional ordering. Even though many transactions may be executed in parallel, the final state must be as if each transaction was processed one-by-one.

However, if the transactions read or write each other’s data, then they must be ordered against each other, with one considered earlier than the other. The one considered “later” must be processed against a version of the database state that includes the writes of the earlier one. In addition, the one considered “earlier” must be processed against a version of the database state that excludes the writes of the later one.

## Locking and logging

Spanner uses TrueTime for this transaction ordering. Google famously uses a combination of GPS and atomic clocks in all of their regions to synchronize time to within a known uncertainty bound. If two transactions are processed during time periods that do not have overlapping uncertainty bounds, Spanner can be certain that the later transaction will see all the writes of the earlier transaction.

>

> Google famously uses a combination of GPS and atomic clocks in all of their regions to synchronize time to within a known uncertainty bound.

Spanner obtains write locks within the data replicas on all the data it will write before performing any write. If it obtains all the locks it needs, it proceeds with all of its writes and then assigns the transaction a timestamp at the end of the uncertainty range of the coordinator server for that transaction. It then waits until this later timestamp has definitely passed for all servers in the system (which is the entire length of the uncertainty range) and then releases locks and commits the transaction. Future transactions will get later timestamps and see all the writes of this earlier transaction.

Thus, in Spanner, every transaction receives a timestamp based on the actual time that it committed, and this timestamp is used to order transactions. Transactions with later timestamps see all the writes of transactions with earlier timestamps, with locking used to enforce this guarantee.

>

> In Spanner, every transaction receives a timestamp based on the actual time that it committed, and this timestamp is used to order transactions.

In contrast, Calvin uses preprocessing to order transactions. All transactions are inserted into a distributed, replicated log before being processed. Clients submit transactions to the preprocessing layer of their local region, which then submits these transactions to the global log via a cross-region consensus process like Paxos. This is similar to a write-ahead log in a traditional, non-distributed database. The order that the transactions appear in this log is the official transaction ordering. Every replica reads from their local copy of this replicated log and processes transactions in a way that guarantees that their final state is equivalent to what it would have been had every transaction in the log been executed one-by-one.

>

> Calvin uses preprocessing to order transactions. All transactions are inserted into a distributed, replicated log before being processed.

## Replication overhead

The design difference between TrueTime vs. preprocessing directly leads to a difference in how the systems perform replication. In Calvin, the replication of transactional input during preprocessing is the only replication that is needed. Calvin uses a deterministic execution framework to avoid *all* cross-replication communication during normal (non-recovery mode) execution aside from preprocessing. Every replica sees the same log of transactions and guarantees not only a final state equivalent to executing the transactions in this log one-by-one, but also a final state equivalent to every other replica.

>

> Calvin uses a deterministic execution framework to avoid all cross-replication communication during normal execution aside from preprocessing.

This requires the preprocessor to analyze the transaction code and “pre-execute” any nondeterministic code (e.g. calls to `sys.random()` or `time.now()`). Once all code within a transaction is deterministic, a replica can safely focus on just processing the transactions in the log in the correct order without concern for diverging with the other replicas. How this effects what types of transactions Calvin supports is discussed at the end of this post.

In contrast, since Spanner does not do any transaction preprocessing, it can only perform replication after transaction execution. Spanner performs this replication via a cross-region Paxos process.

>

> Since Spanner does not do any transaction preprocessing, it can only perform replication after transaction execution. Spanner performs this replication via a cross-region Paxos process.

## The cost of two-phase commit

Another key difference between Spanner and Calvin is how they commit multi-partitioned transactions. Both Calvin and Spanner partition data into separate shards that may be stored on separate machines that fail independently from each other. In order to guarantee transaction atomicity and durability, any transaction that accesses data in multiple partitions must go through a commit procedure that ensures that every partition successfully processed the part of the transaction that accessed data in that partition.

Since machines may fail at any time, including during the commit procedure, this process generally takes two rounds of communication between the partitions involved in the transaction. This two-round commit protocol is called “two-phase commit” and is used in almost every ACID-compliant distributed database system, including Spanner. This two-phase commit protocol can often consume the majority of latency for short, simple transactions since the actual processing time of the transaction is much less than the delays involved in sending and receiving two rounds of messages over the network.

The cost of two-phase commit is particularly high in Spanner because the protocol involves three forced writes to a log that cannot be overlapped with each other. In Spanner, every write to a log involves a cross-region Paxos agreement, so the latency of two-phase commit in Spanner is at least equal to three times the latency of cross-region Paxos.

>

> In Spanner, every write to a log involves a cross-region Paxos agreement, so the latency of two-phase commit in Spanner is at least equal to three times the latency of cross-region Paxos.

## Determinism is durability

In contrast to Spanner, Calvin leverages deterministic execution to avoid two-phase commit. Machine failures do not cause transactional aborts in Calvin.

>
> Calvin leverages deterministic execution to avoid two-phase commit.

Instead, after a failure, the machine that failed in Calvin re-reads the input transaction log from a checkpoint, and deterministically replays it to recover its state at the time of the failure. It can then continue on from there as if nothing happened. As a result, the commit protocol does not need to worry about machine failures during the protocol, and can be performed in a single round of communication (and in some cases, zero rounds of communication—see the original paper for more details).

## Performance implications

At this point, I think I have provided enough details to make it possible to present a theoretical comparison of the bottom line performance of Calvin vs. Spanner for a variety of different types of requests. This comparison assumes a perfectly optimized and implemented version of each system.

### *Transactional write latency*

A transaction that is “non-read-only” writes at least one value to the database state. In Calvin, such a transaction must pay the latency cost of preprocessing, which is roughly the cost of running cross-region Paxos to agree to append the transaction to the log. After this is complete, the remaining latency is the cost of processing the transaction itself, which includes the zero or one-phase commit protocol for distributed transactions.

In Spanner, there is no preprocessing latency, but it still has to pay the cost of cross-region Paxos replication at commit time, which is roughly equivalent to the Calvin preprocessing latency. Spanner also has to pay the commit wait latency discussed above (which is the size of the time uncertainty window), but this can be overlapped with replication. It also pays the latency of two-phase commit for multi-partition transactions.

Thus, Spanner and Calvin have roughly equivalent latency for single-partition transactions, but Spanner has worse latency than Calvin for multi-partition transactions due to the extra phases in the transaction commit protocol.

>

> Spanner and Calvin have roughly equivalent latency for single-partition transactions, but Spanner has worse latency than Calvin for multi-partition transactions due to the extra phases in the transaction commit protocol.

### *Snapshot read latency*

Both Calvin and Spanner keep around older versions of data and read data at a requested earlier timestamp from a local replica without any Paxos-communication with the other replicas.

Thus, both Calvin and Spanner can achieve very low snapshot-read latency.
>
> Both Calvin and Spanner can achieve very low snapshot-read latency.

### *Transactional read latency*

Read-only transactions do not write any data, but they must be linearizable with respect to other transactions that write data. In practice, Calvin accomplishes this via placing the read-only transaction in the preprocessor log. This means that a read-only transaction in Calvin must pay the cross-region replication latency. In contrast, Spanner only needs to submit the read-only transaction to the leader replica(s) for the partition(s) that are accessed in order to get a global timestamp (and therefore be ordered relative to concurrent transactions). Therefore, there is no cross-region Paxos latency—only the commit time (uncertainty window) latency.

Thus, Spanner has better latency than Calvin for read-only transactions submitted by clients that are physically close to the location of the leader servers for all partitions accessed by that transaction.

>

> Spanner has better latency than Calvin for read-only transactions submitted by clients that are physically close to the location of the leader servers.

## Scalability

Both Spanner and Calvin are both (theoretically) roughly-linearly scalable for transactional workloads for which it is rare for concurrent transactions to be accessing the same data. However, major differences begin to present themselves as the conflict rate between concurrent transactions starts to increase.

Both Spanner and Calvin, as presented in the paper, use locks to prevent concurrent transactions from interfering with each other in impermissible ways. However, the amount of time they hold locks for an identical transaction is substantially different. Both systems need to hold locks during the commit protocol. However, since Calvin’s commit protocol is shorter than Spanner’s, Calvin reduces the lock hold time at the end of the transaction. On the flip side, Calvin acquires all locks that it will need at the beginning of the transaction, whereas Spanner performs all reads for a transaction before acquiring write locks. Therefore, Spanner reduces lock time at the beginning of the transaction.

However, this latter advantage for Spanner is generally outweighed by the former disadvantage, since, as discussed above, the latency of two-phase commit in Spanner involves at least three iterations of cross-region Paxos. Furthermore, Spanner has an additional major disadvantage relative to Calvin in lock-hold time: Spanner must also hold locks during replication (which, as mentioned above, is also a cross-region Paxos process). The farther apart the regions, the larger the latency of this replication, and therefore, the longer Spanner must hold locks.

>

> Spanner must hold locks during replication. The farther apart the regions, the larger the latency of replication, and therefore, the longer Spanner must hold the locks.

In contrast, Calvin does its replication during preprocessing, and therefore does not need to hold locks during replication. This leads to Calvin holding locks for much shorter periods of time than Spanner, allowing it to process more conflicting concurrent transactions in parallel

>
> Calvin can process more conflicting concurrent transactions in parallel.

A second difference that can affect scalability is the following: Calvin requires only a single Paxos group for replicating the input log. In contrast, Spanner requires one independent Paxos group per shard, with proportionally higher overhead.

Calvin has higher throughput scalability than Spanner for transactional workloads where concurrent transactions access the same data. This advantage increases with the distance between datacenters.

>

> Calvin has higher throughput scalability than Spanner for transactional workloads where concurrent transactions access the same data.

## Limitations on transaction types

In order to implement deterministic transaction processing, Calvin requires the preprocessor to analyze transactions and potentially “pre-execute” any non-deterministic code to ensure that replicas do not diverge. This implies that the preprocessor requires the entire transaction to be submitted at once. This highlights another difference between Calvin and Spanner—while Spanner theoretically allows arbitrary client-side interactive transactions (that may include external communication), Calvin supports a more limited transaction model.

In particular, the client-side interactive transaction model of SQL, also known as session transactions, is a poor fit for Calvin. The public version of Spanner supports client-side interactive transactions, although mutations must explicitly reference the primary key of each row.

>

> The client-side interactive transaction model of SQL is a poor fit for Calvin.

There are some subtle but interesting differences between Calvin and Spanner in rare situations where every single replica for a shard is unavailable, or if all but one are unavailable, but these differences are out of scope for this post.

## Conclusion

I’m biased in favor of Calvin, but in going through this exercise, I found it very difficult to find cases where an ideal implementation of Spanner theoretically outperforms an ideal implementation of Calvin. The only place where I could find that Spanner has a clear performance advantage over Calvin is for latency of read-only transactions submitted by clients that are physically close to the location of the leader servers for the partitions accessed by that transaction. Since any complex transaction is likely to touch multiple partitions, this is almost impossible to guarantee in a real-world setting.

>

> It is very difficult to find cases where an ideal implementation of Spanner would outperform an ideal implementation of Calvin.

Many real-world workloads do not require client-side interactive transactions, only need transactional support for writes, and are satisfied performing reads against snapshots (after all, this is the default isolation model of many SQL systems). Therefore, It seems to me that Calvin is the better fit for modern applications.

*FaunaDB is a globally replicated, strongly consistent operational database, inspired by Calvin. To learn more about how FaunaDB is built, request our [technical white paper](https://fauna.com/whitepaper).*
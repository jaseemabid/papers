# Kafka: a Distributed Messaging System for Log Processing

Kafka is a distributed messaging system for collecting and delivering high
volumes of log data with low latency. The paper describes a reasonably simple
model designed for very low latency and high throughput.

The key abstraction in Kafka is a `topic`. Producers publish their records to a
topic, and consumers subscribe to one or more topics. A topic is divided into
multiple partitions and each `broker` stores one or more of those partitions.
Producers append records to these logs and consumers subscribe to changes. Each
record is a key/value pair. The key is used for assigning the record to a log
partition (unless the publisher specifies the partition directly).

Kafka embraces simple disk based persistence a quite elegantly. Of the many
interesting techniques described, one of particular interest is that individual
messages published to a topic doesn't have any primary key. Messages are written
to segment files and each message is addressed by its logical offset in the
segment file. This avoids the overhead of maintaining auxiliary, seek intensive
random access index structures that map the message ids to the actual message
locations. This makes writes fast and reads even faster because it can send a
block of a segment file over the network without even copying it to userspace
with the sendfile API. This model can also make use of the OS page cache since
consumers are typically requesting for blocks written by producers very
recently.

Kafka brokers are stateless. Unlike most other messaging systems, in Kafka, the
information about how much each consumer has consumed is not maintained by the
broker, but by the consumer itself. This make consumer management as well as
message deletion trickier. Consumers coordinate among themselves with zookeeper.

In general, Kafka only guarantees at-least-once delivery. If an application
cares about duplicates, it must add its own de-duplication logic, either using
the offsets or some unique key within the message. Kafka guarantees that
messages from a single partition are delivered to a consumer in order. However,
there is no guarantee on the ordering of messages coming from different
partitions.

Interestingly there is no fault tolerance with replication. If a broker goes
down, any message stored on it not yet consumed becomes unavailable. If the
storage system on a broker is permanently damaged, any unconsumed message is
lost forever. Paper leaves
[replication](http://kafka.apache.org/documentation.html#replication) as future
work, but this seems to be implemented now.

Details about the system in production at linkedIn and some benchmarks follow.
Kafka is compared to ActiveMQ and RabbitMQ. Frequency numbers are orders of
magnitude higher than that of ActiveMQ, and at least 2 times higher than
RabbitMQ. Publisher throughput is significantly higher for Kafka because the
Kafka producer doesn't wait for acknowledgments from the broker, while ActiveMQ
and RabbitMQ does. Paper argues that this is a valid optimization for the log
aggregation case, as data must be sent asynchronously to avoid introducing any
latency into the live serving of traffic. Batching of messages and efficient
storage format also gives Kafka some advantage. Kafka consumers outperform
primarily due to stateless nature and efficient transport with sendfile.

## References
  - [Paper](http://research.microsoft.com/en-us/um/people/srikanth/netdb11/netdb11papers/netdb11-final12.pdf)
  - [Kafka Docs](http://kafka.apache.org/documentation.html)
  - [The Log: What every software engineer should know about real-time data's unifying abstraction | LinkedIn Engineering](https://engineering.linkedin.com/distributed-systems/log-what-every-software-engineer-should-know-about-real-time-datas-unifying)
  - [Benchmarking Apache Kafka: 2 Million Writes Per Second (On Three Cheap Machines) | LinkedIn Engineering](https://engineering.linkedin.com/kafka/benchmarking-apache-kafka-2-million-writes-second-three-cheap-machines)

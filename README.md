# kafka-streams

https://www.confluent.io/blog/crossing-streams-joins-apache-kafka/

There are two main abstractions in the Streams API: A KStream is a stream of key-value pairs—a similar model as used for a Kafka topic. The records in a KStream either come directly from a topic or have gone through some kind of transformation—for example there is a filter method that takes a predicate and returns another KStream that only contains those elements that satisfy the predicate. KStreams are stateless, but they allow for aggregation by turning them into the other core abstraction: a KTable, which is often described as a “changelog stream.” A KTable holds the latest value for a given message key and reacts automatically to newly incoming messages.

A nice example that juxtaposes KStream and KTable is counting visits to a website by unique IP addresses. Let’s assume we have a Kafka topic containing messages of the following type: (key=IP, value=timestamp). A KStream contains all visits by all IPs, even if the IP is recurring. A count on such a KStream sums up all visits to a site including multiple visits from the same IP. A KTable, on the other hand, only contains the latest message and a count on the KTable represents the number of distinct IP addresses that visited the site.

Joins
Taking a leaf out of SQLs book, Kafka Streams supports three kinds of joins:

Inner Joins: Emits an output when both input sources have records with the same key.

Left Joins: Emits an output for each record in the left or primary input source. If the other source does not have a value for a given key, it is set to null.

Outer Joins: Emits an output for each record in either input source. If only one source contains a key, the other is null.

Another important aspect to consider are the input types. The following table shows which operations are permitted between KStreams and KTables:

Inner Joins: Emits an output when both input sources have records with the same key.

Left Joins: Emits an output for each record in the left or primary input source. If the other source does not have a value for a given key, it is set to null.

Outer Joins: Emits an output for each record in either input source. If only one source contains a key, the other is null.


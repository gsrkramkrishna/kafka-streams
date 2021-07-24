# kafka-streams

https://www.confluent.io/blog/crossing-streams-joins-apache-kafka/

There are two main abstractions in the Streams API: A KStream is a stream of key-value pairs—a similar model as used for a Kafka topic. The records in a KStream either come directly from a topic or have gone through some kind of transformation—for example there is a filter method that takes a predicate and returns another KStream that only contains those elements that satisfy the predicate. KStreams are stateless, but they allow for aggregation by turning them into the other core abstraction: a KTable, which is often described as a “changelog stream.” A KTable holds the latest value for a given message key and reacts automatically to newly incoming messages.

A nice example that juxtaposes KStream and KTable is counting visits to a website by unique IP addresses. Let’s assume we have a Kafka topic containing messages of the following type: (key=IP, value=timestamp). A KStream contains all visits by all IPs, even if the IP is recurring. A count on such a KStream sums up all visits to a site including multiple visits from the same IP. A KTable, on the other hand, only contains the latest message and a count on the KTable represents the number of distinct IP addresses that visited the site.

<b>Joins:</b><br>
Taking a leaf out of SQLs book, Kafka Streams supports three kinds of joins:

<b>Inner Joins:</b><br> Emits an output when both input sources have records with the same key.

<b>Left Joins:</b><br> Emits an output for each record in the left or primary input source. If the other source does not have a value for a given key, it is set to null.

<b>Outer Joins:</b><br> Emits an output for each record in either input source. If only one source contains a key, the other is null.

Another important aspect to consider are the input types. The following table shows which operations are permitted between KStreams and KTables:

<b>Inner Joins:</b><br> Emits an output when both input sources have records with the same key.

<b>Left Joins:</b><br> Emits an output for each record in the left or primary input source. If the other source does not have a value for a given key, it is set to null.

<b>Outer Joins:</b><br> Emits an output for each record in either input source. If only one source contains a key, the other is null.

==========================================================================================

The <b>stream-table duality</b> describes the close relationship between streams and tables.

Stream as Table: A stream can be considered a changelog of a table, where each data record in the stream captures a state change of the table. A stream is thus a table in disguise, and it can be easily turned into a “real” table by replaying the changelog from beginning to end to reconstruct the table. Similarly, aggregating data records in a stream will return a table. For example, we could compute the total number of pageviews by user from an input stream of pageview events, and the result would be a table, with the table key being the user and the value being the corresponding pageview count.
Table as Stream: A table can be considered a snapshot, at a point in time, of the latest value for each key in a stream (a stream’s data records are key-value pairs). A table is thus a stream in disguise, and it can be easily turned into a “real” stream by iterating over each key-value entry in the table.<br>

Let’s illustrate this with an example. Imagine a table that tracks the total number of pageviews by user (first column of diagram below). Over time, whenever a new pageview event is processed, the state of the table is updated accordingly. Here, the state changes between different points in time – and different revisions of the table – can be represented as a changelog stream (second column).

../_images/streams-table-duality-02.jpg
Because of the stream-table duality, the same stream can be used to reconstruct the original table (third column):

../_images/streams-table-duality-03.jpg


# kafka-streams

https://www.confluent.io/blog/crossing-streams-joins-apache-kafka/

There are two main abstractions in the Streams API: A KStream is a stream of key-value pairs—a similar model as used for a Kafka topic. The records in a KStream either come directly from a topic or have gone through some kind of transformation—for example there is a filter method that takes a predicate and returns another KStream that only contains those elements that satisfy the predicate. KStreams are stateless, but they allow for aggregation by turning them into the other core abstraction: a KTable, which is often described as a “changelog stream.” A KTable holds the latest value for a given message key and reacts automatically to newly incoming messages.

A nice example that juxtaposes KStream and KTable is counting visits to a website by unique IP addresses. Let’s assume we have a Kafka topic containing messages of the following type: (key=IP, value=timestamp). A KStream contains all visits by all IPs, even if the IP is recurring. A count on such a KStream sums up all visits to a site including multiple visits from the same IP. A KTable, on the other hand, only contains the latest message and a count on the KTable represents the number of distinct IP addresses that visited the site.

<b>Joins:</b><br>
Taking a leaf out of SQLs book, Kafka Streams supports three kinds of joins:<br>

<img src="https://cdn.confluent.io/wp-content/uploads/inner-left-outer.jpg" width="460" height="345"> <br><br>

<b>Inner Joins:</b><br> Emits an output when both input sources have records with the same key.

<b>Left Joins:</b><br> Emits an output for each record in the left or primary input source. If the other source does not have a value for a given key, it is set to null.

<b>Outer Joins:</b><br> Emits an output for each record in either input source. If only one source contains a key, the other is null.

Another important aspect to consider are the input types. The following table shows which operations are permitted between KStreams and KTables:<br>

<table>
<tr>
<td style="width: 115px;"><b>Primary Type &nbsp; &nbsp;&nbsp;</b></td>
<td style="width: 139px;"><b>Secondary Type&nbsp;</b></td>
<td style="width: 89px;"><b>Inner Join</b></td>
<td style="width: 78px;"><b>Left Join</b></td>
<td style="width: 96px;"><b>Outer Join</b></td>
</tr>
<tr>
<td style="width: 115px;"><span style="font-weight: 400;">KStream</span></td>
<td style="width: 139px;"><span style="font-weight: 400;">KStream</span></td>
<td style="width: 89px;"><span style="font-weight: 400;">Supported</span></td>
<td style="width: 78px;"><span style="font-weight: 400;">Supported</span></td>
<td style="width: 96px;"><span style="font-weight: 400;">Supported</span></td>
</tr>
<tr>
<td style="width: 115px;"><span style="font-weight: 400;">KTable</span></td>
<td style="width: 139px;"><span style="font-weight: 400;">KTable</span></td>
<td style="width: 89px;"><span style="font-weight: 400;">Supported</span></td>
<td style="width: 78px;"><span style="font-weight: 400;">Supported</span></td>
<td style="width: 96px;"><span style="font-weight: 400;">Supported</span></td>
</tr>
<tr>
<td style="width: 115px;"><span style="font-weight: 400;">KStream</span></td>
<td style="width: 139px;"><span style="font-weight: 400;">KTable</span></td>
<td style="width: 89px;"><span style="font-weight: 400;">Supported</span></td>
<td style="width: 78px;"><span style="font-weight: 400;">Supported</span></td>
<td style="width: 96px;"><span style="font-weight: 400;">N/A</span></td>
</tr>
<tr>
<td style="width: 115px;"><span style="font-weight: 400;">KStream</span></td>
<td style="width: 139px;"><span style="font-weight: 400;">Global KTable</span></td>
<td style="width: 89px;"><span style="font-weight: 400;">Supported</span></td>
<td style="width: 78px;"><span style="font-weight: 400;">Supported</span></td>
<td style="width: 96px;"><span style="font-weight: 400;">N/A</span></td>
</tr>
</table>

The <b>stream-table duality</b> describes the close relationship between streams and tables.<br>

<b>Stream as Table:</b><br> A stream can be considered a changelog of a table, where each data record in the stream captures a state change of the table. A stream is thus a table in disguise, and it can be easily turned into a “real” table by replaying the changelog from beginning to end to reconstruct the table. Similarly, aggregating data records in a stream will return a table. For example, we could compute the total number of pageviews by user from an input stream of pageview events, and the result would be a table, with the table key being the user and the value being the corresponding pageview count.<br>

<b>Table as Stream:</b><br> A table can be considered a snapshot, at a point in time, of the latest value for each key in a stream (a stream’s data records are key-value pairs). A table is thus a stream in disguise, and it can be easily turned into a “real” stream by iterating over each key-value entry in the table.<br>

Let’s illustrate this with an example. Imagine a table that tracks the total number of pageviews by user (first column of diagram below). Over time, whenever a new pageview event is processed, the state of the table is updated accordingly. Here, the state changes between different points in time – and different revisions of the table – can be represented as a changelog stream (second column).<br>

<img src="https://docs.confluent.io/platform/current/_images/streams-table-duality-02.jpg" width="460" height="345"> <br><br>


Because of the stream-table duality, the same stream can be used to reconstruct the original table (third column):<br>

<img src="https://docs.confluent.io/platform/current/_images/streams-table-duality-03.jpg" width="460" height="345"><br><br>

The same mechanism is used, for example, to replicate databases via change data capture (CDC) and, within Kafka Streams, to replicate its so-called state stores across machines for fault tolerance. The stream-table duality is such an important concept for stream processing applications in practice that Kafka Streams models it explicitly via the KStream and KTable abstractions, which we describe in the next sections.<br>


<b>GlobalKTable</b><br>

Like a <b>KTable</b>, a <b>GlobalKTable</b> is an abstraction of a <b>changelog stream</b>, where each data record represents an update.<br>

A GlobalKTable differs from a KTable in the data that they are being populated with, i.e. which data from the underlying Kafka topic is being read into the respective table. Slightly simplified, imagine you have an input topic with 5 partitions. In your application, you want to read this topic into a table. Also, you want to run your application across 5 application instances for maximum parallelism.<br>

If you read the input topic into a KTable, then the “local” KTable instance of each application instance will be populated with data from only 1 partition of the topic’s 5 partitions.<br>
If you read the input topic into a GlobalKTable, then the local GlobalKTable instance of each application instance will be populated with data from all partitions of the topic.<br>

<b>Benefits of global tables:</b><br>

&#9679; &emsp; More convenient and/or efficient joins: Notably, global tables allow you to perform star joins, they support “foreign-key” lookups (i.e., you can lookup data in the table not just by record key, but also by data in the record values), and they are more efficient when chaining multiple joins. Also, when joining against a global table, the input data does not need to be co-partitioned.<br>
&#9679; &emsp;</p>Can be used to “broadcast” information to all the running instances of your application.<br>

<b>Downsides of global tables:</b><br>

&#9679; &emsp; Increased local storage consumption compared to the (partitioned) KTable because the entire topic is tracked.<br>
&#9679; &emsp; Increased network and Kafka broker load compared to the (partitioned) KTable because the entire topic is read.



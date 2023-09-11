# SCARY

SCARY is a software tool aimed at providing infrastructure owners the ability to mimic production workloads to a secondary server under test. The outcomes are recorded data on the execution of production queries and the execution on the target server. With this data, infrastructure owners are able to make informed choices about changing their production environments on the only benchmark that matters, their production workload.

The target server can be different hardware, configuration, maybe a version bump, or maybe a compatible product.

## Terminology

* Base - the server being treated as a reference point. Here is where the queries come from.
* Base Secondary - a hot spare of the current Base server. SCARY will use this server when needing further information that involves load, like executing the query and getting exact timing and query plan information.
* Target - the server being tested against.
* Recording - data generated by SCARY goes into a recording server. This could be the same as a Base or Target, but a 3rd location may be more suitable.

## Types of comparisons

Types of comparisons can include:

1. MariaDB upgrades (major or minor)
1. MySQL / MariaDB comparisons (and others?)
1. Effects of changing global variables (underrated need; hard to otherwise accomplish)
1. Effects of a different hardware/software environment
1. Effects of partitioning, System Versioned tables, data type changes, user plugins

## Supported Versions:

Base/Base Secondary:
* MySQL-5.0+; or
* any MariaDB version

Target Version:
* MariaDB-10.2+ currently. EXPLAIN FORMAT=JSON is being used to record. We aim to lift this limitation.

Recording:
* MariaDB-10.2+

With some testing around table syntax compatibility, this limitation may be removed.

## Components

### Agent

The initial implementation does a `SELECT FROM INFORMATION_SCHEMA.PROCESSLIST` on the Base server. It's envisaged that after some time, the sampling will be sufficient to be considered a full representation of the Base server.

Limitations:
* the Sending Data state is used to identify SELECT queries only. Repeating update queries isn't safe enough.

### SCARY (Processor)

The SCARY Processor take the data from the agent, stores this to the Recording server, and dispatches queries to the Target Server. As data on the Base may be incomplete, missing results and complete timing/query information.

### Kafka Message Log

Communication from the Agent to the SCARY Processor is via a Kafka Message Log. This is to ensure a reliable persistent message data that makes the Agent free of complexity and delays, and provides a work-queue buffer for the SCARY processor.

### Other sources

It's hoped other proxy implementations, like ProxySQL and MaxScale, can be extended to provide a rich set of data on proxied queries, which will limit the SCARY Processor to performing the Target-based queries and recording.

A message queue (via Kafka) from SCARY to the Agent and other sources will include a message digest of queries that have sufficient sampling and need not be recorded further.

### Recording Server

Data is stored on the Recording Server in a number of tables.

Test: start/stop time and description test being run.
Globalvars: global variable information at the start/finish of the test (for consistency checks), of both Base and Target servers
Globalstatus: global status information at the start/finish of the test of both Base and Target servers
Queries: the recorded queries, their execution times and query plan on both Base and Target servers.
Query: digest information of the query and the number of further desired samples of this query.

## FAQ

Q. Why SCARY?
A. SQL Comparator Analysis, Reassure Yourself (SCARY), but really software changes are SCARY, so lets own the word.

Q. How did it happen?
A. Its been on the list as something that was needed for a while. As many know, production workloads are hard to reproduce, so we have to accept them as being the only valid test. With funding from Wikimedia Foundation the work started from the ground up under an open source license.

Q. How is it licensed?
A. SCARY components in this repository are available under the APACHE-2.0 license.

Q. Who developed this?
A. MariaDB Foundation.

Q. Can I contribute?
A. Yes! Create an issue and or pull request on what you would like to change.

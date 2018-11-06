# Microsoft Azure Data Explorer (Kusto) Kafka Sink 
This repository contains the source code of the Kafka To ADX Sink.


## Setup

### Clone

```bash
git clone git://github.com/Azure/kafka-sink-azure-kusto.git
cd ./kafka-sink-azure-kusto
```

### Build

Need to build locally with Maven 

#### Requirements

* JDK >= 1.8 [download](https://www.oracle.com/technetwork/java/javase/downloads/index.html)
* Maven [download](https://maven.apache.org/install.html)

Building locally using Maven is simple:

```bash
mvn clean compile assembly:single
```

Which should produce a Jar complete with dependencies.

### Deploy 

Deployment as a Kafka plugin will be demonstrated using a docker image for convenience,
but production deployment should be very similar (detailed docs can be found [here](https://docs.confluent.io/current/connect/userguide.html#installing-plugins))

#### Run Docker
```bash
docker run --rm -p 3030:3030 -p 9092:9092 -p 8081:8081 -p 8083:8083 -p 8082:8082 -p 2181:2181  -v C:\kafka-sink-azure-kusto\target\kafka-sink-azure-kusto-0.1.0-jar-with-dependencies.jar:/connectors/kafka-sink-azure-kusto-0.1.0-jar-with-dependencies.jar landoop/fast-data-dev 
```

#### Verify 
connect to container and run:
`cat /var/log/broker.log /var/log/connect-distributed.log | grep -C 4 i kusto`

#### Add plugin 
Go to `http://localhost:3030/kafka-connect-ui/#/cluster/fast-data-dev/` and using the UI add Kusto Sink (NEW button, then pick kusto from list)
example configuration:

```config
name=KustoSinkConnector 
connector.class=com.microsoft.azure.kusto.kafka.connect.sink.KustoSinkConnector 
kusto.sink.flush_interval_ms=300000 
key.converter=org.apache.kafka.connect.storage.StringConverter 
value.converter=org.apache.kafka.connect.storage.StringConverter 
tasks.max=1 
topics=testing1 
kusto.tables.topics_mapping=[{'topic': 'testing1','db': 'daniel', 'table': 'KafkaTest','format': 'json', 'mapping':'Mapping'}] 
kusto.auth.authority=XXX 
kusto.url=https://ingest-mycluster.kusto.windows.net/ 
kusto.auth.appid=XXX 
kusto.auth.appkey=XXX 
kusto.sink.tempdir=/var/tmp/ 
kusto.sink.flush_size=1000
```

#### Create Table and Mapping
Very similar to (Event Hub)[https://docs.microsoft.com/en-us/azure/data-explorer/ingest-data-event-hub#create-a-target-table-in-azure-data-explorer]

#### Publish data
In container, you can run interactive cli producer like so:
```bash
/usr/local/bin/kafka-console-producer --broker-list localhost:9092 --topic testing1
```

or just pipe file (which contains example data)
```bash
/usr/local/bin/kafka-console-producer --broker-list localhost:9092 --topic testing1 < file.json
```

#### Query Data
Make sure no errors happend duting ingestion
```
.show ingestion failures
```
See that newly ingested data becomes available for querying
```
KafkaTest | count
```

## Need Support?
- **Have a feature request for SDKs?** Please post it on [User Voice](https://feedback.azure.com/forums/915733-azure-data-explorer) to help us prioritize
- **Have a technical question?** Ask on [Stack Overflow with tag "azure-data-explorer"](https://stackoverflow.com/questions/tagged/azure-data-explorer)
- **Need Support?** Every customer with an active Azure subscription has access to [support](https://docs.microsoft.com/en-us/azure/azure-supportability/how-to-create-azure-support-request) with guaranteed response time.  Consider submitting a ticket and get assistance from Microsoft support team
- **Found a bug?** Please help us fix it by thoroughly documenting it and [filing an issue](https://github.com/Azure/kafka-sink-azure-kusto/issues/new).

# Contribute

We gladly accept community contributions.

- Issues: Please report bugs using the Issues section of GitHub
- Forums: Interact with the development teams on StackOverflow or the Microsoft Azure Forums
- Source Code Contributions: If you would like to become an active contributor to this project please follow the instructions provided in [Contributing.md](CONTRIBUTING.md).

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/). For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.

For general suggestions about Microsoft Azure please use our [UserVoice forum](http://feedback.azure.com/forums/34192--general-feedback).
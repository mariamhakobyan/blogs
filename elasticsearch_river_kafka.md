#Mixing two piece of best technologies - ElasticSearch and Kafka
----

##Introduction

###Elasticsearch
[ElasticSearch](http://www.elasticsearch.com/) is a highly scalable, distributed, real time search engine with a REST API that is hard not to love. The core of Elasticsearch’s intelligent search engine is largely another software project: Lucene. It is perhaps easiest to understand elasticsearch as a piece of infrastructure built around Lucene’s Java libraries. Elasticsearch itself provides a more useable and concise REST API, scalability, and operational tools on top of Lucene’s search implementation. It also allows you to start with one machine and scale to hundreds, and supports distributed search deployed over Amazon EC2's cloud hosting.
Plugins in Elasticsearch are a way to enhance the basic elasticsearch functionality in a custom manner. They range from adding custom mapping types, custom analyzers, native scripts, custom discovery and more. There are multiple types of plugins, in this article we will explore the river plugin, which is used to index a stream of data from one source into Elasticsearch. Particularly, we will see how to index stream of data from Kafka into Elasticsearch.

###Kafka
[Apache Kafka](http://kafka.apache.org/) is a publish-subscribe messaging system rethought as a distributed commit log. It was originally developed at LinkedIn and later on became a part of Apache project. Kafka is fast - a single Kafka broker can handle hundreds of megabytes of reads and writes per second from thousands of clients. The main reason why it’s so fast is that it uses [Zero Copy](http://en.wikipedia.org/wiki/Zero-copy) and works in a partitioning mechanism. Applications that use zero copy request that the kernel copy the data directly from the disk file to the socket, without going through the application. Zero copy greatly improves application performance and reduces the number of context switches between kernel and user mode. Other advantage is that consumers keep the index of read data, not Kafka itself. It is scalable - can be elastically and transparently expanded without downtime, durable - messages are persisted on disk and replicated within the cluster to prevent data loss, and distributed by design - it has a modern cluster-centric design based on multiple brokers and partitions.

----

##Initiative

Assume you have an event-driven asynchronous application, which produces thousands of events per second. The application uses Kafka distributed messaging system, and puts all application related events into Kafka. So the use case is to have all this data available in ElasticSearch to be able to perform analytics, search on it later on. Elasticsearch allows clients to build custom plugins, to add any additional functionality to Elasticsearch using provided Java API, so the perfect solution was to have our own plugin that will do this for us. Using plugins, it's possible to add new functionality to Elasticsearch without having to create a fork of Elasticsearch itself. You then run the plugin in elasticsearch itself, without the need to start up a separate application/process.
In this article, we will go through the steps and show how to create the Elasticsearch plugin to put all data from Kafka into Elasticsearch. The plugin is an open source project and available in GitHub in the [following repository](https://github.com/mariamhakobyan/elasticsearch-river-kafka). You can look through the codebase while following the steps discussed in this article.

----

##Building the elasticsearch-river-kafka plugin

The requirements of building elasticsearch river plugin for kafka are the following:
* Elasticsearch installation
* Kafka installation
* Maven
* JDK

The plugin is simply a Zip file that contains one or more Java jar files with compiled code and resources. After setting up the Maven project including .elasticsearch and kafka dependencies, the next step is to use a maven assembly plugin in pom.xml to create the zip file (for more details take a look [here](https://github.com/mariamhakobyan/elasticsearch-river-kafka/blob/master/pom.xml)). After setting up all the configuration related stuff, here are the actual steps of building the plugin:

####Step1. 
We need to write a class, KafkaRiverPlugin, that extends the AbstractPlugin class, and implement the following methods:

```java
@Override
pulic String name() {
    return "river-kafka";
}

@Override
public String description() {
    return "River Kafka Plugin";
}
```

The name is used in Elasticsearch to identify the plugin, for example when printing the list of loaded plugins. 

####Step 2:
Now we need to tell Elasticsearch about our plugin, which is done by adding the fully qualified class name of the plugin to a special ***es-plugin.properties*** file on the classpath, usually stored under ***src/main/resources/es-plugin.properties***:

```java
plugin=org.elasticsearch.plugin.river.kafka.KafkaRiverPlugin
```

When Elasticsearch starts up, the Elasticsearch ***org.elasticsearch.plugins.PluginManager*** will scan the current classpath looking for plugin configuration files and instantiate the referenced plugins.

####Step 3:
To add our custom functionality to the plugin, we need to create a module. Elasticsearch uses Guice to wire together all the components of the server (More details about the Guice Framework - [here](https://code.google.com/p/google-guice/)). While loading all the plugins, Elasticsearch invokes (via reflection) a method called onModule() with a parameter that extends the AbstractModule class.

```java
public void onModule(RiversModule module) {
    module.registerRiver("kafka", KafkaRiverModule.class);
}
```
The above line of code tells Guice that for this module the River class implementation will be the KafkaRiver class. We also tell KafkaRiver to be created during Guice initialization only once.

```java
public class KafkaRiverModule extends AbstractModule {
    @Override
    protected void configure() {
        bind(River.class).to(KafkaRiver.class).asEagerSingleton();
    }
}
``

Now, when we have the plugin setup, it’s time to dig into the actual implementation of our custom logic.

####Step 4:
A river should implement the River interface and extend the AbstractRiverComponent. The River interface contains only two methods: start, called when the river is started and close, called when the river is closed. The AbstractRiverComponent is just a helper that initializes the logger for the river and stores the river name and the river settings on two instance members. The river constructor is annotated with the Guice @Inject annotation, so that all the needed dependencies can be injected into the river. The following dependencies are injected: RiverName, RiverSettings and Client, where Client is a client pointing to the node where the river is allocated.

```java
public class KafkaRiver extends AbstractRiverComponent implements River {

    private KafkaConsumer kafkaConsumer;
    private ElasticSearchProducer elasticsearchProducer;
    private RiverConfig riverConfig;

    private Thread thread;

    @Inject
    protected KafkaRiver(final RiverName riverName, final RiverSettings riverSettings, final Client client) {
        super(riverName, riverSettings);

        riverConfig = new RiverConfig(riverName, riverSettings);
        kafkaConsumer = new KafkaConsumer(riverConfig);
        elasticsearchProducer = new IndexDocumentProducer(client, riverConfig, kafkaConsumer);
    }

    .......
}
```

The nice thing here is you have a control over the injected properties, so you can retrieve the settings which were passed by the user while creating the river, and do any operation with the client.
River specific properties, which were passed by the user, need to be extracted from the RiverSettings, and this is done in RiverConfig class. In this plugin the settings specified by the user will usually contain elasticsearch and kafka specific properties.

```java
public RiverConfig(RiverName riverName, RiverSettings riverSettings) {

    // Extract kafka related configuration
    if (riverSettings.settings().containsKey("kafka")) {
        Map<String, Object> kafkaSettings = (Map<String, Object>) riverSettings.settings().get("kafka");

        topic = (String) kafkaSettings.get("topic");
        zookeeperConnect = XContentMapValues.nodeStringValue(kafkaSettings.get("zookeeper.connect"), "localhost");
    } else {
        topic = "test-topic";
        zookeeperConnect = "localhost";
    }

    // Extract ElasticSearch related configuration
    if (riverSettings.settings().containsKey("index")) {
        Map<String, Object> indexSettings = (Map<String, Object>) riverSettings.settings().get("index");
        indexName = XContentMapValues.nodeStringValue(indexSettings.get("index"), riverName.name());
    } else {
        indexName = riverName.name();
    }
}
``

####Step 5:
As mentioned earlier, we need to override the start and close methods as well:

```java
@Override
public void start() {
    try {
        final KafkaWorker kafkaWorker = new KafkaWorker(kafkaConsumer, elasticsearchProducer, riverConfig);

        thread = EsExecutors.daemonThreadFactory(settings.globalSettings(), "Kafka River Worker").newThread(kafkaWorker);
        thread.start();
    } catch (Exception ex) {
        throw new RuntimeException(ex);
    }
}

@Override
public void close() {
    elasticsearchProducer.closeBulkProcessor();
    kafkaConsumer.shutdown();

    thread.interrupt();
}
```

Basically river starts a new thread, and inside this thread reds messages from Kafka stream and puts those messages into Elasticsearch. The custom logic of reading messages from Kafka is implemented in KafkaConsumer class, and the part of indexing the data into Elasticsearch in ElasticsearchProducer, which we will explore later.
There are generally two ways to implement kafka consumer, either using High Level Consumer API, which keeps track of offset automatically, or the Simple Consumer API where you manually need to handle the offsets, leader brokers etc. The Simple API generally is more complicated, and you should only use it if there is a need for it. For elasticsearch kafka river we use the High Level API, because we do not need to care about the offsets, we need to stream all the data from kafka to Elasticsearch, and on top of that, this API automatically enables the river to read kafka messages from multiple brokers and multiple partitions.

----

##Installing Kafka and sending messages

Once the plugin is written and packaged, it can easily be added to any Elasticsearch installation in a single command. But before that, we need to start the Kafka server and produce some messages, so we see them being consumed by the river plugin.
 
Here are the steps necessary to perform to produce messages into Kafka: 
1. Install Kafka (See Apache Kafka Quick Start Guide for instructions on how to Download and Build.). We will execute all the steps locally.
1. First, start a local instance of the zookeeper server:
```java
bin/zookeeper-server-start.sh config/zookeeper.properties
```
1. Now start the Kafka server:
```java
bin/kafka-server-start.sh config/server.properties
```
1. Then we need to create a topic called “test”:
```java
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic test
```
1. Kafka comes with a command line producer client that will take input from command line and send it out as messages to the Kafka cluster. By default each line will be sent as a separate message. Let’s produce some messages:
```java
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test 
This is a message
This is another message
``

If you wanted to make sure that the messages are really being sent to Kafka server, you could also run a command line consumer client, which would receive the messages, but we will not go into details here, because our plugin will consume those messages using Java API.

----

##Installing the plugin in Elasticsearch

Now, that we have the messages being produced, we can install the plugin into our Elasticsearch instance. Elasticsearch can be downloaded from [Elasticsearch Download](http://www.elasticsearch.org/download/) page.

1. Install the plugin with a single command:
```java
cd ELASTICSEARCH_HOME
.bin/plugin --install <plugin-name> --url file:////$PLUGIN-PATH/elasticsearch-river-kafka-1.2.1-SNAPSHOT-plugin.zip
```
where the <plugin-name> should be the name of the plugin. Note that we use the --url option for the plugin command to get the file locally instead of trying to download it from an online repository, which is another option.

1. We can now start Elasticsearch and see that our plugin gets loaded:
```java
~/elasticsearch-1.4.0/bin $ ./elasticsearch
    [2014-12-27 18:12:47,862][INFO ][node    ] [Mahkizmo] version[1.4.0], pid[9233], build[bc94bd8/2014-11-05T14:26:12Z]
    [2014-12-27 18:12:47,863][INFO ][node    ] [Mahkizmo] initializing ...
    [2014-12-27 18:12:47,886][INFO ][plugins ] [Mahkizmo] loaded [river-kafka], sites []
```

----

##Configuring the plugin

Basically, at this poit the plugin is installed and running, but we still need to deploy the river itself. To create a river, we execute the following curl request:

```java
curl -XPUT 'localhost:9200/_river/kafka-river/_meta' -d '
{
    "type" : "kafka",
    "kafka" : {
       "zookeeper.connect" : "localhost", 
       "zookeeper.connection.timeout.ms" : 10000,
       "topic" : "test",
       "message.type" : "json"
   },
   "index" : {
       "index" : "kafka-index",
       "type" : "status",
       "bulk.size" : 5,
       "concurrent.requests" : 1,
       "action.type" : "index"
    }
}'
```

The first section is about kafka properties, e.g. zookeeper server host, topic name etc. The second section contains the properties defined by the user for elasticsearch index itself, e.g. index-name, bulk.size etc. In order to index the data from kafka to Elasticsearch, the river plugin uses the Elasticsearch Bulk API, which makes it possible to perform many index/delete operations in a single API call. This can greatly increase the indexing speed. You should note, that type "kafka" must match with the string previously provided when registering the KafkaRiverModule to the RiversModule.

The detailed description of each parameter:

* ***zookeeper.connect*** (optional) - Zookeeper server host. Default is: localhost
* ***zookeeper.connection.timeout.ms*** (optional) - Zookeeper server connection timeout in milliseconds. Default is: 10000
* ***topic*** (optional) - The name of the topic where you want to send Kafka message. Default is: elasticsearch-river-kafka
* ***message.type*** (optional) - The kafka message type, which then will be inserted into ES keeping the same type. Default is: json. The following options are available:
    * ***json***: Inserts json message into ES separating each json property into ES document property. 
    Example:
        ```java
        "_source": {
        "name": "John",
        "age": 28
        }
       ```
    * ***string***: Inserts string message into ES as a document, where the key name is value, and the value is the received message. 
    Example:
        ```java
        "_source": {
              "value": "received text message"
        }
         ```
* ***index*** (optional) - The name of elasticsearch index. Default is: kafka-index
* ***type*** (optional) - The mapping type of elasticsearch index. Default is: status
* ***bulk.size*** (optional) - The number of messages to be bulk indexed into elasticsearch. Default is: 100
* ***concurrent.requests*** (optional) - The number of concurrent requests of indexing that will be allowed. A value of 0 means that only a single request will be allowed to be executed. A value of 1 means 1 concurrent request is allowed to be executed while accumulating new bulk requests. Default is: 1
* ***action.type*** (optional) - The action type against how the messages should be processed. Default is: index. The following options are available:
    * ***index***: Creates documents in ES with the value field set to the received message.
    * ***delete***: Deletes documents from ES based on id field set in the received message.
    * ***raw.execute***: Execute incoming messages as a raw query.

As a conclusion we see, that this plugin allows to index different types of messages coming from Kafka (json and string), and as well to do several types of operations, such as indexing, deleting the indexed data or executing the raw data that comes as a message. These, as well as the possibility to control the bulk size, concurrent request numbers, are powerful features provided by the plugin.

----

##Bulk API usage in the plugin

Here is how the Bulk API is used in ElasticsearchProducer class to index the messages:

```java
private void createBulkProcessor(final KafkaConsumer kafkaConsumer) {
    bulkProcessor = BulkProcessor.builder(client,
            new BulkProcessor.Listener() {
                @Override
                public void beforeBulk(long executionId, BulkRequest request) {
                    logger.info("Index: {}: Going to execute bulk request composed of {} actions.", 
                        riverConfig.getIndexName(), request.numberOfActions());
                }

                @Override
                public void afterBulk(long executionId, BulkRequest request, BulkResponse response) {
                    logger.info("Index: {}: Executed bulk composed of {} actions.", 
                        riverConfig.getIndexName(), request.numberOfActions());

                    // Commit the kafka messages offset, only when messages have been successfully
                    // inserted into ElasticSearch
                    kafkaConsumer.getConsumerConnector().commitOffsets();
                }

                @Override
                public void afterBulk(long executionId, BulkRequest request, Throwable failure) {
                    logger.warn("Index: {}: Error executing bulk.", failure, riverConfig.getIndexName());
                }
            })
            .setBulkActions(riverConfig.getBulkSize())
            .setFlushInterval(TimeValue.timeValueHours(12))
            .setConcurrentRequests(riverConfig.getConcurrentRequests())
            .build();
}
``

The code above creates the BulkProcessor and configures it to execute the bulk when specified number of documents are ready to be indexed or when 12 hours have passed from the last bulk execution, so any remaining messages get flushed to elasticsearch even if the number of messages has not reached. 

The following code adds an index request to the previously created BulkProcessor:

```java
bulkProcessor.add(Requests.indexRequest(riverConfig.getIndexName()).
                        type(riverConfig.getTypeName()).
                        id(UUID.randomUUID().toString()).
                        source(jsonMessageMap));
``

----

##Updating the plugin

Additionally, if you are in development phase, and you change the logic and want to update the plugin, you can do this by removing and installing the plugin again in Elasticsearch. To remove the plugin from Elasticsearch, execute:

```java
cd $ELASTICSEARCH_HOME
./bin/plugin -remove <plugin-name>
```

To see the indexed data in Elasticsearch you ca execute:

```java
curl -XGET 'localhost:9200/kafka-index/_search?pretty=1'
```

----

##Summary

This article showed how to work with Elasticsearch, and how to create plugins, particularly a Kafka plugin which reads stream of messages and indexes them into Elasticsearch. As you saw, it’s relatively easy to add any functionality to the existing Elasticsearch running instance in a custom manner. The most important learnings were that you should use Bulk API as much as possible, because it makes indexing much faster. Also, if you only need to stream data from Kafka brokers to Elasticsearch, you can simply use the High Level API from Kafka. 
The elasticsearch kafka river plugin, that we walked through in this article, is open source project, and available in [Elasticsearch official website](http://www.elasticsearch.org/guide/en/elasticsearch/reference/current/modules-plugins.html) as a plugin. It is available in GitHub in the [following repository](https://github.com/mariamhakobyan/elasticsearch-river-kafka). The readers are welcome to explore, try out and give feedback on it.
# Udemy Course: Apache Kafka Series - Kafka Connect Hands-on Learning

* [Course Link](https://www.udemy.com/course/kafka-connect/)
* [Course Repo](https://github.com/simplesteph/kafka-connect-github-source)
* [Lecturers Website](https://courses.datacumulus.com/)

## Section 3: Kafka Connect Concepts

### Lecture 6. What is Kafka Connect?

* Kafka-Connect is all about code & connectors re-use
* Why Kafka Connect and Streams?
* Four common kafka use cases:
    * Source => Kafka (ProducerAPI) : Kafka Connect Source
    * Kafka => Kafka (Consumer,Producer API) : Kafka Streams
    * Kafka => Sink (Consumer API) : Kafka Connect Sink
    * KAfka => A00 (Consumer API)
* Simplify and Improve getting data in and out of kafka
* simplify transforming data within Kafka without relying on external libs
* Why Kafka Connect?
    * Programmers always want to import data from the same sources: DBs,JDBC,SAP HANA,BlockChain,Cassandra,MongoDB,Titter,IOT,FTP
    * Programmers always want to store data in the same sinks: S3,ElasticSearch,HDFS,JDBC,SAP HANA,Cassandra,DynamoDB,HBase,Redis,mongoDB,
* Don't reinvent the wheel

### Lecture 7. Kafka Connect Architecture Design

* Connect Cluster stands between Sources and Kafka Cluster or between Kafka Cluster  and Sinks
* Connect Cluster has Workers (much like Brokers)
* Streams Apps input/output data to the Kafka Cluster 

### Lecture 8. Connectors, Configuration, Tasks, Workers

* Kafka Connect high Level
    * Source COnnectors to get data from Common Data Sources
    * Sink Connectors to publish that data in Common Data Stores
    * make it easy for non expert devs to quickly get their data reliable to Kafka
    * Part of the ETL pipeline
    * Scaling made easy from small pipelines to company-wide pipelines
    * Re-usable code
* [Connectors](https://www.confluent.io/hub/)
* A Kafka Connect Cluster has multiple loaded Connectors
    * Each Connector is a re-usable piece of code (java jars)
    * many connectors exist in the open source world
* Connectors + User Configuration => Tasks
    * A task is linked to a connector configuration
    * A job configuration may spawn multiple tasks
* Tasks are executed by Kafka Connect Workers (servers)
    * a worker is a single java process
    * a worker can be standalone or in a cluster

### Lecture 9. Standalone vs Distributed Mode

* Kafka Connect Workers Standalone vs Distributed Mode:
* Standalone:
    * A single process runs our connectors and tasks
    * Configuration is bundled with your process
    * Very easy to get started with useful for development and testing
    * No fault tolerant, no scalability, hard to mpnitotr
* Distributed:
    * Multiple workers run your connectors and tasks
    * Configuration is submitted using a REST API
    * Easy to scale, fault tolerant (rebalancing in case of worker dies)
    * useful for production deployment of connectors

### Lecture 10. Distributed Architecture in Details

* Kafka Connect Cluster contains Workers on which  Connector taskss run
* Tasks of a Connector run on different workers
* a worker can run multiple tasks from different connectors
* if a connect cluster worker dies  the cluster rebalances (lik when we lose a consumer in a consumer group)
* remaining workers take up its tasks. rules are followed

## Section 4: Setup and Launch Kafka Connect Cluster

### Lecture 11. Important information about installation

* we must have docker on our machine

### Lecture 14. Docker on Linux (Ubuntu as an example)

* we already have docker
* we need o install ocker-engine and docker-compose
* follow official docker instructions (docker was built for linux)
* we will use a docker-compose yaml file to start the connect cluster
```
version: '2'

services:
  # this is our kafka cluster.
  kafka-cluster:
    image: landoop/fast-data-dev:cp3.3.0
    environment:
      ADV_HOST: 127.0.0.1         # Change to 192.168.99.100 if using Docker Toolbox
      RUNTESTS: 0                 # Disable Running tests so the cluster starts faster
    ports:
      - 2181:2181                 # Zookeeper
      - 3030:3030                 # Landoop UI
      - 8081-8083:8081-8083       # REST Proxy, Schema Registry, Kafka Connect ports
      - 9581-9585:9581-9585       # JMX Ports
      - 9092:9092                 # Kafka Broker

  # we will use elasticsearch as one of our sinks.
  # This configuration allows you to start elasticsearch
  elasticsearch:
    image: itzg/elasticsearch:2.4.3
    environment:
      PLUGINS: appbaseio/dejavu
      OPTS: -Dindex.number_of_shards=1 -Dindex.number_of_replicas=0
    ports:
      - "9200:9200"

  # we will use postgres as one of our sinks.
  # This configuration allows you to start postgres
  postgres:
    image: postgres:9.5-alpine
    environment:
      POSTGRES_USER: postgres     # define credentials
      POSTGRES_PASSWORD: postgres # define credentials
      POSTGRES_DB: postgres       # define database
    ports:
      - 5432:5432                 # Postgres port
```

* we use a ready image for the cluster from landoop because its preloaded with a bunch of connectors
* the ADV_HOST is set to localhost for local development
* we have port setting for the services
* apart from kafka the image installs eleasticsearch and postgres
* there are 2 bash files. `kafka-connect-tutorial-sources.sh` from which we cp commands
* in the folder where is our compose yaml file we run `sudo docker-compose up kafka-cluster`
* because the yaml file has the default name 'docker-compose.yml' we dont need to spec it with -f. also we use kafka-cluster to run only the kfka pat of it (omit elasticsearch and prostgres)
* once the containers are launched we can visit the Lndoop UI at localhost:3030 in browser
* if we click on connector we see one active for logs. if we click on new we see a whole bunch of options

## Section 5: Troubleshooting Kafka Connect

### Lecture 20. Where to view logs?

* in standalone mode they are on the terminal
* in distributed mode (using ladoop image)
  * they will be at URL localhost:3030/logs
  * take file 'connect-distributed.log' look at ERROR
* to check if there is a new image of landoop `docker pull landoop/fast-data-dev` 
* we can see a demo at [githubhttps://github.com/lensesio/fast-data-dev]()

## Section 6: Kafka Connect Source - Hands On

### Lecture 22. Kafka Connect Source Architecture Design

* Kafka Connect Cluster will connect both to Sinks and Sources

### Lecture 23. FileStream Source Connector - Standalone Mode - Part 1

* Goal
  * read a file and load the content directly into Kafka
  * run in a connector in standalone mode (useful for development)
* between the file and the topic we put a Connector without a schema 
* we open the 'kafka-connect-tutorial-sources.sh' file to sse the commands
* if we havent started our connect cluster we start it with `docker-compose up kafka-cluster` to start the cluster
* our first example is in /source/demo-1/
* there we have text file to load a file-stream-standalone.properties file for the connector and a worker.properties
* the 'worker.properties' contains props to configure a worker in standalone mode
  * bootstrap servers are for kafka connection
  * we see a bunch of converters for key and value
  * internal converters usually match the external ones
  * the REST API is where kafka connect standalone is supposed to connect
  * for standalone worker we set a standalone offset and how often we want to commit it
* these are for data serialization as kafka topics accept byte streams
```
# from more information, visit: http://docs.confluent.io/3.2.0/connect/userguide.html#common-worker-configs
bootstrap.servers=127.0.0.1:9092
key.converter=org.apache.kafka.connect.json.JsonConverter
key.converter.schemas.enable=false
value.converter=org.apache.kafka.connect.json.JsonConverter
value.converter.schemas.enable=false
# we always leave the internal key to JsonConverter
internal.key.converter=org.apache.kafka.connect.json.JsonConverter
internal.key.converter.schemas.enable=false
internal.value.converter=org.apache.kafka.connect.json.JsonConverter
internal.value.converter.schemas.enable=false
# Rest API
rest.port=8086
rest.host.name=127.0.0.1
# this config is only for standalone workers
offset.storage.file.filename=standalone.offsets
offset.flush.interval.ms=10000
```

* the actual file import connector config file is presented below
  * name is the name of the connector (uniue)
  * the file it sould run (jar)
  * tasks when it is for source connector we keep it as 1
  * the last 2 are specific to the connector (topic and filename)
```
# These are standard kafka connect parameters, need for ALL connectors
name=file-stream-demo-standalone
connector.class=org.apache.kafka.connect.file.FileStreamSourceConnector
tasks.max=1
# Parameters can be found here: https://github.com/apache/kafka/blob/trunk/connect/file/src/main/java/org/apache/kafka/connect/file/FileStreamSourceConnector.java
file=demo-file.txt
topic=demo-1-standalone
```

### Lecture 24. FileStream Source Connector - Standalone Mode - Part 2

* the command we need to run for linux is `sudo docker run --rm -it -v "$(pwd)":/tutorial --net=host landoop/fast-data-dev:cp3.3.0 bash`
* what we do is runing a shell command in a running container seting a volume (a connection) between the localhost and the container. we actually mount the pwd to a folder in the running container
* we should run the command from demo-1 folder.
* we get into shell of container
* in the container shell we create a topic `kafka-topics --zookeeper localhost:2181 --create --topic demo-1-standalone --partitions 3 --replication-factor 1`
* we confirm the creation of the topic if we go to the localhost:3030 in browser in topics ui we see the topic we created
* we cd into tutorial
* in there we will run `connect-standalone worker.properties file-stream-demo-standalone.properties` to start the connector passing in the 2 properties files
* connect standalone is a stadard kafka command to run a connector. first we always need to pass the worker.properties
* in the log we see that the connector is created\
* our topic is empty because the text file is empty
* as we add lines to the file and hit enter and save the topic is filled
* we stop connector (ctrl+C)
* we see that the connector creates a standalone.offset file
* its a binary file so that when connector restarts it knows from where to read an

### Lecture 25. FileStream Source Connector - Distributed Mode

* We want to read a file and load the content directly into Kafka
* Run in distributed mode on our already set-up kafka connect cluster
* we will run the connector with schema enabled
* we will see how to configure a connector in distributed mode
* learn about kafka connect cluster
* understand schema configuration option 
* if we have closed the bash terminal connection we reconnect `docker run --rm -it --net=host landoop/fast-data-dev:cp3.3.0 bash`
* we create a topic for our example `kafka-topics --create --topic demo-2-distributed --partitions 3 --replication-factor 1 --zookeeper 127.0.0.1:2181`
* we go to the UI at localhost:3030 and to the connect UI
* we see runing connectors and the connector-topic mapping
* click NEW => File
* we see the params, and we can validate them
* in dource/demo-2 we have a 'file-stream-demo-distributed.properties' file with the connector config
* in general connector property files are the same for standalone and distributed mode
  * we need connector name,class(jar file), numOfTasks
  * we set filename and topic name
  * the key,value serielizers
  * schema is enabled
```
name=file-stream-demo-distributed
connector.class=org.apache.kafka.connect.file.FileStreamSourceConnector
tasks.max=1
# Parameters can be found here: https://github.com/apache/kafka/blob/trunk/connect/file/src/main/java/org/apache/kafka/connect/file/FileStreamSourceConnector.java
file=demo-file.txt
topic=demo-2-distributed
# Added configuration for the distributed mode:
key.converter=org.apache.kafka.connect.json.JsonConverter
key.converter.schemas.enable=true
value.converter=org.apache.kafka.connect.json.JsonConverter
value.converter.schemas.enable=true
```
* in distributed mode we dont need the worker.properties file
* we cp the properties file contents in to the UI
* we click CREATE and see the connector running with one task
* we click on it and see the task the topic and the config
* we see the topic is empty
* we log in our docker image and create a filte with some data to stream in the topic
* `sodo docker ps` to see the running containers and get containrr ID
* `sudo docket exec -it <CONTAINER ID> bash`
* we create the input file and push data into it
```
thouch demo-file.txt
echo "hi" >> demo-file.txt
```
* if we look into the topic we see that for key and value a schema is used (mush like a mongoDB doc) and it is a JSON objec
* we run ` kafka-console-consumer.sh --topic demo-2-distributed --from-beginning --bootstrap-server localhost:9092` from host and confirm the json struct `{"schema":{"type":"string","optional":false},"payload":"hi"}`

### Lecture 26. List of Available Connectors

* there is a framework to create connectors
* the best place to find connectors is the [Confluent website](https://www.confluent.io/hub)
* also google 
* many connectors are bundled with landoop docker image
* to add our own we need to map our volumes to the landoop docker image at start [instructions](https://github.com/lensesio/fast-data-dev#Enable%20additional%20connectors)

### Lecture 27. Twitter Source Connector - Distributed Mode - Part 1

* we want to gather data from twitter using kafka connect in distributed mode
* we ll gather real data
* we ll use [Eneco implmentation](https://github.com/Eneco/kafka-connect-twitter)
* we need a formal developer account
* create an app and go to keys
* we go to source/demo-3 and edit file 'source-twitter-distributed.properties' and cp the keys from twitter app. additional props are the usual (topic,name,seriazer type, java class)
* in this connector is important to enable schema
* also we set the search terms and lang of tweets to pull

### Lecture 28. Twitter Source Connector - Distributed Mode - Part 2

* we cp from the ....source.sh file
* being in the container terminal we create a topic `kafka-topics --create --topic demo-3-twitter --partitions 3 --replication-factor 1 --zookeeper 127.0.0.1:2181`
* in host we start a consumer `kafka-console-consumer --topic demo-3-twitter --bootstrap-server 127.0.0.1:9092`
* we go to ui at localhost:3030. we go to connector UI => New => Twitter
* cp the properties => CREATE
* we see that the consumer is already getting from topic
* we check tipic UI and see the schema of the tweets

## Section 7: Kafka Connect Sink - Hands On

### Lecture 30. Kafka Connect Sink Architecture Design

* connect sink are connectors that output data to data sinks
* the archotecute in a cluster are the same with sources 
* they run on >= workers as tasks

### Lecture 31. ElasticSearch Sink Connector - Distributed Mode - Part 1

* we will going to start an ElasticSearch instance to Sink the data to
* we will use Docker to start our ElasticSearch instance
* we will show how to do it with docker but as we have Elasticsearch on the machine we will run it locally
* Εlasticsearch will act as Data Sink. it consumes JSON
* We'll use ElasticSearch Sink in Distributed Mode
  * start an ES instance using Docker
  * Sink a topic with multiple parttiions in ElasticSearch
  * Run in distributed mode with multiple tasks
* ElasticSearchSink connector stands between twitter topic and ES
* we ll learn about `tasks.max` param
* we ll use the same docker compose file to launch instance
* from the dir whre composer file resides we run `sudo docker-compose up kafka-cluster elasticsearch postgres` to run all. we have eleasticsearch and cluster running so we start only prostgres
* we confirm elasticsearch is running visiting localhost:9200 in browser
* we start kibana and fire up console for ease of use
* we go to sink/demo-elasticsearch and peek in the 'sink-elastic-twitter-distributed.properties'
  * most of props are the same
  * now we fire 2 tasks to demeonstrate parallelism
  * we use topics not topic as we can sink from multiple topics
  * the rest is elastic specific
```
# Basic configuration for our connector
name=sink-elastic-twitter-distributed
connector.class=io.confluent.connect.elasticsearch.ElasticsearchSinkConnector
# We can have parallelism here so we have two tasks!
tasks.max=2
topics=demo-3-twitter
# the input topic has a schema, so we enable schemas conversion here too
key.converter=org.apache.kafka.connect.json.JsonConverter
key.converter.schemas.enable=true
value.converter=org.apache.kafka.connect.json.JsonConverter
value.converter.schemas.enable=true
# ElasticSearch connector specific configuration
# # http://docs.confluent.io/3.3.0/connect/connect-elasticsearch/docs/configuration_options.html
connection.url=http://elasticsearch:9200
type.name=kafka-connect
# because our keys from the twitter feed are null, we have key.ignore=true
key.ignore=true
# some (dummy) settings we need to add to ensure the configuration is accepted
# The bug is tracked at https://github.com/Landoop/fast-data-dev/issues/42
topic.index.map="demo-3-twitter:index1"
topic.key.ignore=true
topic.schema.ignore=true

```

### Lecture 32. ElasticSearch Sink Connector - Distributed Mode - Part 2

* note that in order to access host elasticsearch from docker container we need to build a network and expose the host giving it an ip. also in thet case we use that ip:9200 and not elasticsearch:9200
* we follow the usual drill
* cp the props and go to 3030 => Connectors => New => ElasticSearchSink from Confluent => Create
* we opt to start a task.max=1 at first and then sale to 2
* we can issue queries in elasticsearch like 
```
{
  "query: {
    "term": {
      "is_retweet": true
    }
  }
}
```

or a query based on freiends count
```
{
  "query: {
    "range": {
      "user.friends_count": {
        "gt":500
      }
    }
  }
}
```

### Lecture 33. Kafka Connect REST API

* up to now we use the landoop UI to deploy kafka connect sources and sinks on the cluster
* the defualt way is through the Kafka Connect Rest API
* All the actions performed by Landoop Kafka Connect UI are actually triggering [REST API calls to Kafka Connect](http://docs.confluent.io/3.2.0/connect/managing.html#common-rest-examples)
* we can see a list of requests in `demo-rest-api.sh`
* enter landoop docker container terminal `docker run --rm -it --net=host landoop/fast-data-dev:cp3.3.0 bash`
* install jq so that we can see json replies formatted `apk update && apk add jq`
* Actions we can do hittion port 8083
  1. Get Worker information `curl -s 127.0.0.1:8083/ | jq`
  2. List Connectors available on a Worker `curl -s 127.0.0.1:8083/connector-plugins | jq`
  3. Ask about Active Connector `curl -s 127.0.0.1:8083/connectors | jq`
  4. Get information about a Connector Tasks and Config `curl -s 127.0.0.1:8083/connectors/source-twitter-distributed/tasks | jq`
  5. Get Connector Status `curl -s 127.0.0.1:8083/connectors/file-stream-demo-distributed/status | jq`
  6. Pause/Resume a Connector
```
  curl -s -X PUT 127.0.0.1:8083/connectors/file-stream-demo-distributed/pause
curl -s -X PUT 127.0.0.1:8083/connectors/file-stream-demo-distributed/resume
```

  7. Get Connector Configuration `curl -s 127.0.0.1:8083/connectors/file-stream-demo-distributed | jq`
  8. Delete our Connector `curl -s -X DELETE 127.0.0.1:8083/connectors/file-stream-demo-distributed`
  9. Create Connector Configuration 

```
  curl -s -X POST -H "Content-Type: application/json" --data '{"name": "file-stream-demo-distributed", "config":{"connector.class":"org.apache.kafka.connect.file.FileStreamSourceConnector","key.converter.schemas.enable":"true","file":"demo-file.txt","tasks.max":"1","value.converter.schemas.enable":"true","name":"file-stream-demo-distributed","topic":"demo-2-distributed","value.converter":"org.apache.kafka.connect.json.JsonConverter","key.converter":"org.apache.kafka.connect.json.JsonConverter"}}' http://127.0.0.1:8083/connectors | jq
```

  10. Update Connector Configuration 
```
  curl -s -X PUT -H "Content-Type: application/json" --data '{"name": "file-stream-demo-distributed", "config":{"connector.class":"org.apache.kafka.connect.file.FileStreamSourceConnector","key.converter.schemas.enable":"true","file":"demo-file.txt","tasks.max":"1","value.converter.schemas.enable":"true","name":"file-stream-demo-distributed","topic":"demo-2-distributed","value.converter":"org.apache.kafka.connect.json.JsonConverter","key.converter":"org.apache.kafka.connect.json.JsonConverter"}}' http://127.0.0.1:8083/connectors | jq
```

### Lecture 34. JDBC Sink Connector - Distributed Mode

* we will start a PostgreSQL instance to sink data into
* we will use Docker to start our PostgreSQL DB
* It will serve as a sik for the sink connector
* we use JDBCSinkConnector in distributed mode
* Tasks
  * Start a PostgreSQL instance with Docker
  * Run in distributed mode with multiple tasks
* Postgres\SQL is running from the dockercompose up command
* in sink/demo-postgres we have the 'sink-postgres-twitter-distributed.properties' file with connector config
  * credentials for the DB and port match with compose file that created the container
  * we also pass a primary key
  * we also pass a whitelist of fileds we wnt to pass in the DB from the topic entries
  * connector cannot handle nested json
  * we also allow connector to create tables
```
# Basic configuration for our connector
name=sink-postgres-twitter-distributed
connector.class=io.confluent.connect.jdbc.JdbcSinkConnector
# We can have parallelism here so we have two tasks!
tasks.max=1
topics=demo-3-twitter
# the input topic has a schema, so we enable schemas conversion here too
key.converter=org.apache.kafka.connect.json.JsonConverter
key.converter.schemas.enable=true
value.converter=org.apache.kafka.connect.json.JsonConverter
value.converter.schemas.enable=true
# JDBCSink connector specific configuration
# http://docs.confluent.io/3.2.0/connect/connect-jdbc/docs/sink_config_options.html
connection.url=jdbc:postgresql://postgres:5432/postgres
connection.user=postgres
connection.password=postgres
insert.mode=upsert
# we want the primary key to be offset + partition
pk.mode=kafka
# default value but I want to highlight it:
pk.fields=__connect_topic,__connect_partition,__connect_offset
fields.whitelist=id,created_at,text,lang,is_retweet
auto.create=true
auto.evolve=true
```
* we open 3030 new => jdbcsink => cp config create
* we install sqlelectron connect to posges on 127.0.0.1 using the user/pass from cnfig files

## Section 8: Writing your own Kafka Connector

### Lecture 35. Goal of the section: GitHubSourceConnector

* Goals - Understand how to make connectors
  * Dependencies
  * ConfigDef
  * Connector class
  * Schema & Struct
  * Source partition & Source Offsets
  * Tasks
* How to deploy
  * Package Jars
  * Run jars in standalone mode
  * deploy jars

### Lecture 36. Finding the code and installing required software

* get [sourcecode](https://github.com/simplesteph/kafka-connect-github-source/archive/v1.1.tar.gz)
* use Jetbrains IntelliJ IDEA
* we need Java 8
* see steps in Kafka course

### Lecture 37. Description of the GitHub Issues API

* Description of Github Issues [API](https://api.github.com)
  * [API Docs](https://developer.github.com/v3/issues/#list-repository-issues)
  * we want to create a stream of ussues and pull request from the repository of our choice
  * the stream should pick any issue update or pull request
  * we will use a REST Client to Query the [API](https://developer.github.com/v3/issues/#parameters-3)
  * the call is `GET /repos/:owner/:repo/issues`
  * we will use [k8s](https://github.com/kubernetes/kubernetes) repo to test `https://api.github.com/repos/kubernetes/kubernetes/issues`
  * we can chain params `https://api.github.com/repos/kubernetes/kubernetes/issues?state=closed&since=2020-01-01T00:00:00Z`

  ### Lecture 38. Using the Maven Archetype to get started

  * The easiest way to get started on a Kafka Connect PRoject is to use an archetype provided [here](https://github.com/jcustenborder/kafka-connect-archtype)
  * we will see how to install it in IntelliJ and use it
  * then we will downgrade Kafka dep to 0.10.2.0 to match the version from Landoop (wtff) just to follow the lectue (there is a new one)
  * in the archetype we see a maven quick install file (also link form prev versions). we should matc these value sin our config files
  * Start IntelliJ
  * New->Project Maven => create from archetype => add 
    * groupID=io.confluent.maven
    * artifactID=kafka-connect-quickstart
    * version=0.10.0.0
  * groupid: com.github.achliopa.kafka
  * artifactid: kafka-connect-project
  * projectname: kafkaconnectproject
* New Window
* we see the project built (do auto imports)
* if i go to src->main-java-><groupID> i see a lot of source files from archetype
  * if i want to create a Sink connector i mod the files with *Sink*
  * if i want to create a Source connector i mod the files with *Source*
* we change in pom.xml kafka version to 0,10.2.0 and do auto import
* in config folder we see 2 example properties files for our connector to use

### Lecture 39. Config Definitions

* first to think about is config definition. what params we want for the connector
* in kafka its called Config Def and its the way to communicate to the user how we want our configuration to be
* we can have mandatory or optional params (with defaults) and validate the types and constraints of our params. we should also document them
```
public static final String OWNER_CONFIG="github.owner"
private static final String OWNER_DOC="Owner of the repository we would like to follow" 
public static ConfigDef conf() {
    return new ConfigDef()
        .define(OWNER_CONFIG,Type.String, Importance.HIGH, OWNER_DOC)
        .define(BATCH_SIZE_CONFIG,Type.Int,100,new BatchSizeValidator(), Importance.LOW,BATCH_SIZE_DOC)
}
```
* the syntax is <PROP>_CONFIG for config and <PROP>_DOC for documentation
* we use '.define()' to define
* at courseRepo project in /src/main/java/<goupid>/Validators we see a compete configuration
* first properties are defined
* its a direct implementation of MySourceConnectorConfig
* in ConfigDef we pass in validators and the signatures of props
* also a set of getters
* also we can check the test code
* add slf4j dependency in pom.xml

### Lecture 40. Connector Class

* Thats the class of our Kafka Connector that is referenced from the configs.
* It should load the config and create a few tasks
* we nedd to implement
```
public String version()
public void start(Map<String,String> map)
public Class<? extends Task> taskClass()
public List<Map<String,String>> taskConfigs(int i)
public void stop()
public ConfigDef config()
```
* we check GitHubSourceConnector java file
* we extend the SourceConnector and in it instatiate a looger and a config object
* when we start() the Connector we map the properties to their values (key,val pairs)
* taskClass() returns the class of the Task
* taskConfigs(i) we define all the tasks config we will have. multiples for paralelized connectors
* our will have just 1 task
originaltrings() takes <Sting,String> and creates config
* stop() we use to stop data conn
* config() just returns the config

### Lecture 41. Writing a schema

* schema is an abstraction that allow us to define how our data structure will look like. it is using primitive types and then the kafka connect framework will  convert it to JSON or Avro as needed
* it is important to correctly design the schema before we start programming
```
public static Schema USER_SCHEMA=
SchemaBuilder.struct().name(SCHEMA_VALUE_USER)
  .version(1)
  .field(USER_URL_FIELD,Schema.STRING_SCHEMA)
  .field(USER_ID_FIELD,Schema.INT32_SCHEMA)
  .field(USER_LOGIN_FIELD,Schema.STRING_SCHEMA)
  .build();
```
* in our example we can see what to put in the schema by looking at the JSON returned by the GithubAPI
* we implement them in GitHubSchemas class
* we start by the definitions of the fields e.g
```
    public static String OWNER_FIELD = "owner";
    public static String REPOSITORY_FIELD = "repository";
    public static String CREATED_AT_FIELD = "created_at";
```
* and then we build them as we ve seen
* we create 4 schemas
```
    // Schema names
    public static String SCHEMA_KEY = "issue_key";
    public static String SCHEMA_VALUE_ISSUE = "issue";
    public static String SCHEMA_VALUE_USER = "user";
    public static String SCHEMA_VALUE_PR = "pr";
```
* we can add a .doc("Documentation") in the builder to add schema docs
* .optional() sets it to optional
* we can Nest schema in a schema

### Lecture 42. Data Model for our Objects

* Data Models are the building blocks 
* it always good practice to have our data structs orgainized in classes in a 'model' package
* in this example Issue,User,..,PullRequest are modeled as POJOs (plain old java objects)
```
public class PullRequest {
  private String url;
  private String htmlUrl;
  get..();
  set..();
}
```
* its very standard Java class. nothing fancy
* the from Json() method receives JSON data based on data received and builds the Object parsing the vals as constructor args
* also we see the nesting in objects

### Lecture 43. Writing our GitHub API HTTP Client

* we will query the GitHub API using the UniRest library that we wrap in our own REST client
* concepts to implement
  * dynamic URL
  * pagination
  * rate throttling
  * error handling
```
public class GitHubAPIHttpClient{
  protected JSONArray getNextissues() throws InterruptedException 
  ptotected HttpResponse<JsonNode> getNextIssuesAPI() throws UnirestException
  protected String constructUrl()
}
```

* constructUrl() builds the URL to hit
* getNextIssuesAPI() returns the response
* getNextissues() returns JSON
* the implemented class is GitHubAPIHttpClient
* we use the connector configuration
* we decypher response from getNextIssuesAPI to extract JSON
* usern/password supported
* we also add unirest maven dependency in pom.xml

### Lecture 44. Source Partition & Source Offsets

* connectors are stateless. the state a.k.a offset is stored in kafka
* source partition and source offset not have to do with kafka. a pointer to where we were reading from the last time
* source partition allos Kafka Connect to know which source we have been reading
* source offsets alow kafka connect to track until when we ve been reading for the source partition we chose
* they are different than partition and offsets in Kafka
* they are for Kafka Connect Source to b used
```
private Map<String,String> sourcePartition(){
  Map<String,String> map = new HashMap<>();
  map.put(OWNER_FIELD,config.getOwnerConfig());
  map.put(REPOSITORY_FIELD,config.getRepoConfig());
  return map;
}
```

* for our example the source is the repository
* for offset we need the timestamp as well
```
    private Map<String, String> sourceOffset(Instant updatedAt) {
        Map<String, String> map = new HashMap<>();
        map.put(UPDATED_AT_FIELD, DateUtils.MaxInstant(updatedAt, nextQuerySince).toString());
        map.put(NEXT_PAGE_FIELD, nextPageToVisit.toString());
        return map;
    }
```

* these 2 params are needed to hit the next url

* they are set in the GitHubSourceTask class
* both are used to generate the sourceRecord like an index to know where to hit next

### Lecture 45. Source Task

* the task does the actual job
* its supposed to initialize, then find where to resume from, and finally poll the source for records
* the class interface looks like
```
public class GitHubSourceTask extends SourceTask {
  public String version()
  public void start(Map<String,String> map)
  public List<SourceRecord> poll() throws InterruptedException
  public void stop()
}
```

* poll is called continuously polling for records in the source
* poll throws interrupted exception and is called continuosuly
* therefore we need to throttle with sleep()
* we initialize last variables passing partition and offset. if there is not we handle it starting from start.
* poll() is running continuouslyreaching the api for data using the Client
* we add the record to a list of records and generate a SourceRecord with key and value. value is built on the model classes from json parsing
* our batches are of 100recs
* record has a topic field so it knows where to go in Kafka

### Lecture 46. Building and running a Connector in Standalone Mode

* we will build an `mvn clean package`
* then we will run it. (check the ./run.sh)
  * `export CLASSPATH="$(find target/ -type f -name '*.jar'|grep '\-packae' | tr '\n' ':')"`
  * build the Dockerfile `docker build . -t simpesteph/kafka-connect-source-github:1.0`
  * eun the docker image `docker run -e CLASSPATH=$CLASSPATH --net=host --rm -t -v $(pwd)/offsets:/kafka-connect-source-github/offsets simplesteph/kafka-connect-source-github:1.0`
* in our project folder wher pom.xml resides we run `mvn clean package`
* the built jars are at `<pwd>/target/kafka-connect-project-1.0-SNAPSHOT-package/share/java/kafka-connect-project`
* we cp the .properties files in ./config
* we run the .run.sh `sudo ./run.sh`
```
#!/usr/bin/env bash
export CLASSPATH="$(find target/ -type f -name '*.jar'| grep '\-package' | tr '\n' ':')"
if hash docker 2>/dev/null; then
    # for docker lovers
    docker build . -t achliopa/kafka-connect-source-github:1.0
    docker run --net=host --rm -t \
           -v $(pwd)/offsets:/kafka-connect-source-github/offsets \
           achliopa/kafka-connect-source-github:1.0
elif hash connect-standalone 2>/dev/null; then
    # for mac users who used homebrew
    connect-standalone config/worker.properties config/GitHubSourceConnectorExample.properties
elif [[ -z $KAFKA_HOME ]]; then
    # for people who installed kafka vanilla
    $KAFKA_HOME/bin/connect-standalone.sh $KAFKA_HOME/etc/schema-registry/connect-avro-standalone.properties config/MySourceConnector.properties
elif [[ -z $CONFLUENT_HOME ]]; then
    # for people who installed kafka confluent flavour
    $CONFLUENT_HOME/bin/connect-standalone $CONFLUENT_HOME/etc/schema-registry/connect-avro-standalone.properties config/MySourceConnector.properties
else
    printf "Couldn't find a suitable way to run kafka connect for you.\n \
Please install Docker, or download the kafka binaries and set the variable KAFKA_HOME."
fi;
```

* we also add a Dockerfile according to the example to build an image
```
FROM confluentinc/cp-kafka-connect:3.2.0

WORKDIR /kafka-connect-source-github
COPY config config
COPY target target

VOLUME /kafka-connect-source-github/config
VOLUME /kafka-connect-source-github/offsets

CMD CLASSPATH="$(find target/ -type f -name '*.jar'| grep '\-package' | tr '\n' ':')" connect-standalone config/worker.properties config/GitHubSourceConnectorExample.properties
```

* the docker build uses as base the 'confluentinc/cp-kafka-connect' image to build our connector
* make sure to mod the properties file for our project

### Lecture 48. Deploying our Connector on the Landoop cluster

* we have used maven to build
* extract the jars
* mount the jars into th elandoop image
* use connect UI to launch the connector
* we use to run the landoop image mounting the jars

`docker run -it --rm -p 2181:2181 -p 3030:3030 -p 8081:8081 -p 8082:8082 -p 8083:8083 -p 9092:9092 -e ADV_HOST=127.0.0.1 -e RUNTESTS=0 -v ~/workspace/projects/IdeaProjects/kafka-connect-project/target/kafka-connect-project-1.0-SNAPSHOT-package/share/java/kafka-connect-project:/connectors/GitHub landoop/fast-data-dev:cp3.3.0`
* once the cluster is loaded if we visit loacalhost:3030 and ConnectorUI our connector is appearing in the list if we clisk new
* we see the config
* we pass conif user/repo/topic like in our project properties file
* we check our topic and see it loaded with data

### Lecture 49. More Resources for Developers

* Sink Connectors have a similar API as Source Connectors. Slight differences but we can tackle them
* check more [examples](https://www.confluent.io/product/connectors)
  * list open source connectors
  * check the code
* [development guide confluent](https://www.confluent.io/wp-content/uploads/Partner-Dev-Guide-for-Kafka-Connect.pdf?x18424)
* [kafka development guide](https://docs.confluent.io/3.2.0/connect/devguide.html)
* [guidelines](https://docs.confluent.io/3.2.0/connect/devguide.html)

## Section 9: Advanced Concepts

### Lecture 50. Setting up Kafka Connect in Production (1/2)

* We know how to use connetors
* we know how to create connectors
* We know how to add connectors
* We know how to use the landoop Kafka Connect UI
* This is OK for Developemnt
* To setup Kafka Connect in production
  * Download Kafka Binaries
  * Setup the connect-distributed properties as needed
  * Add the jars where needed (plugins path or classpath)
  * (optional) setup the landoop kafka connect UI
* we will interact with Kafka Connect using the REST API, or the UI if we set it up
 *[Donwload](https://kafka.apache.org/downloads) the latest stable Kafka binary
 * we untar it and in config/ we have all .properties files
 * we will see the connect-distributed.properties file
 * in there we will see the props needed to run a connect in an already running kafka cluster
 * setup.sh from course notes explains how to setup connect distributed on a running cluster
 ```
 #!/bin/bash

# 0. Make sure Java is installed on your machine https://java.com/en/download/
# 1. Download Kafka at https://kafka.apache.org/downloads (>= 0.11.0.1)
# 2. Unzip Kafka to a directory of your choice (for example ~/kafka)

# 3. Open a terminal or text editor on the Kafka directory
# 4. Edit the file at config/connect-distributed.config
nano config/connect-distributed.properties
vi config/connect-distributed.properties
atom config/connect-distributed.properties

# 5. change bootstrap.servers to the correct ones
# 6. change rest.port=8083 to rest.port=8084 (or any available port)
# 7. Optional (if you want a separate cluster) -
  # 7a. Set offset.storage.topic=connect-offsets-2
  # 7b. Set config.storage.topic=connect-configs-2
  # 7c. Set status.storage.topic=connect-status-2
  # 7d. Set group.id=connect-cluster-2
# 8. Set plugin.path=/directory/of/your/choice
# 9. Adjust any other settings that matters to your setup (see doc)
# 10. Place any connector (jars) you need in the plugin.path sub-directory

# 11. In order to add connect workers, duplicate the connect-distributed file and change the rest.port if running on the same host. The rest of the settings are the same

# 12. Optionally, setup Kafka Connect UI from Landoop
  # Instructions are at: https://github.com/Landoop/kafka-connect-ui
  # You can build from source or use docker for that (see their github)
 ```

 * setup the bootstrap server. connect program needs to find kafka
 * group-id. if we leave connect-cluster we will join the cluster created for us by landoop. we give a fresh name like 'connect-cluster-2'
 * converters can be overriden in connectors so we leave them
 * internal keys we dont touch
 * then we see a topic for storing offset config etc. we mod them adding a -2 to not create a conflict with landoop
 * in production we should consider replication factor
 * flash interval usualy 60000  (1minute)
 * uncomment the rest.port (if we run landoop we should pick 8084 instead of 8083
 * we also uncommnent and set the plugin.path to the kars folder
 * we first cp the file in our projects and mod it to keep the originam clean
 * from our project folder where we hav ethe properties file we run `connect-distributed.sh ./connect-distributed.properties`
 * we know it is running if we hit the REST API with success `curl localhost:8084`
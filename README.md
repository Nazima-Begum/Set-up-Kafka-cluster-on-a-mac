# Set-up-Kafka-cluster-on-a-mac

Steps followed to do hands on kafka cluster setup on mac:

1.	**Download and extract kafka**
-      Visit the Apache Kafka website: https://kafka.apache.org/downloads
-	Download the latest version of Apache Kafka for Scala 2.13 (or any other preferred version).
-	Extract the downloaded Kafka archive to a directory of your choice.
  
2.	**Start zookeeper**
-	Open a terminal window
-	Navigate to the Kafka directory where you extracted the Kafka archive
-	Start Zookeeper by running the following command:
```bin/zookeeper-server-start.sh config/zookeeper.properties```
NOTE: 
if there’s error /kafka-run-class.sh: line 347: /usr/bin/java/bin/java: Not a directory
Then Set JAVA_HOME and Update Kafka scripts
i. check : echo $JAVA_HOME // if this is pointing to default path usr/bin/java then add below to your .bash_profile file (user/nazbegum/.bash_profile 
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk-<version>.jdk/Contents/Home
save and run this command in terminal to reflect is : source ~/.bash_profile
ii. Navigate to kafka directory and open bin/kafka-run-class.sh and edit 
```
if [ -z "$JAVA" ]; then
    JAVA="/usr/bin/java"
Fi


if [ -z "$JAVA" ]; then
    JAVA="$JAVA_HOME/bin/java"
fi
```

3.	**Start multiple kafka brokers**
-	To start the first kafka broker (we can use the existing config/server.properties)
```bin/kafka-server-start.sh config/server.properties```
-	But to have multiple kafka broker need to do following changes:
i.	Navigate to kafka directory and duplicate config/server.properties file for each aaditional broker we want to create, eg: create server-1.properties, server-2.properties etc
ii.	Configure broker properties (in each of the above newly created server properties files)
broker.id : set unique broker id for each broker
listeners : set the listener configuration to use different ports for each broker, eg: PLAINTEXT://localhost:9092, PLAINTEXT://localhost:9093, PLAINTEXT://localhost:9094
iii.	log.dirs to specify different log directories for each broker, eg: tmp/kafka-logs-1, tmp/kafka-logs-2
-	Now start additional kafka brokers
```bin/kafka-server-start.sh config/server-1.properties```
```bin/kafka-server-start.sh config/server-2.properties```
Add more commands for additional brokers as needed
NOTE: this multiple kafka brokers had to be setup as otherwise it is not possible to create a topic with replication factor > 1 (see below snapsot)
 

4.	**Create Topic with partition count = 3 and replication factor = 3**
```bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 3 --partitions 3 --topic test-topic```

5.	**Verify topic creation**
```bin/kafka-topics.sh --describe --bootstrap-server localhost:9092 --topic test-topic```

6.	Start multiple consumer in consumer group (eg: here 3 consumers)
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test-topic --group test-consumer-group 
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test-topic --group test-consumer-group 
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test-topic --group test-consumer-group
-	Verify the created consumers
bin/kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group test-consumer-group –describe


7.	**Start producer**
```bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test-topic```

Note: once producer started producing messages – it was observed that all messages were ending up with same consumer
In other words all messages were getting produced on same partition

Reason: Default Partitioning Strategy: If the producer is not explicitly setting a partition key for messages, Kafka uses the default partitioning strategy, which is round-robin. In some cases, this may lead to messages being sent to the same partition, especially if there are fewer producers or if the messages are produced at a slow rate

Hence Stopped the current producer and re-created by specifying the 2 properties as below:
```bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test-topic --property "parse.key=true" --property "key.separator=:"```
sample messages produced are
>key1:msg1
>key2:msg2
Now messages were getting evenly distributed among the consumers
(after the 10 messages from earlier had produced 6 more new messages with this key partitioning and they got distributed as 2 + 2 + 2 among 3 consumers)
 

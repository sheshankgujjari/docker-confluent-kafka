***********************  CONFLUENT-CLI-COMMANDS  *****************************************************

Start all the services
```
$ confluent start
```
Retrieve their status
```
$ confluent status
```
Open the log file of a service
```
$ confluent log connect
```
Access runtime stats of a service
```
$ confluent top kafka
```
Discover the availabe Connect plugins
```
$ confluent list plugins
```
or list the predefined connector names
```
$ confluent list connectors
```
Load a couple connectors
```
$ confluent load file-source
$ confluent load file-sink
```
Get a list with the currently loaded connectors
```
$ confluent status connectors
```
Check the status of a loaded connector
```
$ confluent status file-source
```
Read the configuration of a connector
```
$ confluent config file-source
```
Reconfigure a connector
```
$ confluent config file-source -d ./updated-file-source-config.json
or reconfigure using a properties file
$ confluent config file-source -d ./updated-file-source-config.properties
```
Figure out where the data and the logs of the current confluent run are stored
```
$ confluent current
```
Unload a specific connector
```
$ confluent unload file-sink
```
Stop the services
```
$ confluent stop
```
Start on a clean slate next time (deletes data and logs of a confluent run)
```
$ confluent destroy
```


************************  PORTS INFORMATION  ********************************************************

Component	Port
Apache Kafka brokers (plain text)	9092
Confluent Control Center	9021
Kafka Connect REST API	8083
REST Proxy	8082
Schema Registry REST API	8081
ZooKeeper  2181




*********************  DOCKER-COMPOSE  ****************************************************************************

git clone https://github.com/confluentinc/cp-docker-images

cd <path-to-cp-docker-images>/examples/kafka-single-node/
```
$ docker-compose up -d
```

```
$ docker-compose logs zookeeper | grep -i binding
```

```
docker-compose logs kafka | grep -i started
```

CREATE TOPIC
```
$ docker-compose exec kafka  \
kafka-topics --create --topic foo --partitions 1 --replication-factor 1 --if-not-exists --zookeeper localhost:32181
```

TOPIC INFORMATION
```
$ docker-compose exec kafka  \
  kafka-topics --describe --topic foo --zookeeper localhost:32181
```

PRODUCE MESAGE
```
$ docker-compose exec kafka  \
  bash -c "seq 42 | kafka-console-producer --request-required-acks 1 --broker-list localhost:29092 --topic foo && echo 'Produced 42 messages.'"
```

READ MESSAGE
```
docker-compose exec kafka  \
  kafka-console-consumer --bootstrap-server localhost:29092 --topic foo --from-beginning --max-messages 42
```


***********************  DOCKER-CLIENT  ***********************************************************************************************

```
$ docker-machine start confluent
```

configure your terminal window to attach it to your new Docker Machine

```
$ eval $(docker-machine env confluent)
```

##############################  ZOOKEEPER  ############################################################

```
$ docker run -d \
    --net=host \
    --name=zookeeper \
    -e ZOOKEEPER_CLIENT_PORT=32181 \
    confluentinc/cp-zookeeper:4.0.0

```


```
$ docker logs zookeeper
```



##############################  KAFKA  ############################################################

```
$ docker run -d \
    --net=host \
    --name=kafka \
    -e KAFKA_ZOOKEEPER_CONNECT=localhost:32181 \
    -e KAFKA_ADVERTISED_LISTENERS=PLAINTEXT://localhost:29092 \
    -e KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR=1 \
    confluentinc/cp-kafka:4.0.0
```


```
$ docker logs kafka
```


##############################  CREATE TOPIC  ############################################################

```
$ docker run \
  --net=host \
  --rm confluentinc/cp-kafka:4.0.0 \
  kafka-topics --create --topic foo --partitions 1 --replication-factor 1 --if-not-exists --zookeeper localhost:32181
```


##############################  VERIFY TOPIC  ############################################################

```
$ docker run \
  --net=host \
  --rm \
  confluentinc/cp-kafka:4.0.0 \
  kafka-topics --describe --topic foo --zookeeper localhost:32181
```



##############################  PUBLISH DATA  ############################################################

```
$ docker run \
  --net=host \
  --rm \
  confluentinc/cp-kafka:4.0.0 \
  bash -c "seq 42 | kafka-console-producer --request-required-acks 1 --broker-list localhost:29092 --topic foo && echo 'Produced 42 messages.'"
```


##############################  READ MESSAGES  ############################################################

```
$ docker run \
  --net=host \
  --rm \
  confluentinc/cp-kafka:4.0.0 \
  kafka-console-consumer --bootstrap-server localhost:29092 --topic foo --from-beginning --max-messages 42

```

##############################  SCHEMA REGISTRY  ############################################################

```
$ docker run -d \
  --net=host \
  --name=schema-registry \
  -e SCHEMA_REGISTRY_KAFKASTORE_CONNECTION_URL=localhost:32181 \
  -e SCHEMA_REGISTRY_HOST_NAME=localhost \
  -e SCHEMA_REGISTRY_LISTENERS=http://localhost:8081 \
  confluentinc/cp-schema-registry:4.0.0

```

```
$ docker logs schema-registry
```

SCHEMA REGISTRY BASH

```
$ docker run -it --net=host --rm confluentinc/cp-schema-registry:4.0.0 bash
```

Note: we need to REST PROXY docker to execute curl commands



##############################  REST PROXY  ############################################################

```
$ docker run -d \
  --net=host \
  --name=kafka-rest \
  -e KAFKA_REST_ZOOKEEPER_CONNECT=localhost:32181 \
  -e KAFKA_REST_LISTENERS=http://localhost:8082 \
  -e KAFKA_REST_SCHEMA_REGISTRY_URL=http://localhost:8081 \
  -e KAFKA_REST_HOST_NAME=localhost \
  confluentinc/cp-kafka-rest:4.0.0

```


Terminal Bash inside the docker

```
$ docker run -it --net=host --rm confluentinc/cp-schema-registry:4.0.0 bash
```

REST Proxy is to create a consumer instance from inside the bash

```
$ curl -X POST -H "Content-Type: application/vnd.kafka.v1+json" \
  --data '{"name": "my_consumer_instance", "format": "avro", "auto.offset.reset": "smallest"}' \
  http://localhost:8082/consumers/my_avro_consumer
```


GET data from topic from inside the bash

```
$ curl -X GET -H "Accept: application/vnd.kafka.avro.v1+json" \
  http://localhost:8082/consumers/my_avro_consumer/instances/my_consumer_instance/topics/bar
```


##############################  EXAMPLES  ##########################################################################################


============================================= Test Schema ==========================================================================

```
$ /usr/bin/kafka-avro-console-producer \
  --broker-list localhost:29092 --topic bar \
  --property value.schema='{"type":"record","name":"myrecord","fields":[{"name":"f1","type":"string"}]}'
```


```
$ curl -X POST -H "Content-Type: application/vnd.kafka.v1+json" \
  --data '{"name": "my_consumer_instance", "format": "avro", "auto.offset.reset": "smallest"}' \
  http://localhost:8082/consumers/my_avro_consumer
```

```
$ curl -X GET -H "Accept: application/vnd.kafka.avro.v1+json" \
  http://localhost:8082/consumers/my_avro_consumer/instances/my_consumer_instance/topics/bar
```


===================================  Customer CorrelationID schema  ==================================================================

```
$ /usr/bin/kafka-avro-console-producer \
  --broker-list localhost:29092 --topic foobar1 \
  --property value.schema='{"type": "record", "name":"ApplicationArea","fields": [{"name": "CorrelationID","type": "string"}]}'
 {"CorrelationID":"test1"}
{"CorrelationID":"test2"}

ctrl+C
```


```
$ curl -X POST -H "Content-Type: application/vnd.kafka.v1+json" \
  --data '{"name": "my_consumer_instance", "format": "avro", "auto.offset.reset": "smallest"}' \
  http://localhost:8082/consumers/my_avro_consumer
```


```
$ curl -X GET -H "Accept: application/vnd.kafka.avro.v1+json"   http://localhost:8082/consumers/my_avro_consumer/instances/my_consumer_instance/topics/foobar1
```

output:
  '
  [{"topic":"foobar1","key":null,"value":{"CorrelationID":"test1"},"partition":0,"offset":0},{"topic":"foobar1","key":null,"value":{"CorrelationID":"test2"},"partition":0,"offset":1}]'


======================================== Customer CorrelationID and Sender record with ID field =============================================

```
$ /usr/bin/kafka-avro-console-producer \
  --broker-list localhost:29092 --topic foobar2 \
  --property value.schema='{"type": "record","name": "ApplicationArea","fields": [{"name": "CorrelationID","type": "string"},{"name": "Sender", "type":{"type": "record","name": "Sender_Schema","fields":[{"name": "ID","type": "string"}]}}]}'
{"CorrelationID": "0b6e690b","Sender": {"ID": "blah"}}
```

```
$ curl -X POST -H "Content-Type: application/vnd.kafka.v1+json" \
  --data '{"name": "foobar2_instance", "format": "avro", "auto.offset.reset": "smallest"}' \
  http://localhost:8082/consumers/foobar2_consumer
```

```
$ curl -X GET -H "Accept: application/vnd.kafka.avro.v1+json"   http://localhost:8082/consumers/foobar2_consumer/instances/foobar2_instance/topics/foobar2
```


****************************  Confluent Control Center (Monitoring)  *****************************************************

```
$ docker-machine ssh confluent
```

```
docker@confluent:~$ mkdir -p /tmp/control-center/data
docker@confluent:~$ exit
```

```
$ docker run -d \
  --name=control-center \
  --net=host \
  --ulimit nofile=16384:16384 \
  -p 9021:9021 \
  -v /tmp/control-center/data:/var/lib/confluent-control-center \
  -e CONTROL_CENTER_ZOOKEEPER_CONNECT=localhost:32181 \
  -e CONTROL_CENTER_BOOTSTRAP_SERVERS=localhost:29092 \
  -e CONTROL_CENTER_REPLICATION_FACTOR=1 \
  -e CONTROL_CENTER_MONITORING_INTERCEPTOR_TOPIC_PARTITIONS=1 \
  -e CONTROL_CENTER_INTERNAL_TOPICS_PARTITIONS=1 \
  -e CONTROL_CENTER_STREAMS_NUM_STREAM_THREADS=2 \
  -e CONTROL_CENTER_CONNECT_CLUSTER=http://localhost:28082 \
  confluentinc/cp-enterprise-control-center:4.0.0
```

```
$ docker logs control-center | grep Started
```

```
$ docker-machine ip confluent
```

Browse
http://<ip-of-docker-host>:9021


Create Topic for monitoring

```
$ docker run \
  --net=host \
  --rm confluentinc/cp-kafka:4.0.0 \
  kafka-topics --create --topic test-monitoring --partitions 1 --replication-factor 1 --if-not-exists --zookeeper localhost:32181
```

Produce 1000 messages

```
$ while true;
do
  docker run \
    --net=host \
    --rm \
    -e CLASSPATH=/usr/share/java/monitoring-interceptors/monitoring-interceptors-4.0.0.jar \
    confluentinc/cp-kafka-connect:4.0.0 \
    bash -c 'seq 10000 | kafka-console-producer --request-required-acks 1 --broker-list localhost:29092 --topic test-monitoring --producer-property interceptor.classes=io.confluent.monitoring.clients.interceptor.MonitoringProducerInterceptor --producer-property acks=1 && echo "Produced 10000 messages."'
    sleep 10;
done
```

Consume:

```
$ OFFSET=0
$ while true;
do
  docker run \
    --net=host \
    --rm \
    -e CLASSPATH=/usr/share/java/monitoring-interceptors/monitoring-interceptors-4.0.0.jar \
    confluentinc/cp-kafka-connect:4.0.0 \
    bash -c 'kafka-console-consumer --consumer-property group.id=qs-consumer --consumer-property interceptor.classes=io.confluent.monitoring.clients.interceptor.MonitoringConsumerInterceptor --bootstrap-server localhost:29092 --topic test-monitoring --offset '$OFFSET' --partition 0 --max-messages=1000'
  sleep 1;
  let OFFSET=OFFSET+1000
done
```



****************************  Delete Docker Containers  ***************************************************************************

```
$ docker rm -f $(docker ps -a -q)
```

```
$ docker-machine ls
```

```
$ docker-machine rm <your machine name>
```

****************************  END  ***************************************************************************










# Práctica BigData Processing
Ignacio Gallegos Sánchez

## Parte Obligatoria (Spark Streaming)

### Kafka

./bin/zookeeper-server-start.sh config/zookeeper.properties
./bin/kafka-server-start.sh config/server.properties

./bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic keepcoding
./bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic keepcoding --from-beginning


### Twitter producer

El producer está en el proyecto paralelo de mi repositorio:

https://github.com/ignaciogallegossanchez/TwitterKafkaProducer

Lo que hace es:
recibe los mensajes de twitter
los encrypta
los envía a kafka

### Sniffer 

## Parte opcional (GraphX)

<No implementada>

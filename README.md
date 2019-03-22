# docker-compose

```yml
version: '2'
services:
  zookeeper:
    image: confluentinc/cp-zookeeper:5.1.1
    #network_mode: host
    ports:
      - 2181:2181
    environment:
      ZOOKEEPER_CLIENT_PORT: 2181
      ZOOKEEPER_TICK_TIME: 2000
    extra_hosts:
      - "moby:127.0.0.1"
      - "default:127.0.0.1"
  kafka:
    image: confluentinc/cp-kafka:5.1.1
    #network_mode: host
    ports:
      - 9092:9092
    depends_on:
      - zookeeper
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://127.0.0.1:9092
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
    extra_hosts:
      - "moby:127.0.0.1"
      - "default:127.0.0.1"
  ksql:
    image: confluentinc/cp-ksql-server:5.1.1
    #network_mode: host
    depends_on:
      - kafka
    ports:
      - 8088:8088
    environment:
      KSQL_BOOTSTRAP_SERVERS: 127.0.0.1:9092
      KSQL_LISTENERS: http://0.0.0.0:8088
      KSQL_KSQL_SERVICE_ID: ksql_service_id
    links:
      - kafka
    extra_hosts:
      - "moby:127.0.0.1"
      - "default:127.0.0.1"
```

# Run KSQL 
```
docker run -it confluentinc/cp-ksql-cli:5.1.1 http://{id_do_host}:8088
```

# Commandos Úteis

### Criar tópico

```bash 
kafka-topics --zookeeper 127.0.0.1:2181 --topic first_topic --create --partitions 3 --replication-factor 1
```

### Listar Tópicos

```bash 
kafka-topics --zookeeper 127.0.0.1:2181 --list
```

### Ver informações detalhadas de um tópico

```bash 
kafka-topics --zookeeper 127.0.0.1:2181 --topic first_topic --describe
```

### Deletar Tópico
```bash 
kafka-topics --zookeeper 127.0.0.1:2181 --topic sedond_topic --delete
```

### Produzir Mensagens para um Tópico
```bash 
kafka-console-producer --broker-list 127.0.0.1:9092 --topic first_topic
```

### Consumir um tópico diretamente
```bash 
kafka-console-consumer --bootstrap-server 127.0.0.1:9092 --topic first_topic
```

### Consumir um tópico a partir de um grupo de consumidores
```bash 
kafka-console-consumer --bootstrap-server 127.0.0.1:9092 --topic first_topic --group my_first_group
```

### Listar grupo de consumidores
```bash 
kafka-consumer-groups --bootstrap-server 127.0.0.1:9092 --list
```

### Exibir informações do grupo
```bash 
kafka-consumer-groups --bootstrap-server 127.0.0.1:9092 --describe --group my_first_group
```

### Resetar os offsets commitados em um tópico
```bash 
kafka-consumer-groups --bootstrap-server localhost:9092 --group my_first_group --reset-offsets --to-earliest --execute --topic first_topic
```

### Produzir Mensagem com Chave e Valor
```bash 
kafka-console-producer --broker-list 127.0.0.1:9092 --topic first_topic --property parse.key=true --property key.separator=,
> key,value
> another key,another value
```

### Consumir Mensagens Exibindo a Chave e Valor
```bash 
kafka-console-consumer --bootstrap-server 127.0.0.1:9092 --topic first_topic --from-beginning --property print.key=true --property key.separator=,
```

### Console Web para Kafka
https://github.com/yahoo/kafka-manager


## kafka ConnectWorkes Standalone vc Distributed Mode
* Standalone:
    * A single process runs your connectors and tasks
    * Configuration is bundled with your process
    * very * easy to get started with
    * Not fault tolerant, no scalability, hard to monitor
* Distributed:
    * Muiltiple workers run your connectors and tasks
    * Configuration is submitted using a REST API
    * Easy to scale, and fault tolerant (rebalancing in case a worker die)
    * useful for production deployments of connectors

## Linger.ms & batch.size
* By default, Kafka tries to send records as soon as possibile
    * It will have up to 5 requests in flight, meaning up to 5 messages individually sent at the same time.
    * After this, it more messages hava to be sent while others are in flight, Kafka is smart and will start batching 
them while they wait to send them all at once.
* This smart batching allows Kafka to increase throughput while maitaining very low latency.
* Batches hava higher compression ratio so better efficiency.

* So how can we control the batching mechanism?
* Linger.ms: Number of milliseconds a produces is willing to wait before sending a batch out (default 0)
* By introducing some lag (for example linger.ms=5), we increase the chances of messages being
send together in a batch.
* If a batch is full (see batch.size) before the end of the linger.ms period, it will be sent to 
Kafka right away.
* **batch.size**: Maximum number of bytes that will be included in a batch. The default is 16kb.
* Increasing a batch size to something like 32kb or 64kb can help increasing the compression, 
throughput, and efficiency of requests.
* Any message that is bigger than the batch size will not be batched.
* A batch is allocated per partition, so make sure that you don't set it to a number that's too
high, otherwise you'll run waste memory.
* Note: you can monitor the average batch size metric using Kafka Producer Metrics.

## High Throughput Producer Demo
* We'll add snappy message compression in our producer
* Snappy is very helpful if your message are text based, for example log lines of JSON documents
* Snappy has a good balance of CPU / compression ratio
* We'll also increase the batch.size to 32kb and introduce a small delay through linger.ms (20 ms)

´´´
properties.setProperty(ProducerConfig.COMPRESION_TYPE_CONFIG, "snappy");
properties.setProperty(ProducerConfig.LINGER_MS_CONFIG, "20");
properties.setProperty(ProducerConfig.BATCH_SIZE_CONFIG, Integer.toString(32*1024)); //32kb batch size
´´´

## Producer Default Partition and how keys are hashed
* By default, your keys area hashed using the "murmur2" algorithm
* It is most likely preferred to not override the behavior of the partitioner, but it is possibile
to do so (partitioner.class)
* The formula is: targetPartition = Utils.abs(Utils.mumur2(record.key())) % numParticions;
* This mean that same key will go to the same partition (we already know this) and adding
partitions to a topic will completely alter the formula.

## Max.block.ms & buffer.memory
* If the producer produces faster than the broker can take, the records will be buffered in memory
* **buffer.memory=33554432 (32mb)** the size of the send buffer
* That buffer will fill up over time and fill back down when the throughput to the broker increases


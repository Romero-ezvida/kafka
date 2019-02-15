# docker-compose

```yml
version: '2'
services:
  zookeeper:
    image: confluentinc/cp-zookeeper:5.1.1
    network_mode: host
    environment:
      ZOOKEEPER_CLIENT_PORT: 32181
      ZOOKEEPER_TICK_TIME: 2000
    extra_hosts:
      - "moby:127.0.0.1"
      - "default:127.0.0.1"
  kafka:
    image: confluentinc/cp-kafka:5.1.1
    network_mode: host
    depends_on:
      - zookeeper
    environment:
      KAFKA_BROKER_ID: 1
      KAFKA_ZOOKEEPER_CONNECT: localhost:32181
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://localhost:29092
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
    extra_hosts:
      - "moby:127.0.0.1"
      - "default:127.0.0.1"
```





# Commandos Úteis

## Criar tópico

```bash 
kafka-topics.sh --zookeeper 127.0.0.1:2181 --topic first_topic --create --partitions 3 --replication-factor 1
```

## Listar Tópicos

```bash 
kafka-topics.sh --zookeeper 127.0.0.1:2181 --list
```

## Ver informações detalhadas de um tópico

```bash 
kafka-topics.sh --zookeeper 127.0.0.1:2181 --topic first_topic --describe
```

## Deletar Tópico
```bash 
kafka-topics.sh --zookeeper 127.0.0.1:2181 --topic sedond_topic --delete
```

## Produzir Mensagens para um Tópico
```bash 
kafka-console-producer.sh --broker-list 127.0.0.1:9092 --topic first_topic
```

## Consumir um tópico diretamente
```bash 
kafka-console-consumer.sh --bootstrap-server 127.0.0.1:9092 --topic first_topic
```

## Consumir um tópico a partir de um grupo de consumidores
```bash 
kafka-console-consumer.sh --bootstrap-server 127.0.0.1:9092 --topic first_topic --group my_first_group
```

## Listar grupo de consumidores
```bash 
kafka-consumer-groups.sh --bootstrap-server 127.0.0.1:9092 --list
```

## Exibir informações do grupo
```bash 
kafka-consumer-groups.sh --bootstrap-server 127.0.0.1:9092 --describe --group my_first_group
```

## Resetar os offsets commitados em um tópico
```bash 
kafka-consumer-groups.sh --bootstrap-server localhost:9092 --group my_first_group --reset-offsets --to-earliest --execute --topic first_topic
```

## Produzir Mensagem com Chave e Valor
```bash 
kafka-console-producer --broker-list 127.0.0.1:9092 --topic first_topic --property parse.key=true --property key.separator=,
> key,value
> another key,another value
```

## Consumir Mensagens Exibindo a Chave e Valor
```bash 
kafka-console-consumer --bootstrap-server 127.0.0.1:9092 --topic first_topic --from-beginning --property print.key=true --property key.separator=,
```

## Console Web para Kafka

https://github.com/yahoo/kafka-manager

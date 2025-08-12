# Level 2: Multi Partition Topics

```bash
# [host:1] run the docker compose file
docker compose up -d

# [host:1] go inside the kafka container
docker exec -it kafka bash

# [kafka:1] create a topic with 3 partitions
kafka-topics.sh --create --topic multi-part-topic --bootstrap-server localhost:9092 --partitions 3 --replication-factor 1

# [kafka:1] describe kafka topic
kafka-topics.sh --describe --topic multi-part-topic --bootstrap-server localhost:9092

# [kafka:2] create a consumer with partition information
kafka-console-consumer.sh --topic multi-part-topic --bootstrap-server localhost:9092 --property print.key=true --property print.partition=true --property key.separator=" : " --from-beginning

# [kafka:1] create a producer without specifying keys
kafka-console-producer.sh --topic multi-part-topic  --bootstrap-server localhost:9092

# [kafka:1] give some input
# > hello
# > world
# > i am a consumer
# > kafka is fun

# [kafka:2] meanwhile the output seems to be like
# Partition:0 : null :  hello
# Partition:0 : null :  world
# Partition:0 : null :  i am a producer
# Partition:0 : null :  kafka is fun


# why are all partitions 0? 
# - kafka-console-producer does not round-robin messages for each send
# - It chooses one partition for the entire session if no key is provided
# - Real producer clients round-robin by default when no key is set

# [kafka:1] exit

# [host:1] run a seperate console producer for messages
for msg in hello world kafka fun; do
  echo $msg | docker exec -i kafka \
    kafka-console-producer.sh --topic multi-part-topic --bootstrap-server localhost:9092
done

# [kafka:2] output, we can see the round robin nature of partitioning

# Partition:0 : null :  hello
# Partition:0 : null :  world
# Partition:0 : null :  i am a producer
# Partition:0 : null :  kafka is fun
# Partition:2 : null : hello
# Partition:0 : null : world
# Partition:1 : null : kafka
# Partition:2 : null : fun

# [kafka:1] create a producer with keys
kafka-console-producer.sh --topic multi-part-topic --bootstrap-server localhost:9092 --property "parse.key=true" --property "key.separator=:"

# [kafka:1] give some input
# >key1:msg-a
# >key2:msg-b
# >key3:msg-c
# >key1:msg-d
# >key2:msg-e
# >key3:msg-f
# >key4:msg-g
# >key4:msg-h

# [kafka:2] output (continued from the previous one, the window is kept alive)

# Partition:2 : key1 : msg-a
# Partition:2 : key2 : msg-b
# Partition:1 : key3 : msg-c
# Partition:2 : key1 : msg-d
# Partition:2 : key2 : msg-e
# Partition:1 : key3 : msg-f
# Partition:0 : key4 : msg-g
# Partition:0 : key4 : msg-h

# Observations:
# 1. Same key → same partition (hash-based mapping)
# 2. Ordering is guaranteed within a partition
# 3. Without keys → default round robin (if using a real producer, not CLI single session)

# [kafka:1, kafka:2] exit

# [host] tear down the cluster
docker compose down
```
# Level 1: Kafka Basics

```bash
# [host] run the docker compose file
docker compose up -d

# [host] go inside the kafka container
docker exec -it kafka bash

# [kafka] create a topic
kafka-topics.sh --create \
  --topic test-topic \
  --bootstrap-server localhost:9092 \
  --partitions 1 \
  --replication-factor 1

# [kafka] list kafka topics
kafka-topics.sh --list --bootstrap-server localhost:9092

# open two terminals side by side [kafka:1] [kafka:2]

# [kafka:1] create a producer
kafka-console-producer.sh --topic test-topic  --bootstrap-server localhost:9092

# [kafka:1] write some data in the producer window
# > hello
# > world
# > i am a producer

# note: (do not stop this process)

# [kafka:2] create a consumer
kafka-console-consumer.sh --topic test-topic --bootstrap-server localhost:9092 --from-beginning 

# output: 
# hello
# world
# i am a producer

# [kafka:1] write some more data
# > kafka is fun

# [kafka:2] visit terminal 2
# the new data can be seen on the screen
# final output:
# hello
# world
# i am a producer
# kafka is fun

# [kafka:1, kafka:2] stop both processes using ctrl+c

# [kafka:2] create a consumer again using the same command

# the data still persists
# output: 

# hello
# world
# i am a producer
# kafka is fun

# [kafka:all] stop everything and exit

# [host] docker compose down
docker compose down
```
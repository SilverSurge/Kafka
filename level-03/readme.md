# Level 3: Consumer Groups

```bash
# [host:1] run the docker compose file
docker compose up

# [kafka:1] create a group with 3 partitions
kafka-topics.sh --create --topic group-test-topic \
  --bootstrap-server localhost:9092 \
  --partitions 3 \
  --replication-factor 1

# [kafka:1] create a consumer with group 'my-group'
kafka-console-consumer.sh --topic group-test-topic \
  --bootstrap-server localhost:9092 \
  --group my-group \
  --property print.partition=true

# [kafka:2] create a consumer with group 'my-group'
kafka-console-consumer.sh --topic group-test-topic \
  --bootstrap-server localhost:9092 \
  --group my-group \
  --property print.partition=true


# [kafka:3] create a producer 
kafka-console-producer.sh --topic group-test-topic --bootstrap-server localhost:9092
# give some input
# stop the process and run again
# give some input
# as discussed before kafka-console-producer does not round-robin messages for each send

# [kafka:1] output
# Partition:1     msg-7
# Partition:1     msg-8
# Partition:1     msg-9
# Partition:1     msg-10
# Partition:0     msg-11
# Partition:0     msg-12
# Partition:0     msg-13

# [kafka:2] output
# Partition:2     msg-1
# Partition:2     msg-2
# Partition:2     msg-3
# Partition:2     msg-4
# Partition:2     msg-5
# Partition:2     msg-6

# [kafka:2] lets kill consumer two

#  [kafka:3] create a producer (repeatition)
kafka-console-producer.sh --topic group-test-topic --bootstrap-server localhost:9092
# give some input
# stop the process and run again
# give some input
# as discussed before kafka-console-producer does not round-robin messages for each send

# [kafka:1] output
# Partition:1     msg-7
# Partition:1     msg-8
# Partition:1     msg-9
# Partition:1     msg-10
# Partition:0     msg-11
# Partition:0     msg-12
# Partition:0     msg-13
# Partition:0     msg-14
# Partition:0     msg-15
# Partition:0     msg-16
# Partition:2     msg-17
# Partition:2     msg-18
# Partition:2     msg-19
# Partition:2     msg-20

# we see rebalancing in action

# [kafka:1] stop the consumer process

# [kafka:1] create a consumer with group 'group-A'
kafka-console-consumer.sh --topic group-test-topic \
  --bootstrap-server localhost:9092 \
  --group group-A \
  --property print.partition=true \
  --from-beginning

# [kafka:1] output
# Partition:0     msg-11
# Partition:0     msg-12
# Partition:0     msg-13
# Partition:0     msg-14
# Partition:0     msg-15
# Partition:0     msg-16
# Partition:1     msg-7
# Partition:1     msg-8
# Partition:1     msg-9
# Partition:1     msg-10
# Partition:2     msg-1
# Partition:2     msg-2
# Partition:2     msg-3
# Partition:2     msg-4
# Partition:2     msg-5
# Partition:2     msg-6
# Partition:2     msg-17
# Partition:2     msg-18
# Partition:2     msg-19
# Partition:2     msg-20

# note overall ordering is not maintained,
# only the ordering per partition is maintained

# [kafka:2] create a consumer with group 'group-B'
kafka-console-consumer.sh --topic group-test-topic \
  --bootstrap-server localhost:9092 \
  --group group-B \
  --property print.partition=true \
  --from-beginning

# [kafka:2] output is same as group-A

# thus each consumer group gets all the messages
# if there are multiple consumers in a consumer group, they are assigned some
# partitions and they recieve msgs from those partitions only

# TODO: try the same thing with `keys`

# [kafka:1, kafka:2, kafka:3] stop all processes and exit

# [host:1] tear down the cluster
docker compose down
```
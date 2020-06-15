---
title: kafka消费者
date: 2019-06-29 15:51:03
tags: kakfa
categories: kafka
---

# kafka消费者

## kafka消费者

一个正常的消费逻辑需要具备以下几个步骤：

1. 配置消费者客户端参数及创建相应的消费者实例。
2. 订阅主题。
3. 拉取消息并消费。
4. 提交消费位移。
5. 关闭消费者实例。

## kafka消费者参数配置介绍

config-key | config-explain 
 - | -
bootstrap.servers | 用来指定连接 Kafka 集群所需的 broker 地址清单，具体内容形式为 host1:port1,host2:post，可以设置一个或多个地址，中间用逗号隔开，此参数的默认值为" "。
group.id | 一般而言，这个参数需要设置成具有一定的业务意义的名称。
key.deserializer & value.deserializer | 与生产者客户端 KafkaProducer 中的 key.serializer和value.serializer 参数对应。消费者从 broker 端获取的消息格式都是字节数组（byte[]）类型，所以需要执行相应的反序列化操作才能还原成原有的对象格式。这两个参数分别用来指定消息中 key 和 value 所需反序列化操作的反序列化器，这两个参数无默认值
client.id | 这个参数用来设定 KafkaConsumer 对应的客户端id，默认值也为“”。如果客户端不设置，则 KafkaConsumer 会自动生成一个非空字符串，内容形式如“consumer-1”、“consumer-2”，即字符串“consumer-”与数字的拼接。
## kafka消费者方法介绍

### Kafka订阅主题方法

#### 1. subscribe()：通过主题订阅

一个消费者可以订阅一个或多个主题，使用subscribe()方法来订阅主题，对于这个方法来说，既可以用集合的形势订阅多个主题，也可以用正则表达式的形势订阅特定模式的主题。

	public void subscribe(Collection<String> topics, 
    ConsumerRebalanceListener listener)
	public void subscribe(Collection<String> topics)
	public void subscribe(Pattern pattern, ConsumerRebalanceListener listener)
	public void subscribe(Pattern pattern)

使用正则方式订阅主题。

	// 根据主题名进行正则匹配
	consumer.subscribe(Pattern.compile("topic-*"));

#### 2. assign()：通过 主题 + 分区 的方式订阅

消费者不仅可以通过 KafkaConsumer.subscribe() 方法订阅主题，还可以直接订阅某些主题的特定分区，在 KafkaConsumer中还提供了一个assign()方法来实现。

	public void assign(Collection<TopicPartition> partitions)

这个方法只接受一个参数 parttions,用来指定所需要订阅的分区集合。

我们简单的看下 TopicPartition 类

	// TopicPartition 类中有两个属性
	// 分区编号
	private final int partition;
	// 主题
    private final String topic;

订阅一个分区编号为0，主题为 topic-demo。

	consumer.assign(Arrays.asList(new TopicPartition("topic-demo"), 0))

## 获取信息

### partitionsFor()：查询指定主题的元数据信息

	public List<PartitionInfo> partitionsFor(String topic)

PartitionInfo 的结构如下

	public class PartitionInfo {
		// 主题名称
	    private final String topic;
		// 分区编号
	    private final int partition;
		// leader 副本所在位置
	    private final Node leader;
		// AR 集合
	    private final Node[] replicas;
		// ISR 集合
	    private final Node[] inSyncReplicas;
		// OSR 集合
	    private final Node[] offlineReplicas;
	}

我们用 assign() 方法来订阅主题(全部分区)的功能。

	List<TopicPartition> partitions = new ArrayList<>();
	List<PartitionInfo> partitionInfos = consumer.partitionsFor(topic);
	if (partitionInfos != null) {
		for (PartitionInfo partitionInfo : partitionInfos) {
			partitions.add(new TopicPartition(partitionInfo.topic, partitionInfo.partition()));		
		}
	}
	consumer.assign(partitions);

## 取消订阅

### 1. unsubscribe()
### 2. subscribe(new ArrayList<String>())
### 3. assign(new ArrayList<TopicPartition>())

## 拉模式获取消息

### poll()

	public ConsumerRecords<K, V> poll(final Duration timeout)
	@Deprecated
	public ConsumerRecords<K, V> poll(final long timeout)

方法中的 timeout 为阻塞时间

ConsumerRecord 类内部结构为

	public class ConsumerRecord<K, V> {
		// 主题
	    private final String topic;
		// 分区编号
	    private final int partition;
		// 偏移量
	    private final long offset;
		// 时间戳
	    private final long timestamp;
		// 时间戳类型
	    private final TimestampType timestampType;
		// key 序列化后的大小
	    private final int serializedKeySize;
		// value 序列化后的大小
	    private final int serializedValueSize;
		// 头信息
	    private final Headers headers;
		// 消息的 key
	    private final K key;
		// 消息的 value
	    private final V value;
		// CRC32 的校验值 (不懂)
	    private volatile Long checksum;
	}

我们在消费消息的时候可以直接对 ConsumerRecord 中感兴趣的字段进行具体的业务逻辑处理。

## records：按照分区维度来消费

	public List<ConsumerRecord<K, V>> records(TopicPartition partition)


## 位移提交

位移提交中三个重要的概念：

1. position：下一次拉取的消息的位置。
2. committed_offset：已经提交过的消费位移。
3. last_consumed_offset：当前消费到的位置。

### position()：获取 position 的值

	public long position(TopicPartition partition)

### committed()：获取 committed_offset 的值

	public OffsetAndMetadata committed(TopicPartition partition)

## 手动提交

开启手动提交功能的前提是消费者客户端参数 `enable.auto.commit` 配置为 `false`

	props.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, false);

手动提交可以细分为 同步提交 和 异步提交。

### commitSync()：同步提交

	public void commitSync()
	public void commitSync(final Map<TopicPartition, OffsetAndMetadata> offsets)


### commitAsync()：异步提交

	public void commitAsync()
	public void commitAsync(OffsetCommitCallback callback)
	public void commitAsync(final Map<TopicPartition, OffsetAndMetadata> offsets, OffsetCommitCallback callback)

## 控制或关闭消费

KafkaConsumer 提供了对消费速度进行控制的方法，在有些应用场景下我们可能需要暂停某些分区的消费而先消费其他分区，当达到一定条件时再恢复这些分区的消费。KafkaConsumer 中使用 pause() 和 resume() 方法来分别实现暂停某些分区在拉取操作时返回数据给客户端和恢复某些分区向客户端返回数据的操作。

	public void pause(Collection<TopicPartition> partitions)
	public void resume(Collection<TopicPartition> partitions)

KafkaConsumer 还提供了一个无参的 paused() 方法来返回被暂停的分区集合，此方法的具体定义如下：

	public Set<TopicPartition> paused()

## 指定位移消费

这里有个重要的概念，当kafka消费者找不到所记录的消费位移时，就会根据消费者客户端参数 `auto.offset.reset` 的配置来决定从何处开始进行消费，默认为 `latest`，表示从分区末尾开始消费信息。这样就会存在消息丢失的问题。如果将该值设置为 `earliest`，那么就会从开始消费。

### seek()

	public void seek(TopicPartition partition, long offset)

TopicPartition：分区。
offset：从分区的哪个位置开始消费。

### assignment()：用来获取消费者所分配到的分区信息

	public Set<TopicPartition> assignment()

### endOffsets()：用来获取指定分区末尾的消息位置。

	public Map<TopicPartition, Long> endOffsets(Collection<TopicPartition> partitions)
	public Map<TopicPartition, Long> endOffsets(Collection<TopicPartition> partitions,
            Duration timeout)

### beginningOffsets()：用来获取指定分区起始的消息位置。

	public Map<TopicPartition, Long> beginningOffsets(Collection<TopicPartition> partitions)
	public Map<TopicPartition, Long> beginningOffsets(Collection<TopicPartition> partitions,
	            Duration timeout)

### seekToBeginning()

	public void seekToBeginning(Collection<TopicPartition> partitions)

### seekToEnd()

	public void seekToEnd(Collection<TopicPartition> partitions)


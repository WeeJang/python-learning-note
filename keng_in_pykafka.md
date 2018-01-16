# 记录pykafka在使用过程中的一些坑

## 消费者组：ConsumerGroup

关于这个，有个比较好的blog : http://www.cnblogs.com/huxi2b/p/6223228.html

这个还没看： http://www.infoq.com/cn/articles/kafka-analysis-part-4

查询消费情况：

```bash
./kafka-consumer-groups.sh --bootstrap-server 192.168.2.64:9092 --describe --group jwgroup
```

## 关于启动时offset	

```bash
pykafka-docs-io : http://pykafka.readthedocs.io/en/latest/usage.html?highlight=offset
```

## assuming "mygroup" has no committed offsets

### starts from the latest available offset

```python
consumer = topic.get_simple_consumer(
    consumer_group="mygroup",
    auto_offset_reset=OffsetType.LATEST
)
consumer.consume()
consumer.commit_offsets()
```

### starts from the last committed offset

```python
consumer_2 = topic.get_simple_consumer(
    consumer_group="mygroup"
)
```

### starts from the earliest available offset

```python
consumer_3 = topic.get_simple_consumer(
    consumer_group="mygroup",
    auto_offset_reset=OffsetType.EARLIEST,
    reset_offset_on_start=True
)
```

---
layout: post
title:  "kafka producer"
date:   2018-06-06 20:05 +0800
categories: 学无止境
tag: 周边
---

* content
{:toc}

最近在看kafka源码，使用了资料《kafka技术内幕》by 郑奇煌，稍微记下笔记，毕竟下笔去记的的时候会加深理解..


主要组件
KafkaProducer - 生产者客户端；
RecordAccumulator-缓存生产者客户端产生的消息；
Sender-读取RecordAccumulator的批量消息，通过NetworkClient发送给服务端。

#### 1.两种发送方式

KafkaProducer.send(xx) : 根据方法返回的Future，可以实现同步或异步发送消息
#### 2. send处理逻辑

```
private Future<RecordMetadata> doSend(ProducerRecord<K, V> record, Callback callback) {
    TopicPartition tp = null;
    try {
        // (1) 校验消息对应的topic是否可用
        long waitedOnMetadataMs = waitOnMetadata(record.topic(), this.maxBlockTimeMs);
        long remainingWaitMs = Math.max(0, this.maxBlockTimeMs - waitedOnMetadataMs);
        //(2) 序列化key,value
        byte[] serializedKey;
        serializedKey = keySerializer.serialize(record.topic(), record.key());
        byte[] serializedValue;
        serializedValue = valueSerializer.serialize(record.topic(), record.value());
        //(3) 选择partition
        int partition = partition(record, serializedKey, serializedValue, metadata.fetch());
        int serializedSize = Records.LOG_OVERHEAD + Record.recordSize(serializedKey, serializedValue);
        //(4) 验证序列化后的key，value长度是否满足发送条件
        ensureValidRecordSize(serializedSize);
        tp = new TopicPartition(record.topic(), partition);
        // ... 省略部分代码
        // (5) 追加消息到RecordAccumulator；如果收集器满了，通知Sender发送消息
        RecordAccumulator.RecordAppendResult result = accumulator.append(tp, timestamp, serializedKey, serializedValue, interceptCallback, remainingWaitMs);
        if (result.batchIsFull || result.newBatchCreated) {
            log.trace("Waking up the sender since topic {} partition {} is either full or getting a new batch", record.topic(), partition);
            this.sender.wakeup();
        }
        return result.future;
      
    }...
}
```
其中：

**第(1)步**：在获取topic的meta时，如果topic的信息未在客户端缓存有会发起获取topic meta的请求到服务端，**超时时间为max.block.ms,默认60s。**
如果往kafka一个未创建的topic发送消息，那么send将阻塞在更新meta的步骤，如果服务包含往不同topic发送消息时，服务的连接很可能被在发送这个不存在的topic消息集时占满，导致其他正常的消息无法发送。
```
private long waitOnMetadata(String topic, long maxWaitMs) throws InterruptedException {
    // add topic to metadata topic list if it is not there already.
    if (!this.metadata.containsTopic(topic))
        this.metadata.add(topic);

    if (metadata.fetch().partitionsForTopic(topic) != null)
        return 0;

    long begin = time.milliseconds();
    long remainingWaitMs = maxWaitMs;
    while (metadata.fetch().partitionsForTopic(topic) == null) {
        int version = metadata.requestUpdate();
        sender.wakeup();
        //发起获取topic meta的请求到服务端，等待maxWatiMs时间
        metadata.awaitUpdate(version, remainingWaitMs);
        long elapsed = time.milliseconds() - begin;
        if (elapsed >= maxWaitMs)
            throw new TimeoutException("Failed to update metadata after " + maxWaitMs + " ms.");
        if (metadata.fetch().unauthorizedTopics().contains(topic))
            throw new TopicAuthorizationException(topic);
        remainingWaitMs = maxWaitMs - elapsed;
    }
    return time.milliseconds() - begin;
}


public synchronized void awaitUpdate(final int lastVersion, final long maxWaitMs) throws InterruptedException {
    if (maxWaitMs < 0) {
        throw new IllegalArgumentException("Max time to wait for metadata updates should not be < 0 milli seconds");
    }
    long begin = System.currentTimeMillis();
    long remainingWaitMs = maxWaitMs;
    //等待更新的meta版本大于我们已知的版本
    while (this.version <= lastVersion) {
        if (remainingWaitMs != 0)
            wait(remainingWaitMs);
        long elapsed = System.currentTimeMillis() - begin;
        if (elapsed >= maxWaitMs)
            throw new TimeoutException("Failed to update metadata after " + maxWaitMs + " ms.");
        remainingWaitMs = maxWaitMs - elapsed;
    }
}
```
**第(2)步**中，分配partition是因为能够将消息按照partition均衡负载，分区也作为并行任务的最小单位。
如果消息中没有key，将会使用round-robin算法得到分区；如果包含key，则对key散列化后，再与分区数目取模得到分区。
**第(3)步**： 验证序列化后的key，value长度是否满足发送条件
```
private void ensureValidRecordSize(int size) {
// max.request.size配置的单个请求最大长度
    if (size > this.maxRequestSize)
        throw new RecordTooLargeException("The message is " + size +
                                          " bytes when serialized which is larger than the maximum request size you have configured with the " +
                                          ProducerConfig.MAX_REQUEST_SIZE_CONFIG +
                                          " configuration.");
//buffer.memory配置的最大消息缓存大小
    if (size > this.totalMemorySize)
        throw new RecordTooLargeException("The message is " + size +
                                          " bytes when serialized which is larger than the total memory buffer you have configured with the " +
                                          ProducerConfig.BUFFER_MEMORY_CONFIG +
                                          " configuration.");
}
```

#### 3.RecordAccumulator - 记录收集器
生产者发送的消息都会被缓存在RecordAccumulator中，通过append方法追加，返回一个结果对象用于判断RecordBatch是否满了，如果满了就开始发送这一批数据。
记录收集器按TopicPartition归类存放消息，消息会按批记录集放在一个双端队列中，每次追加时首先取出队列中最后一个批记录集，往记录集中添加消息。如果队列中不存在批记录，或者上一个批记录满了，将会创建一个新的批记录集。满的记录集将等待发送。

![whats]({{ '/styles/images/post/18060601.png' | prepend: site.baseurl  }})

```
Deque<RecordBatch> dq = getOrCreateDeque(tp);
synchronized (dq) {
    if (closed)
        throw new IllegalStateException("Cannot send after the producer is closed.");
    RecordBatch last = dq.peekLast();
    if (last != null) {
        FutureRecordMetadata future = last.tryAppend(timestamp, key, value, callback, time.milliseconds());
        if (future != null)
            return new RecordAppendResult(future, dq.size() > 1 || last.records.isFull(), false);
    }
}

// we don't have an in-progress record batch try to allocate a new batch
int size = Math.max(this.batchSize, Records.LOG_OVERHEAD + Record.recordSize(key, value));
ByteBuffer buffer = free.allocate(size, maxTimeToBlock);
synchronized (dq) {
    // Need to check if producer is closed again after grabbing the dequeue lock.
    if (closed)
        throw new IllegalStateException("Cannot send after the producer is closed.");
    RecordBatch last = dq.peekLast();
    //取出队列中最后一个批记录集，往记录集中添加消息。如果队列中不存在批记录，或者上一个批记录满了，将会创建一个新的批记录集
    if (last != null) {
        
        FutureRecordMetadata future = last.tryAppend(timestamp, key, value, callback, time.milliseconds());
        //如果batch满了，无法插入新记录，返回的future为null，需要重新创建一个batch
        if (future != null) {
            // Somebody else found us a batch, return the one we waited for! Hopefully this doesn't happen often...
            free.deallocate(buffer);
            return new RecordAppendResult(future, dq.size() > 1 || last.records.isFull(), false);
        }
    }
    MemoryRecords records = MemoryRecords.emptyRecords(buffer, compression, this.batchSize);
    RecordBatch batch = new RecordBatch(tp, records, time.milliseconds());
    FutureRecordMetadata future = Utils.notNull(batch.tryAppend(timestamp, key, value, callback, time.milliseconds()));

    dq.addLast(batch);
    incomplete.add(batch);
    return new RecordAppendResult(future, dq.size() > 1 || batch.records.isFull(), true);
```


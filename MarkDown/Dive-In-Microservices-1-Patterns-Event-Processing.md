**Event processing** is a model which allows you to decouple your microservices by using a message queue. Rather than connect directly to a service which may or may not be at a known location, you broadcast and listen to events which exist on a queue, such as [Redis](https://redis.io/), [Amazon SQS](https://aws.amazon.com/sqs/), [RabbitMQ](https://www.rabbitmq.com/), [Apache Kafka](https://kafka.apache.org/), and a whole host of other sources.

The _message queue_ is a highly distributed and scalable system, and it should be capable of processing millions of messages so we do not need to worry about it not being available. At the other end of the queue, there will be a worker who is listening for new messages pertaining to it. When it receives such a message, it processes the message and then removes it from the queue.

Due to the async nature of the event processing pattern there needs to be a requirement to handle failures in a programmable way,

**Event processing with at least once delivery**

One of first and basic sync mechanism is to request for delivery, we add the message to the queue and then wait for an ACK from the queue to let us know that the message has been received. Of course, we would not know if the message has been delivered but receiving the ACK should be enough for us to notify the user and proceed. There is always the possibility that the receiving service cannot process the message which could be due to a direct failure or bug in the receiving service or it could be that the message which was added to the queue is not in a format which can be read by the receiving service. We need to deal with both of these issues independently, with handling errors discussed next.

**Handling Errors**

It is not uncommon for things to go wrong with distributed systems and is the essential factor in microservice based software design. As per above scenario, if a valid message can not be processed one standard approach is to retry processing the message, normally with a delay. It is important to append the error every time we fail to process a message as it gives us the history of what went wrong, it also provides us with the capability to understand how many times we have tried to process the message because after we exceed this threshold we do not want to continue to retry we need to move this message to a second queue where we can use it for diagnostic information.

**Debugging the failures with Dead Letter Queue**

It is most common practice to remove the message from queue once it is processed. The purpose of the dead letter queue is so that we can examine the failed messages on this queue to assist us with debugging the system. Since we can append the error details to the message body, we know what the error is and we know where the history lies if we need it.

**Working with idempotent transactions**

While many message queues nowadays offer _At Most Once Delivery_ in addition to the _At Least Once_, the latter option is still the best for large throughput of messages. To deal with the fact that the receiving service may receive a message twice it needs to be able to handle this in its own logic. One of the common methods for ensuring that the message is not processed twice is to log the message ID in a transactions table. If the message has already been processed and if it will be disposed.

**Working with the ordering of messages**

One of the common issue while handling failures with retry is receiving a message out of sequence or  in an incorrect order, which will end up with inconsistent data in the database. One potential way to avoid this issue is to again leverage the transaction table and to store the message dispatch\_date in addition to the id. When the receiving service receives a message then it can not only check if the current message has been processed it can check that it is the most recent message and if not discard it.

**Working with**  **atomic**  **transactions**

This is the common issue found when moving the legacy systems to microservices. While storing data, a database can be _atomic_: that is, all operations occur or none do. Distributed transactions do not give us the same kind of transaction that is found in a database. When part of a database transaction fails, we can roll back the other parts of the transaction. By using this pattern we would only remove the message from the queue if the process succeeded so when something fails, we keep retrying. This gives us a kind of eventually consistent transaction.

Unfortunately, there is no one solution fits all with messaging we need to tailor the solution which matches the operating conditions of the service.

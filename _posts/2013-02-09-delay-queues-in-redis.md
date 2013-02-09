---
layout: post
title: "Delay Queues in Redis (with Grails example)"
description: ""
category: 
tags: []
---
{% include JB/setup %}

I needed a way to handle delayed processing of messages in a distributed system. Since I already had Redis running (but no message queue other than Kafka), I used a sorted set as a simple delay queue.

The basic approach is to insert each message into the sorted set with a score equal to the \[unix\] time the message should become available for processing ("ready").

    redis> ZADD delayqueue <future_timestamp> "messsage"
    
Get all "ready" messages with a range query from (time) zero to now, then delete the messages. To avoid multiple processing and lost messages, run this in a transaction:

    redis> MULTI
    redis> ZRANGEBYSCORE delayqueue 0 <current_timestamp>
    redis> ZREMRANGEBYSCORE delayqueue 0 <current_timestamp>
    redis> EXEC
  
Note that this implementation requires messages to be unique per queue.

Here's a quick implementation as a Grails service:

{% highlight groovy %}
import redis.clients.jedis.Jedis

/**
 * Handles the delaying of queued messages for later retrieval.
 *
 * Messages are unique per queue, and are sorted by the time they [will] have become available.
 */
class DelayQueueService {
    def redisService
    
    /**
     * Queue a message for later retrieval. Messages are unique per queue and are deleted upon retrieval. If a given
     * message already exists, it is updated with the new delay.
     *
     * @param queue Queue name
     * @param message
     * @param delay Time in seconds the message should be delayed
     */
    def queueMessage(String queue, String message, Integer delay) {
        Double time = System.currentTimeMillis()/1000 + delay

        redisService.withRedis { Jedis redis ->
            redis.zadd(queue, time, message)
        }
    }

    /**
     * Retrieve messages that are no longer delayed. Deletes messages on read.
     *
     * @param queue Queue name
     */
    def getMessages(String queue) {
        Double startTime = 0
        Double endTime = System.currentTimeMillis() / 1000

        return redisService.withRedis { Jedis redis ->
            def t = redis.multi()
            def response = t.zrangeByScore(queue, startTime, endTime)
            t.zremrangeByScore(queue, startTime, endTime)
            t.exec()
            return response.get()
        }
    }
}
{% endhighlight %}
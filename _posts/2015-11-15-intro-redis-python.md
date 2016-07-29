---
title: An introduction to Redis data types with Python
date: 2015-11-15 00:00:00 Z
categories:
- development
- architecture
tags:
- python
layout: post
author: Jose A. Dianes
comments: true
---

In this post, we will go thourhg a similar set of commands as those described in the [Redis Data Types introduction](http://redis.io/topics/data-types-intro) but using the [redis-py](https://github.com/andymccurdy/redis-py) Python client from a [Jupyter notebook](https://jupyter.org).  

Remember that [Redis](http://redis.io) is a server, and it can be access in a distributed way by multiple clients in an [Enterprise System](https://en.wikipedia.org/wiki/Enterprise_software). This notebook acts as a single client, and is just for educative purposes. The full power of Redis comes when used in an enterprise architecture!  

While working with Redis with Python, you will notice that many operations on Redis data types are also available for the Python data types that we get as a result of some operations (e.g. lists). However we have to keep in mind that they operate at very different levels. Using the Redis server operations, and not the local Python equivalents, is the way to go for enterprise applications in order to keep our system **scalability** and **availability** (i.e. large sets, concurrent access, etc).

## Starting up

Interacting with a running [Redis](http://redis.io/topics/quickstart) server from a Jupyter notebook using Python is as easy as installing [redis-py](https://github.com/andymccurdy/redis-py) for your Python distribution and then import the module as follows.


{% highlight python %}
import redis
{% endhighlight %}

Then we can obtain a reference to our server. Asuming that we are running our Redis server and our Jupyter notebook server in the same host, with the default Redis server port, we can do as follows.


{% highlight python %}
r = redis.StrictRedis(host='localhost', port=6379, db=0)
{% endhighlight %}

## Getting and settings Redis [*Keys*](http://redis.io/topics/data-types-intro#redis-keys)

Now we can use `r` to send Redis commands. For example, we can [SET](http://redis.io/commands/set) the value of the **key** `my.key` as follows.  


{% highlight python %}
r.set('my.key', 'value1')
{% endhighlight %}




    True



In order to check our recently set key, we can use [GET](http://redis.io/commands/get) and pass the name of the key.  


{% highlight python %}
r.get('my.key')
{% endhighlight %}




    'value1'



We can also check for the existence of a given key.


{% highlight python %}
r.exists('my.key')
{% endhighlight %}




    True




{% highlight python %}
r.exists('some.other.key')
{% endhighlight %}




    False



If we want to set multiple keys at once, we can use [MSET](http://redis.io/commands/mset) and pass a Python dictionary as follows.


{% highlight python %}
r.mset({'my.key':'value2', 'some.other.key':123})
{% endhighlight %}




    True




{% highlight python %}
r.get('my.key')
{% endhighlight %}




    'value2'




{% highlight python %}
r.get('some.other.key')
{% endhighlight %}




    '123'



We can also increment the value of a given key in an atomic way.


{% highlight python %}
r.incrby('some.other.key',10)
{% endhighlight %}




    133



Notice how the resulting type has been changed to integer!

### Setting keys to expire

With **redis-py** we can also set keys with limited time to live.


{% highlight python %}
r.expire('some.other.key',1)
r.exists('some.other.key')
{% endhighlight %}




    True



Let's wait for a couple of seconds for the key to expire and check again.


{% highlight python %}
from time import sleep
sleep(2)
r.exists('some.other.key')
{% endhighlight %}




    False



Finally, **del** is a reserved keyword in the Python syntax. Therefore redis-py uses 'delete' instead.


{% highlight python %}
r.delete('my.key')
{% endhighlight %}




    1



## Redis [*Lists*](http://redis.io/topics/data-types-intro#redis-lists)

Redis lists are linked lists of keys. We can insert and remove elements from both ends.

The [LPUSH](http://redis.io/commands/lpush) command adds a new element into a list, on the left.


{% highlight python %}
r.lpush('my.list', 'elem1')
{% endhighlight %}




    1L



The [RPUSH](http://redis.io/commands/rpush) command adds a new element into a list, on the right. 


{% highlight python %}
r.rpush('my.list', 'elem2')
{% endhighlight %}




    2L



Finally the [LRANGE](http://redis.io/commands/lrange) command extracts ranges of elements from lists.  


{% highlight python %}
r.lrange('my.list',0,-1)
{% endhighlight %}




    ['elem1', 'elem2']




{% highlight python %}
r.lpush('my.list', 'elem0')
{% endhighlight %}




    3L




{% highlight python %}
r.lrange('my.list',0,-1)
{% endhighlight %}




    ['elem0', 'elem1', 'elem2']



The result is returned as a Python list. We can use [LLEN](http://redis.io/commands/llen) to check a Redis list lenght without requiring to store the result of `lrange` and then use Python's `len`.


{% highlight python %}
r.llen('my.list')
{% endhighlight %}




    3



We can push multiple elements with a single call to push.


{% highlight python %}
r.rpush('my.list','elem3','elem4')
{% endhighlight %}




    5L




{% highlight python %}
r.lrange('my.list',0,-1)
{% endhighlight %}




    ['elem0', 'elem1', 'elem2', 'elem3', 'elem4']



Finally, we have the equivalent pop operations for both, right and left ends.


{% highlight python %}
r.lpop('my.list')
{% endhighlight %}




    'elem0'




{% highlight python %}
r.lrange('my.list',0,-1)
{% endhighlight %}




    ['elem1', 'elem2', 'elem3', 'elem4']




{% highlight python %}
r.rpop('my.list')
{% endhighlight %}




    'elem4'




{% highlight python %}
r.lrange('my.list',0,-1)
{% endhighlight %}




    ['elem1', 'elem2', 'elem3']



### Capped Lists

We can also [TRIM](http://redis.io/commands/ltrim) Redis lists with redis-py. We need to pass three arguments: the name of the list, and the start and stop indexes.


{% highlight python %}
r.lpush('my.list','elem0')
r.ltrim('my.list',0,2)
{% endhighlight %}




    True




{% highlight python %}
r.lrange('my.list',0,-1)
{% endhighlight %}




    ['elem0', 'elem1', 'elem2']



Notice as the last element has been dropped when triming the list. The `lpush`/`ltrim` sequence is a common pattern when inserting in a list that we want to keep size-fized.


{% highlight python %}
r.delete('my.list')
{% endhighlight %}




    1



## Redis [*Hashes*](http://redis.io/topics/data-types-intro#redis-hashes)  

The equivalent of Python dictionaries are Redis *hashes*, with field-value pairs. We use the command [HMSET](http://redis.io/commands/hmset).


{% highlight python %}
r.hmset('my.hash', {'field1':'value1',
                   'field2': 1234})
{% endhighlight %}




    True



We can also set individual fields.


{% highlight python %}
r.hset('my.hash','field3',True)
{% endhighlight %}




    0L



We have methods to get individual and multiple fields from a hash.


{% highlight python %}
r.hget('my.hash','field2')
{% endhighlight %}




    '1234'




{% highlight python %}
r.hmget('my.hash','field1','field2','field3')
{% endhighlight %}




    ['value1', '1234', 'True']



The result is returned as a list of values.

Increment operations are also available for hash fields.


{% highlight python %}
r.hincrby('my.hash','field2',10)
{% endhighlight %}




    1244L



## Redis [Sets](http://redis.io/topics/data-types-intro#redis-sets)

Redis Sets are unordered collections of strings. We can easily add multiple elements to a Redis set in redis-py as follows by using its implementation of [SADD](http://redis.io/commands/sadd).


{% highlight python %}
r.sadd('my.set', 1, 2, 3)
{% endhighlight %}




    3



As a result, we get the size of the set. If we want to check the elements within a set, we can use [SMEMBERS]().


{% highlight python %}
r.smembers('my.set')
{% endhighlight %}




    {'1', '2', '3'}




{% highlight python %}
type(r.smembers('my.set'))
{% endhighlight %}




    set



Notice that we get a Python *set* as a result. That opens the door to all sort of Python set operations. However, we can operate directly within the Redis server space, and still do things as checking an element membership using [SISMEMBER](http://redis.io/commands/sismember). This is the way to go for enterprise applications in order to keep our system **scalability** and **availability** (i.e. large sets, concurrent access, etc).


{% highlight python %}
r.sismember('my.set', 4)
{% endhighlight %}




    False




{% highlight python %}
r.sismember('my.set', 1)
{% endhighlight %}




    True



The [SPOP](http://redis.io/commands/spop) command extracts a random element (and we can use [SRANDMEMBER](http://redis.io/commands/srandmember) to get one or more random elements without extraction).


{% highlight python %}
elem = r.spop('my.set')
{% endhighlight %}


{% highlight python %}
r.smembers('my.set')
{% endhighlight %}




    {'1', '2'}




{% highlight python %}
r.sadd('my.set',elem)
{% endhighlight %}




    1




{% highlight python %}
r.smembers('my.set')
{% endhighlight %}




    {'1', '2', '3'}



Or if we want to be specific, we can just use [SREM](http://redis.io/commands/srem).


{% highlight python %}
r.srem('my.set',2)
{% endhighlight %}




    1




{% highlight python %}
r.smembers('my.set')
{% endhighlight %}




    {'1', '3'}



### Set operations

In order to obtain the intersection between two sets, we can use [SINTER](http://redis.io/commands/sinter).


{% highlight python %}
r.sadd('my.other.set', 'A','B',1)
{% endhighlight %}




    3




{% highlight python %}
r.smembers('my.other.set')
{% endhighlight %}




    {'1', 'A', 'B'}




{% highlight python %}
r.sinter('my.set','my.other.set')
{% endhighlight %}




    {'1'}



That we get as a Python set. Alternatively, we can directly store the result as a new Redis set by using [SINTERSTORE](http://redis.io/commands/sinterstore).


{% highlight python %}
r.sinterstore('my.intersection','my.set','my.other.set')
{% endhighlight %}




    1




{% highlight python %}
r.smembers('my.intersection')
{% endhighlight %}




    {'1'}



Similar operations are available for union and difference. Moreover, they can be applied to more than two sets. For example, let's create a union set with all the previous and store it in a new Redis set.


{% highlight python %}
r.sadd('my.intersection','batman')
r.sunionstore('my.union','my.set','my.other.set','my.intersection')
{% endhighlight %}




    5




{% highlight python %}
r.smembers('my.union')
{% endhighlight %}




    {'1', '3', 'A', 'B', 'batman'}



Finally, the number of elements of a given Redis set can be obtained with [SCARD](http://redis.io/commands/scard).


{% highlight python %}
r.scard('my.union')
{% endhighlight %}




    5



Let's clean our server before leaving this section.


{% highlight python %}
r.delete('my.set','my.other.set','my.intersection','my.union')
{% endhighlight %}




    4



## Redis [sorted Sets](http://redis.io/topics/data-types-intro#redis-sorted-sets)

In a Redis sorted set, every element is associated with a floating point value, called the score. Elements within the set are then ordered according to these scores. We add values to a sorted set by using the oepration [ZADD](http://redis.io/commands/zadd). 


{% highlight python %}
r.zadd('my.sorted.set', 1, 'first')
r.zadd('my.sorted.set', 3, 'third')
r.zadd('my.sorted.set', 2, 'second')
r.zadd('my.sorted.set', 4, 'fourth')
r.zadd('my.sorted.set', 6, 'sixth')
{% endhighlight %}




    1



Sorted sets' scores can be updated at any time. Just calling ZADD against an element already included in the sorted set will update its score (and position).

It doesn't matter the order in which we insert the elements. When retrieving them using [ZRANGE](http://redis.io/commands/zrange), they will be returned as a Python list ordered by score.


{% highlight python %}
r.zrange('my.sorted.set',0,-1)
{% endhighlight %}




    ['first', 'second', 'third', 'fourth', 'sixth']



And if we want also them in reverse order, we can call [ZREVRANGE](http://redis.io/commands/zrevrange). 


{% highlight python %}
r.zrevrange('my.sorted.set',0,-1)
{% endhighlight %}




    ['sixth', 'fourth', 'third', 'second', 'first']



Even more, we can slice the range by score by using [ZRANGEBYSCORE](http://redis.io/commands/zrangebyscore).


{% highlight python %}
r.zrangebyscore('my.sorted.set',2,4)
{% endhighlight %}




    ['second', 'third', 'fourth']



A similar schema can be used to remove elements from the sorted set by score using [ZREMRANGEBYSCORE](http://redis.io/commands/zremrangebyscore).


{% highlight python %}
r.zremrangebyscore('my.sorted.set',6,'inf')
r.zrange('my.sorted.set',0,-1)
{% endhighlight %}




    ['first', 'second', 'third', 'fourth']



It is also possible to ask what is the position of an element in the set of the ordered elements by using [ZRANK](http://redis.io/commands/zrank) (or [ZREVRANK](http://redis.io/commands/zrevrank) if we want the order in reverse way).


{% highlight python %}
r.zrank('my.sorted.set','third')
{% endhighlight %}




    2



Remember that ranks and scores have the same order but different values! If what we want is the score, we can use [ZSCORE](http://redis.io/commands/zscore). 


{% highlight python %}
r.zscore('my.sorted.set','third')
{% endhighlight %}




    3.0



Finally, there also a series of operations that operate on a sorted set in a lexicographical basis. They work when all the elements in the ser are inserted with the same value. For example, we can list the elements in the set sliced by its inital with [ZRANGEBYLEX](http://redis.io/commands/zrangebylex).


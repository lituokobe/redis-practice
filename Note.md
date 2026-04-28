# Redis note
## Deploy redis with docker
```bash
docker run -d \
  --name redis-stack-new \
  -p 6380:6379 \
  -p 8001:8001 \
  -e REDIS_ARGS='--requirepass StrongPassword!' \
  redis/redis-stack:latest
```
- Exposes two ports:
  - 6379: Redis database (with modules like RediSearch, RedisJSON, etc.)
  - 8001: Redis Insight — the built-in web-based GUI for monitoring and data exploration

## Use Redis-CLI
```bash
docker exec -it redis-stack redis-cli
```
Redis is a key-value based database, operate with "GET", "SET", "DEL", "EXISTS", etc.
```redis-cli
> SET name bbn
"OK"

> GET name
"bbn"

> set name Bbn
"OK"

> get name
"Bbn"

> set age 450
"OK"

> get age
"450"

> DEL age
(integer) 1

> GET age
(nil)

> EXISTS age
(integer) 0

> EXISTS name
(integer) 1
```

```redis-cli
> KEYS *
1) "name"

> FLUSHALL # delete all
"OK"

> KEYS *
(empty list or set)

> EXPIRE name 12 # set expiration time for the key
(integer) 1

> TTL name # check time to live (2 seconds passed)
(integer) 10

> SETEX age 12 40 # directly create a value with 12 seconds of expiration
"OK"

> ttl age
(integer) 10
```

```redis-cli
> LPUSH letter 1 # add element to a list head
(integer) 1

> LRANGE letter 0 -1 # get all the element of a list
1) "1"

> LPUSH letter bbn
(integer) 2

> LRANGE letter 0 -1
1) "bbn"
2) "1"

> LPUSH letter 2 3 5
(integer) 5

> LRANGE letter 2 4
1) "2"
2) "bbn"
3) "1"

> LRANGE letter 0 -1
1) "5"
2) "3"
3) "2"
4) "bbn"
5) "1"

> RPUSH letter 4 # add an element to the list tail
(integer) 6

> LRANGE letter 0 -1
1) "5"
2) "3"
3) "2"
4) "bbn"
5) "1"
6) "4"

> LPOP letter # delete the head element of a list
"5"

> LRANGE letter 0 -1
1) "3"
2) "2"
3) "bbn"
4) "1"
5) "4"

> RPOP letter # delete the tail element of a list
"4"

> LRANGE letter 0 -1
1) "3"
2) "2"
3) "bbn"
4) "1"

> LTRIM letter 1 3 # only keep 1 - 3 element of the list
"OK"

> LRANGE letter 0 -1
1) "2"
2) "bbn"
3) "1"
```

```redis-cli
> SADD course 1 a 8.6 # add elments to a set, disorganized, unduplicated data structure
(integer) 3

> SMEMBERS course # check the set
1) "1"
2) "a"
3) "8.6"

> SISMEMBER course bbn # check if an element is in the set
(integer) 0

> SREM course 1  # Remove an element of the set
(integer) 1

> SMEMBERS course
1) "a"
2) "8.6"
```

```redis-cli
> ZADD result 680 BBN 650 DDF 664 Dany # create a sorted set where element has scores and ranks from small to big
(integer) 3

> ZRANGE result 0 -1 # check all the elements
1) "DDF"
2) "Dany"
3) "BBN"

> ZRANGE result 0 -1 withscores
1) "DDF"
2) "650"
3) "Dany"
4) "664"
5) "BBN"
6) "680"

> zscore result BBN #check score of an element
"680"

> zrank result DDF # check the ranking of an element
(integer) 0

> zrevrank result DDF # check the reverse rank of an element
(integer) 2

> zrem result DDF # remove an element
(integer) 1

> zrange result 0 -1
1) "Dany"
2) "BBN"
```

```redis-cli
> HSET person name BBN # create a hash set of key-value pair
(integer) 1

> HSET person age 100
(integer) 1

> HGET person name
"BBN"

> HGETALL person # get all the pairs in the set
1) "name"
2) "BBN"
3) "age"
4) "100"

> HDEL person age # delete a pair
(integer) 1

> HGETALL person
1) "name"
2) "BBN"

> HEXISTS person age # check if a pair exists
(integer) 0

> HKEYS person # get all the keys
1) "name"

> HLEN person # get the length of the hash set
(integer) 1
```

```redis-cli
> SUBSCRIBE bbn # scubscribe a channel
Use Pub/Sub tool to subscribe to channels.

> publish bbn good # publish something on to the channel, other cli that subsribe will see the published content
(integer) 0

> publish bbn nice
(integer) 1

> publish bbn beautiful
(integer) 1
```
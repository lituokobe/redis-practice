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
```redis-cli
> XADD BBN * course redis
"1777383534135-0"

> XADD BBN * course docker #add streem, * add the id for the item
"1777383556829-0"

> XADD BBN * course git
"1777383564863-0"

> XLEN BBN
(integer) 3

> XRANGE BBN - + # Check the full stream
1) 1) "1777383534135-0"
   2) 1) "course"
      2) "redis"
2) 1) "1777383556829-0"
   2) 1) "course"
      2) "docker"
3) 1) "1777383564863-0"
   2) 1) "course"
      2) "git"

> XREAD COUNT 2 BLOCK 1000 STREAMS BBN 0 # Read 2 items, blcok for 1000ms if no item, from 0th
1) 1) "BBN"
   2) 1) 1) "1777383534135-0"
         2) 1) "course"
            2) "redis"
      2) 1) "1777383556829-0"
         2) 1) "course"
            2) "docker"

> XGROUP CREATE BBN group1 0 #create consumer group from current stream
"OK"

> XINFO GROUPS BBN
1) 1) "name"
   2) "group1"
   3) "consumers"
   4) "0"
   5) "pending"
   6) "0"
   7) "last-delivered-id"
   8) "0-0"
   9) "entries-read"
   10) "null"
   11) "lag"
   12) "null"

> XGROUP CREATECONSUMER BBN group1 consumer1
(integer) 1

> XGROUP CREATECONSUMER BBN group1 consumer2
(integer) 1

> XINFO GROUPS BBN
1) 1) "name"
   2) "group1"
   3) "consumers"
   4) "2"
   5) "pending"
   6) "0"
   7) "last-delivered-id"
   8) "0-0"
   9) "entries-read"
   10) "null"
   11) "lag"
   12) "null"
```
```redis-cli
> GEOADD city 115.543 49.2324 Shijiazhuang # create a location with lattitude and altitude
(integer) 1

> GEOADD city 35.5423 42.2324 Qinhuangdao
(integer) 1

> GEOPOS city Shijiazhuang # show all the locations
1) 1) "115.54300099611282349"
   2) "49.23239982415405791"

> GEODIST city Shijiazhuang Qinhuangdao # get the distance (by m by default)
"5964997.5180"

> GEODIST city Shijiazhuang Qinhuangdao KM # get the distance by km
"5964.9975"

> GEOSEARCH city FROMMEMBER Qinhuangdao BYRADIUS 6000 KM # search by radius
1) "Qinhuangdao"
2) "Shijiazhuang"
```
```redis-cli
> PFADD course git docker redis # create a hyperlog, whose base number is the count of unique values
(integer) 1

> PFADD course
(integer) 0

> PFCOUNT course
(integer) 3

> PFADD course nginx
(integer) 1

> PFCOUNT course
(integer) 4

> PFADD course 2 python git go
(integer) 1

> PFMERGE result course course2 # merge to hyperlog
"OK"

> PFcount result
(integer) 7
```
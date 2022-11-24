# Introduction to Redis
오픈소스 프로젝트인 Redis 에 대해서 배워보자.

Redis는 in-memory 데이터베이스이다. cache, db, message broker, streaming engine으로 사용될 수 있다.  

## Datastructure

1. strings
2. hashes
3. lists
4. sets
5. sorted sets with range queries
6. bitmaps
7. hyperloglogs
8. geospatial indexed
9. streams

## built-in component
1. replication
2. Luascripting
3. LRU eviction
4. transactions
5. on-dist persistence
6. Redis Sentinal (non-clustered High Availability)
7. Redis Cluster


## Atomic Operations
1. appending to a string
2. incrementing the value in a hash
3. pushing an element to a list
4. computing set intersection
5. computing set union
6. computing set difference
7. getting the member with highest ranking in a sorted set

## etc
1. dumping the dataset to disk
2. appending each command to a disk-based log
3. asynchronous replication

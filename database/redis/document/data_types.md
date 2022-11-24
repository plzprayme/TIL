# Redis data types
Redis는 data structure server 이다.  
Redis는 native data type의 collection을 제공하여 caching, queuing, event processing 문제를 해결할 수 있도록 돕는다.  


## Strings
byte sequence로 표현되는 가장 기본적인 data type이다.  
캐싱에 자주 사용되지만 counter를 구현해서 비트 단위 작업도 수행할 수 있다.

### Basic Commands
* Get, Set
  * SET
  * SETNX (Key가 없을 때만 저장한다. lock을 구현할 때 유용하게 사용할 수 있다.) ref: https://stackoverflow.com/questions/56589374/single-instance-redis-lock-with-setnx
  * GET
  * MGET (Multiple GET) 
* Counter
  * INCRBY (음수와 함께 사용하면 DECR 가능. 원자성 보장)
  * INCRBYFLOAT (소수 카운터)
* Bitwise 
  * chckout [bitmaps data type docks](https://redis.io/docs/data-types/bitmaps/) 

whole command: https://redis.io/commands/?group=string

### Examples
* store and then retrieve a string
```shell
> SET user:1 salvatore
OK
> GET user:1
"salvatore"
```

* Store a serialized JSON string and set TTL 100
```shell
> SET ticket:27 "\"{'username': 'priya', 'ticket_id': 321}\"" EX 100
```

* Increment a counter
```shell
> INCR views:page:2
(integer) 1
> INCRBY views:page:2 10
(integer) 11
```

### Limits
512MB 까지 저장할 수 있다.

### Performance
O(1) 이지만 SUBSTR, GETRANGE, SETRANGE 명령어는 조심해서 사용하자. 이 커맨드들은 O(N)이다.

### Alternatives
serialized string을 저장할거라면, Redis hashes, RedisJSON 사용을 고려해보자



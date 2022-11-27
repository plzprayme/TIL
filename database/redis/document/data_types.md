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

## lists
Redis lists느 string value의 linked list이다.  
스택과 큐를 구현할 때 많이 사용하고, background worker system의 queue management에 많이 사용된다.

### Example
* FIFO queue
```shell
> LPUSH work:queue:ids 101
(integer) 1
> LPUSH work:queue:ids 237
(integer) 2
> RPOP work:queue:ids
"101"
> RPOP work:queue:ids
"237"
```
* FILO stack
```shell
> LPUSH work:queue:ids 101
(integer) 1
> LPUSH work:queue:ids 237
(integer) 2
> LPOP work:queue:ids
"237"
> LPOP work:queue:ids
"101"
```

* list 길이 확인
```shell
> LLEN work:queue:ids
(integer) 0
```

* 원자성을 보장하며 A 리스트에서 pop하고 B 리스트에 push 하기
```shell
> LPUSH board:todo:ids 101
(integer) 1
> LPUSH board:todo:ids 273
(integer) 2
> LMOVE board:todo:ids board:in-progress:ids LEFT LEFT
"273"
> LRANGE board:todo:ids 0 -1
1) "101"
> LRANGE board:in-progress:ids 0 -1
1) "273"
```

* LPUSH 호출 후 LTRIM을 호출하여 list length 제한하기
```shell
> LPUSH notifications:user:1 "You've got mail!"
(integer) 1
> LTRIM notifications:user:1 0 99
OK
> LPUSH notifications:user:1 "Your package will be delivered at 12:01 today."
(integer) 2
> LTRIM notifications:user:1 0 99
OK
```

### Limits
list의 길이는 2^32 - 1 을 초과할 수 없다.

### Basic commands
* LPUSH: head에 새로운 element를 추가한다.
* RPUSH: tail에 새로운 element를 추가한다.
* LPOP, RPOP: head/tail 의 원소를 제거하며 리턴한다.
* LLEN: list.length();
* LMOVE: 원자성을 보장하며 하나의 리스트의 원소를 다른 리스트로 전송
* LTRIM: list를 명시한 range만큼으로 줄여버린다.

#### Blocking commands
* BLPOP: LPOP과 같은 동작을 하는데, list가 비어 있으면 element가 push 될 때 까지 block 되거나 timeout 시간까지 block 된다.
* BLMOVE: 위와 같다.

### Performance
O(1) 인데, LINDEX, LINSERT, LSET 같은 커맨드들은 O(N)이다. 거대한 list를 다룰 때는 조심해서 사용해야 한다.

## Sets
set은 순서가 보장되지 않는 unique string 의 컬렉션이다. 
* IP 같은 unique item을 Tacking할 때 유용하게 사용 할 수 있다.
* 관계를 표현할 수 있다. (모든 유저의 권한 set)
* union, intersection 등과 같은 일반적인 set operation을 사용할 수 있다.

### Examples
* 123유저와 456 유저의 Favorite Book ID를 저장한다.
```shell
> SADD user:123:favorites 347
(integer) 1
> SADD user:123:favorites 561
(integer) 1
> SADD user:123:favorites 742
(integer) 1
> SADD user:456:favorites 561
(integer) 1
```
* 123 유저가 책 742, 299를 좋아하는지 체크
```shell
> SISMEMBER user:123:favorites 742
(integer) 1
> SISMEMBER user:123:favorites 299
(integer) 0
```

* 유저 123, 456이 공통으로 좋아하는 책 확인
```shell
> SINTER user:123:favorites user:456:favorites
1) "561"

* set size
```shell
> SCARD user:123:favorites
(integer) 3
```

### Limits
set의 member size는 2^32-1 로 제한된다ㅏ.

### Basic commands
* SADD: member 추가
* SREM: member 제거
* SISMEMBER: set에 member가 있는지 확인 (test membership)
* SINTER: 2개 이상의 set에 공통으로 존재하는 member인지 확인
* SCARD: set size
* SMEMBERS: get all
* SSCAN: iterate member (SSCAN key cursor [MATCH patter] [COUNT count]

### Performance
거의 대부분 O(1)이다. 거대한 set에 대해서 SMEMBERS 커맨드를 사용할 때는 주의해야 한다.  
SMEMBERS는 O(n)이고 모든 set을 single response로 리턴한다. 모든 member를 조회해야 할 때는 SSCAN을 사용하는 것을 권장한다. 

### Alternatives
아주 큰 Set을 저장하려면 메모리를 많이 사용해야 한다.  
만약 메모리 사용에 걱정이 있고 완벽하게 정밀한 데이터가 필요 없다면 [Bloom filter or Cuckoo filter](https://redis.io/docs/stack/bloom/)를 고려해보자

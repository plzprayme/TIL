# Redis Patterns

## Bulk Loading
Redis protocol과 data bulk write 해보자.  

### Bulk loading using the Redis protocol  
normal redis client로 bulk loading 하는 것은 좋은 생각이 아니다.  
1. 모든 명령어에 RTT 만큼 레이턴시를 가진다.  
2. 파이프라인을 통해 응답을 받는 동시에 새로운 명령어를 쓰면서 속도를 증가시킬 수 있다.  

아주 소수의 클라이언트만 논블록킹 I/O를 지원하며, 모든 클라이언트들이 최대 throughput 만큼의 응답을 처리할 수 있을정도로 효율적이지 않다.  
그러니까 Redis protocol을 사용하자.

다수의 명령어들이 있는 텍스트 파일을 netcat 을 이요해서 REDIS에게 먹여보자.

`(cat commands.txt; sleep 10) | nc localhost 6379 > /dev/null`

빠르긴 한데 신뢰할 수 없는 방법이다. 모든 명령어가 잘 적용되었는지 알 수 없고 에러 체크가 안되기 때문이다.  
2.6 이상에서는 redis-cli 라는 유틸리티 툴이 지원되며 pipe mode 를 통해 bulk loading을 효율적으로 할 수 있다.  

```shell
$ cat data.txt | redis-cli --pipe

---
---

All data transferred. Wating for the last reply...
Last reply received from server.
errors: 0, replies: 1000000
```

에러 메세지 구경하기, 커맨드, 파이프라인, netcat 퍼포먼스 비교하기 `#TODO` 

### Generating Redis Protocol
Redis protocol은 아주 쉽게 생성하고 파싱할 수 있다. 

```shell
*<args><cr><lf>
$<len><cr><lf>
<arg0><cr><lf>
<arg1><cr><lf>
...
<argN><cr><lf>
```

<cr> == "\r" (ASCII 13)
<lf> == "\n" (ASCII 10)

SET key value 커맨드를 만드는 예제를 보자

```shell
*3<cr><lf>
$3<cr><lf>
SET<cr><lf>
$3<cr><lf>
key<cr><lf>
$5<cr><lf>
value<cr><lf>

or

"*3\r\n$3\r\nSET\r\n$3\r\nkey\r\n$5\r\nvalue\r\n"
```

ruby 프로그램

```ruby
def gen_redis_proto(*cmd)
    proto = ""
    proto << "*"+cmd.length.to_s+"\r\n"
    cmd.each{|arg|
        proto << "$"+arg.to_s.bytesize.to_s+"\r\n"
        proto << arg.to_s+"\r\n"
    }
    proto
end

puts gen_redis_proto("SET","mykey","Hello World!").inspect

(0...1000).each{|n|
	STDOUT.write(gen_redis_proto("SET", "Key#{n}", "Value#{n}"))
}
```

```shell
$ ruby proto.rb | redis-cli --pipe
All data transferred. Waiting for the last reply...
Last reply received from server.
errors: 0, replies: 1000
```

### How the pipe mode works under the hood

* `redis-cli --pipe` 서버에게 최대한 빠르게 데이터를 전송한다.
* 데이터를 읽을 수 있게 됨과 동시에 파싱을 시도한다.
* STDIN에 더 이상 읽을 데이터가 없다면 랜덤한 20byte sting으로 구성된 ECHO 명령어를 전송한다. 이 명령어가 그대로 돌아오는 것이 보장되어야 한다.
* 응답을 받으면 랜덤 스트링을 매칭해본다. 매칭되면 성공으로 간주한다.


## Distributed Locks with Redis
서로 다른 다수의 프로세스들이 공유하는 분산 락을 구현해보자

다수의 라이브러리들이 DLM(Distributed Lock Manger)를 구현하고 있다. 모든 라이브러리들이 서로 다른 접근을 사용한다.  
canonical algorithm 인 Redlock에 대해서 알아보자. 아마도 vanilla single instance approach 보다 안전할 것이다.  

### Safety and Liveness Guarantees
분산락을 위한 세 가지 속성을 보장해야한다.

1. Saftey property: Mutual exclusion 이다. 반드시 하나의 클라이언트만 lock을 잡고 있을 수 있다.
2. Liveness property A: Deadlock free. lock을 잡던 클라이언트가 깨지더라도 항상 lock을 acquire 할 수 있어야 한다.
3. Liveness property B: Fault tolerance. Redis node가 up되면 클라이언트는 lock을 acquire / release 할 수 있다.  

### Why Failover-based Implementations Are Not Enough
우리가 이루고 싶은 것을 이해하기 위해서, 가장 {  } 한 레디스 분산락 라이브러리를 분석해보자.

레디스로 자원에 락을 거는 가장 쉬운 방법은 인스턴스에 키를 생성하는 것이다. 키는 대부분 TTL이 설정되어 있다.  
TTL이 초과되거나 키를 삭제하면 자원이 release된다.  

이 방식은 Redis master가 장애상황에 빠지면 전체 서버의 SPOF가 된다. Redis master가 장애에 빠지지 않도록 replication을 scale-out할 수 있지만, 이것은 mutual exclusion을 구현할 수 없다.

이 방식의 race condition을 살펴보자

1. Client A 가 master에 acquire한다.
2. master가 key를 replica에 복사하기 전에 죽는다.
3. replica가 master로 부터 promoted된다.
4. Client B가 Client A가 이미 lock해둔 자원에 대해 acquire한다. < SAFTEY VIOLATION


### Correct Implementation with a Single Instance
분산 락 알고리즘의 근-본을 살펴보자.

**acquire the lock**
`SET 

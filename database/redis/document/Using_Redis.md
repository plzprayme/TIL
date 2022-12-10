# Using Redis

## Client-side caching
네트워크 I/O가 비싸니까 로컬 메모리에 데이터를 저장해두고 쓰자. 
다수의 인스턴스를 운영하고 단일 DB를 운영한다고 했을 때, 모든 인스턴스에 live data 를 서빙해야한다.

### The Redis implementation of client-side Caching
`CLIENT TRACKING` 을 사용하면 된다. TRACKING 은 두 가지 모드가 있는다.

#### default mode
서버가 접근한 클라이언트를 기억하고 있다.  
그리고 키가 변경되었을 때 저장되어 있는 client들에게 invalidation message를 전송한다. 

default mode의 sequence는 다음과 같다.
<img width="342" alt="스크린샷 2022-12-10 오전 11 52 25" src="https://user-images.githubusercontent.com/34934883/206825241-a4997138-a919-46fc-8b82-554edc37c957.png">

그런데 문제가 있다. 10,000의 클라이언트와 연결을 맺고 수백만의 키들을 유지하고 있다고 생각해보자.  
이런 서버가 동작하려면 아주 비싼 CPU와 Memory가 필요할 것이다.  

그래서 Redis는 2가지 아이디어 구현을 통해 Memory CPU 사용량을 핸들링한다.  

1. 클라이언트의 리스트를 저장하는 single global table을 유지한다. 해당 테이블은 Invalidation Table 라고 부른다. entry max 만큼 저장할 수 있고, 새로운 키가 입력되면 키가 삭제된 것으로 여기며 클라이언트들에게 invalidation message 를 전송한다. 이 방식을 통해 클라이언트가 같은 키를 가지고 있더라도 메모리를 회수할 수 있다.  
2. invalidation table에 클라이언트 구조 포인터를 실제로 저장할 필요는 없다. 우리는 레디스 클라이언트의 고유 ID를 저장한다. 클라이언트와 연결이 끊기면 garbage collector가 메모리를 확보할 것 이다.
3. single key namespace를 사용한다. database number를 구분하지 않는다. 예를들어 database 2에서 foo를 캐싱하고, database 3에서 foo를 수정하더라도 invalidation message는 전송될 것이다. 이를 통해 memory usage를 절약하고 구현 복잡도를 낮췄다.

#### Two Connections mode
Redis 6이 지원하는 RESP3 프로토콜을 사용한다면, 같은 커넥션에서 쿼리와 invalidation message를 받을 수 있다.  
대부분의 클라이언트들이 invalidate message connection, data connection을 분리한다. 이를 통해 invalidation message를 client ID로 식별되는 다른 커넥션으로 redirect 할 수 있다. 다수의 data connection이 invalidation message를 하나의 connection으로 redirection 할 수 있다. 이 방법은 connection pooling을 구현할 때 유용하다. two connections model은 RESP2에서도 지원한다.

RESP2 로 구현한 example을 살펴보자.
example은 tracking redirecting을 활성화, asking for a key, modified key의 invalidation message를 받는 시나리오이다.  

시작하기위해 일단 첫번째 커넥션을 연다. 이 커넥션은 invalidation, connection ID 요청, invalidation message를 받는 Pub/Sub channel 등 여러 용도로 사용된다.  

```shell
(Connection 1 -- used for invalidations)
CLIENT ID
:4
SUBSCRIBE __redis__:invalidate
*3
$9
subscribe
$20
__redis__:invalidate
```

data connection에서 이제 tracking 을 활성화하자

```shell
(Connection 2 -- data connection)
CLIENT TRACKING on REDIRECT 4
+OK

GET foo
$3
bar
```

클라이언트는 로컬 메모리에 "foo" => "bar" 를 캐싱하게 되었다.  
이제 다른 클라이언트에서 "foo" 를 변경해보자

```shell
(Some other unrelated connection)
SET foo bar
+OK
```
그러면 invalidations connection은 invalidation message를 받게 된다.

```shell
(Connecction 1 -- used for invalidations)
*3
$7
message
$20
__redis__:invalidate
*1
$3
foo
```

클라이언트에서는 캐싱 슬롯에서 해당 키가 있는지 확인하고 더 이상 유효한 데이터가 아니라고 판단하여 버린다.  
Pub/Sub 메세지의 세번째 원소는 single key가 아니라 array로 이루어진 single element라는 것에 주목해야한다.  array를 전송하면 모든 원소에 대해서 validate을 single message에서 할 수 있다. 

client-side caching을 구현하기 위해서는 RESP2, Pub/Sub connection의 이해가 중요하다. Pub/Sub과 REDIRECGT를 사용하여 Trick을 부리는 것과 같기 때문이다. 반면에 RESP3 를 사용하면 invalidation message가 connection에 의해서 전송된다.

#### Opt-in chaching
클라이언트는 선택한 키만 캐시할 수 있다. 이건 캐시를 추가할 때  더 많은 bandwidth가 필요하지만 서버에 저장되는 data의 양과 클라이언트가 수신하는 invalidation message는 줄어든다.  

`CLIENT TRACKING on REDIRECT 1234 OPTIN`



### broadcasting mode
서버가 클라이언트를 기억하지 않는다. 따라서, 서버의 메모리를 사용하지 않는다.  
대신에 클라이언트가 subscribe 하고 있으며 매칭되는 키를 다룰 때 마다 notification message를 받는다.  

...

## Redis pipelining
multiple command 들의 각각 response를 기다리지 않도록 해서 퍼포먼스를 향상 시키는 기술이다.  
이 기술의 문제점을 어떻게 해결했는지 알아보자

### Req/Res protocols and RTT(Routd-Trip Time)
Redis는 TCP의 Server - client model의 프로토콜을 사용한다. 그래서 요청마다 
Redis는 TCP의 Server - client model의 프로토콜을 사용한다.  ㅇㅡㅇ답을 받아야만 한다.




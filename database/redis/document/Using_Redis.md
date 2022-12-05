# Using Redis

## Client-side caching
high performance service를 제공하기 위해서 client-side caching을 고려할 수 있다.  
일반적인 서버는 DB와 직접 연결을 맺지만, Client-side caching 의 경우에는 서버 자체의 local cache 즉, 메모리에 저장하게 된다.  
이를 통해 DB connection이 없어지고 빨라진다.  

로컬에는 DB와 비교하여 아주 적은 메모리를 가지고 있을테니까 local cache는 너무 크지 않아야한다.  
자주 접근하는 데이터를 캐시로 유지한다면 데이터 획득 지연 시간이 크게 감소하고, DB 부하도 줄일 수 있다.  
자주 변경되지 않는 데이터를 저장하는 것도 유용하다.

결론
* small latency
* less database query


### CS의 두가지 골칫덩이
이 패턴의 문제 중 하나는 데이터가 유효기간이 지나면 유저에게 보여주지 않도록 보장하는 것이다.  
그래서 Redis 는 Time to live 라는 고정적인 유효시간을 설정하는 아이디어를 도입했다.  
기본적으로는 TTL이 지나면 데이터 유효성을 고려할 필요가 없다. 더 복잡한 시스템을 설계하고자 한다면 Pub/Sub 을 사용하여 listen client 들에게 메세지를 전송하는 패턴을 사용할 수 있다.  
Pub/Sub은 구독중인 모든 클라이언트에게 메세지를 전송하기 때문에 bandwidth 측면을 고려해야한다.  

하여튼 좋은 퍼포먼스를 내기 위해서는 반드시 client-side caching 기법을 사용해야한다.  
그래서 Redis 6에서부터 client-side caching 을 지원한다. 이걸 사용하면 고가용성에 효율적인 서버를 간단한 구현만으로 구축할 수 있다.  

### The Redis implementation of client-side Caching

ref: https://docs.spring.io/spring-data/redis/docs/current/reference/html/

## 10.4 Connecting to Redis
IoC Container 를 통해서 Redis와 연결한다. `RedisConnection`, `RedisConnectionFactory`인터페이스가 커넥션을 가져온다.  

### 10.4.1 RedisConnection and RedisConnectionFacotry
`RedisConnectionFacotory`는 `RedisConnection`에 의해서 생성된다.  
`RedisConnection`이 레디스와 상호작용하는데에 필수적인 블록을 만든다.  

### Redis Transaction
`multi, exec, discard` 커맨드로 트랜잭션을 지원한다. 이 메서드들은 `RedisTemplate`의 메서드이다.  
그런데 이 메서드들은 모든 명령어가 단일 connection에서 수행되는 것을 보장하지 않는다.  
모든 명령어가 단일 connection에서 수행되길 원한다면 `SessionCallback` 인터페이스를 사용하자

```java
// execute a Transaction
List<Object> txResults = redisTemplate.execute(new SessionCallback<List<Object>>() {
  public List<Object> execute(RedisOperations operations) throws DataAccessException {
    operations.multi();
    operations.opsForSet().add("key", "value1");
    // This will contain the results of all operations in the transaction
    return operations.exec();
  }
});


#### @Transactional Support
기본적으로는 지원안해준다. 만약 사용하고 싶다면 RedisTemplate 에 setEnableTransactionSupport(true)를 설정해야한다.  
만약 옵션을 
활성화하면 



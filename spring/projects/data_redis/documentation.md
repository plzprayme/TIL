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
```

#### @Transactional Support
기본적으로는 지원안해준다. 만약 사용하고 싶다면 RedisTemplate 에 setEnableTransactionSupport(true)를 설정해야한다.  
만약 옵션을 활성화하면 현재 트랜잭션의 RedisConnection 이 `ThreadLocal` 에 바인딩된다.  
만약 오류없이 트랜잭션이 끝나면 EXEC 메서드가 호출되는 것과 같고 에러가 발생했다면 DISCARD 가 호출된 것과 같을 것이다.

## Redis Cache
일단 `RedisCacheManager`을 빈으로 등록해줘야한다.  
```java
@Bean
public RedisCacheManager cacheManager(RedisConnectionFactory connectionFactory) {
    return RedisCacheManager.create(connectionFactory);
}
```

`RedisCacheManageBuilder` 로도 생성할 수 있다.

```java
RedisCacheManager cm = RedisCacheManager.builder(connectionFactory)
    .cacheDefaults(defaultCacheConfig())
    .withInitialCacheConfigurations(singletonMap("prodefined", defaultCacheConfig().disableCachingNullValues()))
    .transactionAware()
    .build();
```

`RedisCacheConfiguration`도 정의해주자. 여기서 TTL, prefix, RedisSerializer 구현체를 설정할 수 있다.  

```java
RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
    .entryTtl(Duration.ofSeconds(1))
    .disableCachingNullValues();
```

기본적으로는 read/write 연산에 lock 을 걸지 않는데 걸고 싶다면 ..

```java
RedisCacheManager cm = RedisCacheManager.build(RedisCacheWriter.lockingRedisCacheWriter(connectionFactory))
	.cacheDefaults(defaultCacheConfig())
	...
```


기본적으로 저장할 때 `prefix::key` 로 저장된다. 이걸 수정할 수 있다
```java
// static key prefix
RedisCacheConfiguration.defaultCacheConfig().prefixKeysWith("( ͡° ᴥ ͡°)"); // prefix가 이걸로 바뀜

The following example shows how to set a computed prefix:

// computed key prefix
RedisCacheConfiguration.defaultCacheConfig().computePrefixWith(cacheName -> "¯\_(ツ)_/¯" + cacheName); // 기존 prefix가 cacheName 으로 들어옴.
```

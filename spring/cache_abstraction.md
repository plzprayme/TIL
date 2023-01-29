ref: https://docs.spring.io/spring-framework/docs/6.0.4/reference/html/integration.html#cache

# Cache Abstraction
3.1 부터 지원한다. [transaction](https://docs.spring.io/spring-framework/docs/6.0.4/reference/html/data-access.html#transaction) 처럼 코드에 영향을 최소로 미친다.  
4.1에서는 JSR-107 annotation의 캐시 확장성이 크게 향상되었으며 더 커스터마이징하기 좋아졌다.  

## Unserstanding the Cache Abstrction
자바 메서드를 캐싱할 수 있게 해준다. 따라서 메서드 코드 수행 횟수를 줄여준다.  
cache target method가 호출 될 때 마다 해당 인자와 함께 호출된 적이 있는지를 체크한다.  
이미 호출된 적 있다면 메서드 코드 수행 없이 캐시를 반환하게 된다. 호출된 적이 없다면 메서드 코드를 수행하고 결과값을 캐싱한다.  

caching abstraction은 save, select 외에도 update, remove 연산을 제공한다.  
cache abstraction은 `org.springfarmework.cache.Cache`, `org.springframework.cache.CacheManager` 인터페이스로 사용할 수 있다.  
그래서 우리가 직접 caching logic을 작성할 필요가 없어진다.  

스프링에서는 일부 구현체를 제공한다. `ConcurrentMap` 기반, Gemfire, Caffeine, Ehcache 등이 있다.  

멀티 프로세스 환경에서 캐시를 운영한다면 직접 핸들링해야한다.  
여러 프로세스가 각자 캐시를 저장하는 것은 상관 없지만 update, remove 등이 발생하면 변경을 전파해줘야한다.  

## Declarative Annotation-based Caching
* `@Cacheable`: 캐싱 타겟 지정
* `@CacheEvict`: 캐시 제거
* `@CachePut`: 캐시 수정
* `@Caching`: 캐시작업 regroup
* `@CacheConfig`: class-level의 캐시셋팅

### The `@Cacheable` Annotation
메서드에 붙이면 캐시가 가능해진다. 

```java
@Cacheable("books")
public Book findBook(ISBN isbn) {}

@Cacheable({"books", "isbns"}) // 여러 캐시에 저장할 수 있다.
public Book findBook(ISBN isbn) {}|
```

#### Default Key Generation
cache 는 key-value store에 저장된다.그러므로 key가 필요하다.  
기본 설정은 다음과 같다.
* 파라미터가 없다면 `SimpleKey.EMPTY` 이다. (SimpleKey == hashCode)
* 파라미터가 하나라면 그 파라미터 인스턴스가 된다.
* 파라미터가 둘 이상이라면 모든 파라미터들의 `SimpleKey()` 이다.

default key gernation을 수정하고 싶다면 `org.springframework.cache.interceptor.KeyGeneratior` 인터페이스를 구현하면 된다.

#### Custom Key Generation Declaration
Cacheable 메서드에 SpEL로 키를 만들 수 있다.
```java
@Cacheable(cacheNames="books", key="#isbn")
public Book findBook(ISBN isbn, boolean checkWearehouse, boolean includeUsed);
```

### Cache Resolution
#### Default Cache Resolution
기본적으로는 CacheResolver가 캐시를 가져오고 CacheManager에 설정된 연산을 사용한다.  

#### Custom Resolution
`@cacheable(cacheResolver="runtimeCacheResolver")` 를 사용해서 커스텀할 수 있다.

#### Synchronized Caching
멀티 스레드 환경에서는 동기화를 시켜주자
`@cachable(cacheNames="foos", sync=true)`

#### Conditional Caching
SpEL 로 true / false 조건식을 입력하면 조건부 캐싱이 가능하다.
`@Cacheable(cacheNames="book", condition="#name.length() < 32")`

특정 조건일 때 캐싱을 하지 않으려면 unless 를 사용하자
`@Cacheable(cacheNames="book", condition="#name.length() < 32", unless=#result.hardback")`

Optional 도 지원한다.
`@Cacheable(cacheNames="book", condition="#name.length() < 32", unless=#result?.hardback")`

### @CachePut Annotation
애너테이션이 붙은 메서드의 결과값으로 캐시가 업데이트 된다.  
사용 예를 들어보면 DB 값은 업데이트 되었는데 인메모리 캐시는 반영이 안되어 있을 수 있기 때문에 이 메서드를 사용해야한다.  
그러니까 Cacheable 이랑 CachePut을 동시에 같은 메서드에 붙이지 말자.
`@CachePut(cacheNames="book", key="#isbn")` 

### @CacheEvict Annotation
CacheEvict 는 메서드 리턴 값에 따른 동작이 없다. 
`@CacheEvict(cacheNames="books", allEntries=true)`

### Caching

# LettuceConnectionFacotry
Lettuce를 사용해서 Redis Connection을 만들기 위해서 `LettuceConnectionFactory`를 빈으로 등록해야한다.  
해당 인스턴스는 `RedisConnectionFacotry` 인터페이스의 `getConnection()` 메서드가 호출 될 때마다 새로운 `LettuceConnection` 을 리턴한다.  
LettuceConnection은 single thread-safe를 공유하는 것이 기본 설정이다.  

shared native connection은 절대 LettuceConnection에 의해서 닫히지 않는다.  

shareNativeConnection이 true로 설정되어 있다면, 일반적인 명령어들은 shared connection을 사용하게 된다.  
tx, blocking 명령어를 수행할 때는 LettuceConnectionProvier에 의해서 선택된 커넥션을 사용한다. shared connection을 사용하지 않는다.  
만약 native connection sharing이 비활성화 되어 있으면 모든 명령어에 대해서 새로운 커넥션을 생성한다.   

LettuceConnectionFacotry는 environmental configuration 과 clienct configuration에 의해서 구성된다.  

# Environmental Configurations
* RedisStandaloneConfiguration
* RedisStaticMasterReplicaConfiguration
* RedisSocketConfiguration
* RedisSentinelConfiguration
* RedisClusterConfiguration

## RedisStandaloneConfiguration
single node의 Redis를 설정할 때 해당 클래스를 정의하자.
host, port, database number, username, password 를 설정할 수 있다.

# ClientConfigurations
## LettuceClientConfiguration
SSL, verify peers, StartTLS, ClientResources, ClientOptions (TimoutOptions), client name, ReadFrom Master/Replica, timeout, period. 

### ClientOptions
여기서 timeout을 설정할 수 있다. TimeoutSource를 커스텀하자


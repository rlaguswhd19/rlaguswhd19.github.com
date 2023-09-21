---
title : "[SPRINGBOOT] HikariCP"
tags :
 - Springboot

---

# HikariCP

히카리 커넥션풀

<br/>

#### connection pool CP

**미리 데이터베이스와 연결시킨 상태를 유지하는 기술**

<br/>

#### 스프링에서의 커넥션 풀

자바에서는 기본적으로 DataSource 인터페이스를 사용하여 커넥션풀을 관리한다.

Spring에서는 사용자가 직접 커넥션을 관리할 필요없이 자동화된 기법들을 제공하는데

SpringBoot 2.0 이전에는 **tomcat-jdbc**를 사용하다,
현재 2.0이후 부터는 **HikariCP**를 기본옵션으로 채택 하고있다.

[히카리 벤치마킹 페이지](https://github.com/brettwooldridge/HikariCP-benchmark)를 보면 성능을 알 수 있다.

![스크린샷 2023-09-19 오전 9 51 13](https://github.com/rlaguswhd19/rlaguswhd19.github.com/assets/46040824/7406df3b-6294-41fe-a289-46f9da4779e3)



히카리의 경우는 Connection 객체를 Wrappring한 PoolEntry가 있고 이를 관리하는 **ConcurrentBag**이라는 구조를 가지고 있다.

`ConcurrentBag은 HikariPool.getConnection() -> ConcurrentBag.borrow()`라는 메서드를 통해 사용 가능한(idle) Connection을 리턴하도록 되어있다.

대략적인 내용은 여기 정리가 잘 되어있다.

[히카리 커넥션풀 데드락](https://techblog.woowahan.com/2664/)

<br/>

#### 히카리 설정 변경 방법

```java
@Bean
@ConfigurationProperties(prefix = "datasource.af-bbs")
public HikariConfig bbsHikariConfig() {
	HikariConfig hikariConfig = new HikariConfig();
	hikariConfig.setMaxLifetime(1800000);
	hikariConfig.setConnectionTimeout(30000);
	hikariConfig.setValidationTimeout(5000);
	hikariConfig.setKeepaliveTime(30000);
	return hikariConfig;
}

@BatchDataSource
@Bean
public DataSource bbsDataSource(@Qualifier("bbsHikariConfig") HikariConfig bbsHikariConfig) {
	return new HikariDataSource(bbsHikariConfig);
}
```

###### [option](https://perfectacle.github.io/2022/09/25/hikari-cp-time-config/#validationTimeout)

* maxLifeTime

  커넥션 풀에서 idle 커넥션이 최대 얼마동안 생존할 수 있냐는 설정이다. (단위는 ms, 기본값은 30분, 최소값은 30초, 0으로 설정하면 무제한)

* connectionTimeout

  커넥션을 맺는데 걸리는 시간을 의미하며 이 시간은 단순히 하나의 물리적인 커넥션을 맺는데 걸리는 시간을 의미하는 게 아니라 [커넥션 풀에서 커넥션을 획득하는데 걸리는 시간](https://github.com/brettwooldridge/HikariCP/blob/dev/src/main/java/com/zaxxer/hikari/pool/HikariPool.java#L136-L176)을 의미한다.

* validationTimeout

  커넥션의 유효성(사용 가능한 상태인지)을 검사하는데 걸리는 최대 마지노선 시간이라고 보면 된다. (기본값은 5초이고, 최소값은 250ms)

* keepaliveTime

  dle connection에 대해서 keepaliveTime에 도달하면 주기적으로 커넥션의 유효성을 검증한다. (최소값은 30분, 기본값은 비활성화(0)이다.)

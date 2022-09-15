---
title : "[SPRINGBOOT] 다중 DB 설정"
tags : 
 - Springboot
---

## 다중 DB 설정

스프링부트 여러 DB를 설정하는 방법을 정리한다.

`@EnableJpaRepostiory` : `JpaRepository`를 활성화하기 위한 애노테이션으로 기본적으로 해당 클래스의 하위 패키지를 스캔한다.

* `basePackages` : 여러 DB를 사용하게 되면 각 DB마다 사용하는 Repository가 다르기 때문에 package 단위로 설정 할 수 있다.

* `entityManagerFactoryRef` : `EntitiyManager`를 설정한다. `Entity` = DB의 Table과 매칭되는 개념으로 보면 된다. `EntityManager`는 `Persistence Context`로 `Entity`를 관리한다. `EntityManager`와 `Persistence Context`는 데이터의 상태 변화를 감지하고 필요한 쿼리를 수행한다. `EntityManagerFactory`는 EntityManager를 만드는 공장인데 Thread Safe하지 않아 상황에 따라서 계속 `EntityManager`를 만들어줘야한다. 공장을 만드는 비용은 굉장히 크기 때문에 DB마다 하나씩 사용한다. 반면에 `EntityManager`를 만드는것은 비용이 많이 들지 않는다. 그래서 `EntityManagerFactory`는 Thread Safe라고 한다.

* `transactionManagerRef` : `TransactionManager`를 설정한다. PlatformTransactionManager는 스프링 트랜잭션 추상화의 핵심 인터페이스로 다양한 환경과 제품에 대응하는 구현 클래스를 제공한다.

  * `DataSourceTransactionManager` : JDBC 및 마이바티스 등의 JDBC 기반 라이브러리로 데이터베이스에 접근하는 경우에 이용한다.
  * `HibernateTransactionManager` : 하이버네이트를 이용해 데이터베이스에 접근하는 경우에 이용한다.
  * `JpaTransactionManager` : JPA로 데이터베이스에 접근하는 경우에 이용한다.

  JPA를 사용하기 위해서 JpaTransactionManager를 만들어준다.

<br/>

`@Primary` : 다중 DB를 설정하게 되면 같은 타입의 여러 `Bean`이 생성되기 때문에 기본적으로 사용한 Bean을 지정해주는 역할을 한다. 스프링의 `Bean`주입은 이름이 아니라 타입이 우선시 되기 때문이다. 그래서 예를 들어 `DataSource`를 `Bean`이름을 다르게해서 여러개 만들더라도 Primary로 지정된 Bean을 우선적으로 불러온다. 이름으로 가져오기 위해서는 `@Qualifier`를 사용해서 의존성을 주입해줘야 한다. 때문에 다중 DB 설정을 할때 다른 설정에는 `@Qualifier`를 사용해서 생성자 함수에 의존성을 주입한다. 하지 않으면 `@Primary`로 지정된 의존성이 주입되기 때문에 주의하자.

<br/>

`@ConfigurationProperties` : *.properties , *.yml 파일에 있는 property를 자바 클래스에 값을 가져와서(바인딩) 사용할 수 있게 해주는 애노테이션으로 객체에 바인딩하거나 생성자에 사용하여 속성값에 바인딩이 가능하다. `DataSource` 생성자에 사용하여 값들을 바인딩하여 만들어준다.

<br/>

###### LiveBoxDbConfig.class

```java
@Configuration
@EnableJpaRepositories(
        basePackages = "com.search.api.repository.livebox",
        entityManagerFactoryRef = "liveBoxDbEntityManager",
        transactionManagerRef = "liveBoxDbTransactionManager"
)
public class LiveBoxDbConfig {
    @Value("${spring.datasource.dialect}")
    private String dialect;
    @Value("${spring.datasource.ddl-auto}")
    private String ddlAuto;

    @Primary
    @ConfigurationProperties(prefix="spring.datasource.live-box-db")
    @Bean
    public DataSource liveBoxDbDataSource() {
        return DataSourceBuilder.create().build();
    }

    @Primary
    @Bean
    public LocalContainerEntityManagerFactoryBean liveBoxDbEntityManager(EntityManagerFactoryBuilder builder, DataSource liveBoxDbDataSource) {
        HashMap<String, Object> properties = new HashMap<>();
        properties.put("hibernate.dialect", dialect);
        properties.put("hibernate.hbm2ddl.auto", ddlAuto);
        properties.put("hibernate.format_sql", "true");

        return builder
                .dataSource(liveBoxDbDataSource)
                .packages("com.search.api.dto.livebox") // DB Entity package 설정
                .properties(properties) // DB 설정값
                .persistenceUnit("LiveBox") // EntityManager에서 사용할 이름 설정
                .build();
    }

    @Primary
    @Bean
    public PlatformTransactionManager liveBoxDbTransactionManager(LocalContainerEntityManagerFactoryBean liveBoxDbEntityManager) {
        return new JpaTransactionManager(Objects.requireNonNull(liveBoxDbEntityManager.getObject()));
    }
}
```

<br/>

###### SearchAdminDbConfig.class

`@Primary`가 없기 때문에 `@Qualifier`를 사용해 Bean을 주입한다. `@Qualifier`가 없으면 `@Primary`로 지정된 Bean이 주입된다.

```java
@Configuration
@EnableJpaRepositories(
        basePackages = "com.search.api.repository.admin",
        entityManagerFactoryRef = "searchAdminDbEntityManager",
        transactionManagerRef = "searchAdminDbTransactionManager"
)
public class SearchAdminDbConfig {
    @Value("${spring.datasource.dialect}")
    private String dialect;
    @Value("${spring.datasource.ddl-auto}")
    private String ddlAuto;

    @ConfigurationProperties(prefix="spring.datasource.search-admin-db")
    @Bean
    public DataSource searchAdminDbDataSource() {
        return DataSourceBuilder.create().build();
    }

    @Bean
    public LocalContainerEntityManagerFactoryBean searchAdminDbEntityManager(EntityManagerFactoryBuilder builder, @Qualifier("searchAdminDbDataSource") DataSource searchAdminDbDataSource) { 
        HashMap<String, Object> properties = new HashMap<>();
        properties.put("hibernate.dialect", dialect);
        properties.put("hibernate.hbm2ddl.auto", ddlAuto);
        properties.put("hibernate.format_sql", "true");

        return builder
                .dataSource(searchAdminDbDataSource)
                .packages("com.search.api.dto.admin")
                .properties(properties)
                .persistenceUnit("SearchAdmin")
                .build();
    }

    @Bean
    public PlatformTransactionManager searchAdminDbTransactionManager(@Qualifier("searchAdminDbEntityManager") LocalContainerEntityManagerFactoryBean searchAdminDbEntityManager) {
        return new JpaTransactionManager(Objects.requireNonNull(searchAdminDbEntityManager.getObject()));
    }
}
```




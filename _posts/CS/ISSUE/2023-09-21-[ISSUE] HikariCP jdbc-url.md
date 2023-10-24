---
title : "[ISSUE] HikariCP jdbc-url"
tags :
 - issue

---

# HikariCP jdbc-url

검색 인덱서를 만드는 과정에서 url과 jdbc-url을 혼용해서 사용하고 있었는데 jdbc-url을 삭제하면서 bbs에서 에러가 발생했다.

bj에서는 에러가 발생하지 않았는데 둘의 차이점을 보면 알 수 있다.



###### bj

```java
@Bean(name = "afSearchDatasourceProperties")
@ConfigurationProperties(prefix = "datasource.af-search")
public DataSourceProperties afSearchDatasourceProperties() {
    return new DataSourceProperties();
}
    
@Bean(name = "afSearchDataSource")
public DataSource dataSource(@Qualifier("afSearchDatasourceProperties") DataSourceProperties properties) {
    return properties.initializeDataSourceBuilder().build();
}
```



###### bbs

```java
@Bean
@ConfigurationProperties(prefix = "datasource.af-bbs")
public DataSource bbsDataSource() {
    return DataSourceBuilder.create().build();
}
```



**bj**와 **bbs**의 DataSource구성을 보면 다르게 되어있다. HikariCP의 경우는 **HikariDataSourceProperties**를 사용한다. Springboot의 경우는 HikariDataSource를 default로 사용하기 때문에 jdbc-url을 사용해야하고 위의 경우는 url로 사용해야한다.

```java
	/**
	 * {@link DataSourceProperties} for Hikari.
	 */
	private static class HikariDataSourceProperties extends MappedDataSourceProperties<HikariDataSource> {

		HikariDataSourceProperties() {
			add(DataSourceProperty.URL, HikariDataSource::getJdbcUrl, HikariDataSource::setJdbcUrl);
			add(DataSourceProperty.DRIVER_CLASS_NAME, HikariDataSource::getDriverClassName,
					HikariDataSource::setDriverClassName);
			add(DataSourceProperty.USERNAME, HikariDataSource::getUsername, HikariDataSource::setUsername);
			add(DataSourceProperty.PASSWORD, HikariDataSource::getPassword, HikariDataSource::setPassword);
		}
}
```

